### DSP监控原理

**DSP采用的是VictoriaMetrics架构存储数据，自动发现和抓取 Pod 的指标，而不是Promethus service**
##### 工作流程

![[Pasted image 20251226005316.png]]
##### 架构对比

```
  | 组件  | Prometheus 架构     | VictoriaMetrics 架构        |
  |-----|-------------------|---------------------------|
  | 客户端 | Prometheus Client | Prometheus Client（兼容）     |
  | 采集器 | Prometheus Server | VictoriaMetrics           |
  | 存储  | Prometheus TSDB   | VictoriaMetrics TSDB（更高效） |
  | 查询  | PromQL            | MetricsQL（兼容 PromQL）      |
  | 展示  | Grafana           | Grafana                   |
```

##### promethus Client原理

```
  1. 指标定义（metrics.go）
     └── prometheus.NewCounterVec() 创建指标对象

  2. 指标注册（metrics.go 的 init()）
     └── prometheus.MustRegister() 注册到全局注册表

  3. 指标更新（业务代码中）
     └── ExternalAPIRequestCounter.WithLabelValues(...).Inc()

  4. 指标暴露（main.go）
     └── promhttp.Handler() 返回所有注册的指标
     └── 格式化为 Prometheus 标准格式

  5. 数据采集
     └── VictoriaMetrics 访问 http://pod:9300/metrics
     └── 获取指标数据并存储
```

##### 监控指标输出

```
  # HELP taskinference_external_api_requests_total External API request count...
  # TYPE taskinference_external_api_requests_total counter
  taskinference_external_api_requests_total{endpoint="/api/chat",method="POST",sta
  tus="200"} 1234
```

**Prometheus 在抓取时，自动添加当前时间戳**

```
  Prometheus 的拉取模型（Pull Model）

  不是 Inc() 主动上传，而是 Prometheus 定期来拉取

  时间线：
  10:00:00  应用启动，counter = 0
  10:00:05  发生1次错误，Inc() → counter = 1（内存中）
  10:00:08  发生1次错误，Inc() → counter = 2（内存中）
  10:00:12  发生1次错误，Inc() → counter = 3（内存中）

  10:00:15  ← Prometheus 来抓取（第1次scrape）
            访问 http://taskinference:9300/metrics
            读取到 counter = 3
            存储：{timestamp: 10:00:15, value: 3}

  10:00:20  发生2次错误，Inc() → counter = 5（内存中）
  10:00:28  发生1次错误，Inc() → counter = 6（内存中）

  10:00:30  ← Prometheus 来抓取（第2次scrape）
            访问 http://taskinference:9300/metrics
            读取到 counter = 6
            存储：{timestamp: 10:00:30, value: 6}

  10:00:45  ← Prometheus 来抓取（第3次scrape）
            读取到 counter = 6（没有新错误）
            存储：{timestamp: 10:00:45, value: 6}
```


```
  存储的是定期采样的快照，不是每次 Inc()

  Prometheus 数据库中存储：

  api_inference_failures_total{error_code="1001", error_msg="JSON解析失败"}
  ├─ timestamp: 1640000000 (10:00:00), value: 0
  ├─ timestamp: 1640000015 (10:00:15), value: 3   ← 采样点1
  ├─ timestamp: 1640000030 (10:00:30), value: 6   ← 采样点2
  ├─ timestamp: 1640000045 (10:00:45), value: 6   ← 采样点3
  ├─ timestamp: 1640000060 (10:01:00), value: 10  ← 采样点4
  └─ ...

  注意：
  - 10:00:05、10:00:08、10:00:12 的 Inc() 调用没有被单独记录！
  - 只记录了 10:00:15 时刻的累计值（3次）
  - 时间戳是 Prometheus 在抓取时添加的，不是应用程序
```









## Promethus介绍

 Prometheus **将所有指标数据存储为时间序列**，即**指标信息**与其**记录时间戳**一起存储；此外，还可以存储称为**标签**的可选键值对。

**时间序列**=指标名称+时间戳+数值，本质是一个随时间变化的指标数据曲线，即属于同一指标和同一组带标签维度的带时间戳的值流

每个时间序列都由其**指标名称**和称为**标签**的可选键值对进行唯一标识。

promethus会定期的去采集，之后会自动生成一系列的**时间戳+数字**

## 指标

指标是一种衡量标准，对于 Web 服务器来说，可能是请求时间；对于数据库来说，可能是 CPU 使用率或活跃连接数等等。
### 指标名称

