### 对外提供接口
```
  对外接口定义：
    1. 路由注册结构 (config_manager/config_server/config_server.go)

  func RegisterRouter(r *gin.Engine) {
        // 路由全路径： /chat_work/api/v1/模块/子模块/方法 eg: /dataIdentifyConfig/v1/labels
        r.GET("/metrics", gin.WrapH(promhttp.Handler()))  // Prometheus指标暴露端点
        health_check.RegisterHealthz(r)                   // 健康检查端点
        chatWorkRoot := r.Group("/dataIdentifyConfig")
        {
                apiV1 := chatWorkRoot.Group("/v1")
                {
                        RegisterChatWorkRouter(apiV1)              // 基础配置API
                        syncGroup := apiV1.Group("/sync")
                        {
                                RegisterConfigSyncRouter(syncGroup)    // 配置同步API
                        }
                }
        }
  }



  文件: config_manager/config_server/router.go

  func RegisterChatWorkRouter(group *gin.RouterGroup) {
        // 测试gin
        group.GET("/test_gin", testGin)                              // GET 
  /dataIdentifyConfig/v1/test_gin
        group.GET("/labels", getLabels)                              // GET 
  /dataIdentifyConfig/v1/labels
        group.GET("/labels/labelCategoryRelation", getRelation)      // GET 
  /dataIdentifyConfig/v1/labels/labelCategoryRelation  
        group.GET("/allTemplateIds", getAllTemplateIds)              // GET 
  /dataIdentifyConfig/v1/allTemplateIds
        group.GET("/categories", getCategories)                      // GET 
  /dataIdentifyConfig/v1/categories
        group.GET("/industry", getIndustry)                          // GET 
  /dataIdentifyConfig/v1/industry
        group.POST("/delWithPrefixes", delWithPrefixes)              // POST 
  /dataIdentifyConfig/v1/delWithPrefixes
  }




  C. 配置同步API (/dataIdentifyConfig/v1/sync/*)

  文件: config_manager/config_server/sync_router.go

  func RegisterConfigSyncRouter(group *gin.RouterGroup) {
        group.PUT("/sync_message", authoritarian.AuthMiddleware(), syncMsg)      // PUT 
  /dataIdentifyConfig/v1/sync/sync_message
        group.GET("/allTemplateIds", authoritarian.AuthMiddleware(), getAllTemplateIds)  // GET 
  /dataIdentifyConfig/v1/sync/allTemplateIds  
        group.GET("/require_sync", GetCategoryTreeInfo)                         // GET 
  /dataIdentifyConfig/v1/sync/require_sync
  }




总共监控十个指标

  监控范围：
  - /dataIdentifyConfig/v1/test_gin ✓
  - /dataIdentifyConfig/v1/labels ✓
  - /dataIdentifyConfig/v1/labels/labelCategoryRelation ✓
  - /dataIdentifyConfig/v1/allTemplateIds ✓
  - /dataIdentifyConfig/v1/categories ✓
  - /dataIdentifyConfig/v1/industry ✓
  - /dataIdentifyConfig/v1/delWithPrefixes ✓
  - /dataIdentifyConfig/v1/sync/sync_message ✓
  - /dataIdentifyConfig/v1/sync/allTemplateIds ✓
  - /dataIdentifyConfig/v1/sync/require_sync ✓

```

### 埋点代码梳理

