
# 工具篇
### Web访问工具

##### web_search

通过 网络搜索代理商的API（例如：Brave Search API或Perplexity Sonar（直连或通过 OpenRouter）搜索网络

主动搜索：给出关键字会主动去搜索，不需要url，

##### web_fetch

直接 GET 某个 URL，然后提取网页的可读正文（HTML → markdown 或纯文本），类似“去广告读正文”的功能。

对话框给出对应url，需要知道url，类似于爬虫获取

# 分布式计算

### PyTorch分布式计算-torchrun

```
torchrun --nproc_per_node 2 train.py
```

表示：**在当前机器上启动 2 个训练进程（通常对应 2 张 GPU）进行并行训练**
nproc：number of processes（进程数）
per_node：每台机器


`torchrun` 是 PyTorch 提供的官方分布式启动工具：
- 启动多个进程
- 自动设置分布式环境变量（rank、world_size 等）
- 管理进程通信（DDP）