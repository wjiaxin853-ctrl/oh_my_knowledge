
### DSP业务原理图

主要抽象为三部分，业务pod埋点，Promethus指标抓取与存储，Grafana面板可视化

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

##### 第一部份：promethus Client原理（业务埋点）

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

##### 第二部分：Promethus指标抓取与存储
