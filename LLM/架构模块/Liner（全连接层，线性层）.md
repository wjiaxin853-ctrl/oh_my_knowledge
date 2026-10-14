Linear 层接受一个输入张量，通过一个[矩阵乘法](https://zhida.zhihu.com/search?content_id=224904932&content_type=Article&match_order=1&q=%E7%9F%A9%E9%98%B5%E4%B9%98%E6%B3%95&zhida_source=entity)和一个偏置向量相加的方式计算输出张量，其中矩阵和偏置向量是 Linear 层的可学习参数。


### 公式
$y = xW^T + b$
- xx 是输入
- WW 是权重 (weight)矩阵
- bb 是偏置 (bias)向量 (vector)
- yy 是输出
### 作用
Linear 层可以==将输入张量通过线性变换投影到其他维度空间中==（可以升维、降维或保持维度），从而提高模型的表现能力

### 使用场景

Linear 层常用于实现神经网络的前向传播，通常与[非线性激活函数](https://zhida.zhihu.com/search?content_id=224904932&content_type=Article&match_order=1&q=%E9%9D%9E%E7%BA%BF%E6%80%A7%E6%BF%80%E6%B4%BB%E5%87%BD%E6%95%B0&zhida_source=entity)（如 ReLU）配合使用，以增强模型的非线性表达能力。