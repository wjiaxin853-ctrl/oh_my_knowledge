
### 数据湖文件存储
![[截屏2026-08-25 20.02.10.png]]

### 对外接口
![[截屏2026-08-25 20.03.01.png]]


### 网关
上传还有下载是两个网关链路

![[截屏2026-08-25 20.04.15.png]]

#### 下载网关

##### 路径A：**AKSK 下载网关**
https://wiki.sangfor.com/pages/viewpage.action?pageId=214363689#heading-6%E6%B5%81%E7%A8%8B%E8%AE%BE%E8%AE%A1
这个文档讲的就是
```
请求方法	GET
请求URL	http://{domain}:{port}/dl/v1/filegw/file?id=${id}
```

 ```                                                                                                 

   解析器 ──AK/SK签名──▶ 下载网关(filegw) ──▶ 存储                                                   

                         只认 fileId，背后对接哪个存储你不需要知道