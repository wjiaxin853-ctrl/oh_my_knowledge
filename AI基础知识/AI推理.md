### 检查硬件配置

##### 检查cpu
```
lscpu
```

##### 内存检测
```
free -h
```

##### 硬盘检测

```
df -h
```
# MAC加速引擎

mac的mps相当于CUDA

##### 查看mac显卡配置
```
system_profiler SPDisplaysDataType
```
##### 测试mac到加速引擎是否可以用

```
import torch
print(f"MPS 是否可用: {torch.backends.mps.is_available()}")
print(f"MPS 是否已编译: {torch.backends.mps.is_built()}")
```

# Nvida

### 检查显卡对应关系

```
python -c "import os, torch; print('CUDA_VISIBLE_DEVICES=', os.environ.get('CUDA_VISIBLE_DEVICES')); print('device_count=', torch.cuda.device_count()); [print(i, torch.cuda.get_device_name(i)) for i in range(torch.cuda.device_count())]"
```
示例输出：
CUDA_VISIBLE_DEVICES= None
device_count= 3
0 NVIDIA GeForce RTX 3090
1 NVIDIA GeForce RTX 3090
2 NVIDIA GeForce RTX 2080 Ti

##### 检测GPU可用
```
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"  
print(f"使用设备：{DEVICE}")
```
### NPU

```
npu-smi info
```
