### 示例图

![[截屏2025-12-06 23.12.19.png|500]]

### 埋点思路
http_client.go里面新定义hettpclient，之前代码是只有gpt_client使用了http_client,当前：
是针对所有对外接口，都统一调用http_client，去计算时延

统计的是http的状态吗，是200，是固定的

增加了methed方法，可以显示get，post请求，

针对endpoint，设置了可变参数的模版，避免过多曲线，导致内存爆炸

## 对外接口位置（4个服务，九个接口）

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
  - 函数: GpuChat (第513行)
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
  - 函数: SearchLabelLabel (第79行)
  - 调用位置: vector_client.go:86

  4. 认证服务 (1个接口)

  文件: internal/router/service/authoritarian/authoritarian.go
  - 接口: POST http://{TokenCheckHost}:{TokenCheckPort}/{TokenCheckUrl}?token={token}
  - 函数: RequestUser (第55行)
  - 调用位置: authoritarian.go:72

  5. 模型列表服务 (1个接口)

  文件: pkg/client_utils/gpt_client.go
  - 接口: GET /v1/models
  - 函数: GetAllModels (第274行)
  - 调用位置: gpt_client.go:276

  6. Apex 连接器服务 (1个接口)

  文件: pkg/apexClientSdk/apex_connector_client.go
  - 接口: Apex 平台配置接口
  - 函数: 初始化配置 (第58行)
  - 调用位置: apex_connector_client.go:58

  7. 限流配置 (仅读取配置)

  文件: internal/router/service/rate_limit/rate_limit.go
  - 函数: init() (第51行)
  - 说明: 仅从 GpuInfo 读取配置，不发起网络请求


```

# 设计时延桶上限问题

```
  1. GPU 客户端超时配置

  - 配置文件：app-gpu-config.yaml
  - 配置值：gpuClientTimeout: 600（秒）= 10分钟
  - 使用位置：http_client.go:141-153
  httpWaitTimeUp := time.Duration(m.GpuInfo.GpuClientTimeout)  // 600秒
  client := &http.Client{
      Timeout: httpWaitTimeUp * time.Second,  // 600秒
  }

  2. 统一 HTTP 客户端超时

  - 默认值：60秒
  - 使用位置：http_client.go:120
  client: &http.Client{Timeout: 60 * time.Second}  // 60秒

  3. Web 服务超时

  - 配置值：timeOut = 10 * 60 * time.Second = 10分钟
  - 使用位置：main.go:61-62
  ReadTimeout:  timeOut,  // 10分钟
  WriteTimeout: timeOut,  // 10分钟

  4. Etcd 连接超时

  - 连接超时：3秒
  - 请求超时：3秒
  - 使用位置：apex_connector_client.go

```



# task_actuator/pkg/client_utils/http_client.go

##### 接口抽象：提供统一接口，便于类型替换和测试。（HTTPClient 接口定义）
![[截屏2025-12-16 11.04.23.png]]

  作用：
  - 定义了 HTTP 客户端应该具备的基本方法
  - 任何实现了这个接口的类型都可以作为 HTTP 客户端使用。
  - 定义了这个接口支持Get方法，Post方法，都会复用到Do


##### 埋点客户端结构：包装原始 HTTP 客户端，自动添加监控埋点
![[截屏2025-12-16 11.06.56.png]]

  作用：
  - 装饰器模式：包装 http.Client，增加埋点功能
  - 是接口的具体实现
  - 持有一个原始的 *http.Client 实例
  - 专门用于埋点

**注册指标监控客户端，方便全局调用**
![[截屏2025-12-16 11.18.34.png]]

**处理路径**

![[截屏2025-12-16 11.21.33.png]]
```
  1. 基本功能

  func buildEndpointLabel(req *http.Request) string {
      path := req.URL.Path          // 提取URL路径部分
      if path == "" {              // 如果路径为空
          path = "/"               // 默认设为根路径
      }
      return path                  // 返回纯路径，不包含参数
  }

  2. 处理示例

  // 输入的完整URL
  fullURL := "http://api.example.com/api/v1/chat?token=abc123&user_id=456"

  // req.URL.Path 提取结果
  path := "/api/v1/chat"  // ❌ 不包含查询参数

  // 返回给监控系统的标签
  buildEndpointLabel(req)  // 返回: "/api/v1/chat"