- 指标名称应明确指出所测量系统的总体特征（例如`http_requests_total`，接收到的 HTTP 请求总数）。
- 指标名称可以使用任何 UTF-8 字符。
- 指标名称应与正则表达式匹配`[a-zA-Z_:][a-zA-Z0-9_:]*`，超出此范围的指标名称在 PromQL 中使用时需要加引号

> [!注意]
> 冒号（“:”）保留用于用户自定义的记录规则。导出程序或直接仪器不应使用冒号。
> 

### 标签

标签允许您捕获同一指标名称的不同实例，我们称之为 Prometheus 的“维度数据模型”。**查询语言允许基于这些维度进行过滤和聚合**。任何标签值的更改（包括添加或删除标签）都会创建一个新的时间序列。

> [!NOTE]
> 只要标签组合发生任何变化，（值修改、新增标签、删除标签），就不再是原来的时间序列，而是生成一条全新的独立序列
> 

#### 标签命名规范
- 标签名称可以使用任何 UTF-8 字符。
- 以`__`（两个下划线）开头的标签名称必须保留供 Prometheus 内部使用。
- 标签名称应与正则表达式匹配`[a-zA-Z_][a-zA-Z0-9_]*`
- 标签值可以包含任何 UTF-8 字符。
- 标签值为空的标签被视为等同于不存在的标签。

#### 标签种类

  标签有两种，自定义标签还有自动生成标签：
  - 自定义标签：在采集的时候自己定义，在指标函数，例如`endpoint`，`status`，`method`
  - 自动生成标签:由服务发现或采集过程自动附加，用于标识目标属性；例如`instance=192.168.1.10:9100`（目标地址）、`job=node_exporter`（采集任务名），`pod=dspm-taskinference-ndkwa`（pod来源）



## Promethus的基本架构




## 存储
基于 Prometheus 数据模型规范，不同维度的时间序列（由 “指标名 + 标签组合” 唯一标识）通过**分层存储结构**实现物理隔离，再通过**标签匹配查询语法**实现逻辑区分
### 样品

**“样品”（Sample）** 是构成时间序列（Time Series）的**最小数据单元**，是 Prometheus 实际采集、存储和查询的 “原子级数据点”。它直接对应 “**某个指标在特定时间点的具体数值”**

#### 样品构成

样品包括：指标名称，标签集，时间戳，数值（该时间点的监控指标具体数值）

#### 样品的生命周期

##### 1.采集生成
Prometheus 通过 “拉取（Pull）” 方式从监控目标（如服务器、应用的 `/metrics` 接口）获取数据，每一次拉取都会为每个指标的每个维度生成一个或多个样品
##### 2.标签处理与序列归属

Prometheus 会检查每个样品的 “指标名 + 标签集”：

- 如果该组合不存在，就创建一条新的时间序列；
- 如果该组合已存在，就将当前样品追加到这条时间序列的末尾（按时间戳排序）。

##### 3.持久化存储

样品会**先存入 Prometheus 的内存中**（用于快速查询近期数据），随后按配置的 “存储策略”（如 `retention_time`）持久化到磁盘（默认是 TSDB 时序数据库格式）。存储时，样品会按 “时间序列” 分组，相同序列的样品会被高效压缩（利用时序数据的连续性特征）。

每个时间序列的存储格式为:时间序列 = 指标名称 + 标签集 + [(时间戳, 数值), (时间戳, 数值), ...]

```
  {
    "metric": {
      "__name__": "taskdispatch_api_latency_seconds_sum",
      "container": "dspm-taskdispatch-task-distribute",
      "endpoint": "metrics",
      "exported_endpoint": "/dataIdentifyConfig/v1/allTemplateIds",
      "instance": "172.17.248.6:9300",
      "job": "dsp/dspm-taskdispatch-pod-monitor",
      "method": "GET",
      "namespace": "dsp",
      "pod": "dspm-taskdispatch-78f79b84d4-kj54j",
      "status": "200"
    },
    "value": [
      1766028592,      // Unix 时间戳 (2025-12-18 10:16:32)
      "10.243487563"   // 数值 (总延迟 10.24 秒)
    ]
  }

```


## 查询

### 符号（指标监控输出）
给定一个指标名称和一组标签，时间序列通常使用以下符号进行标识：

```
<metric name>{<label name>="<label value>", ...}
```

例如，带有指标名称`api_http_requests_total`和标签的时间序列`method="POST"`可以`handler="/messages"`这样写：

```
api_http_requests_total{method="POST", handler="/messages"}
```

