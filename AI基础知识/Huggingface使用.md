### **1. 安装依赖**  

```
pip install -U huggingface_hub
```

临时使用清华源
```
pip install -U huggingface_hub -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### **2. 设置环境变量**  



_Linux_

```
export HF_ENDPOINT=https://hf-mirror.com
```

_Windows Powershell_

```
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

建议将上面这一行写入 `~/.bashrc`。  
### **3.1 下载模型**

> [!NOTE]
> 每次都需要在终端执行这个镜像源命令！！！
> 

```
 hf download Qwen/Qwen3.5-4B --local-dir Qwen3.5-4B
```

hf download 模型名称（仓库直接复制） --local-dir 模型下载相对路径

默认支持断点下载，不要删除.cache即可

##### 下载显示日志

```
HF_DEBUG=1 hf download openbmb/MiniCPM-V-2 --local-dir ./models/MiniCPM-V-2
```

### 3.2 下载数据集

```
from huggingface_hub import snapshot_download

LOCAL_DIR = "/Users/wjx/Desktop/datasets/Evaluate/ceval"

snapshot_download(
    repo_id="ceval/ceval-exam",
    repo_type="dataset",
    local_dir=LOCAL_DIR,
)

print(f"C-Eval 下载完成：{LOCAL_DIR}")
```