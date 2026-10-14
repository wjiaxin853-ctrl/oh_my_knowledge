
# 配置外网模型转发到内网

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