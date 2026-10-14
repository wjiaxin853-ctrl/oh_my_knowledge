
### 最近一小时是否有日志上报

  **"日志上报"是指什么**                                                                                                                                                                                              
  在这个上下文中，**"日志上报"指的是从各个数据源采集的日志流入 Pulsar 消息队列**：                                                                                                                                    
```
  数据源（STA、数据库审计等）

      ↓

  采集日志

      ↓

  上报到 Pulsar Topic（http_log）← 这就是"日志上报"

      ↓

  消费处理
```

  **怎么判断日志上报是否正常**

  **不能只看"消费速率"，要同时看"写入速率"和"消费速率"：**

### DSP日志是否严重堆积

**堆积（Backlog）** = 消息队列中等待被消费的消息数量

```
  日志流入 Pulsar Topic                                     

      ↓

  消息堆积在队列中

      ↓

  消费端处理消息

      ↓

  消息被消费，堆积减少
```

**Topic** = Pulsar 消息队列中的"主题"或"频道"
Topic 的完整路径：persistent://10001001/default/http_log

```
  persistent:// → 持久化消息（不会丢失）

  10001001 → 租户 ID（DSP 的租户）

  default → 命名空间（默认命名空间）

  http_log → Topic 名称（HTTP 日志
```
  
```
  **"严重堆积"的含义：**

  - 消息进入速度 > 消费速度

  - 导致队列中的消息越来越多

  - 最终可能导致系统性能下降、内存溢出等问题
```


```
curl http://pulsar-proxy.pulsar.svc.cluster.local/admin/v2/persistent/10001001/default/http_log/partitioned-stats -H 'Authorization: Bearer ${token}' | jq .subscriptions
```

  调用 Pulsar Admin API 获取 http_log topic 的统计信息， /partitioned-stats 返回该 topic 所有 partition 的详细统计

##### 判断推积是否严重方式

```
  **方法** **1：直接看堆积数量**
  msgBacklog = 50000 条
  判断：

  - 如果消费速率 = 1000 msg/s

  - 需要 50000 / 1000 = 50 秒才能消费完

  - 如果允许延迟 < 50 秒，则不严重

  - 如果允许延迟 < 10 秒，则严重 ❌

  **方法** **2：看堆积增长趋势**

  时间    堆积数量    趋势

  09:00   1000       ↗ 增长

  09:05   5000       ↗ 增长

  09:10   15000      ↗ 增长

  09:15   30000      ↗ 增长（严重堆积）❌


  **方法** **3：看消费速率** **vs** **写入速率**


  消费速率 = 1000 msg/s

  写入速率 = 1200 msg/s

  差值 = 200 msg/s（消费跟不上）

  
  如果持续 1 小时：

  堆积增长 = 200 msg/s × 3600s = 720000 条 ❌ 严重

```

##### 什么是topic还有Subscription
![[截屏2026-03-18 14.32.15.png]]

### 巡检项 3：检测GPT推理任务是否堆积严重（部分支持）
  DSP日志堆积：                                                                                                                                                                                                 

    快递站 = http_log（普通 HTTP 日志）

    里面放的是：用户访问日志、API 请求日志等
  

  GPT推理任务堆积：

    快递站 = GPT 推理相关的 topic（具体名称未知）

    里面放的是：等待 GPT 处理的推理任务

               比如：风险定性、数据识别、应用名称推理等


  **GPT** **推理任务的流程：**

  日志触发推理需求

      ↓

  推理任务写入 Pulsar（某个 GPT topic）

      ↓

  GPT 服务消费任务并处理

      ↓

  返回推理结果

  

  如果 GPT 服务处理慢或者挂了，任务就会在 Pulsar 里堆积。

### 是否修改了默认的日志审计配置范围
![[截屏2026-03-18 15.25.24.png]]





