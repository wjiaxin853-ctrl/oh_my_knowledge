
### 配置到本地pycharm的jyputer内核

##### 进入auto-dl的终端，开放jyputer服务
```
jupyter lab --ip 0.0.0.0 --port 8890 --no-browser --allow-root
```

![[截屏2026-10-01 15.09.06.png]]

执行后回输出url还有token，记录下来
```
http://127.0.0.1:8890/lab?token=【这里是新token】
```
##### mac，开放端口转发
```
ssh -L 8890:127.0.0.1:8890 -p 50427 root@connect.bjb1.seetacloud.com
```
![[截屏2026-10-01 15.10.53.png]]
##### pycharm配置连接

![[截屏2026-10-01 15.11.05.png]]