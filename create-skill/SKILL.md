---
name: create-skill
description: Use when the user wants to create a new Claude Code skill, build a new slash command with TDD, or establish reusable agent behavior patterns.
---

# Create Skill

你正在执行 **创建新 Skill** 任务：

> $ARGUMENTS

---

## Step 1 — 读取 writing-skills 指南

```bash
cat $(ls -d ~/.npm/_npx/*/node_modules/superpowers-mcp/skills/writing-skills/SKILL.md 2>/dev/null | tail -1)
```

如果上面命令无输出，跳过并使用内置知识执行下面步骤。

---

## Step 2 — 执行 TDD 创建流程

按 RED → GREEN → REFACTOR 三阶段创建 skill：

### RED（基线测试）
- 用 subagent 在**没有 skill 的情况下**跑压力场景
- 记录 agent 的自然行为和理由（逐字记录）

### GREEN（写最小化 Skill）
- 在 `~/.claude/skills/<skill-name>/SKILL.md` 创建 skill 文件
- 文件必须包含：
  ```yaml
  ---
  name: skill-name-with-hyphens
  description: Use when [具体触发条件和症状]
  ---
  ```
- description 用第三人称，以 "Use when..." 开头，只描述触发条件，**不描述流程**
- 用 skill 重跑场景，验证 agent 遵循规则

### REFACTOR（关闭漏洞）
- 找出新的理由/漏洞，逐一补充到 skill 中
- 重复测试直到 skill 足够健壮

---

## Step 3 — 完成

- 列出创建的文件路径
- 说明 skill 的触发场景
- 告知用户需要手动验证的点
