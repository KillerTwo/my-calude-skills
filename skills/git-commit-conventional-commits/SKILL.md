---
name: git-commit-conventional-commits
description: Use when preparing a Git commit with Conventional Commits, especially when the working tree may contain unrelated changes, only the current task should be staged, the message must be in Chinese, and the output needs a concrete description.
---

# Git Commit Conventional Commits

## Overview

Use this skill when preparing a general-purpose Git commit.

Core goals:
- Commit only the current task's changes
- Use a valid Conventional Commits title
- Write the commit message in Chinese
- Avoid vague words with no concrete meaning
- Always provide a useful `description`

## When to Use

Use when:
- A commit message needs to follow Conventional Commits
- The working tree may contain unrelated edits
- The staged scope must stay limited to the current task
- A Chinese commit message is required
- The commit output should include a concrete `description`

Do not use when:
- The user explicitly wants a non-Conventional-Commits format
- The user explicitly wants to batch multiple unrelated tasks into one commit

## Core Rules

### 1. Commit only the current task

Before staging or committing, identify the files that belong to the current task.

Never:
- Mix unrelated changes into the same commit
- Stage extra files just because they are already modified
- Clean up or revert unrelated work unless the user asked for it

### 2. Stage precisely when nothing is staged yet

If the current task is not staged yet, stage it first.

Use this standard:

> If the current task has not been staged yet, stage only the newly added files or explicitly involved files for this task before committing. Do not pull unfinished or unrelated changes into the same commit.

Preferred behavior:
- Stage only files that belong to the current task
- Prefer narrow staging over broad staging
- Check the staged scope before continuing

Avoid:
- `git add .`
- `git add -A`
- Any bulk staging command that may capture unrelated changes

### 3. Write the commit title in Chinese

The commit title should be concise, specific, and written in Chinese.

### 4. Ban vague filler words

Do not use empty wording such as:
- 收口
- 优化一下
- 调整细节
- 完善功能
- 处理问题

These phrases are unacceptable unless they are followed by a concrete object and a concrete outcome. Prefer direct wording that states what changed.

### 5. Always include `description`

The output must include a `description` section.

`description` should explain:
- The direct purpose of the commit
- The main change points
- The impact, constraints, or compatibility notes

Do not repeat the title with different wording. Add detail.

## Commit Format

Use this title format:

```text
type(scope): 中文摘要
```

Common `type` values:

| type | Meaning |
|------|---------|
| `feat` | 新功能 |
| `fix` | 缺陷修复 |
| `refactor` | 重构 |
| `docs` | 文档变更 |
| `test` | 测试相关 |
| `build` | 构建或依赖调整 |
| `chore` | 杂项维护 |
| `perf` | 性能优化 |

`scope` guidance:
- Use a real module name when one exists
- Omit `scope` if no meaningful module applies
- Do not invent a vague scope just to fill the format

Examples:

```text
feat(export): 支持出库单列表按列配置导出
fix(login): 修复登录态过期后页面重复跳转问题
refactor(order): 拆分订单校验逻辑并减少重复判断
docs(skill): 补充通用 git 提交技能说明
```

## Output Template

```text
staged_scope:
- 本次准备提交的文件或模块范围

commit:
- type(scope): 中文摘要

description:
- 说明本次提交的直接目标
- 说明核心改动点
- 说明影响范围、限制条件或兼容性事项
```

## Workflow

1. Inspect the working tree and identify the current task's files.
2. Exclude unrelated changes.
3. If nothing relevant is staged yet, stage only the files that belong to this task.
4. Re-check the staged diff.
5. Write a Chinese Conventional Commits title.
6. Write a concrete `description`.
7. Commit only after the staged scope is correct.

## Good vs Bad

Good:

```text
fix(inventory): 修复库存同步失败后状态未回滚的问题

description:
1. 在库存同步异常时补充状态回滚逻辑，避免任务长期停留在处理中
2. 调整异常分支的状态更新顺序，并补充失败日志
3. 不影响正常同步流程，仅修复失败场景下的数据一致性问题
```

Bad:

```text
fix: 收口库存逻辑
```

Why bad:
- The wording is vague
- The object is unclear
- The result is unclear
- No `description`

## Common Mistakes

- Using broad staging and pulling unrelated changes into the commit
- Writing titles with words like “优化” or “调整” but no concrete object
- Repeating the title in `description` without adding detail
- Using a fake or meaningless `scope`
- Committing incomplete intermediate work together with the target change

## Final Rule

If the boundary of "current task changes" is unclear, stop and narrow the staged scope first.

Prefer under-committing to mixing unrelated work.
