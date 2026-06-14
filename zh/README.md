# AI-Native Workspace

> **一套分层框架,把任意 AI 编程 Agent 变成你的个人操作系统。**
>
> 适用于 Claude Code、Codex、Cursor,或任何能读 markdown 的 agent。

**这不是又一个 `.cursorrules` 模板。** 它是一套有主见的架构,用来搭建真正在跑的科研/工程工作区 —— AI agent 是主要操作者,人类是 agent 不可用时的兜底。

---

## 为什么需要这个框架

市面上大多数 "Claude Code workflow" repo 落入两类:

- **dotfiles 倾倒** —— 贴我的 `.claude/`、我的规则、我的 prompt。难泛化。
- **单生态绑定** —— 只为单一 agent + 单一模型 + 单一提供商设计。换一家就崩。

本框架基于不同的前提:

> **Workspace 是操作系统,Agent 是进程,Model 是内核。**
>
> 任何一层都可以独立替换,其他层不用重写。

### 多层解耦 —— 真正的差异化

| 层 | 单生态方案 | 本框架 |
|----|-----------|--------|
| **模型 (Model)** | 只能用 Claude(或只能用 GPT) | DeepSeek / Claude / Qwen / OpenAI / Gemini / 本地模型 —— 改 `.env` 即可切换 |
| **Agent** | 只能用 Claude Code | Claude Code / Codex CLI / Cursor / 自研 LangGraph agent |
| **协议 (Protocol)** | 只能用 Anthropic MCP | 文件协议(CLAUDE.md)+ CLI + MCP(可选) |
| **应用 (Application)** | 厂商提供的模板 | 你自己的项目、PKM、工作流 |

**为什么这很重要**: 明年 Anthropic 出了更强的 agent,你不用重写工作区。DeepSeek 比 Claude 便宜,你不用重写工作流。用腻了 MCP,你的知识库不会丢。

**框架是协议级的,不是产品级的。**

### AI-first,人类可读

CLAUDE.md **优先为 agent 消费而优化** —— 结构化到 AI agent 加载后就能直接干活。但它不是 agent 独占的:同一份文件人类完全能读,当 agent 不可用时,人类可以进入 agent 层、用同样的协议操作 workspace。

- **分层认知加载**:全局 → 工作区 → 子项目
- **渐进式上下文**:先看索引,按需深入
- **结构化连续性**:跨会话状态存在 CLAUDE.md 和项目文档里,不在聊天历史里
- **工作流契约**:会话结束协议、恢复协议、审计链

人类不只是拿附带收益 —— 他们是**兜底操作者**。当 agent 层挂掉时(API 挂了、模型退化、agent 处理不了的边界情况),人类读同一份 CLAUDE.md,从 agent 断掉的地方接手。

---

## 四层架构

```
┌─────────────────────────────────────────────────────┐
│  应用层 (Application)                                │
│  项目 · Inbox(草稿缓冲)· Wiki(沉淀后的知识库)        │
├─────────────────────────────────────────────────────┤
│  协议层 (Protocol)                                   │
│  CLAUDE.md 约定 · CLI 工具 · MCP Servers             │
├─────────────────────────────────────────────────────┤
│  智能体层 (Agent)                                     │
│  Claude Code · Codex · Cursor · 任意能读 md 的 agent  │
├─────────────────────────────────────────────────────┤
│  模型层 (Model)                                      │
│  DeepSeek · Claude · Qwen · OpenAI · 本地模型          │
└─────────────────────────────────────────────────────┘
```

每层单一职责。边界靠**文件约定**强制执行,不靠代码 —— 所以任何能读 markdown 的 agent 都能跑这个框架。

完整设计见 [ARCHITECTURE.md](ARCHITECTURE.md)。

---

## 仓库结构

```
ai-native-workspace/
├── README.md                  ← 你在这里(英文)
├── zh/README.md               ← 中文版(本文件)
├── ARCHITECTURE.md            ← 4 层设计详解 + anti-patterns(英文)
├── zh/ARCHITECTURE.md         ← 中文版
├── BOOTSTRAP.md               ← 给 agent 读的 bootstrap 指令(英文)
├── zh/BOOTSTRAP.md            ← 中文版
├── LICENSE                    ← MIT
├── examples/
│   └── sample-workspace/      ← 最小可运行样例(英文)
└── zh/examples/
    └── sample-workspace/      ← 最小可运行样例(中文)
```