### LOgrich的富化topic

  **一、整体架构（用大白话）**                                                                                                                                                                                        

  想象一个快递分拣中心：                                                                                                                                                                                          

  - **采集点** = 快递收件点（STA设备、ZTNA网关）                                                                                                                                                                      

  - **Pulsar Topic** = 传送带                                                                                                                                                                                         

  - **富化服务** = 分拣工人（给快递贴标签、分类）                                                                                                                                                                     

  - **入库服务** = 仓库管理员（把快递存到仓库）                                                                                                                                                                       

  **二、5个Topic的完整流程（结合代码）**                                                                                                                                                                            

  

  **📍** **Topic** **1:** **collect-sta-http-log** **(采集侧)**

  

  **位置**：collect/default/collect-sta-http-log

  

  **作用**：STA设备采集的原始HTTP日志

  

  **代码证据**：

  // pulsar.go 第27-31行

  var topicPrefixMap = map[string]string{

      "persistent://10001001/default/ztna_access_log": "ATRUST_SYSLOG",

      "persistent://10001001/default/http_log":        "STA_HTTP_LOG",

  }

  

  **大白话**：

  - STA设备（流量分析设备）抓到HTTP请求后，发到这个topic

  - 这是**采集侧的传送带**，日志还没进入DSP系统

  - 消费者：collect-datalake-transfer-prc（数据湖传输服务）

  

  **为什么要监控**：

  - 如果这里堆积 → STA设备在疯狂发日志，但传输服务处理不过来

  - 堆积严重 → 日志会丢失或延迟进入DSP

  

  ---

  **📍** **Topic** **2:** **http_log** **(DSP入口)**

  

  **位置**：10001001/default/http_log

  

  **作用**：进入DSP系统的原始HTTP日志（未富化）

  

  **代码证据**：

  // pulsar.go 第29行

  "persistent://10001001/default/http_log": "STA_HTTP_LOG",

  

  **大白话**：

  - 传输服务把日志从采集侧搬到DSP侧，放到这个topic

  - 这是**DSP的原始日志入口**，日志还是"裸奔"状态（没有用户名、部门等信息）

  - 消费者：dsp_http_log（富化服务的订阅名）

  

  **为什么要监控**：

  - 如果这里堆积 → 富化服务处理不过来

  - 富化服务是单点，堆积会影响所有后续流程

  

  ---

  **📍** **Topic** **3:** **ztna_access_log** **(ZTNA入口)**

  

  **位置**：10001001/default/ztna_access_log

  

  **作用**：ZTNA零信任网关的访问日志

  

  **代码证据**：

  // pulsar.go 第28行

  "persistent://10001001/default/ztna_access_log": "ATRUST_SYSLOG",

  

  **大白话**：

  - ZTNA网关（零信任访问控制）记录的用户访问日志

  - 这是**另一个DSP入口**，与http_log并行

  - 消费者：**也是** **dsp_http_log**（与http_log共享同一个富化服务）

  

  **为什么要监控**：

  - 如果这里堆积 → 富化服务处理不过来（与http_log共享消费者）

  - ZTNA日志堆积会影响零信任访问的审计和告警

  

  ---

  **📍** **Topic** **4:** **dsp_http_log** **(富化后)**

  

  **位置**：10001001/default/dsp_http_log

  

  **作用**：富化后的HTTP日志（已添加用户名、部门、资产等信息）

  

  **代码证据**：

  # pulsar_output.yaml 第11-14行

  output_topic_cfg:

    tenant: "$tenant"           # 10001001

    namespace: "default"

    topic: "$dstLogType"        # dsp_http_log

  

  **大白话**：

  - 富化服务处理完后，把"贴好标签"的日志放到这里

  - 这是**富化后的中转站**，日志已经有完整信息了

  - 消费者：datalake-ingest-backend-prc（数据湖入库服务）

  

  **为什么要监控**：

  - 如果这里堆积 → 入库服务写ClickHouse太慢

  - 堆积严重 → 查询不到最新日志，影响实时告警

  

  ---

  **📍** **Topic** **5:** **dsp_business_http_log** **(业务日志)**

  

  **位置**：10001001/default/dsp_business_http_log

  

  **作用**：业务HTTP日志（富化后，供多个引擎消费）

  

  **代码证据**：

  从Topic列举.md可知，有4个消费者：

  - dsc-data-ingest(ck_risk_http_log) - 入库CK风险日志表

  - dsp-subscribe-apiMerge-http-log - API合并服务

  - alpp - ALPP引擎

  - dsc-data-ingest(pg_http_business) - 入库PG业务日志表

  

  **大白话**：

  - 富化服务把业务相关的HTTP日志单独放到这里

  - 这是**多路消费的分发点**，4个服务同时消费

  - 任何一个消费者慢了，都会导致堆积

  

  **为什么要监控**：

  - 如果这里堆积 → 4个消费者中至少有1个处理不过来

  - 堆积严重 → 风险检测、API分析、告警都会延迟