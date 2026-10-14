仓库地址：
http://code.sangfor.org/SDC/SDL/SBDL/datalake-file-storage-svc

分支名称：
v1.0.0-datalake-base-20250330-gopath

### 删除清理接口

![[截屏2026-03-10 19.53.29.png]]

  **路由注册**   
src/soapa-app/pkg/datalake-file-storage-svc/server/node/router.go

```
func New(e *gin.Engine, c *Client) {  
    fileGroup := e.Group("/v1/collect/file")  
    fileGroup.POST("/upload", MakeHandlerFunc(c.UploadFile))  
    fileGroup.GET("/download", MakeHandlerFunc(c.DownloadFile))  
    fileGroup.POST("/delete", MakeHandlerFunc(c.DeleteFiles))  
  
    nodeGroup := e.Group("/v1/collect/node", customLogger())  
    nodeGroup.GET("/info", MakeHandlerFunc(c.InfoResp))  
  
    cleanGroup := e.Group("/v1/collect/clean", authMiddleware(c.AppCfg.NodeOp))  
    cleanGroup.GET("/earliest", MakeHandlerFunc(c.Earliest))  
    cleanGroup.GET("/count", MakeHandlerFunc(c.Count))  
    cleanGroup.DELETE("", MakeHandlerFunc(c.Delete))  
}
```

代码有/v1/collect/clean删除接                                                                                                                                                                                              


  **接口实现**

  

  /Users/wjx/Documents/dsp/datalake-file-storage-svc/src/soapa-app/pkg/datalake-file-storage-svc/server/node/clean_interface.go — Earliest()、Count()、Delete() 三个方法

  

  **请求参数结构**

  

  /Users/wjx/Documents/dsp/datalake-file-storage-svc/src/soapa-app/pkg/datalake-file-storage-svc/core/common/defend.go：

  

  type CleanDeleteReq struct {

      RecordDate int64 `form:"recordDate" json:"recordDate" binding:"required"`

  }

  

  form:"recordDate" 说明参数名是 recordDate，通过 query string 传递。

  

  **单文件删除接口**

  

  /Users/wjx/Documents/dsp/datalake-file-storage-svc/src/soapa-app/pkg/datalake-file-storage-svc/server/node/server_interface.go 第243行 DeleteFiles() 方法




### 实现定期清理接口
                                                                                                                    
```

  **路线1：利用内置清理机制（推荐，零开发）**

  数据湖本身已经有定时删除，只需要确认/调整配置：


  在应用平台上查看当前数据湖的配置（**单行命令**）：

  kubectl get configmap -n datagateway -o yaml | grep -A5 fileExpiredDay
  

  如果 fileExpiredDay 已经是合理的值（比如3天），说明捞取过的文件3天后会自动清理，不需要额外脚本。

  

  **路线2：写外部清理脚本调用** **DELETE** **接口**

  

  问题是这个接口需要 **AKSK** **v4** **签名认证**，不是简单 curl 能调的。需要：

  1. 先从 k8s configmap 或 pod 里获取 clean 接口的 AK/SK

  2. 用 Go 或 Python 实现 AKSK v4 签名算法

  3. 带签名调用 DELETE 接口

  

  这比较复杂，而且数据湖自己已经有这个能力。
```




### 三天过期清理配置路径

![[截屏2026-03-11 09.57.56.png]]
