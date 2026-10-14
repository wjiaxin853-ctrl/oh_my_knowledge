![[截屏2025-12-07 16.31.48.png|500]]


```
  ---
  📋 指标十八完整代码走读

  阶段一：指标定义

  文件: /root/dsp/taskinference/task_actuator/internal/monitor/metrics.go

  1. 指标声明（176-183行）

  // 指标十八: taskinference对外提供api接口的成功率 Counter
  APIRequestCounter = prometheus.NewCounterVec(
      prometheus.CounterOpts{
          Name: "taskinference_api_requests_total",
          Help: "Total number of API requests, labeled by endpoint and status code.",
      },
      []string{"endpoint", "status"},  // 标签维度：接口路径、状态码
  )

  说明：
  - 指标类型: CounterVec - 带标签的计数器
  - 指标名称: taskinference_api_requests_total
  - 标签1: endpoint - 记录API接口路径
  - 标签2: status - 记录HTTP状态码
  - 用途: 统计每个接口的调用次数，用于计算成功率

  ---
  阶段二：操作函数定义

  文件: /root/dsp/taskinference/task_actuator/internal/monitor/metrics.go

  2. 操作函数（281-287行）

  // 指标十八: RecordAPIRequest 记录API请求
  func RecordAPIRequest(endpoint string, statusCode int) {
      APIRequestCounter.WithLabelValues(
          endpoint,                    // 标签1：接口路径
          strconv.Itoa(statusCode),   // 标签2：状态码转字符串
      ).Inc()                         // 计数器加1
  }

  说明：
  - 参数1: endpoint - API端点路径（如 /task_actuator/api/v1/run_static_task）
  - 参数2: statusCode - HTTP状态码（如 200, 422, 500）
  - 操作: 将状态码整数转为字符串，然后对应的计数器+1
  - 调用时机: 每次API请求完成后调用一次

  ---
  阶段三：Prometheus注册

  文件: /root/dsp/taskinference/task_actuator/internal/monitor/metrics.go

  3. 注册到Prometheus（289-310行）

  // 注册指标
  func init() {
      prometheus.MustRegister(
          APILatencyHistogram,           // 指标十七
          APIRequestCounter,             // 指标十八 ← 在这里注册
      )
  }

  说明：
  - 执行时机: 服务启动时自动执行（init() 函数）
  - 作用: 将指标十八注册到Prometheus，使其可以被抓取
  - 顺序: 在所有指标的最后注册

  ---
  阶段四：中间件过滤逻辑

  文件: /root/dsp/taskinference/task_actuator/middleware/handler.go

  4. 判断是否为推理API（13-17行）

  // 指标十四|指标十七|指标十八 判断是否为推理API
  // 只记录 /task_actuator/api/v1/* 路径下的请求
  func isInferenceAPI(path string) bool {
      return strings.HasPrefix(path, "/task_actuator/api/v1/")
  }

  说明：
  - 作用: 过滤出需要统计的API接口
  - 规则: 只统计以 /task_actuator/api/v1/ 开头的路径
  - 结果:
    - ✅ /task_actuator/api/v1/run_static_task → 返回 true
    - ❌ /check → 返回 false

  ---
  阶段五：中间件埋点（核心）

  文件: /root/dsp/taskinference/task_actuator/middleware/handler.go

  5. Gin中间件主逻辑（19-55行）

  // GinLoggerHandler 日志中间件
  func GinLoggerHandler() gin.HandlerFunc {
      return func(c *gin.Context) {
          // 步骤1: 记录请求开始时间（22-27行）
          startTime := time.Now()
          requestPath := c.Request.URL.Path
          if c.Request.URL.RawQuery != "" {
              requestPath = requestPath + "?" + c.Request.URL.RawQuery
          }

          // 步骤2: 执行实际的API处理逻辑（29-30行）
          c.Next()  // ← 这里调用 TaskStaticRun/TaskDynamicRun 等处理函数

          // 步骤3: 计算请求处理时长（32-33行）
          duration := time.Since(startTime)

          // 步骤4: 获取HTTP响应状态码（37-38行）
          responseCode := c.Writer.Status()
          logger.Info("%s - \"%s %v %v\" %v %s",
              c.ClientIP(),
              c.Request.Method,
              requestPath,
              c.Request.Proto,
              responseCode,
              http.StatusText(responseCode))

          // 步骤5: 判断是否需要记录指标（40-53行）
          if isInferenceAPI(c.Request.URL.Path) {  // ← 过滤：只统计推理API
              endpoint := c.FullPath()
              if endpoint == "" {
                  endpoint = c.Request.URL.Path
              }

              // 步骤6: 记录指标十七（时延）
              monitor.RecordAPILatency(endpoint, responseCode, duration)

              // 步骤7: 记录指标十八（请求计数）← 指标十八埋点位置
              monitor.RecordAPIRequest(endpoint, responseCode)
          }
      }
  }

  说明：
  - 第22行: 记录请求开始时间
  - 第30行: 执行业务逻辑（API Handler）
  - 第33行: 计算请求耗时
  - 第37行: 获取HTTP状态码
  - 第38行: 打印访问日志
  - 第42行: 判断是否为推理API（调用 isInferenceAPI）
  - 第43-46行: 获取接口端点路径
  - 第49行: 记录指标十七（时延）
  - 第52行: 记录指标十八（请求计数） ← 核心埋点

  ---
  阶段六：路由注册

  文件: /root/dsp/taskinference/task_actuator/internal/router/register_router.go

  6. 注册被统计的6个API（10-22行）

  // RegisterRouter initialize routing information
  func RegisterRouter(r *gin.Engine) {
      r.GET("/check", v1.TaskCheck)  // ❌ 不统计（不在 /task_actuator/api/v1/ 下）

      apiVersion1 := r.Group("/task_actuator/api/v1")
      {
          // ✅ 以下6个接口都会被指标十八统计
          apiVersion1.POST("/run_static_task", v1.TaskStaticRun)       // 接口1
          apiVersion1.POST("/run_dynamic_task", v1.TaskDynamicRun)     // 接口2
          apiVersion1.POST("/chat_model", authoritarian.AuthMiddleware(), v1.CommonChatJob) // 接口3
          apiVersion1.POST("/stream/chat", authoritarian.AuthMiddleware(),
  limit.RateLimiter(limit.LimiterCfg), v1.StreamChatJob)  // 接口4
          apiVersion1.POST("/chat_model_risk", v1.CommonChatJob)        // 接口5
          apiVersion1.POST("/run_category_plus", v1.TaskCategoryPlusRun)// 接口6
      }
  }

  说明：
  - 第12行: /check 不会被统计（不符合路径前缀）
  - 第15-20行: 6个业务API接口，全部会被统计

  ---
  阶段七：API Handler示例

  文件: /root/dsp/taskinference/task_actuator/internal/router/api/v1/api.go

  7. 静态分类API示例（22-36行）

  // TaskStaticRun POST 静态任务流
  func TaskStaticRun(c *gin.Context) {
      var taskParams Configs.TaskParams
      if err := c.ShouldBindJSON(&taskParams); err != nil {
          c.JSON(http.StatusUnprocessableEntity, gin.H{"code": public.FailCode, "error":
  err.Error()})
          // ↑ 返回状态码 422，中间件会记录：
          //   monitor.RecordAPIRequest("/task_actuator/api/v1/run_static_task", 422)
          return
      }

      logger.Info("TaskStaticRun taskParams: %+v", taskParams)
      // 2. 执行任务
      resultStr := staticdata.HandleStaticTask(taskParams)
      // 5. 返回结果
      logger.Info("TaskStaticRun resultStr: %+v", resultStr)
      c.JSON(http.StatusOK, gin.H{"code": public.SuccessCode, "message": resultStr})
      // ↑ 返回状态码 200，中间件会记录：
      //   monitor.RecordAPIRequest("/task_actuator/api/v1/run_static_task", 200)
  }

  说明：
  - 第26行: 参数绑定失败 → 返回 422 → 指标十八记录失败次数
  - 第35行: 处理成功 → 返回 200 → 指标十八记录成功次数

  ---
  📊 完整执行流程图


  客户端发起请求: POST /task_actuator/api/v1/run_static_task
      ↓
  【1】Gin Router 路由匹配
      └─ 匹配到 TaskStaticRun Handler
      ↓
  【2】中间件 GinLoggerHandler - 请求前
      ├─ handler.go:22 - 记录 startTime = time.Now()
      └─ handler.go:23-27 - 构建 requestPath
      ↓
  【3】handler.go:30 - c.Next() 执行业务逻辑
      ├─ api.go:23 - TaskStaticRun 开始
      ├─ api.go:25 - 参数绑定 ShouldBindJSON
      │   ├─ 成功 → 继续
      │   └─ 失败 → api.go:26 返回 422
      ├─ api.go:32 - 调用 HandleStaticTask 处理业务
      └─ api.go:35 - 返回 200 OK
      ↓
  【4】中间件 GinLoggerHandler - 请求后
      ├─ handler.go:33 - 计算 duration = time.Since(startTime)
      ├─ handler.go:37 - 获取 responseCode = c.Writer.Status()
      │                  (可能是 200/422/500)
      ├─ handler.go:38 - 打印访问日志
      └─ handler.go:42 - if isInferenceAPI(path):
          ├─ handler.go:15 - 检查路径前缀
          │   └─ "/task_actuator/api/v1/run_static_task"
          │      匹配 "/task_actuator/api/v1/" → 返回 true
          ├─ handler.go:43-46 - 获取 endpoint
          │   └─ endpoint = "/task_actuator/api/v1/run_static_task"
          ├─ handler.go:49 - monitor.RecordAPILatency(...)  ← 指标十七
          └─ handler.go:52 - monitor.RecordAPIRequest(endpoint, responseCode)
              ↓
              【5】metrics.go:282 - RecordAPIRequest 函数
                  ├─ metrics.go:283-285 - APIRequestCounter.WithLabelValues(
                  │
  endpoint="/task_actuator/api/v1/run_static_task",
                  │                           status="200"
                  │                       )
                  └─ metrics.go:286 - .Inc()  ← 计数器 +1
                      ↓
                      【6】Prometheus内部更新
                          └─ taskinference_api_requests_total{
                                 endpoint="/task_actuator/api/v1/run_static_task",
                                 status="200"
                             } 增加 1
      ↓
  【7】返回响应给客户端
      ↓
  【8】Prometheus定时抓取（每15秒）
      ├─ GET http://taskinference:9300/metrics
      └─ 获取所有指标数据
      ↓
  【9】Grafana查询展示
      └─ PromQL: sum(rate(taskinference_api_requests_total[5m]))

  ---
  🔢 代码统计

  | 项目          | 文件                 | 行号      | 说明                           |
  |-------------|--------------------|---------|------------------------------|
  | 指标定义        | metrics.go         | 176-183 | 声明 APIRequestCounter         |
  | 操作函数        | metrics.go         | 281-287 | RecordAPIRequest()           |
  | 注册指标        | metrics.go         | 308     | prometheus.MustRegister(...) |
  | 过滤函数        | handler.go         | 13-17   | isInferenceAPI()             |
  | 埋点位置        | handler.go         | 52      | monitor.RecordAPIRequest()   |
  | 路由注册        | register_router.go | 15-20   | 6个被统计的API                    |
  | API Handler | api.go             | 22-36等  | 业务处理，返回状态码                   |

  ---
  📝 关键要点总结

  埋点逻辑顺序

  1. 服务启动 → metrics.go:308 注册指标
  2. 请求到达 → handler.go:22 开始计时
  3. 业务处理 → handler.go:30 调用 c.Next()
  4. 获取状态码 → handler.go:37 获取响应状态
  5. 过滤判断 → handler.go:42 检查是否推理API
  6. 埋点记录 → handler.go:52 调用 RecordAPIRequest
  7. 计数器+1 → metrics.go:286 Inc()

  数据维度

  - endpoint维度: 区分6个不同的API接口
  - status维度: 区分成功(2xx)、客户端错误(4xx)、服务器错误(5xx)

  成功率计算

  成功率 = 状态码为2xx的请求数 / 总请求数 * 100%

  ---
  完整的指标十八代码走读完成！ ✅

```