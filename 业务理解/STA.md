

![[Pasted image 20261021164914.png]]


# 获得原始sta日志

### 1、kvd内容自动删除关闭 
```
 vi /etc/sinfor/fw/sync_dsp.ini 
```
 
默认kvd_delete_after_upload 为1， 代表kvd的大小为100时上报会自动删除 

```
kvd_delete_after_upload = 0
```

```
/home/sharpknife/bin/supervisorctl restart sharpknife-loguploader:loguploader-dsp 
```
  
  上面3个操作 我想变成shell脚本，但是需要校验磁盘空间小于80%时kvd_delete_after_upload设置为0，若大于80%则将kvd_delete_after_upload设置为1，然后重启服务

### 2、STA的KVD 存储路径

```
/fwlog/data/metadata/raw/20261010/
```

莫俊永个人环境：
10.74.167.18 root/Admin@123#!
在/root/fwlog/data/metadata/raw/20261012/http/part-0-http-dsp-all

### 3.读取kvd的命令

```
/root/kvd_reader 0.data
```

kvd_reader是一个二进制文件



### 原始日志格式：

##### http原始日志格式：

```
{

  "accept-charset": "",

  "afver": "TS3.0.107.14061 Build20260702",

  "attacks": "",

  "audit_status": "1",

  "cdn-src-ip": "",

  "clientip": "",

  "connid": "181340493",

  "cookie": "",

  "custom": "{\"encryptType\":-1,\"encPayloadType\":-1,\"encReqType\":-1,\"encRspType\":-1,\"urlHash\":0}",

  "device_id": "EF5B5D5F",

  "dip0": "170400169",

  "dip1": "0",

  "dip2": "0",

  "dip3": "0",

  "dport": "15501",

  "dst_ip": "10.40.25.169",

  "dst_mac": "58:69:6c:f2:a0:f1",

  "duration": "2",

  "dzone": "2",

  "errcode": "0",

  "expires": "",

  "file_access_success": "1",

  "file_transfer_type": "2",

  "full_attack_info": "{\"matched_id\" : \"\", \"correlation_name\" : \"\"}",

  "host": "agent.nmic.cma:15501",

  "http_url": "agent.nmic.cma:15501/metrics/api/v2/agent/config/list?id=9480829d624b4e2c8743da5d01bdbeec\u0026source=agent\u0026api_key=d01160d21c5111ec8b6248dc2d058668",

  "in_dev": "eth8",

  "iptype": "4",

  "method": "GET",

  "payload": "",

  "referer": "",

  "req_body": "",

  "req_body_len": "0",

  "req_content_type": "",

  "req_head": "GET /metrics/api/v2/agent/config/list?id=9480829d624b4e2c8743da5d01bdbeec\u0026source=agent\u0026api_key=d01160d21c5111ec8b6248dc2d058668 HTTP/1.1\r\nConnection: close\r\nHost: agent.nmic.cma:15501\r\nAccept-Encoding: gzip\r\n\r\n",

  "req_len": "210",

  "req_ts": "1791647745566",

  "rsp_body": "[{\"fileName\":\"656e8e3f15da3943d538b416_system_cpu\",\"md5\":\"0ebcd36db88fad4373779f9218d52b6c\",\"enabled\":true,\"deleted\":false,\"ipList\":[]},{\"fileName\":\"656e8e3f15da3943d538b416_system_disk\",\"md5\":\"a307b0c2cd9e49d4437b1e737ce30ee4\",\"enabled\":true,\"deleted\":false,\"ipList\":[]},{\"fileName\":\"656e8e3f15da3943d538b416_system_load\",\"md5\":\"9613fe6e9bb1946acdaef9177c49a72e\",\"enabled\":true,\"deleted\":false,\"ipList\":[]},{\"fileName\":\"656e8e3f15da3943d538b416_system_memory\",\"md5\":\"44d1c44aecb893cc82405fb07c72251b\",\"enabled\":true,\"deleted\":false,\"ipList\":[]},{\"fileName\":\"656e8e3f15da3943d538b416_system_net\",\"md5\":\"bf8ae767ed46c8379e500d6d6403140e\",\"enabled\":true,\"deleted\":false,\"ipList\":[]}]",

  "rsp_body_len": "681",

  "rsp_content_type": "application/json",

  "rsp_head": "HTTP/1.1 200 OK\r\nServer: openresty\r\nDate: Sat, 10 Oct 2026 15:55:45 GMT\r\nContent-Type: application/json\r\nTransfer-Encoding: chunked\r\nConnection: close\r\nVary: Accept-Encoding\r\nSet-Cookie: JSESSIONID=pdeQZhPaCUGSvtS9o74c8rGIpvqZH17yjjdrwa_W; path=/\r\nX-Frame-Options: SAMEORIGIN\r\nX-Content-Type-Options: nosniff\r\nX-XSS-Protection: 1; mode=block\r\nX-Download-Options: noopen\r\nStrict-Transport-Security: max-age=31536000; includeSubDomains\r\nCache-Control: max-age=0, no-cache, no-store, must-revalidate\r\nContent-Security-Policy: default-src 'self'; font-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; connect-src 'self' ws:; img-src 'self' data:\r\nContent-Encoding: gzip\r\n\r\n",

  "rsp_len": "1402",

  "rsp_ts": "1791647745568",

  "server": "openresty",

  "session_state": "3",

  "set-cookie": "[ \"JSESSIONID=pdeQZhPaCUGSvtS9o74c8rGIpvqZH17yjjdrwa_W; path=\\/\" ]",

  "sip0": "170416223",

  "sip1": "0",

  "sip2": "0",

  "sip3": "0",

  "sport": "33684",

  "src_ip": "10.40.88.95",

  "src_mac": "e8:61:1f:3b:50:34",

  "status_code": "200",

  "syn_ack_seq": "1568437644",

  "syn_seq": "779030311",

  "szone": "2",

  "ts": "1791647745579",

  "type": "1",

  "uid": "3090496296",

  "uri": "/metrics/api/v2/agent/config/list?id=9480829d624b4e2c8743da5d01bdbeec\u0026source=agent\u0026api_key=d01160d21c5111ec8b6248dc2d058668",

  "user-agent": "",

  "x-forwarded-for": "",

  "x-real-ip": ""

}
```




admin/Fufy{7_x>d_P
Fufy{7_x>d_Psangfornetwork