```

实现示例：
在cache_client.go里面，定义的对外接口
![[截屏2025-12-16 11.29.48.png|300]]
```
  1. 实际 URL 路径示例

  // cache_client.go 中的接口路径：
  完整URL1: "http://redis-service/dataIdentifyConfig/v1/industry"
  完整URL2: "http://redis-service/dataIdentifyConfig/v1/allTemplateIds"
  完整URL3:
  "http://redis-service/dataIdentifyConfig/v1/categories?templateIds=1,2,3"
  完整URL4: "http://redis-service/dataIdentifyConfig/v1/labels/labelCategoryRelati
  on?labelIds=10,20,30"

  2. buildEndpointLabel 提取结果

  func buildEndpointLabel(req *http.Request) string {
      path := req.URL.Path
      if path == "" {
          path = "/"
      }
      return path
  }

  提取后的标签：
  - URL1 → /dataIdentifyConfig/v1/industry
  - URL2 → /dataIdentifyConfig/v1/allTemplateIds
  - URL3 → /dataIdentifyConfig/v1/categories
  - URL4 → /dataIdentifyConfig/v1/labels/labelCategoryRelation
```


### doWithMetrics 函数（第41-79行）
![[截屏2025-12-16 12.43.26.png]]

**resp, err := client.Do(req)**：执行http请求
  **statusCode := 0  // 默认设置为0**
```
  - 0 不是标准的HTTP状态码
  - 在监控中用来表示网络错误或连接失败
  0=网络错误, 200=成功, 500=服务器错误等
```


### MetricsHTTPClient 方法实现（第81-105行）
![[截屏2025-12-16 12.50.46.png]]

作用：执行任意HTTP请求并自动记录指标
![[截屏2025-12-16 12.54.04.png]]


```
  if contentType != "" {
      req.Header.Set("Content-Type", contentType)
  }
  - 当 contentType 不为空时，设置请求头
  - 告诉服务器请求体的MIME类型
```

```

  | 特性           | GET请求 | POST请求 |
  |--------------|-------|--------|
  | 请求体          | ❌ 没有  | ✅ 有    |
  | Content-Type | ❌ 不需要 | ✅ 需要   |
  | 数据位置         | URL参数 | 请求体    |
  | 用途           | 查询资源  | 提交数据   |
  | 幂等性          | ✅ 幂等  | ❌ 非幂等  |

  简单说：GET没有身体（body），所以不需要穿衣服（Content-Type）！
```


### 统一客户端
![[截屏2025-12-16 12.57.23.png]]

  - 创建一个带监控功能的HTTP客户端
  - 使用统一配置（60秒超时）
  - 返回单例实例（虽然这里不是严格单例）

其中gpu_client调用的还是使用的之前的实例，最后也会调用埋点
![[截屏2025-12-16 13.04.10.png]]



![[截屏2025-12-16 13.04.53.png]]
将标准客户端包装成带监控的客户端，赋予监控超能力，gpu仍然是对应yaml文件的时间配置，不是60秒（**scripts/chart/templates/app-gpu-config.yaml**）


#### gpt_client.go 变动

![[截屏2025-12-16 14.29.25.png]]

修改为当前的客户端，修改计算时间位置，仅仅统计http接口的时间，也就是调用模型接口时间，影响不大
```
  变动前（包含初始化）：

  总时间 = GetGpuSvc() + 构建请求头 + HTTP调用 + 响应处理
         = 1ms + 5ms + 2000ms + 10ms = 2016ms
  日志记录："GPT处理用时：2016ms"

  变动后（排除初始化）：

  总时间 = HTTP调用 + 响应处理
         = 2000ms + 10ms = 2010ms
  日志记录："GPT处理用时：2010ms"
```
变动二：

![[截屏2025-12-16 14.32.14.png]]