---
name: test-skill
description: Use when the user wants to test whether a Claude Code skill works correctly — running stress scenarios, comparing with/without skill behavior, and producing a test report.
---

# Test Skill

你正在执行 **测试 Skill** 任务：

> $ARGUMENTS

---

## Step 1 — 找到目标 Skill

从 $ARGUMENTS 中提取 skill 名称，读取 skill 文件：

```bash
# 查看全局 skill 列表
ls ~/.claude/skills/

# 读取指定 skill（将 SKILL_NAME 替换为目标 skill）
cat ~/.claude/skills/SKILL_NAME/SKILL.md
```

如果找不到，检查 superpowers skills：
```bash
ls $(ls -d ~/.npm/_npx/*/node_modules/superpowers-mcp/skills/ 2>/dev/null | tail -1)
cat $(ls -d ~/.npm/_npx/*/node_modules/superpowers-mcp/skills/SKILL_NAME/SKILL.md 2>/dev/null | tail -1)
```

---

## Step 2 — 设计压力测试场景

根据 skill 类型设计测试场景：

- **规则/纪律类 skill**：设计压力场景（时间压力 + 沉没成本 + 权威压力 + 疲劳感），验证 agent 在压力下是否仍遵守规则
- **技巧类 skill**：设计应用场景，验证 agent 是否正确应用技巧
- **参考文档类 skill**：设计信息检索场景，验证 agent 是否能找到并正确使用信息

---

## Step 3 — 派发 Subagent 执行测试

用 Agent 工具派发 subagent，给出：
1. 一个**带有 skill** 的场景（应通过）
2. 一个**没有 skill** 的场景（基线，预期失败）

对比两组结果，记录差异。

---

## Step 4 — 测试报告

输出：
- ✅/❌ 测试是否通过
- 发现的漏洞或不清晰的地方
- 建议改进点（交给用户决定是否执行 /fix-skill）
