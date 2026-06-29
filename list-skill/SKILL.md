---
name: list-skill
description: Use when the user wants to see all available Claude Code skills and slash commands, or asks "what skills do I have", "list skills", "show available commands"
---

# List Skills

## Overview

Lists all available skills and slash commands registered in the current Claude Code session.

## How to Use

When invoked via `/list-skill`, read the `system-reminder` injected in the current context — it contains the full list of available skills under the "The following skills are available" section.

Format the output as a markdown table with two columns: **Skill** (`/name`) and **Description**.

## Output Format

```markdown
| Skill | 描述 |
|-------|------|
| `/skill-name` | description from frontmatter |
```

## Sources of Skills

Skills come from two sources:
1. `~/.claude/commands/*.md` — user-defined slash commands
2. Built-in skills from Claude Code and installed plugins (e.g. superpowers-mcp)

## Quick Reference

- Command files location: `~/.claude/commands/`
- Skill files location: `~/.claude/skills/`
- To create a new skill: `/create-skill`
