## 1.配置访问wiki工具
![[截屏2025-11-27 19.19.50.png]]
## 2.spec规范同步
回答一下上午大家关于Spec的问题

1，为什么不用opensepc？
open-spec定义的是一套SOP（command），面向开发过程的产出管理，适用于个人开发；（prompt写的比较好，可以参考）
sangfor-spec定义的是SKILL+角色+SOP的一套工程方案，面向开发过程的规范管理/质量约束，适用于企业开发；

2，Spec编写规范
体系目前并没有推出正式的规范，大家可以先参考claude的规范

3，Spec评审流程/规范
体系目前并没有推出正式的规范，初步管理方案是参考TMG标准管理规范
![[WeChatWorkScreenshot_c70339d2-d1cf-4898-a8c3-ec91ba3c15ae.png]]
![[WeChatWorkScreenshot_ec197aeb-e661-43a3-9b43-90431b3230ba.png]]
![[WeChatWorkScreenshot_c6988536-ee35-40e5-a99e-e2b3dbf29f11.png]]![[WeChatWorkScreenshot_17a973d1-2676-404b-a125-8e08ed821d1f.png]]简单来说，先参考claude的规范来写skill 和 command

## 3.统计tocken用量
##### 方式一：
```
大家统计一下10月份的token用量，用于后续预估模型资源
CC参考统计方式：
方式一：pip install claude-monitor && claude-monitor --view monthly
方式二：pip install ccusage && ccusage monthly
codex 目前还没有很好的统计方式
```

##### 方式二：在线网站CRS
```
http://10.65.245.59:3000/admin-next/api-stats
```


## 4，CoStrict CLI

```
 npm install -g @costrict/cs --registry=http://npm.uedc.sangfor.com.cn:80
```

```
🚀 CoStrict CLI 抢先体验

将 AI 的力量带到你的终端

CoStrict CLI 是一款终端 AI 助手，让你在命令行中直接使用体验 CoStrict 的能力。无需切换窗口，无需复制粘贴，让 AI 成为你开发流程中的原生工具。

✨ 核心特性
- 🎯 多模型支持 - 支持登录 CoStrict 使用同等的模型体验
- 🔧 终端原生 - 为命令行开发者量身打造，无缝融入工作流
- 🛠 强大工具 - 内置文件操作、代码分析、Shell 执行等能力
- 🔌 高度可扩展 - 支持 MCP 协议、Skill 能力，轻松集成自定义工具

📦 快速开始

1. 安装

内网: npm install -g @costrict/cs --registry=http://npm.uedc.sangfor.com.cn:80
外网: npm install -g @costrict/cs --registry=https://registry.npmjs.org

2. 启动

终端输入：cs
（要求 nodejs 版本 v20 以上）

3. 登录认证

在交互界面中输入：/auth
可选择 CoStrict 按提示完成登录流程

4. 切换模型

/model

5. 切换 Agent

/agents

根据任务需求选择不同的 Agent 配置

6. 清理会话

/clear

🏢 内网环境配置

如果你在内网环境中使用，需要配置内网服务地址：
export COSTRICT_BASE_URL=https://shenma.sangfor.com.cn
```


如果/auth不可以，就尝试/login


![[截屏2026-01-29 20.23.22.png]]
 https://shenma.sangfor.com.cn 



### DSP知识库插件

dsp知识库已经上线了，通过以下命令进行安装（当前仅支持claude code和csc）
/plugin marketplace add git@git.sangfor.com:ai-native/cospowers/cospowers.git
/plugin install cospowers
/plugin marketplace add git@git.sangfor.com:ai-native/cospowers/dept-cospowers/cospowers-dsp.git
/plugin install cospowers-dsp
/reload-plugins
使用自然语言提问即可（比较细节的提问可能识别不到，显式调用/kb-query即可），有问题欢迎反馈

附（csc安装）：
npm install -g @costrict/csc --registry=http://npm.uedc.sangfor.com.cn:80
将以下 alias 添加到全局中
export COSTRICT_BASE_URL="https://shenma.sangfor.com.cn"
alias csc-bypass="IS_SANDBOX=1 csc --permission-mode bypassPermissions"




https://daedalus.rd.sangfor.com/home
隐性知识库，缺失或是积累沉淀的知识可以上传到这里，kb-quer可以查询到，让知识库变得越来越好用

![[企业微信截图_1042467f-7b74-4bae-b2c9-9c218714456a.png]]


### DSP升级排查skill

DSP 升级问题排查 skill dsp-upgrade-triage 已上线， 安装 cospowers-dsp 即可使用 https://git.sangfor.com/ai-native/cospowers/dept-cospowers/cospowers-dsp
以后遇到升级问题可以先用这个 skill  跑跑， 过程中有任何问题直接反馈给我

### 集成仓库更换包号skill
有时间测试一下这个自动提集成仓库的skill：npx ainative@latest install skill https://git.sangfor.com/85555/skill.git:dsp-release:85555
需要配置 ~/.qianliu/config.json ,使用例子：

查一下某分支有哪些仓库未同步到集成仓库
把某个/某些仓库的某分支更新到集成仓库（更新集成版本、发 MR）
把新仓库加入集成自动化服务/同步清单
同步所有仓库某分支的最新版本到集成仓库
查看某次发布任务的进度、失败原因或 MR 链接
手动操作：http://10.74.167.67:8989/