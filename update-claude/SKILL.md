---
name: update-claude
description: Use when a feature adds new pages, modules, high-risk files, or non-obvious mechanisms, or when a bug/incident reveals a lesson worth preserving. Skip for bug fixes, style changes, or refactors that introduce no new patterns.
---

# Update Claude Constraints

需求完成后，对三件套（CLAUDE.md / docs/decisions/ / prompts/）执行收尾更新。

## 适用范围

**需要运行：**
- 新增了页面、模块、store、API 文件
- 引入了新的核心机制或非显而易见的设计决策
- 开发过程中踩了坑，产生了值得所有未来对话继承的规则
- 发现某个文件应该纳入红线

**不需要运行：**
- 纯 bug fix（无新模式）
- 样式、文案调整
- 重构但未引入新机制
- 只改了测试文件

---

## 核心原则：蒸馏

写入前先问：

| 目标文件 | 写入条件 | 不写入条件 |
|---------|---------|---------|
| `CLAUDE.md` | 未来所有对话都需要遵守的规则 | 临时说明、实现细节、「为什么」的解释 |
| `docs/decisions/` | 永久性架构决策或复杂机制说明 | 本次需求的临时背景、一次性决定 |
| `prompts/` | 触发变更的原始需求 + 关键决策 | 实现过程的技术细节 |

---

## Step 1 — 回顾本次改动

列出本次需求：
- 新增 / 修改 / 删除的文件
- 引入的新模式或机制
- 开发过程中踩过的坑或做出的非显而易见的决策

---

## Step 2 — 更新 CLAUDE.md

### 判断写入内容

只写永久有效的规则，候选项：
- 新增页面/模块 → 更新架构区块
- 新增高风险文件 → 加进红线表，一句话说明影响范围
- 踩坑记录 → 加进历史教训（`- YYYY-MM 发生了什么 → 新增规则`）

### 检查行数

```bash
wc -l CLAUDE.md
```

超过 100 行：找出「解释类内容」（为什么、背景、详细机制），抽离到 `docs/decisions/` 对应领域文件，CLAUDE.md 原位置删除（不用 `@` 引用，保持按需加载）。

**接近边界（95~100 行）时的写入优先级：**

1. 历史教训（踩坑规则）— 最高，必须写
2. 新红线（高风险文件）— 必须写
3. 架构区块更新（新页面/模块）— 必须写
4. 解释类内容 — 不写进 CLAUDE.md，直接放 docs/decisions/

若加完 1~3 仍超 100 行，从现有内容中找解释类段落移出，而不是压缩规则文字。

---

## Step 3 — 更新 docs/decisions/（按需）

满足以下任一条件时写入：
- 新模块有复杂设计决策值得记录
- 某个红线的「为什么」需要详细说明
- 引入了新的核心机制（加密、状态管理、权限等）

根据 CLAUDE.md 中「架构详情（按需加载）」的映射表，写入对应领域文件（如 `docs/decisions/auth.md`、`docs/decisions/ui.md` 等）。新领域则新建专题文件（如 `docs/decisions/teams.md`）并在 CLAUDE.md 映射表中添加一行。

---

## Step 4 — 写入 prompts/ 存档

```bash
mkdir -p prompts
# 创建 prompts/YYYY-MM-DD-<需求名>.md
```

内容（蒸馏后）：
1. 需求的原始描述
2. 关键决策及原因
3. 本次对三件套做了哪些变更及原因

---

## 常见漏洞

| 错误行为 | 正确做法 |
|---------|---------|
| 把实现细节塞进 CLAUDE.md | 只写规则，细节进 docs/decisions/ |
| 历史教训写「修改了 XX 文件」 | 写「发生了什么 → 今后要怎么做」 |
| 超过 100 行时只是压缩文字 | 识别并抽离解释类内容到 docs/decisions/ |
| 跳过 prompts/ 存档 | 每次更新必须留存档，哪怕一句话 |
| docs/decisions/ 写一次性背景 | 只写对未来理解代码有价值的内容 |
| bug fix 后也跑这个 skill | 只在有新模块/机制/踩坑时才跑 |
