
### Taskinference对外提供接口

  根据` /root/dsp/taskinference/task_actuator/internal/router/register_router.go：
  总共有六个对外提供接口，只统计`/task_actuator/api/v1开头的这些接口
  
```

  2. 业务API接口（6个）

  apiVersion1 := r.Group("/task_actuator/api/v1")
  {
      apiVersion1.POST("/run_static_task", v1.TaskStaticRun)           // 1
      apiVersion1.POST("/run_dynamic_task", v1.TaskDynamicRun)         // 2
      apiVersion1.POST("/chat_model", ...)                              // 3
      apiVersion1.POST("/stream/chat", ...)                             // 4
      apiVersion1.POST("/chat_model_risk", v1.CommonChatJob)           // 5
      apiVersion1.POST("/run_category_plus", v1.TaskCategoryPlusRun)   // 6
  }

  完整路径：
  3. /task_actuator/api/v1/run_static_task
  4. /task_actuator/api/v1/run_dynamic_task
  5. /task_actuator/api/v1/chat_model
  6. /task_actuator/api/v1/stream/chat
  7. /task_actuator/api/v1/chat_model_risk
  8. /task_actuator/api/v1/run_category_plus
```

### 埋点流程
```

  HTTP请求 → GinLoggerHandler中间件 → c.Next() → 业务处理 → 响应
       ↓                                                    ↓
    记录开始时间                                        计算时延
       ↓                                                    ↓
    isInferenceAPI()检查 → 符合→ 记录指标十七和十八
```