##### 指标定义
```
  TaskDispatch 项目 - 指标十七完整梳理

  一、指标定义

  文件: D:\DSP\grafana\taskdispatch\pkg\monitor\metirc.go
  行号: 10-18

  指标十七：API时延Histogram

  // 指标十七: taskdispatch 对外提供API时延 Histogram
  var APILatencyHistogram = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
                Name:    "taskdispatch_api_latency_seconds",  // 指标名称
                Help:    "TaskDispatch API request latency in seconds.",  // 指标描述
                Buckets: []float64{0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10},
  // 分桶配置
        },
        []string{"endpoint", "status", "method"},  // 标签维度
  )

  参数详解：
  - prometheus.NewHistogramVec: 创建向量Histogram，支持多标签
  - Name: "taskdispatch_api_latency_seconds" -
  遵循Prometheus命名规范：application_metric_type_unit
  - Buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10] -
  12个桶从5ms到10s
    Buckets: []float64{0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10}

  Bucket的原理:
  桶边界: [0, 0.005]  (0.005, 0.01]  (0.01, 0.025]  ...  (5, 10]  (10, +∞)
  观测值: 0.003s   ✓ 桶1
  观测值: 0.008s   ✓ 桶2
  观测值: 0.015s   ✓ 桶3
  观测值: 0.8s     ✓ 桶6
  观测值: 12s      ✓ 桶12 (最后一个无限桶)

  桶数量统计: 实际有 13个桶 (12个有限桶 + 1个无限桶)

    - 0.005s (5ms): 快速响应
    - 0.01s (10ms): 正常响应
    - 0.025s (25ms): 轻慢响应
    - 0.05s (50ms): 可接受响应
    - 0.1s (100ms): 预警响应
    - 0.25s (250ms): 慢响应
    - 0.5s (500ms): 异常响应
    - 1s: 严重异常
    - 2.5s, 5s, 10s: 极端异常
  - 标签: {"endpoint", "status", "method"} -
  支持按接口、状态码、HTTP方法维度监控

  
  指标十八：API请求总数Counter

  // 指标十八: taskdispatch 对外提供API请求总数 Counter
  var APIRequestCounter = prometheus.NewCounterVec(
        prometheus.CounterOpts{
                Name: "taskdispatch_api_requests_total",  // 指标名称
                Help: "Total number of API requests, labeled by endpoint, status code, and 
  method.",  // 指标描述
        },
        []string{"endpoint", "status", "method"},  // 标签维度
  )

  参数详解：
  - prometheus.NewCounterVec: 创建向量Counter，支持多标签
  - Name: "taskdispatch_api_requests_total" - Counter类型通常以_total结尾
  - 标签: 与Histogram保持一致的标签维度，便于关联分析
  ---
  二、指标注册

  文件: D:\DSP\grafana\taskdispatch\pkg\monitor\metirc.go
  行号: 48-53

  自动注册机制

  func init() {
        prometheus.MustRegister(
                APILatencyHistogram,  // 注册时延指标
                APIRequestCounter,    // 注册计数指标
        )
  }

  为什么用init()函数：
  - 自动执行: 包导入时自动执行，无需手动调用
  - 注册时机: 确保指标在使用前已经注册到Prometheus
  - prometheus.MustRegister:
  如果注册失败会panic，确保应用不会在没有指标的情况下运行


  ---
  三、指标记录函数

  文件: D:\DSP\grafana\taskdispatch\pkg\monitor\metirc.go
  行号: 31-37

  // 指标十七: taskdispatch 对外提供API时延
  func RecordAPILatency(endpoint string, statusCode int, method string, 
  duration time.Duration) {
        APILatencyHistogram.WithLabelValues(
                endpoint,
                strconv.Itoa(statusCode),    // 状态码转字符串
                method,
        ).Observe(duration.Seconds())     // 记录时延值（秒）
  }

  参数说明:
  - endpoint: API路由模板（如 /dataIdentifyConfig/v1/labels）
  - statusCode: HTTP状态码（200/404/500等）
  - method: HTTP方法（GET/POST/PUT/DELETE）
  - duration: 请求总耗时（Go的time.Duration类型）

最后会生成

 taskdispatch_api_requts_total{endpoint="/api/users",method="GET",status="200"} 5
 

```

### 中间件实现

  ##### pkg/middleware/metrics_middleware.go

