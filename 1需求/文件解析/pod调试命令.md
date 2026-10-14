### 挂载目录对应关系

![[截屏2026-08-05 15.50.34.png]]

### 验证模块是否可以导入

文本清洗模块
```
python3 -c 'from unstructed_parser.cleaner.cleaner import clean_text; print("OK")'
```

API模块
```
cd /app && python3 -c " from unstructed_parser.api import create_app from unstructed_parser.main import app print('API 模块导入成功 ✓') "
```

### helm部署


```
cd /root/dsp/UnstructedParser/helm部署
```

helm部署升级
/root/wjx/fileparser/helm部署/dsp-fileparser-3.0.36-feature-develop3.0.36-wjx-refactor-942de188.tgz
```
helm upgrade  fileparser  ./dsp-fileparser-3.0.36-feature-develop3.0.36-wjx-refactor-942de188.tgz  -n dsp -f ./values-10-65-245-10.yaml
```

首次部署需要安装：
```
helm install fileparser ./dsp-fileparser-3.0.36-feature-develop3.0.36-wjx-refactor-22634d65.tgz -n dsp -f ./values-10-65-245-10.yaml
```



查看升级状态

```
helm list -n dsp | grep fileparser
```

查看pod状态

```
kubectl get pods -n dsp | grep fileparser
```


### 开启调试模式

获取pod名字
```
POD=$(kubectl get pods -n dsp -o name | grep unstructed | head -1 | cut -d/ -f2)
```

##### 调用ocr

打开调试模式，ocr调用配置从yaml获取
```
kubectl set env deploy/dsp-fileparser-deploy -n dsp DSPMODE=debug
```
关闭调试模式：kubectl set env deploy/fileparser -n dsp DSPMODE=

修改配置文件，增加cor配置

```
kubectl edit cm -n dsp dsp-fileparser-system-config
```
host改成模型平台ip

增加数据湖配置

ak: "lk2KxbgHyhNtynF3"  
sk: "c4c2c21e0772d2d3d56ccded38ddc887eda8dcebda8ed1ef231770fe01ee6d3e"




重启验证配置生效
```
kubectl rollout restart deploy/dsp-fileparser-deploy -n dsp
```

```
POD=$(kubectl get pods -n dsp -o name | grep unstructed | head -1 | cut -d/ -f2)
```

```
kubectl exec $POD -n dsp -c parser -- printenv DSPMODE
```

```
kubectl exec $POD -n dsp -- cat /etc/unstruct/system_config.yaml | grep -A3 model_platform
```

#### 数据湖AKSK

##### 开启本地aksk认证
```
kubectl edit cm -n datagateway file-download-svc
```

需要把 local 的 enable 从 false 改成 true，这样就启用本地 AK/SK 认证了

```
          local:                                                                 
            enable: false
  改成：
          local:
            enable: true

```

##### 重启网关服务
```
kubectl rollout restart deployment -n datagateway file-download-svc
```

查看重启状态
```
kubectl get pods -n datagateway | grep file-download-svc
```

##### 验证aksk生效

```
kubectl get cm -n datagateway file-download-svc -o yaml | grep -A15 "aksk"
```








### 测试api服务

开启pod的seivice服务，暴露成node节点ip，调试用

```
kubectl patch svc fileparser -n dsp --type=json -p='[{"op":"replace","path":"/spec/type","value":"NodePort"},{"op":"add","path":"/spec/ports/0/nodePort","value":30449}]'
```
验证ip修改：
```
kubectl get svc fileparser -n dsp
```
   # 期望输出:                                                                                                                                                                                                    

```
  fileparser   NodePort   192.188.16.95   <none>   8000:30449/TCP   28h

```

修改回来成集群IP

```
kubectl patch svc fileparser -n dsp --type=json -p='[ {"op":"replace","path":"/spec/type","value":"ClusterIP"}, {"op":"remove","path":"/spec/ports/0/nodePort"} ]'
```

##### 宿主机调用api接口传入md5

```
curl -X POST http://172.16.250.58:8000/internal/parse -F "md5=9fa123cb15040b5e06b4897c09cbec51"
```
##### 其他平台调用接口

```
curl -X POST "http://10.65.245.95:30449/internal/parse" -F "file=@/root/dsp/UnstructedParser/demo/source_1.pdf"
```


### 测试AKSK认证

```
kubectl rollout restart deploy/fileparser -n dsp
```

