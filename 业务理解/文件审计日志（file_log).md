# 总览
![[Pasted image 20261020153449.png]]
## 各种设备渠道上报日志

dsp主要用到的这个协议的是http，ftp，xdla的


  • **STA 路径**：设备上报不同协议的文件日志；采集服务根据日志里的 source 识别协议，再按协议规则转换、补充字段。

  • **XDLA 路径**：服务接收已经按 TMG 文件审计格式组织好的 log.json，把字段映射成 DSP 的 file_log。XDLA 是一种日志接入/封装格式，不是 source 里的协议类型。


 ![[Pasted image 20261021112839.png]]



### sta线路到dsp-file-log
![[Pasted image 20261021164920.png]]

### xdla线路到dsp-file-log

![[Pasted image 20261022104914.png]]

![[Pasted image 20261022111822.png]]

![[Pasted image 20261022152320.png]]