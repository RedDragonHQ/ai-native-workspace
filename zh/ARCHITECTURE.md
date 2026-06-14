# 架构

> **四层如何协同、边界为什么重要、以及忽略边界时会怎么崩。**

本文档给想理解设计的人读。如果你是被要求搭 workspace 的 agent,请读 [`zh/BOOTSTRAP.md`](BOOTSTRAP.md)。

---

## 核心主张

> AI agent 的 workspace 是一个操作系统。Agent 是进程,Model 是内核,Protocol 是 syscall,Application 是用户程序。

这不是比喻 —— 这是设计约束。每层单一职责,对相邻层有清晰接口。违反边界 = 锁定 + 脆弱,或两者兼得。

---

## 四层详解

### 第 4 层:Application(应用)

**这层有什么**:你的项目、inbox、wiki。

```
workspace/
├── CLAUDE.md              ← 工作区级认知配置
├── inbox/                 ← 草稿缓冲
│   ├── README.md          ← inbox 文件格式
│   └── TODO.md            ← 跨会话流动清单
├── projects/              ← 具体项目
│   ├── project-a/
│   │   └── CLAUDE.md      ← 项目级认知配置
│   └── project-b/
│       └── CLAUDE.md
└── wiki/                  ← 沉淀后的知识库
    └── MANIFEST.md        ← 渐进加载索引
```

**单一职责**:承载工作产出和它积累的知识。

**与下层的边界(Protocol)**:应用层只通过第 3 层定义的协议与外界交互,绝不直接调 model API。

**核心设计:inbox 缓冲区**

大多数知识管理框架假设你捕获时就知道信息该放哪。**这是错的**。你几乎永远不知道 —— 这是 bug 报告、设计决策、会议记录、还是纯噪音?

inbox 说:**先捕获,后分类**。一条 inbox 文件的生产成本低(几行字)、承诺也低(明天可以删)。只有当某条 inbox 项被引用 3 次以上,它才升级到 `wiki/`。

这跟 David Allen GTD 的 capture 步骤同理,应用到 AI agent 工作流:**在捕获点降低摩擦**。

**核心设计:通过 MANIFEST 渐进加载**

Context 窗口有限。200 篇 wiki 不可能全进 context。解法:`MANIFEST.md` 是索引,每篇一行摘要。Agent 先读 MANIFEST(200 行),基于摘要挑 2-5 篇相关文档,只加载那些。

跟数据库索引 / 图书馆目录卡同理:**扫便宜,取贵**。

### 第 3 层:Protocol(协议)

**这层有什么**:约定、CLI 工具、MCP server,让 agent 与外界交互。

```
协议类型:
┌─────────────────────────────────────────────┐
│  文件协议(CLAUDE.md 约定)                      │ ← 永远有
│  CLI 工具(git / curl / docker / node / python)│ ← 通常有
│  MCP Servers(SearXNG / 自定义 / ...)         │ ← 按需加
└─────────────────────────────────────────────┘
```

**单一职责**:定义 agent 与外界的交互方式。

**与下层的边界(Agent)**:agent 用协议但不实现协议。换 agent,保留协议。

**与上层的边界(Application)**:应用层只用协议与外界交互,它不知道这些协议背后是 MCP、CLI 工具,还是 curl 包装。

**核心设计:文件协议作为最小公分母**

MCP 强大但可选。CLI 工具强大但不总可用。**markdown 文件永远能读**。框架的核心配置(CLAUDE.md)是 markdown —— 意味着任何能读文本的 agent 都能部分跑这个框架,即便零集成。

这是 AI agent 框架的 Postel 定律:**在接收上宽容**。

### 第 2 层:Agent(智能体)

**这层有什么**:读 CLAUDE.md 并决定下一步做什么的软件。

已测试:
- Claude Code(Anthropic)—— 集成最成熟
- Codex CLI(OpenAI)—— 稍改 CLAUDE.md 即可
- Cursor —— 通过 `.cursorrules` 适配可用
- 自研 LangGraph agent —— 自定义 CLAUDE.md parser 即可

