---
name: fix-skill
description: Use when the user wants to improve, patch, or fix an existing Claude Code skill — closing loopholes, updating rules, or refining the description.
---

# Fix Skill

你正在执行 **改进 Skill** 任务：

> $ARGUMENTS

---

## Step 1 — 读取目标 Skill

从 $ARGUMENTS 中提取 skill 名称和改进目标，读取当前内容：

```bash
# 查看全局 skill 列表
ls ~/.claude/skills/

# 读取指定 skill（将 SKILL_NAME 替换为目标 skill）
cat ~/.claude/skills/SKILL_NAME/SKILL.md
```

如果是 superpowers skill：
```bash
cat $(ls -d ~/.npm/_npx/*/node_modules/superpowers-mcp/skills/SKILL_NAME/SKILL.md 2>/dev/null | tail -1)
```

---

## Step 2 — 分析改进目标

根据 $ARGUMENTS 描述的改进方向，评估：
- 需要修改哪些部分（description / overview / 步骤 / 示例 / 漏洞表）
- 是否有未关闭的理由/漏洞
- description 是否符合规范：第三人称、以 "Use when..." 开头、只写触发条件不写流程

---

## Step 3 — 执行改进（改前必须列出变更计划）

在修改前，列出：
1. 所有将要变更的内容
2. 变更原因

等待用户确认后再修改文件。

改进时遵守：
- 不删除已有的有效规则，只补充或澄清
- description 最多 1024 字符，尽量简洁
- 保留原有的 `name` slug（除非用户明确要重命名）

---

## Step 4 — 验证改进

改完后，用 subagent 跑一个快速场景验证改动是否生效：
- 场景应覆盖被改进的部分
- 确认 agent 行为符合预期

---

## Step 5 — 完成

- 列出所有修改的文件和变更摘要
- 说明潜在副作用
- 建议用户运行 `/test-skill` 进行完整测试