**`BOOTSTRAP.md` 是最关键的文件。** AI agent 读它来搭建一个全新的 workspace。把内容粘到空目录里的 Claude Code / Codex / Cursor,agent 会自动帮你把结构建起来。

---

## 快速开始

### 方式 A:让 agent bootstrap

1. 创建一个新目录
2. 在该目录打开 Claude Code / Codex / Cursor
3. 把 [`BOOTSTRAP.md`](../BOOTSTRAP.md)(或 [`zh/BOOTSTRAP.md`](BOOTSTRAP.md))粘进第一条消息
4. Agent 会扫描你的目录并搭建结构

### 方式 B:直接拷贝样例

```bash
cp -r examples/sample-workspace ~/my-workspace
cd ~/my-workspace
$EDITOR CLAUDE.md          # 按你的场景调整
```

或中文版:

```bash
cp -r zh/examples/sample-workspace ~/my-workspace
cd ~/my-workspace
$EDITOR CLAUDE.md
```

### 然后呢?

用起来。框架的奥义在日复一日的使用中浮现:

- **第 1 天**:每次会话结束写 `inbox/YYYY-MM-DD-thing.md`
- **第 2 周**:`inbox/TODO.md` 自然成为你的跨会话状态
- **第 2 个月**:inbox 越来越挤时,`wiki/` 自然浮现
- **第 3 个月+**:CLAUDE.md 和项目文档开始沉淀你的偏好、决策、教训

没有启动仪式。架构是日常实践,不是安装流程。

---

## 核心思想(TL;DR)

1. **Inbox 是缓冲区** —— 低摩擦捕获,任何东西都可以先丢进 `inbox/`,以后再分类
2. **Wiki 是沉淀层** —— 当某个 inbox 条目被引用 3 次以上,它就升级到 `wiki/`
3. **MANIFEST 是索引** —— 不要全量读 wiki,先读 MANIFEST.md(每篇一行摘要),再按需深入
4. **Anti-patterns 是护栏** —— 框架既由它允许的事定义,也由它禁止的事定义

---

## 反模式(摘几条)

| ❌ 不要 | ✅ 要 |
|--------|------|
| 上来就搞 multi-agent | 单 agent 起步,只有 context 真不够了才拆 |
| 启动时全量加载所有文件 | 读索引,按需深入 |
| 跳过 inbox 直接写 wiki | Inbox 是缓冲区,让沉淀自然发生 |
| 写 500 行 CLAUDE.md | ≤ 100 行;细节放子项目 CLAUDE.md |
| 把 TODO.md 当指令执行 | 它是草稿 —— 引用前先验证 |
| 不经确认就自动写 inbox 文件 | 提示用户,不要突袭 |

完整列表见 [ARCHITECTURE.md](ARCHITECTURE.md)。

---

## 什么时候该用 / 不该用

**该用**:

- 你每天用 AI coding agent 做多个项目,context 丢失是真实痛点
- 你试过 `.cursorrules` / `.claude/CLAUDE.md`,但过了单项目规模就崩
- 你想要一个跨模型/agent/provider 都能活的工作流
- 你做研究或基础设施工作,知识要跨月累积

**不该用**:

- 你只做单个短期项目(一份简单的 CLAUDE.md 就够了)
- 你不需要跨会话记忆(直接用 chat history)
- 你不是天天用 AI agent(框架的红利来自持续实践)

---

## 来源

本框架提取自一个真实运行的研究 workspace,跑了 6 个多月、日更使用,同时支撑 8+ 个项目(ML 研究、多 agent 系统、基础设施、安全研究、PKM)。在反复踩坑中迭代 —— 反模式列表比功能列表长,因为每一条都是真摔出来的。

workspace 内部代号 **RedDragon Workspace**。本次发布剥离了个人内容,只留架构。

---

## 贡献

欢迎 Issue 和 PR,尤其:

- 适配其他 agent(Aider、Continue 等)的 adapter
- 特定领域的 workspace 样例(研究、SWE、内容创作)
- BOOTSTRAP.md / ARCHITECTURE.md 的其他语种翻译

---

## License

MIT