**单一职责**:推理、规划、工具编排。

**与下层的边界(Model)**:agent 调 model API,但不关心哪个 model 响应。同样的 prompt,不同的后端。

**与上层的边界(Protocol)**:agent 用协议但不拥有协议。换 agent,保留协议。

**核心设计:agent 无关的配置**

大多数 workflow 框架为单一 agent 设计。Anthropic 官方文档假设 Claude Code。Cursor rules 假设 Cursor。本框架的 CLAUDE.md 设计为**任何能解析 markdown 的 agent 都能读** —— 当前所有 agent 和绝大多数未来 agent 都满足。

"CLAUDE.md" 这个名字来自 Anthropic 的约定,但内容是 agent 无关的。如果你的 agent 用不同配置文件名(`.cursorrules`、`AGENTS.md` 等),直接把内容粘过去就行。

### 第 1 层:Model(模型)

**这层有什么**:给 agent 推理能力的 LLM。

```
已测试的模型提供方:
├── DeepSeek(Anthropic 兼容 API)—— 日常驱动,低成本
├── Anthropic Claude(直连或经 Bedrock/Vertex)
├── Qwen(阿里,经 DashScope 走 Anthropic 兼容)
├── OpenAI GPT 系列
├── 本地模型(ollama / vLLM)
└── 第三方中转(OpenRouter 等)
```

**单一职责**:推理能力。

**与上层的边界(Agent)**:model 接收 prompt、返回 completion。它不知道项目、inbox、wiki。

**核心设计:模型切换 = 改环境变量**

因为框架在多个 provider 上使用 Anthropic 兼容 API 约定,换模型就是:

```bash
# 日常:便宜快
source daily.sh     # 设 ANTHROPIC_BASE_URL=deepseek

# 重活:强推理
source strong.sh    # 设 ANTHROPIC_BASE_URL=claude-relay
```

同样的 agent、同样的 workspace、同样的工作流。不同的模型。**模型是最可丢弃的一层** —— 这正是重点,因为模型能力和定价每季度都在变。

---

## 多层解耦为什么重要

单生态方案(单一 agent + 单一模型 + 单一提供商)在以下情况 OK:

- 你永远满意当前提供商
- 你只为单一场景构建
- 你不在乎可移植性

**它在以下情况崩**:

- **提供商涨价**(2024 OpenAI 涨过,还会再发生)
- **提供商质量下滑**(2025 几家都发生过)
- **更强的模型出了**(每 6 个月一次)
- **你的用例扩展**(从 SWE 起步,然后加研究,再加内容)
- **你选的 agent 砍功能**(Cursor 多次 pivot 过)

4 层框架前期多花一点搭建成本。回报是:

| 年份 | 单生态 | 多层 |
|------|--------|------|
| 第 1 年 | 起步轻松,进展顺利 | 略多搭建,进展顺利 |
| 第 2 年 | 进展顺利 | 进展顺利 |
| 第 3 年 | 锁在旧提供商上;换就重写 | 换一层,其他保留 |
| 第 4 年 | 锁定 | 可移植 |

这就是经典的 build-vs-buy 权衡,应用在 AI 工作流上。

---

## 反模式完整列表

反模式和功能一样定义这个框架。下面每一条都对应本框架提取自的那个 workspace 里真实发生过的失败。

### Agent 层

| ❌ 不要 | ✅ 要 | 为什么 |
|---------|------|-------|
| 上来就搞 multi-agent | 单 agent 起步,只有 context 真不够了才拆 | Multi-agent 加协调开销,90% 工作负载用不到 |
| 启动时全量加载所有文件 | 读索引,按需深入 | Context 窗口有限,全量加载浪费 |
| 写 500 行 CLAUDE.md | ≤ 100 行;细节放项目级 CLAUDE.md | 根 CLAUDE.md 是目录,不是书 |
| 不经确认自动写 inbox | 提示用户,不要突袭 | 写入是承诺,突袭写入制造债务 |
| 把 TODO.md 当指令执行 | 它是草稿 —— 引用前先验证 | TODO 积累垃圾,不验证就执行放大错误 |

