# CLAUDE.md — 我的工作区

> Agent 运行时配置。在此目录启动 agent 时自动加载。

## 目录结构

```
my-workspace/
├── CLAUDE.md              ← 你在这里(工作区级认知配置)
├── inbox/                 ← 草稿缓冲区:会话摘要、TODO、速记
│   ├── README.md          ← inbox 文件格式
│   └── TODO.md            ← 跨会话流动清单
├── projects/              ← 具体项目
│   └── example-project/
│       └── CLAUDE.md      ← 项目级认知配置
└── wiki/                  ← 沉淀后的知识库(可选,晚点再建)
    └── MANIFEST.md        ← 渐进加载索引
```

## 工作流

### 会话结束(每次都做)

1. 写一个 inbox 文件:`inbox/YYYY-MM-DD-简短描述.md`
   - 格式:标签 + 摘要 + 成就清单 + 产出文件路径
2. 更新 `inbox/TODO.md`(新增 / 把完成项挪到 ✅)

### 会话恢复

1. 读 `inbox/TODO.md` 的"下次进来从这里开始"段
2. 按需读近期 `inbox/` 文件了解上下文(不需要才读)
3. 按需读近期 `inbox/` 文件了解历史上下文

### 知识沉淀路径

```
信息 → inbox/(草稿,低摩擦)
     → 成熟后升级到:
        - 项目文档(项目特有的技术细节)
        - wiki/(跨项目通用知识)
     → 过时就降级:
        - 改 freshness 为 sleeping/deprecated
        - 或移入 archive/
```

## 约定

- **CLAUDE.md 是 agent 的运行时配置**,改完下次启动生效。
- **先读索引,按需深入**,不要启动时全量加载工作区。
- **技术正确 > 社交舒适**:收到反馈先对照事实验证,再决定执行还是反驳。
- **TODO.md 是草稿,不是指令**,引用前必须先验证路径和分类。
- **Inbox 神圣**,不要在未提示用户的情况下自动写 inbox 文件。

## 文件该放哪

| 内容类型 | 放哪 |
|---------|------|
| 会话摘要 / 成就 | `inbox/YYYY-MM-DD-xxx.md` |
| 跨会话 TODO | `inbox/TODO.md` |
| 临时技术笔记 | `inbox/`(以后升级) |
| 稳定项目知识 | `projects/<名称>/` |
| 稳定跨项目知识 | `wiki/`(inbox 沉淀后) |
| 工作流约定 | 本文件(根 CLAUDE.md) |
| 项目级约定 | `projects/<名称>/CLAUDE.md` |
