
### 数据集结构
##### 预训练数据集
文件名字：pretrain_t2t_mini.jsonl(预训练数据)

```
{"text": "给我生成一首有关秋天的诗歌。秋日早晨，清风拂面。金色的叶子，似火在燃烧..."}
{"text": "北京是中华人民共和国的首都，也是中国的政治、文化、科技和国际交往中..."}
...
```

**特点**：就是**纯粹的文本**，没有`问题`、`选项`、`答案`这些人工标注。比如文章，书籍，网页等。

==模型学习预测下一个 token==

- 每一行是一个 JSON 对象
- 只有一个字段 text
- 内容是纯文本，模型学习预测下一个 token

训练目标 : Next Token Prediction (下一个词预测)

- 输入: "今天天气很好"
- 目标: 预测下一个最可能的词

##### 指令微调数据集

```
{"conversations": [
  {"role": "user", "content": "请介绍一下北京"},
  {"role": "assistant", "content": "北京是中国的首都."，"reasoning_content": "可选"}
   ...  
]}
```

- 每一行是一个 JSON 对象
- 包含 conversations 字段
- 对话格式，包含 user (用户) 和 assistant (助手) 角色
- 有些还包含 reasoning_content (思考过程)

##### 常见的指令微调格式
###### 1.单轮指令微调格式
```
{
  "prompt": "解释什么是机器学习",
  "response": "机器学习是人工智能的一个分支..."
}
```
###### 2.多轮对话格式（ShareGPT格式) - **最常用**
```
{
  "conversations": [
    {"role": "user", "content": "你好，能帮我写首诗吗？"},
    {"role": "assistant", "content": "当然可以！请问你想写什么主题的诗？"},
    {"role": "user", "content": "关于秋天的"},
    {"role": "assistant", "content": "秋风萧瑟天气凉，草木摇落露为霜..."}
  ]
}
```
###### 3.带推理链的格式 (CoT - Chain of Thought)
```
{
  "conversations": [
    {"role": "user", "content": "小明有5个苹果，给了小红2个，又买了3个，现在有几个？"},
    {"role": "assistant", 
     "content": "小明现在有6个苹果。",
     "reasoning_content": "1. 开始时小明有5个苹果\n2. 给小红2个：5-2=3\n3. 又买3个：3+3=6\n所以答案是6个"}
  ]
}
```

###### 4.偏好对齐格式 (RLHF)
```
{
  "prompt": "如何快速致富？",
  "chosen": "通过努力工作、学习和投资，逐步积累财富...",
  "rejected": "买彩票、赌博或者投机取巧..."
}
```

### 预训练

预训练参数介绍

**--batch_size**：每次 GPU 处理的 样本数量。
所需内存 ≈ batch_size × 模型参数 × 2(梯度) × 数据大小

