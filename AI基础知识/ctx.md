
# 配置外网模型转发到内网（APIkey的方式）

### mac 配置
![[截屏2026-10-10 12.29.58.png]]

mac的omp代理端口是20092,

### linux配置
![[截屏2026-10-10 12.31.02.png]]

其中这些baseurl还有apikey都是从配置：

mac上执行“
```
cat ~/.omp/agent/models.yml
```

获得：
![[截屏2026-10-10 12.31.53.png]]


### 端口转发
```
ssh -N -o ExitOnForwardFailure=yes -o ServerAliveInterval=15 -o ServerAliveCountMax=3 -R 20092:127.0.0.1:20092 root@10.74.167.125
```

后台执行
```
ssh -f -N -o ExitOnForwardFailure=yes -o ServerAliveInterval=15 -o ServerAliveCountMax=3 -R 20092:127.0.0.1:20092 root@10.74.167.125
```


# 配置chatgpt的auth授权到内网

#### 本地mac执行端口转发
```
ssh -N \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=15 \
  -o ServerAliveCountMax=3 \
  -R 127.0.0.1:21926:127.0.0.1:51926 \
  root@10.74.167.125
```

其中这个 51926是 mac 的代理端口，ctx重启会改变，用 scutil --proxy 重新确认即可
当前的是：

```
<dictionary> {
  ExceptionsList : <array> {
    0 : *zhihu.com,*zhimg.com,*jd.com,100ime-iat-api.xfyun.cn,*360buyimg.com,localhost,*.local,127.*,10.*,172.16.*,172.17.*,172.18.*,172.19.*,172.2*,172.30.*,172.31.*,192.168.*
  }
  FTPPassive : 1
  HTTPEnable : 1
  HTTPPort : 51926
  HTTPProxy : 127.0.0.1
  HTTPSEnable : 1
  HTTPSPort : 51926
  HTTPSProxy : 127.0.0.1
  ProxyAutoConfigEnable : 0
  SOCKSEnable : 1
  SOCKSPort : 51926
  SOCKSProxy : 127.0.0.1
}
```

### 远端的ctx导入本地的auth凭证

##### 开启导入凭证
![[截屏2026-10-15 09.48.42.png]]
##### 设置代理环境变量

```
export HTTP_PROXY=http://127.0.0.1:21926
export HTTPS_PROXY=http://127.0.0.1:21926
export NO_PROXY=localhost,127.0.0.1,::1
```

##### 查看登陆状态
```
codex -c 'model_provider="openai"' login status
```

显示 **使用 ChatGPT 登录**就代表成功

###### 登陆codex

```
codex -c 'model_provider="openai"' -m gpt-6-astra
```

## 补充：重启后快速使用

### 本地 Mac 执行（每次开机或网络代理重启后）

先开启 Mac 网络代理软件的系统代理，再执行下面命令，自动读取当前代理端口并后台转发，无需手动修改 `51926`。

```bash
bash "/Users/wjx/Desktop/知识库/AI基础知识/codex-mac-tunnel.sh"
```

首次使用脚本前，先关闭旧的手动 `21926` 转发窗口。使用期间保持 Mac 在线、代理运行。

### 远端执行（首次配置一次，已配置过无需重复）

```bash
cat >> /root/.bashrc <<'EOF'

# Codex through the Mac SSH proxy tunnel
codex() {
  HTTP_PROXY=http://127.0.0.1:21926 \
  HTTPS_PROXY=http://127.0.0.1:21926 \
  NO_PROXY=localhost,127.0.0.1,::1 \
  command codex -c 'model_provider="openai"' -m gpt-6-astra "$@"
}
EOF

source /root/.bashrc
```

### 远端执行（以后新终端直接启动）

```bash
codex
```