```
  1. 监控范围控制函数

  func isConfigAPI(path string) bool {
        // 排除健康检查端点
        if path == "/healthz" {
                return false
        }
        // 排除metrics暴露端点
        if path == "/metrics" {
                return false
        }
        // 只监控配置管理API
        return strings.HasPrefix(path, "/dataIdentifyConfig/v1/")
  }

  设计理由：
  - 排除健康检查: /healthz是监控系统频繁调用的端点，监控无意义
  - 排除metrics端点: /metrics是指标暴露端点，避免自我监控循环
  - 专注业务: 只监控/dataIdentifyConfig/v1/*核心业务API

  2. 核心中间件函数

  func MetricsMiddleware() gin.HandlerFunc {
        return func(c *gin.Context) {
                // 记录请求开始时间
                startTime := time.Now()
                requestPath := c.Request.URL.Path

                // 处理请求（继续执行后续的中间件和handler）
                c.Next()

                // 只对配置管理API进行指标记录
                if isConfigAPI(requestPath) {
                        // 获取endpoint（使用路由模板路径）
                        endpoint := c.FullPath()
                        if endpoint == "" {
                                // 如果FullPath为空（404等情况），使用实际请求路径
                                endpoint = requestPath
                        }

                        // 获取状态码和方法
                        statusCode := c.Writer.Status()
                        method := c.Request.Method

                        // 计算请求时延
                        duration := time.Since(startTime)

                        // 指标十七：记录API时延
                        monitor.RecordAPILatency(endpoint, statusCode, method, duration)

                        // 指标十八：记录API请求总数
                        monitor.RecordAPIRequest(endpoint, statusCode, method)
                }
        }
  }

  关键函数选择理由：

  time.Now() 和 time.Since()

  startTime := time.Now()
  // ... 业务逻辑处理
  duration := time.Since(startTime)
  - time.Now(): 获取当前系统时间，纳秒级精度
  - time.Since(): 计算从指定时间到现在的时间差，返回time.Duration
  - 高精度: 纳秒级别，满足API性能监控精度要求

  c.Next()

  c.Next()
  - Gin中间件核心函数: 执行后续所有中间件和业务处理函数
  - 阻塞执行: 等待所有后续逻辑执行完成后返回
  - 完整时延: 确保记录的是从请求开始到响应结束的完整时间

  c.FullPath() vs c.Request.URL.Path

  endpoint := c.FullPath()
  if endpoint == "" {
        endpoint = requestPath
  }
  - c.FullPath(): 获取路由模板路径，如/dataIdentifyConfig/v1/labels/:id
  - c.Request.URL.Path: 获取实际请求路径，如/dataIdentifyConfig/v1/labels/123
  - 优先使用模板: 模板路径更便于聚合分析，404等路由失败时回退到实际路径

  c.Writer.Status()

  statusCode := c.Writer.Status()
  - 获取响应状态码: 200、400、500等
  - 用于标签分析: 按成功/失败分类监控API状态

  strconv.Itoa(statusCode)

  monitor.RecordAPILatency(endpoint, strconv.Itoa(statusCode), method, duration)
  - 类型转换: 将int状态码转为string
  - Prometheus要求: 标签值必须是string类型
```





### 中间件注册
  文件: D:\DSP\grafana\taskdispatch\config_manager\config_server\gin.go
  行号: 29-98
  
```
  func InitGin() *gin.Engine {
        gin.SetMode(gin.ReleaseMode)

        r := gin.New()

        setGinEngine(r)

        r.Use(ginLogger(), ginRecovery())
        r.Use(middleware.MetricsMiddleware()) // 指标十七和十八：API监控中间件
        return r
  }
```

  注册顺序重要性：
  1. ginLogger(): 记录访问日志
  2. ginRecovery(): 异常恢复
  3. MetricsMiddleware(): 性能监控

  为什么放在最后：
  - 完整时延: 确保记录的是包括日志和恢复处理的完整时间
  - 错误捕获: 可以记录到所有可能的错误响应
    
    
### 路由注册和Metrics端点暴露

  文件: D:\DSP\grafana\taskdispatch\config_manager\main.go
  行号: 14-29