```
POD=$(kubectl get pods -n dsp -o name | grep unstructed | head -1 | cut -d/ -f2)
```
无签名
```
kubectl exec $POD -n dsp -- python3 -c "import os; os.environ['NO_PROXY'] = '*'; import requests; r = requests.post('http://localhost:8000/internal/parse', files={'file': ('source_1.pdf', open('/tmp/test.pdf', 'rb'), 'application/pdf')}, timeout=120); d = r.json(); print('=== 状态码:', r.status_code); print('=== file_name:', d.get('file_name')); print('=== file_type:', d.get('file_type')); print('=== status_code:', d.get('status_code')); print('=== error_msg:', d.get('error_msg')); print('=== txt_content 长度:', len(d.get('txt_content', ''))); print('=== OCR 输出全文 ==='); print(d.get('txt_content'))"
```

带签名
```
kubectl exec $POD -n dsp -- python3 -c "import os;os.environ['NO_PROXY']='*';import requests;from unstructed_parser.common.aksk import Signature;req=requests.Request('POST','http://localhost:8000/internal/parse',files={'file':('source_1.pdf',open('/tmp/test.pdf','rb'),'application/pdf')});Signature(ak='testak',sk='testsk').signature(req);print('=== Authorization 头:',req.headers.get('Authorization','')[:80],'...');r=requests.Session().send(req.prepare(),timeout=120);d=r.json();print('=== 状态码:',r.status_code);print('=== file_name:',d.get('file_name'));print('=== status_code:',d.get('status_code'));print('=== error_msg:',d.get('error_msg'));print('=== txt_content 长度:',len(d.get('txt_content','')));print('=== OCR 输出全文 ===');print(d.get('txt_content'))"
```

其他平台
```
cd /root/dsp/UnstructedParser && .venv/bin/python -c " import requests from unstructed_parser.common.aksk import Signature req = requests.Request('POST', 'http://10.65.245.95:30449/internal/parse', files={'file': ('source_1.pdf', open('demo/source_1.pdf','rb'), 'application/pdf')}) Signature(ak='testak', sk='testsk').signature(req) r = requests.Session().send(req.prepare(), timeout=180) print(r.status_code, r.text[:200])"
```


### 测试办公类文件

接口测试：
docx
```
curl -s -X POST "http://10.65.245.95:30449/internal/parse" -F "file=@/root/dsp/UnstructedParser/demo/001_立项调研报告.xlsx" | python3 -c "import sys,json; print(json.load(sys.stdin)['txt_content'])"
```


### 查询etcd的模型平台配置

```
kubectl -n dsp exec fileparser-76bc6b9f48-65thm -c parser -- python3 -c "import etcd3; c = etcd3.client(host='etcd-etcd-biz-headless.etcd.svc.cluster.local', port=2379); v, _ = c.get('/apex/connector/platformAI/openapi'); print(v)"
```

### 修改etcd配置端口
备份原值
```
kubectl exec -n etcd etcd-etcd-biz-0 -- sh -c "ETCDCTL_API=3 etcdctl get $KEY" | tee /tmp/openapi_backup.txt
```
**改成 30080 + http**
```
kubectl exec -n etcd etcd-etcd-biz-0 -- sh -c "ETCDCTL_API=3 etcdctl put /apex/connector/platformAI/openapi '{\"ak\":\"84f8d882b7c6cf946a9e78511df1d4abb1212b176a735d501336a3ead2c90ae8\",\"sk\":\"1a4347ff38567ef7474091fb1090875ba6487cdccbf754d8aa053b023413ef12\",\"platformType\":\"platformAI\",\"businessType\":\"openapi\",\"protocol\":\"http\",\"host\":\"10.87.67.99\",\"port\":30080}'"
```
检查
```
kubectl exec -n etcd etcd-etcd-biz-0 -- sh -c "ETCDCTL_API=3 etcdctl get /apex/connector/platformAI/openapi"
```
### 查询对接的模型平台是那个
进入pod。执行python3
```
from fileparser.clients.ocr.ocr_client import OcrClient
from fileparser.clients.vlm.vlm_client import VlmClient
print(OcrClient().base_url)
print(VlmClient().base_url)
```

   [OCR] base_url = https://10.87.67.99:443     ← ✅                                                                                                                                                              
   [VLM] base_url = https://10.87.67.99:443     ← ✅