### 查询规则
##### 针对UTF-8名称查询规范

在 PromQL 中，查询具有 UTF-8 名称的指标需要略微不同的语法。UTF-8 名称必须用引号括起来**并**移到花括号中：

`{"my.metric"}`

如果标签名称包含与旧版不兼容的字符，则必须用引号括起来：

`{"metric.name", "my.label.name"="bar"}`

**度量名称可以出现在大括号内的任何位置，但规范建议将其放在第一个位置。**


# 埋点

### 度量指标（四个）

#### Counter
计数器是一个**只能递增或重置**的指标值，也就是说，其值不能小于先前的值。它可以用于诸如**请求数、错误数**等指标。或者在重启时重置为零



##### Inc（）

- 非持久化存储
- Prometheus 计数器（Counter）
- 本质是一个内存中的变量（比如 `float64` 类型的累加器）
- 不是覆盖，是累加，`count = 100`，每次请求执行 `count += 1`，变成 `101`、`102`...，历史累积值持续保留

**Inc() 只做一件事：在应用程序内存中把计数器 +1**

```
  一、Inc() 函数 - 用于 Counter 指标


  Inc() = Increment（递增），用于 Counter 类型指标，表示"发生一次事件，计数器+1"。


  示例1：指标十四 - 错误统计（行247-256）

  func RecordDynamicInferenceError(taskErr *globalpkg.TaskError) {
      if taskErr != nil && taskErr.Code != globalpkg.Success.Code {
          InferenceAPIFailureCounter.WithLabelValues(
              strconv.Itoa(taskErr.Code),  // 错误码
              taskErr.Msg,                  // 错误原因
          ).Inc()  // ← 每次调用这个函数，对应错误码的计数器 +1
      }
  }

  工作流程：
  第1次错误：error_code=1001, error_msg="JSON解析失败"
  → Inc() 调用
  → Counter 从 0 变为 1

  第2次相同错误：error_code=1001, error_msg="JSON解析失败"
  → Inc() 调用
  → Counter 从 1 变为 2

  第3次不同错误：error_code=1002, error_msg="GPU推理超时"
  → Inc() 调用
  → 这是一个新的标签组合，创建新的 Counter，值为 1

  存储结果：
  api_inference_failures_total{error_code="1001", error_msg="JSON解析失败"} = 2
  api_inference_failures_total{error_code="1002", error_msg="GPU推理超时"} = 1

```


#### Gauge（测量）

计量值是一个**可以上升也可以下降的数值**。它可以用于衡量集群中的 **Pod 数量、队列中的事件数量**等指标。

PromQL 函数（例如`max_over_time`` get_guest_metrics``min_over_time`和`avg_over_time``get_guest_metrics`）可用于计量指标。


#### Histogram（直方图）

直方图可用于任何**基于桶值进行计数的计算值**。桶的边界可由开发人员配置。一个常见的例子是响应请求所需的时间，称为**延迟**。

#####  桶的计数累积计算

> [!NOTE]
> **桶（Bucket）本质是 “累积桶”，即每个桶统计的是 “时延 ≤ 该桶上限（`le` 值）” 的所有请求次数，而非 “两个桶之间区间” 的次数****

例如：假设我们要观察 API 请求的处理时间。直方图允许我们将请求时间存储在不同的桶中，而不是单独存储每个请求的请求时间。我们可以定义不同的桶来存储处理时间，例如1、2、3、4**lower or equal 0.3和5。**
这些就是我们的桶，一旦计算出某个请求的处理时间`le 0.5`，它就会被添加到**所有桶边界值大于该值的桶的计数中**。`le 0.7``le 1``le 1.2`

假设对端点“/ping”的请求 1 耗时 0.25 秒。则各桶的计数值将为：

|桶|数数|
|---|---|
|0 - 0.3|1|
|0 - 0.5|1|
|0 - 0.7|1|
|0 - 1|1|
|0 - 1.2|1|
|0 - +无穷大|1|

注意：默认情况下会添加 +Inf 桶。

（由于直方图显示的是累积频率，因此所有大于目标值的桶都加 1）

对端点“/ping”的第二个请求耗时0.4秒。桶的计数值将如下所示。

| 桶        | 数数  |
| -------- | --- |
| 0 - 0.3  | 1   |
| 0 - 0.5  | 2   |
| 0 - 0.7  | 2   |
| 0 - 1    | 2   |
| 0 - 1.2  | 2   |
| 0 - +无穷大 | 2   |