```
	// 初始化Prometheus，指标暴露
	go func() {
		http.Handle("/metrics", promhttp.Handler())
		logger.Info("Metrics endpoint started on :9300/metrics")
		if err := http.ListenAndServe("0.0.0.0:9300", nil); err != nil {
			logger.Error("Failed to start metrics server: %v", err)
		}
	}()
```

  
埋点流程
```
---
  `七、完整执行流程链路`

  1. `应用启动`
     `↓`
  2. `init()函数自动注册Prometheus指标`
     `↓`
  3. `InitGin()创建引擎并注册MetricsMiddleware`
     `↓`
  4. `HTTP请求到达 → MetricsMiddleware`
     `↓`
  5. `startTime := time.Now() (记录开始时间)`
     `↓`
  6. `c.Next() (执行业务逻辑)`
     `↓`
  7. `计算duration := time.Since(startTime)`
     `↓`
  8. `isConfigAPI()判断是否监控`
     `↓`
  9. `获取endpoint、statusCode、method`
     `↓`
  10. `RecordAPILatency()记录到Histogram`
     `↓`
  11. `RecordAPIRequest()记录到Counter`
      `↓`
  12. `通过/metrics端点暴露给Prometheus`



 
  `---`
`路由注册机制`
 
  `10.2 过滤规则`

  `不统计的路径:`
  - `/healthz - 健康检查端点`
  - `/metrics - Prometheus采集端点`

  `统计的路径:`
  - `/dataIdentifyConfig/v1/* 下的所有业务API`

  `10.3 标签维度`

  `| 标签       | 来源                | 示例值`
  `|`
  `|----------|-------------------|-------------------------------|`
  `| endpoint | c.FullPath()      | /dataIdentifyConfig/v1/labels |`
  `| status   | c.Writer.Status() | 200, 404, 500                 |`
  `| method   | c.Request.Method  | GET, POST, PUT, DELETE        |`

  `10.4 Bucket时间分布`

  `0.005秒 (5ms)`
  `0.01秒 (10ms)`
  `0.025秒 (25ms)`
  `0.05秒 (50ms)`
  `0.1秒 (100ms)`
  `0.25秒 (250ms)`
  `0.5秒 (500ms)`
  `1秒`
  `2.5秒`
  `5秒`
  `10秒`
  `+Inf (无穷大)`

  `---`
  `十一、Grafana查询示例`

  `查询1：计算QPS（每秒请求数）`

  `rate(taskdispatch_api_latency_seconds_count{endpoint="/dataIdentifyCon`
  `fig/v1/labels"}[5m])`

  `查询2：计算P99时延`

  `histogram_quantile(0.99,` 
    `rate(taskdispatch_api_latency_seconds_bucket{`
      `endpoint="/dataIdentifyConfig/v1/labels"`
    `}[5m])`
  `)`

  `查询3：成功请求的平均时延`

  `rate(taskdispatch_api_latency_seconds_sum{status="200"}[5m])`
  `/`
  `rate(taskdispatch_api_latency_seconds_count{status="200"}[5m])`

  `---`
  `十二、验证命令`

  # `查看metrics输出`
  `curl http://0.0.0.0:9876/metrics | grep`
  `taskdispatch_api_latency_seconds`

  # `预期输出示例：`
  # `HELP taskdispatch_api_latency_seconds TaskDispatch API request` 
  `latency in seconds.`
  # `TYPE taskdispatch_api_latency_seconds histogram`
  # `taskdispatch_api_latency_seconds_bucket{endpoint="/dataIdentifyConfi`
  `g/v1/labels",method="GET",status="200",le="0.01"} 5`
  # `taskdispatch_api_latency_seconds_bucket{endpoint="/dataIdentifyConfi`
  `g/v1/labels",method="GET",status="200",le="0.1"} 42`
  # `taskdispatch_api_latency_seconds_bucket{endpoint="/dataIdentifyConfi`
  `g/v1/labels",method="GET",status="200",le="+Inf"} 100`
  # `taskdispatch_api_latency_seconds_sum{endpoint="/dataIdentifyConfig/v`
  `1/labels",method="GET",status="200"} 3.456`
  # `taskdispatch_api_latency_seconds_count{endpoint="/dataIdentifyConfig`
  `/v1/labels",method="GET",status="20`

### `对外指标梳理（/dataIdentifyConfig/v1/*）`
```
  API接口定义位置

  位置1：主业务路由（7个接口）

  文件：
  D:\DSP\grafana\taskdispatch\config_manager\config_server\router.go

  函数： RegisterChatWorkRouter(r *gin.RouterGroup) （第20-30行）

  // 第20-30行
  func RegisterChatWorkRouter(r *gin.RouterGroup) {
        r.GET("/test_gin", testGin)                                      // 1.
   测试接口
        r.GET("/labels", getLabels)                                      // 2.
   获取标签
        r.GET("/labels/labelCategoryRelation", getRelation)              // 3.
   标签分类关系
        r.GET("/allTemplateIds", getAllTemplateIds)                      // 4.
   获取所有模板ID
        r.GET("/categories", getCategories)                              // 5.
   获取分类
        r.GET("/industry", getIndustry)                                  // 6.
   获取行业
        r.POST("/delWithPrefixes", delWithPrefixes)                      // 7.
   删除前缀
  }

  ---
  位置2：同步路由（3个接口）

  文件： D:\DSP\grafana\taskdispatch\config_manager\config_server\sync_r
  outer.go

  函数： RegisterConfigSyncRouter(syncGroup *gin.RouterGroup)
  （第16-21行）

  // 第16-21行
  func RegisterConfigSyncRouter(syncGroup *gin.RouterGroup) {
        syncGroup.PUT("/sync_message", authoritarian.AuthMiddleware(),
  syncMsg)           // 8. 同步消息
        syncGroup.GET("/allTemplateIds", authoritarian.AuthMiddleware(),
  getAllTemplateIds) // 9. 获取所有模板ID（sync版本）
        syncGroup.GET("/require_sync", GetCategoryTreeInfo)
               // 10. 获取同步信息
  }

  ---
  位置3：路由注册入口

  文件： D:\DSP\grafana\taskdispatch\config_manager\config_server\config
  _server.go

  函数： RegisterRouter(r *gin.Engine) （第19-33行）

  这个函数将上述两个路由组注册到Gin引擎：

  // 第19-33行
  func RegisterRouter(r *gin.Engine) {
        // 路由全路径： /chat_work/api/v1/模块/子模块/方法
        health_check.RegisterHealthz(r)

        chatWorkRoot := r.Group("/dataIdentifyConfig")       // 根路径
        {
                apiV1 := chatWorkRoot.Group("/v1")               // v1版本分组
                {
                        RegisterChatWorkRouter(apiV1)                // 
  注册主业务路由（7个接口）
                        syncGroup := apiV1.Group("/sync")            // sync子分组
                        {
                                RegisterConfigSyncRouter(syncGroup)      // 注册同步路由（3个接口）
                        }
                }
        }
  }

  ---
  完整路径构成

  根据路由分组结构，完整路径为：

  /dataIdentifyConfig  (chatWorkRoot)
    └─ /v1  (apiV1)
        ├─ /test_gin                          →
  /dataIdentifyConfig/v1/test_gin
        ├─ /labels                            →
  /dataIdentifyConfig/v1/labels
        ├─ /labels/labelCategoryRelation      →
  /dataIdentifyConfig/v1/labels/labelCategoryRelation
        ├─ /allTemplateIds                    →
  /dataIdentifyConfig/v1/allTemplateIds
        ├─ /categories                        →
  /dataIdentifyConfig/v1/categories
        ├─ /industry                          →
  /dataIdentifyConfig/v1/industry
        ├─ /delWithPrefixes                   →
  /dataIdentifyConfig/v1/delWithPrefixes
        └─ /sync  (syncGroup)
            ├─ /sync_message                  →
  /dataIdentifyConfig/v1/sync/sync_message
            ├─ /allTemplateIds                →
  /dataIdentifyConfig/v1/sync/allTemplateIds
            └─ /require_sync                  →
  /dataIdentifyConfig/v1/sync/require_sync

  ---
  总结

  - router.go（第20-30行）：定义了7个主业务API
  - sync_router.go（第16-21行）：定义了3个同步相关API
  - config_server.go（第19-33行）：将上述路由组合注册，形成完整路径

  所有接口都在 /dataIdentifyConfig/v1/
  路径下，这也是为什么我们的中间件使用 strings.HasPrefix(path, 
  "/dataIdentifyConfig/v1/") 来过滤需要监控的API。
