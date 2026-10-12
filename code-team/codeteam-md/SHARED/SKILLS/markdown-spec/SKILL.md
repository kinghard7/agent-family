---
name: markdown-spec
description: 团队 Markdown 文档规范。撰写任务卡、记忆文件、报告时遵循统一的标题层级、表格、代码块与 append-only 日志写法。
---

# Markdown 文档规范

## 标题层级
- 文档顶部一个 `#` 一级标题。
- 章节用 `##`，子项用 `###`，不跳级。

## 表格
- 表头与分隔线对齐，列含义清晰。

## 代码块
- 用三反引号标注语言（如 ` ```bash `、` ```mermaid `）。

## append-only 日志
- 交接记录、决策日志、纠错案例**只追加不修改**。
- 每条带时间戳与角色，格式：`- [YYYY-MM-DD 角色] 内容`。

## frontmatter
- 任务卡 `meta.md` 使用 YAML frontmatter 记录状态机字段（id/status/owner/from/updated）。
