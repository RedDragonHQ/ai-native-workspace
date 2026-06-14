# CLAUDE.md —— example-project

> 本项目 agent 运行时配置。cd 进本项目时自动追加加载。
> 继承自工作区根 `CLAUDE.md`。

## 项目概述

一段话说明这个项目是什么。

## 目录结构

```
example-project/
├── CLAUDE.md              ← 你在这里
├── README.md              ← 给人读的文档
├── src/                   ← 源代码
└── tests/                 ← 测试
```

## 关键文件

| 文件 | 用途 |
|------|------|
| `src/main.py` | 入口 |
| `src/core.py` | 核心逻辑 |
| `tests/test_core.py` | 主测试文件 |

## 如何运行

```bash
# 开发
python -m src.main

# 测试
pytest tests/
```

## 项目专属约定

*(只适用于本项目,工作区通用的放根 CLAUDE.md)*

- 因为 Z 所以偏好 X 而非 Y
- 做 A 之前必须先检查 B
- 本项目用 C 库,因为 D

## 已知踩坑

*(本项目里被坑过的事)*

- 坑 1 → 怎么避免
- 坑 2 → 怎么避免
