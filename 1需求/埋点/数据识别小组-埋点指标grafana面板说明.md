### 指标定义
![[截屏2025-12-25 00.22.29.png]]


### promethus原理

![[Pasted image 20251226005316.png]]

### Grafan面板说明

#### 数据识别模块（模型平台）

**埋点服务**：针对taskinference还有taskdispatch服务
![[截屏2025-12-25 00.21.51.png]]
#### 数据识别模块（应用平台）

**埋点服务**：针对apollo服务

#### 接口筛选功能
![[截屏2025-12-25 00.27.40.png|300]]


### 埋点指标说明

#### 指标十四：【taskinference】动态任务（API）推理失败原因统计

说明：是针对动态任务推理涉及到的业务错误码进行个数统计

```
  错误码体系 (constants.go:44-65):
  var (
        Success                = TaskError{Code: 0, Msg: "成功"}
        UnknownErr             = TaskError{Code: 1, Msg: "未知错误"}
        GptErr                 = TaskError{Code: 1001, Msg: "GPT服务异常"}
        BizCodeErr             = TaskError{Code: 1002, Msg: "代码逻辑异常"}
        EmptyJsonErr           = TaskError{Code: 1003, Msg: "返回空json"}
        FieldNameMatchErr      = TaskError{Code: 1004, Msg: "字段名不匹配"}
        CategoryErr            = TaskError{Code: 1005, Msg: "结果分类错误"}
        DataCorruption         = TaskError{Code: 1006, Msg: "数据损坏"}
        GetCategoryResultError = TaskError{Code: 1007, Msg: "获取分类结果错误"}
        // ... 更多错误定义
```
![[截屏2025-12-25 00.23.07.png]]
#### 指标十五：【taskinference】静态任务（数据库字段）推理失败原因统计

说明：是针对静态推理涉及到的业务错误码进行个数统计

```
  错误码体系 (constants.go:44-65):
  var (
        Success                = TaskError{Code: 0, Msg: "成功"}
        UnknownErr             = TaskError{Code: 1, Msg: "未知错误"}
        GptErr                 = TaskError{Code: 1001, Msg: "GPT服务异常"}
        BizCodeErr             = TaskError{Code: 1002, Msg: "代码逻辑异常"}
        EmptyJsonErr           = TaskError{Code: 1003, Msg: "返回空json"}
        FieldNameMatchErr      = TaskError{Code: 1004, Msg: "字段名不匹配"}
        CategoryErr            = TaskError{Code: 1005, Msg: "结果分类错误"}
        DataCorruption         = TaskError{Code: 1006, Msg: "数据损坏"}
        GetCategoryResultError = TaskError{Code: 1007, Msg: "获取分类结果错误"}
        // ... 更多错误定义
```
![[截屏2025-12-25 00.23.20.png]]

#### 指标十六：【taskinference】调用外部接口时延

针对以下七个接口
```
  1. 缓存配置服务 (4个接口)

  文件: pkg/client_utils/cache_client.go

  | 接口     | 方法  | URL                                      | 调用位置
     |
  |--------|-----|------------------------------------------|--------------------|
  | 获取配置   | GET | /configs                                 | cache_client.go:32 |
  | 批量获取模板 | GET | /templateIds?templateIds={ids}           | cache_client.go:54
   |
  | 批量获取标签 | GET | /labelIds?labelIds={ids}                 | cache_client.go:66
   |
  | 获取单个模板 | GET | /templates?templateId={id}&token={token} | cache_client.go:78
   |

  2. GPU 模型推理服务 (1个接口)

  文件: pkg/client_utils/gpt_client.go
  - 接口: POST http://{GPU_IP}:{GPU_Port}{GPU_Uri}
  - 调用位置:
    - internal/router/service/common_chat/common_chat_task.go:46
    - internal/router/service/common_chat/common_chat_task.go:128
    - internal/router/service/dynamic_data/api_predict_tasks/api_predict_task.go:279
    - internal/router/service/dynamic_data/api_predict_tasks/api_predict_task.go:416
    - internal/router/service/category_plus/reason/category_plus_reason.go:495
    - internal/router/service/static_data/reason/task_category.go:202
    - internal/router/service/static_data/reason/task_category.go:585
    - internal/router/service/static_data/reason/task_category.go:653

  3. 向量检索服务 (1个接口)

  文件: pkg/client_utils/vector_client.go
  - 接口: POST /vsearch/field_label
  - 调用位置: vector_client.go:86

  4. 认证服务 (1个接口)

  文件: internal/router/service/authoritarian/authoritarian.go
  - 接口: POST http://{TokenCheckHost}:{TokenCheckPort}/{TokenCheckUrl}?token={token}
  - 调用位置: authoritarian.go:72
```
![[截屏2025-12-25 00.20.14.png]]
#### 指标十九：【taskinference】调用外部接口成功率

成功状态码：2XX
统计方法：统计所有接口的状态码还有次数，在grafana面板计算成功率，
**公式：某一个接口2XX状态码成功次数/某一个接口所有调用次数**

![[截屏2025-12-25 00.20.35.png]]
#### 指标十七：【Apollo，taskdispatch，taskinference】对外提供api接口的时延
##### 【Apollo】五个

```
统计接口：
  统一前缀: /apollo/internal/risk_visualization/

  具体路径:
  - /apollo/internal/risk_visualization/data_category - 数据分类查询
  - /apollo/internal/risk_visualization/data_source_by_ids - 数据源ID查询
  - /apollo/internal/risk_visualization/data_label_by_ids - 标签ID查询
  - /apollo/internal/risk_visualization/data_label_by_pages - 标签分页查询
  - /apollo/internal/risk_visualization/data_level - 数据级别查询
```
![[截屏2025-12-25 00.18.28.png]]
##### 【taskdispatch】十个
统计接口：
```

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
![[截屏2025-12-25 00.19.27.png]]
##### 【taskinference】六个
统计接口：
只统计`/task_actuator/api/v1开头的这些接口
  
```

业务API接口（6个）

  apiVersion1 := r.Group("/task_actuator/api/v1")
  {
      apiVersion1.POST("/run_static_task", v1.TaskStaticRun)           // 1
      apiVersion1.POST("/run_dynamic_task", v1.TaskDynamicRun)         // 2
      apiVersion1.POST("/chat_model", ...)                              // 3
      apiVersion1.POST("/stream/chat", ...)                             // 4
      apiVersion1.POST("/chat_model_risk", v1.CommonChatJob)           // 5
      apiVersion1.POST("/run_category_plus", v1.TaskCategoryPlusRun)   // 6
  }



```

![[截屏2025-12-25 00.21.02.png]]
#### 指标十八：【Apollo，taskdispatch，taskinference】对外提供api接口的成功率

成功状态码：2XX
统计方法：统计所有接口的状态码还有次数，在grafana面板计算成功率，

统计接口同指标十七
##### 【apollo】

![[截屏2025-12-25 00.18.43.png]]
##### 【taskdispatch】

![[截屏2025-12-25 00.19.41.png]]


##### 【taskinference】
![[截屏2025-12-25 00.21.15.png]]