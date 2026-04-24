# Installation Guide

## Claude Code Skills Directory

Claude Code 的个人技能通常安装到：

```text
~/.claude/skills/
```

Windows 常见路径：

```text
C:\Users\<用户名>\.claude\skills\
```

## Install a Single Skill

以 `git-commit-conventional-commits` 为例。

### Windows PowerShell

在仓库根目录执行：

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force ".\skills\git-commit-conventional-commits" "$HOME\.claude\skills\"
```

### macOS / Linux

```bash
mkdir -p ~/.claude/skills
cp -R ./skills/git-commit-conventional-commits ~/.claude/skills/
```

## Verify Installation

确认以下几点：
- 目标路径存在 `~/.claude/skills/git-commit-conventional-commits/`
- 目标目录内存在 `SKILL.md`
- `SKILL.md` frontmatter 包含 `name` 和 `description`

## Update a Skill

如果本地已存在同名技能，可直接覆盖复制：

### Windows PowerShell

```powershell
Copy-Item -Recurse -Force ".\skills\git-commit-conventional-commits" "$HOME\.claude\skills\"
```

### macOS / Linux

```bash
cp -R ./skills/git-commit-conventional-commits ~/.claude/skills/
```

## Uninstall a Skill

### Windows PowerShell

```powershell
Remove-Item -Recurse -Force "$HOME\.claude\skills\git-commit-conventional-commits"
```

### macOS / Linux

```bash
rm -rf ~/.claude/skills/git-commit-conventional-commits
```