由于 0.4 小于 0.5，因此所有小于等于 0.5 的桶的计数都会增加。

##### 桶的时延计算举例

**时间 10:05:00**：

```
桶    计数
0.05  500
0.1   800    # 包含前面的500
0.3   950    # 包含前面的800  
0.5   980    # 包含前面的950
+Inf  1000   # 包含前面的980
```

**时间 10:10:00**（新增100个请求）

```
- le="0.05": 500 + 40 = 540

- le="0.1": 800 + (40+30) = 870

- le="0.3": 950 + (40+30+20) = 1040

- le="0.5": 980 + (40+30+20+5) = 1075

- le="+Inf": 1000 + 100 = 1100
```

**第一步计算：rate（）**
时间窗口：300秒（10:05到10:10）

```
rate(le="0.05") = (540-500)/300 = 40/300 = 0.1333 请求/秒
rate(le="0.1")  = (870-800)/300 = 70/300 = 0.2333 请求/秒
rate(le="0.3")  = (1040-950)/300 = 90/300 = 0.3000 请求/秒
rate(le="0.5")  = (1075-980)/300 = 95/300 = 0.3167 请求/秒
rate(le="+Inf") = (1100-1000)/300 = 100/300 = 0.3333 请求/秒
```

**第二步：计算P90**

**总请求速率** = rate(le="+Inf") = 0.3333 请求/秒
**P90对应的请求速率** = 0.3333 × 0.9 = 0.3000 请求/秒

**查找对应的桶**

| 桶上限  | 累计速率   | 是否包含P90点(0.3000)  |
| ---- | ------ | ----------------- |
| 0.05 | 0.1333 | 0.1333 < 0.3000 ❌ |
| 0.1  | 0.2333 | 0.2333 < 0.3000 ❌ |
| 0.3  | 0.3000 | 0.3000 = 0.3000 ✅ |

**正好命中**！le="0.3"桶的累计速率(0.3000)正好等于P90所需的速率(0.3000)。

所以：**P90延迟 = 0.3秒**

第四步：生成时延曲线
Grafana在每个时间点（比如每分钟）都会重复这个过程：

```
10:00:00 → 计算9:55-10:00的rate() → P90 = 0.15秒
10:01:00 → 计算9:56-10:01的rate() → P90 = 0.18秒  
10:02:00 → 计算9:57-10:02的rate() → P90 = 0.22秒
10:03:00 → 计算9:58-10:03的rate() → P90 = 0.35秒 ← 性能下降
10:04:00 → 计算9:59-10:04的rate() → P90 = 0.28秒
10:05:00 → 计算10:00-10:05的rate() → P90 = 0.20秒
...
```

把这些点连接起来，就是**P90时延曲线**！

```
P90延迟(秒)
0.40 │                    •
0.35 │                   ↗
0.30 │           •     ↗
0.25 │         ↗ ↘   •
0.20 │       •     ↘
0.15 │     •         •
0.10 │   •
     └─────────────────
      10:00  10:02  10:04

```

**P90含义**：代表大多数用户体验：90%的用户感受就是这个延迟以下

假设一个接口的延迟分布：

```
P50  = 0.15秒  ← 一半用户比这快，一半比这慢
P90  = 0.25秒  ← 90%的用户 ≤ 0.25秒
P95  = 0.35秒  ← 95%的用户 ≤ 0.35秒  
P99  = 0.80秒  ← 99%的用户 ≤ 0.80秒
```

用户分布分析：

```
用户比例    延迟范围
50%用户     ≤ 0.15秒  (很快)
40%用户     0.15-0.25秒 (正常)
5%用户      0.25-0.35秒 (有点慢)  ← P90到P95之间
4%用户      0.35-0.80秒 (很慢)     ← P95到P99之间
1%用户       > 0.80秒  (极慢)     ← 最差的1%
```

也就是百分之九十的用户都是正常的，

理想状态：四个曲线间距很小，说明所有用户体验好，并且稳定
间距很大：有很快用户还有很慢用户



##### observe（）

用于 Histogram（直方图） 和 Summary（摘要）类型的指标。用来记录时延，大小。

将普通函数转化成Observer接口


```
type ObserverVec interface {
	GetMetricWith(Labels) (Observer, error)
	//安全获取指定标签的Observer，返回error
	
	GetMetricWithLabelValues(lvs ...string) (Observer, error)
	//按标签值顺序获取Observer
	
	With(Labels) Observer
	WithLabelValues(...string) Observer
	CurryWith(Labels) (ObserverVec, error)
	MustCurryWith(Labels) ObserverVec

	Collector
}
```

