<div align="center">

# 🌌 Oh My Knowledge

### ✨ 我的个人知识库 · My Personal Knowledge Garden

**保持好奇，持续探索；把零散的发现，慢慢连成自己的知识宇宙。**

📖 学习笔记　·　🧠 思考沉淀　·　🛠️ 动手实践　·　🌱 持续生长

![Knowledge Garden](https://img.shields.io/badge/Knowledge-Garden-6C63FF?style=for-the-badge)
![Personal Wiki](https://img.shields.io/badge/Personal-Wiki-00A896?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Growing-F4A261?style=for-the-badge)

**[🗺️ 知识地图](#-知识地图) · [🌟 精选入口](#-精选学习入口) · [📂 仓库结构](#-仓库结构) · [🔖 如何使用](#-如何使用)**

</div>

---

## 👋 欢迎来到我的知识花园

这里是我的 **个人知识库**，用来记录学习、整理思考、积累经验，也收藏那些值得回头再看的灵感。

它不局限于某一门技术：既有 AI 与大语言模型的原理和实验，也有后端开发、计算机基础、数学、工程实践与日常问题记录。我希望这里不仅能回答“学过什么”，也能留下“为什么这样理解”“遇到问题如何解决”。

> 🌱 **知识库不是一次性写完的作品，而是一座持续生长的花园。**  
> 从概念到实践，从问题到复盘，慢慢把知识连接起来。

## 🗺️ 知识地图

以下入口均对应仓库中**已经存在**的目录。随着新领域加入，这张地图也会继续扩展。

| 领域 | 在这里可以找到 | 入口 |
| :--- | :--- | :--- |
| 🤖 **大语言模型** | Transformer、MiniMind、预训练、微调、RAG、模型架构、论文笔记 | [LLM](LLM/) |
| 🧠 **AI 基础与工具** | AI 推理、Hugging Face、Ollama、环境搭建 | [AI 基础知识](AI基础知识/) |
| ⚡ **推理与 Agent** | vLLM、Agent 概念与相关实践 | [AI 推理](AI推理/) · [Agent](Agent/) |
| 💻 **开发经验** | Python、Git、Docker、Kubernetes、数据库、调试与部署 | [开发经验](开发经验/) |
| 🏗️ **后端与计算机基础** | API、接口、缓存、进程、计算机组成原理 | [后端基础](后端基础知识/) · [计算机基础](计算机基础知识/) |
| 📐 **数学基础** | 线性代数与矩阵运算 | [线性代数](线性代数/) |
| 🛠️ **工程实践** | 需求分析、设计记录、问题排查与 Bug 复盘 | [需求记录](1需求/) · [Bug](Bug/) · [问题记录](问题记录/) |
| 🗒️ **日常与业务记录** | 每日任务、业务理解、信息同步 | [每日任务](1每日任务/) · [业务理解](业务理解/) · [信息同步](信息同步/) |
| 🎨 **辅助工具与资料** | Excalidraw 草图、Copilot 提示词与对话、博客摘录 | [Excalidraw](Excalidraw/) · [Copilot](copilot/) · [博客](博客/) |

<details>
<summary><strong>📦 其他专项记录</strong></summary>

仓库还包含 [内网配置](内网配置/) 和 [美的自动捞取文件](美的自动捞取文件/) 等特定场景的操作记录。部分资料依赖特定环境或业务背景，阅读和使用时请自行核对适用范围。

</details>

## 🌟 精选学习入口

不必从头到尾阅读整个仓库。可以从一个问题、一项技术或一条学习路线开始。

### 🤖 AI 与大模型：从原理走向实践

| 想了解什么 | 推荐入口 |
| :--- | :--- |
| 大模型的基本概念 | [LLM 概念词](LLM/1概念词.md) · [深入理解 LLMs](LLM/深入理解LLms.md) |
| Transformer 为什么重要 | [Transformer 专题](LLM/transformer/) · [手撕 Transformer](LLM/transformer/手撕transformer.md) |
| 如何准备数据与训练模型 | [训练数据准备](LLM/动手学习大模型/4.训练数据准备.md) · [预训练](LLM/动手学习大模型/5.预训练.md) |
| 如何进行模型微调 | [监督微调](LLM/监督微调.md) · [微调笔记](LLM/动手学习大模型/5.1%20微调（Fine-tuning）.md) |
| 如何动手做模型实验 | [MiniMind 学习笔记](LLM/从零开始训练一个大语言模型-Minimind.md) · [Notebook 实践](LLM/动手学习大模型/) |
| 推理、评估与应用 | [模型推理](LLM/动手学习大模型/6.推理.md) · [模型评估](LLM/模型评估.md) · [RAG](LLM/动手学习大模型/10.RAG.md) |
| 模型内部组件与论文 | [架构模块](LLM/架构模块/) · [经典论文](LLM/经典论文/) |

<details>
<summary><strong>🧭 展开：LLM 参考学习路线</strong></summary>

```text
基础概念与发展
       ↓
Transformer / 模型架构
       ↓
数据准备 → 预训练 → 监督微调
       ↓
MiniMind / Notebook 动手实验
       ↓
模型推理 → 模型评估
       ↓
提示学习 / RAG / 论文阅读
```

这是一条可选的学习路线，不代表所有内容必须按顺序完成。

</details>

### 💻 开发与工程：从知识到解决问题

- **开发基础**：[Python](开发经验/11.python基础.md) · [Git](开发经验/05.Git命令.md) · [Linux](开发经验/04.linux命令.md)
- **环境与部署**：[Docker](开发经验/03.Docker.md) · [Kubernetes](开发经验/05.Kubernetes运行.md) · [模型部署](开发经验/13.模型部署.md)
- **数据与服务**：[数据库基础](开发经验/14.数据库基础.md) · [Redis](开发经验/18.redis相关.md) · [API](后端基础知识/API篇.md)
- **观测与排障**：[Grafana 埋点](开发经验/02.Grafana埋点.md) · [日志、调试与性能分析](开发经验/15.日志,调试与性能分析.md) · [问题记录](问题记录/)
- **工程记录**：[需求与设计](1需求/) · [每日任务](1每日任务/) · [业务理解](业务理解/)

### 📚 基础与工具：补全知识拼图

- **计算机基础**：[计算机组成原理](计算机基础知识/计算机组成原理.md) · [进程与端口](后端基础知识/进程与端口.md)
- **数学基础**：[矩阵运算](线性代数/矩阵运算.md)
- **AI 工具**：[Ollama](AI基础知识/Ollama.md) · [Hugging Face](AI基础知识/Huggingface使用.md) · [vLLM](AI推理/vLLM/1.vLLM.md)
- **辅助记录**：[Excalidraw](Excalidraw/) · [Copilot 自定义提示词](copilot/copilot-custom-prompts/)

## 📂 仓库结构

```text
oh_my_knowledge/
├── README.md             # 知识库首页
├── LLM/                  # 大语言模型、训练实践与论文
├── AI基础知识/           # AI 工具、推理与环境
├── AI推理/               # vLLM 等推理专题
├── Agent/                # Agent 知识
├── 开发经验/             # 开发、运维与工具实践
├── 后端基础知识/         # API、缓存、接口等
├── 计算机基础知识/       # 计算机原理
├── 线性代数/             # 数学笔记
├── 1需求/                # 需求与设计记录
├── 1每日任务/            # 日常工作与复盘
├── Bug/                  # Bug 相关记录
├── 问题记录/             # 问题追踪
├── 业务理解/             # 业务知识
├── 信息同步/             # 信息与会议记录
├── 内网配置/             # 特定环境配置
├── 美的自动捞取文件/     # 专项操作记录
├── Excalidraw/           # 绘图草稿
├── copilot/              # Copilot 对话与提示词
├── 博客/                 # 博客资料
└── 截图/                 # 笔记配图
```

> 这里只展示一级目录。目录与笔记会随学习和实践持续调整。

## 🔖 如何使用

**在线探索**：从 [知识地图](#-知识地图) 选择主题，或直接进入对应目录查看笔记。

**本地阅读**：用支持 Markdown 的编辑器打开 `.md` 文件；`.ipynb` 文件可使用 Jupyter 等兼容工具查看。

```bash
git clone https://github.com/wjiaxin853-ctrl/oh_my_knowledge.git
cd oh_my_knowledge
```

**动手实验**：部分 Notebook、命令和配置依赖特定环境。运行前请确认依赖、权限、数据与适用场景，不要直接在生产环境执行未经验证的操作。

## 🌱 关于记录与成长

这个仓库保存的是我在不同阶段的理解、实践与思考，并不追求每篇笔记一开始就是完整教程。

- **记录**：把学到的知识用自己的语言表达出来。
- **连接**：在技术、基础理论与真实问题之间建立联系。
- **实践**：通过代码、实验和排查验证想法。
- **复盘**：留下遇到的问题与解决过程。
- **迭代**：持续修订，让知识库和自己一起成长。

部分工程笔记与操作记录可能包含特定业务背景，请在公开分享或复用前注意信息脱敏和环境差异。

欢迎通过 [Issues](https://github.com/wjiaxin853-ctrl/oh_my_knowledge/issues) 交流与反馈。

---

<div align="center">

### ✨ Stay Curious · Keep Learning · Keep Building

**以好奇心为起点，以长期积累为答案。**

<sub>📚 Made for learning, thinking & growing.</sub>

</div>
