

![[截屏2025-12-07 17.43.20.png|500]]


### 埋点思路：
针对动态的，一个api动态推理就是一个错误码，在task_actuator/internal/router/service/dynamic_data/api_predict_tasks/api_predict_task.go这个文件里面的HandleApiRich（）函数里面最后会有一个api分类结果之类的日志，在这里可以尝试统一埋点。
在FieldDescriptionPredicted这个结构体（prompt_utils.go）里面有ErrInfo字段，里面有TaskError里面是如下代码，在这里面获取错误码
```
type TaskError struct {
	Code    int      `json:"code"`
	Msg     string   `json:"msg"`
	Details []string `json:"-"`
}
```

### 根据业务定义错误码去统计（错误码来源）

```
  3. TaskError结构分析 (pkg/constants.go:8-26)

  type TaskError struct {
        Code    int      `json:"code"`    // 错误码
        Msg     string   `json:"msg"`     // 错误消息  
        Details []string `json:"-"`       // 详细信息（不序列化）
  }

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
  )

  错误码分类:
  - 0: 成功状态
  - 1: 未知错误
  - 1001-1999: 前端只展示错误码（系统级错误）
  - 2001-2999: 前端展示具体错误（业务级错误）
```


### api_predict_task.go

![[截屏2025-12-15 23.44.45.png]]
if语句：只有当存在错误信息，并且错误状态吗不是成功的时候才记录
使用defer原因：
保证是 finalErrInfo 的最终值：GptErr
```
  1️⃣ 避免过程噪音

  // ❌ 如果统计中间错误，会产生噪音
  "参数验证错误"    → 100次
  "模型调用失败"    → 80次
  "JSON解析异常"    → 20次
  总错误数：200次  ← 这个数字没有业务意义

  // ✅ 只统计最终结果错误
  "推理任务最终失败" → 15次  ← 这个数字有明确的业务意义
```



![[截屏2025-12-15 23.44.25.png]]

  3️⃣ JSON解析是特殊情况
  // JSON解析是唯一一个：
  // - 发生在业务逻辑之前
  // - 错误类型不是 *cons.TaskError
  // - 需要显式转换的地方
  
由于这个是json解析（解码）失败，正好对应
JsonUnmarshalErr       = TaskError{Code: 1014, Msg: "json序列化异常"}
但是不是*cons.TaskError格式的报错，所以需要采用pkg/constants.go里面的new方法将go'标准库错误转化为业务状态码错误，确保finalErrInfo始终是*cons.TaskError类型



![[截屏2025-12-15 23.43.38.png]]

  - 关键点：从最终推理结果中提取错误信息
  - apiResponseReturn.Fields.ErrInfo 包含了推理过程的所有错误
  - **将错误赋值给 finalErrInfo，等待defer处理**



![[截屏2025-12-16 00.23.11.png]]

```

  🚨 什么是 Panic？

  1️⃣ Panic 的定义

  - Go 语言的运行时异常：类似于 Java 的 RuntimeException、Python 的 Exception
  - 程序崩溃：遇到 panic 时，当前 goroutine 会立即终止
  - 不可恢复：除非使用 recover()，否则整个程序会退出
    
    
此处增加nil检查，防止出现程序崩溃
```



```
  场景1：所有子任务结果都是 nil

  subTaskSuccess = {
      1: nil,
      2: nil,
      3: nil
  }

  有兜底的情况：

  SetSubTaskSuccessFlag() 返回 UnknownErr
  ↓
  finalErrInfo = UnknownErr
  ↓
  defer: monitor.RecordDynamicInferenceError(UnknownErr)  ← 埋点成功！

  无兜底的情况：

  SetSubTaskSuccessFlag() 返回 nil
  ↓
  finalErrInfo = nil
  ↓
  defer: if finalErrInfo != nil...  ← 条件不满足，不埋点！

```


![[截屏2025-12-16 00.34.06.png]]
防止不存在错误信息的时候，会遗漏埋点
对应的是
```
 cons.BizCodeErr = TaskError{Code: 1002, Msg: "代码逻辑异常"}
```


### 错误信息结构体定义

![[截屏2025-12-15 23.48.05.png]]

![[截屏2025-12-15 23.48.28.png]]

![[截屏2025-12-15 23.48.51.png]]

![[截屏2025-12-15 23.49.14.png]]