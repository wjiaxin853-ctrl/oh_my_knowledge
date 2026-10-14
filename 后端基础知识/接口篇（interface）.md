### 客户端（Client）与服务端（Server）

###### **您的服务（客户端）：**

1. 发起请求的一方

2. 主动调用外部API

3. 处理响应结果

4. 需要处理网络异常、超时、重试等

5. 对调用结果负责

###### **外部接口（服务端）：**

- 接收请求的一方
    
- 被动响应API调用
    
- 返回数据或状态
    
- 保证接口可用性和性能

## 接口的方法

```
1. **GET**：只读操作，获取数据
    
2. **POST**：创建新资源或执行复杂操作
    
3. **PUT**：完整更新现有资源
    
4. **PATCH**：部分更新资源
    
5. **DELETE**：删除资源
```


## 接口动态参数
```
  实际 endpoint 示例

  无动态参数的路径：
  /apollo/internal/dashboard/datasources/link
  → /apollo/internal/dashboard/datasources/link

  /apollo/internal/dashboard/datacategories/template
  → /apollo/internal/dashboard/datacategories/template

  带数字ID的路径：
  /apollo/internal/dashboard/datasources/123
  → /apollo/internal/dashboard/datasources/{id}

  /apollo/internal/dashboard/scan_tasks/456
  → /apollo/internal/dashboard/scan_tasks/{id}

  /apollo/internal/dashboard/datasourcegroups/789
  → /apollo/internal/dashboard/datasourcegroups/{id}

  多级动态参数：
  /apollo/internal/dashboard/datasources/123/scan
  → /apollo/internal/dashboard/datasources/{id}/scan

  /apollo/internal/catalog/data_fields/456/status
  → /apollo/internal/catalog/data_fields/{id}/status

  带UUID的路径：
  /apollo/internal/tasks/550e8400-e29b-41d4-a716-446655440000/logs
  → /apollo/internal/tasks/{uuid}/logs

```


## 接口状态码（HTTP状态码）

### 状态码定义

当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含 HTTP 状态码的信息头（server header）用以响应浏览器的请求。

HTTP 状态码的英文为 **HTTP Status Code**。

### 状态码分类

- **1xx（信息性状态码）**：表示接收的请求正在处理。
- **2xx（成功状态码）**：表示请求正常处理完毕。
- **3xx（重定向状态码）**：需要后续操作才能完成这一请求。
- **4xx（客户端错误状态码）**：表示请求包含语法错误或无法完成。
- **5xx（服务器错误状态码）**：服务器在处理请求的过程中发生了错误。

#### 常见状态码
| 状态码 |        状态码英文名称        |                          中文描述                           |
| :-: | :-------------------: | :-----------------------------------------------------: |
| 100 |       Continue        |                      继续。客户端应继续其请求                       |
| 200 |          OK           |                   请求成功。一般用于GET与POST请求                   |
| 403 |       Forbidden       |                 服务器理解请求客户端的请求，但是拒绝执行此请求                 |
| 404 |       Not Found       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 500 | Internal Server Error |                     服务器内部错误，无法完成请求                      |