### Protocol 层

| ❌ 不要 | ✅ 要 | 为什么 |
|---------|------|-------|
| 核心工作流依赖 MCP | 文件协议是底线,MCP 是天花板 | MCP 好用但可选;你的核心工作流不该依赖跑着的 server |
| 所有 CLI 都自己造 | 先用 git/curl/docker | 重造标准工具浪费时间、困惑协作者 |
| 在脚本里硬编码 API key | 用 env 文件 + env 变量切换 | 密钥属于 env,不属于源码 |

### Application 层

| ❌ 不要 | ✅ 要 | 为什么 |
|---------|------|-------|
| 跳过 inbox 直接写 wiki | Inbox 是缓冲区,让沉淀自然发生 | 过早 wiki 化 = 孤儿半成品文档 |
| 把事实散落在随机文件里 | 稳定事实放进项目文档或 wiki | CLAUDE.md 放工作流,事实应该稳定 |
| 让 TODO 永远留着 | 每周归档或删过时项 | 不缩水的 TODO 清单是负罪感清单 |
| 让单个项目霸占根 CLAUDE.md | 项目细节搬到项目 CLAUDE.md | 根 CLAUDE.md 跨项目,项目 CLAUDE.md 项目内 |

### Model 层

| ❌ 不要 | ✅ 要 | 为什么 |
|---------|------|-------|
| 硬编码单一模型 | 用 env 变量切换 | 模型每季度变,env 切换零成本 |
| 假设模型质量稳定 | 定期用真实任务测试 | 退化会发生,无声退化最糟 |
| 把成本当固定值 | 按任务类型追踪成本 | 便宜模型在多数任务上 80% 效果、5% 成本 |

---

## 何时不适用本框架

以下场景本框架过度:

- **单个短期项目** —— 一份简单 CLAUDE.md 够了
- **一次性任务** —— 没有跨会话状态,就不需要 inbox/wiki
- **不用 AI agent 的工作流** —— 不是每天用 AI agent,框架的日常实践就回报不了
- **已有成熟约定的团队** —— 你团队已有跑通的工作流,不要为理论换掉它

本框架在以下场景回报:

- **你每天用 AI agent** —— 实践会复利
- **你跨多个项目工作** —— 跨项目协调是 inbox/MANIFEST 的战场
- **你的工作累积知识** —— 研究、基础设施、PKM
- **你在乎可移植性** —— 不想被单一提供商锁死

---

## 来源与谱系

本框架提取自 **RedDragon Workspace** —— 一个从 2025 年起真实运行的研究 workspace:

- 8+ 个活跃项目,跨 ML 研究、多 agent 系统、基础设施、安全研究、PKM
- 6+ 个月每日 agent 驱动使用
- 多次模型切换(OpenAI → Claude → DeepSeek),工作区从未重写
- 150+ inbox 文件、60+ wiki 条目

框架是**沉淀物**,不是设计。本 repo 里的每条约定都对应一次真实失败。反模式列表比功能列表长,因为每一条都是伤疤。

谱系:
- **PKM 原则** —— 来自 Zettelkasten、PARA、Johnny Decimal
- **GTD** —— 来自 David Allen(inbox 缓冲区是纯 GTD capture)
- **渐进披露** —— 来自信息架构
- **Agent-first 设计** —— 来自观察 Claude Code / Codex 实际怎么读文件(先索引,后详情)
- **多 provider API 设计** —— 来自 DeepSeek、Qwen 等采用的 Anthropic 兼容 API 约定

---

## 延伸阅读

如果你对这个框架有共鸣,可能也会喜欢:

- **Anthropic 官方 CLAUDE.md 文档** —— 基线,Claude 专属但扎实
- **`awesome-claude-code`** —— 社区整理的 CLAUDE.md 样例集
- **MindStudio "Personal AI OS" 系列** —— 类似想法,SaaS 视角(闭源但写得不错)
- **Johnny Decimal** —— 最早的 "文件夹结构 = 认知基础设施" 框架
- **Building a Second Brain(Tiago Forte)** —— 本框架继承的 PKM 原则
