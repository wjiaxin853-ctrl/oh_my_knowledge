
### 埋点思路

在
task_actuator/internal/router/service/static_data/reason/task_category.go里面有静态推理字段，静态推理是一个列表映射，是一个字段一个错误码，需要用循环去提取，是map映射。

需要分开提取错误码




### 埋点代码

![[截屏2025-12-16 00.40.29.png]]



  解释：
  - 触发时机：在 dealResult 函数中，结果处理完成后
  - 调用方式：统一收集错误信息
  - 数据来源：reasonResultMap（推理结果映射）


![[截屏2025-12-16 00.44.11.png]]
遍历错误，直接收集


针对分类级别错误码
![[截屏2025-12-16 00.43.33.png]]


![[截屏2025-12-16 00.45.25.png]]









针对字段级错误


![[截屏2025-12-16 00.45.56.png]]




```






















		
	
	
	
	
	
	
	
	
	
	
	
| 类别      | 具体场景            | 项目中的表现             |
  |---------|-----------------|--------------------|
  | 网络/连接   | 连接失败、网络不通       | err != nil         |
  | 认证/权限   | API Key无效、权限不足  | resCode == 401/403 |
  | 请求格式    | Prompt格式错误、参数错误 | resCode == 400/422 |
  | 服务限流    | GPU服务过载、并发限制    | resCode == 429     |
  | GPU服务异常 | 模型推理失败、内部错误     | resCode >= 500     |
  | 数据处理    | JSON解析失败、格式错误   | tErr != nil        |
  | 业务逻辑    | 字段匹配失败、分类不存在    | 业务检查失败             |
  | 资源限制    | 模板不存在、被禁用       | 模板状态检查失败           |
  | 超时      | 处理时间过长          | resCode == 408/504 |
  | 未知错误    | 其他未预期错误         | default case       |

  🔍 错误码获取方式

  1. HTTP状态码 - 来自外部调用

  // 在 client_utils/gpt_client.go 中获取 resCode
```
  resBody, resCode, err := client_utils.GpuChat(&gpuChatParams, *httpClient)

  // 调用失败时记录指标二
  if err != nil || resCode != 200 {
      monitor.IncStaticFieldFailure(staticLabelPhase, resCode, "")
      return err
  }
```
  | HTTP状态码 | 含义    | 监控映射           |
  |---------|-------|----------------|
  | 401     | 认证失败  | auth_failed    |
  | 403     | 权限不足  | auth_failed    |
  | 400     | 请求错误  | bad_request    |
  | 429     | 限流    | quota_limit    |
  | 500     | 服务器错误 | internal_error |
  | 503     | 服务不可用 | internal_error |

  2. 内部错误码 - 来自业务逻辑（constants.go）

```
  // 使用TaskInference定义的业务错误码
  if err != nil {
      return globalpkg.GptErr.New(fmt.Sprintf("GPT服务异常, resCode: %v",
  resCode))
  }
```
  | 错误码  | 名称                | 可能的失败场景     |
  |------|-------------------|-------------|
  | 1001 | GptErr            | GPU服务异常     |
  | 1003 | EmptyJsonErr      | GPU返回空结果    |
  | 1004 | FieldNameMatchErr | GPU返回字段名不匹配 |
  | 1005 | CategoryErr       | 分类结果错误      |
  | 1014 | JsonUnmarshalErr  | JSON解析失败    |
  | 2001 | TemplateDisable   | 模板被禁用       |
  | 2002 | TemplateDelete    | 模板被删除       |
  | 2003 | TemplateEmpty     | 模板内容为空      
  
  
  ![[截屏2025-12-02 20.53.51.png]]
  
  ![[截屏2025-12-02 20.54.06.png]]
  ![[截屏2025-12-02 20.54.19.png]]![[截屏2025-12-02 20.54.41.png]]