实际使用：
用于记录时延数据，将其放到对应的桶内
```
	APILatencyHistogram.WithLabelValues(
		endpoint,
		strconv.Itoa(statusCode),
		method,
	).Observe(duration.Seconds())
```

达成效果：
```
taskdispatch_api_latency_seconds_bucket{endpoint="/api/test",status="200",method="GET",le="2.5"} 3  ←

taskdispatch_api_latency_seconds_sum{endpoint="/api/test",status="200",method="GET",le="1"} 238

taskdispatch_api_latency_seconds_count{endpoint="/api/test",status="200",method="GET",le="1"} 238
```
22.680515866000142
##### 记录计算方式

```
  Buckets: []float64{0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10}
```

  这里定义了 11个bucket边界，Prometheus会自动添加 +Inf bucket，所以实际有：
  - 12个bucket（11个自定义 + 1个+Inf）
同时也会增加**sum:所有请求时延总和；_count：请求总数**

```
  二、Observe() 函数 - 用于 Histogram/Summary 指标

  定义和作用

  Observe(value) = 观察/记录一个具体的数值，用于 Histogram 和 Summary 类型指标，记录数值的分布情况。

  源码中的使用示例

  示例1：指标六 - 动态标签检出时延（行207-210）

  func ObserveDynamicLabelLatency(endpoint, status string, latency float64) {
      DynamicLabelLatencyHistogram.WithLabelValues(endpoint, status).Observe(latency)
      //                                                              ↑
      //                                                        记录这次的时延值
  }

  工作流程：
  // 实际调用（在其他代码中）
  start := time.Now()
  // ... 执行动态标签推理 ...
  takeTime := time.Since(start)
  monitor.ObserveDynamicLabelLatency(endpoint, status, takeTime.Seconds())
  //                                                    ↑
  //                                              例如：1.5 秒

  Histogram 的工作原理：
  // 指标定义（行56-64）
  DynamicLabelLatencyHistogram = prometheus.NewHistogramVec(
      prometheus.HistogramOpts{
          Name:    "dsp_dataidentify_dynamic_label_latency",
          Buckets: prometheus.ExponentialBuckets(1, 1.5, 15),
          // Buckets = [1ms, 1.5ms, 2.25ms, 3.375ms, ..., 约437ms]
      },
      []string{"endpoint", "status"},
  )

  多次调用后的存储：
  假设调用了5次 Observe()：
  - Observe(0.5)  // 0.5秒
  - Observe(1.2)  // 1.2秒
  - Observe(2.0)  // 2.0秒
  - Observe(0.8)  // 0.8秒
  - Observe(3.5)  // 3.5秒

  Prometheus 会自动统计分布：
  dsp_dataidentify_dynamic_label_latency_bucket{le="0.005"} = 0    (≤5ms的有0个)
  dsp_dataidentify_dynamic_label_latency_bucket{le="0.01"} = 0    (≤10ms的有0个)
  dsp_dataidentify_dynamic_label_latency_bucket{le="1.0"} = 2     (≤1秒的有2个)
  dsp_dataidentify_dynamic_label_latency_bucket{le="2.0"} = 3     (≤2秒的有3个)
  dsp_dataidentify_dynamic_label_latency_bucket{le="5.0"} = 5     (≤5秒的有5个)
  dsp_dataidentify_dynamic_label_latency_bucket{le="+Inf"} = 5    (所有的)

  dsp_dataidentify_dynamic_label_latency_sum = 8.0    (总和：0.5+1.2+2.0+0.8+3.5)
  dsp_dataidentify_dynamic_label_latency_count = 5    (总次数)
  
```

##### Sum

这个是所有这个指标的时间的总和，不是随着本地时间增加，而是所有指标的时间的总和，也就正常有指标会一直增加，没有指标会不变，增长较为缓慢证明指标例如接口的请求很快，累积叫增长会变慢。

##### Count


> [!NOTE]
> 考虑指标爆炸，endpoint到变化参数会导致内存很多key，内存爆炸，考虑endpoint到最大数值是多少个，是否可以枚举，状态吗有多少个，指标想成考虑内存性能占用到边界，衡量性能，

#### Summary

摘要也能衡量事件，是直方图的替代方案。它们成本更低，但会丢失更多数据。摘要是在应用层计算的，因此无法聚合同一进程多个实例的指标。当事先不知道指标的分组情况时，可以使用摘要，但强烈建议尽可能使用直方图而非摘要。