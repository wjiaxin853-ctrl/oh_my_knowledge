# seq2seq 翻译模型（英→中）学习实验

对应学习问题：**什么是 seq2seq / RNN 如何编码解码 / 长文本的瓶颈 / Attention 如何改善**。
用 WMT 官方 News Commentary v14 英中平行语料，亲手训练两个模型对比验证。

## 目录结构

```
1基于seq2seq的翻译模型/
├── dataset/
│   └── news-commentary-v14.en-zh.tsv    # 32 万句对（en\tzh，tab 分隔）
├── 04_seq2seq_翻译.py                   # 训练脚本（GRU Encoder-Decoder，可开关 Attention）
├── 05_seq2seq_对比.py                   # 加载两个 checkpoint，并排对比翻译
├── checkpoints/
│   ├── seq2seq_attn_best.pt             # 带 Attention（best valid loss）
│   └── seq2seq_noattn_best.pt           # 无 Attention 对照组
└── figures/
    ├── 04_seq2seq_损失曲线_attn.png     # Attention 版 20 epoch 损失曲线
    ├── 04_seq2seq_损失曲线_noattn.png   # 无 Attention 对照组
    └── 04_seq2seq_对齐热力图_attn.png   # 源词↔目标词对齐热力图（软对齐可视化）
```

## 数据

- **来源**：WMT 官方 News Commentary v14（新闻评论平行语料），32 万句对
- **格式**：每行 `英文\t中文`，tab 分隔；原始语料含长句与噪声
- **清洗**：去空句/超长句/长度比失衡/重复（`clean_pairs`），短句配置后剩 3.4 万句

## 脚本用法

```bash
# 训练带 Attention 模型（短句配置，20 epoch，约 4.5 分钟）
python3 04_seq2seq_翻译.py --max-samples 30000 --max-en-len 12 --max-zh-len 18 --epochs 20

# 训练无 Attention 对照组（同参数加 --no-attention）
python3 04_seq2seq_翻译.py --max-samples 30000 --max-en-len 12 --max-zh-len 18 --epochs 20 --no-attention

# 并排对比两个已训练模型（翻译同一批验证句）
python3 05_seq2seq_对比.py
```

注意：05 的长度参数需与训练时一致（默认按短句配置 12/18），否则词表对不上 checkpoint。

## 实验结果（3 万句 / 20 epoch / hidden 256 / teacher forcing 0.5）

| 指标 | 无 Attention | 带 Attention |
|------|-------------|--------------|
| best valid loss | 5.446（epoch 4） | **5.176**（epoch 5） |
| 训练耗时 | ~3.4 分钟 | ~4.4 分钟 |
| 参数量 | 7.49M | 7.82M |

两模型均在第 5~6 epoch 后过拟合（train loss 持续下降、valid 回升）——checkpoint 自动保留 best。

**同一句对比（验证集，模型未见过）**：

| 英文 | 参考 | 无 Attention | 带 Attention |
|------|------|-------------|--------------|
| many wealthier countries have followed this path | 许多富裕国家就表现出这一模式 | 许多国家的经不不不不。 | 许多国家都有这一这一这一点。 |
| in any case, the iraqi army is a defeated force | 无论如何，伊军都只能算是战败部队 | 在在，，，，，，是是是是是的。 | 在何，，，，只是一个失败的的。 |
| india's urban awakening | 印度的城市觉醒 | 印度的的的的的 | 印度的新兴市场 |
| the economic crisis is getting worse | （自编） | 经济危机的危机已经一个危机。 | 经济危机已经显著。 |
| europe is being cautious | （自编） | 欧洲的欧洲是欧洲的一个。 | 欧洲就是。 |

## 核心结论（对应学习问题③④）

1. **信息瓶颈真实存在**：无 Attention 的解码器只靠一个定长向量，失去源句指引后
   极易"空转"——反复输出"的/是/在/经济"等高频词，长句后半段尤其严重。
2. **Attention 缓解瓶颈**：带 Attention 模型每步动态聚焦源句相关词（`c_t = Σ α_tj h_j`），
   输出重复明显减少、语义贴合度更高，valid loss 更低（5.176 vs 5.446）。
3. **可解释性**：`figures/04_seq2seq_对齐热力图_attn.png` 展示了 α 权重——
   每个中文输出字符对哪些英文源词"注意力更强"，就是软对齐。
4. 翻译质量整体仍有限（小模型 + 3 万句），但"有无 Attention"的差距方向明确。

## 为什么效果还不够好（改进方向）

- **数据量**：3 万句远低于实用门槛（工业级 百万~亿级）
- **过拟合**：teacher forcing 0.5 固定、无正则；可加 dropout、label smoothing、tf 退火
- **词表**：英文 top-2 万词 + 中文按字；业界用 SentencePiece/BPE 子词
- **模型**：单层 GRU hidden 256；可加深、加大、换 LSTM
- **解码**：greedy → beam search；训练 → 大 Transformer（你下一个学习目标）
