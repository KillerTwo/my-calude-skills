# git-commit-conventional-commits

一个用于 Claude Code 的本地技能示例。

作用：
- 约束 Git 提交只包含当前任务的变更
- 使用中文 Conventional Commits 提交标题
- 强制补充具体的 `description`
- 避免“收口”等空泛描述词

## 目录结构

```text
skills/
└─ git-commit-conventional-commits/
   ├─ SKILL.md
   └─ README.md
```

## 安装到 Claude Code 技能目录

Claude Code 的个人技能通常放在：

```text
~/.claude/skills/
```

在 Windows 上，通常对应：

```text
C:\Users\<你的用户名>\.claude\skills\
```

安装方式是把整个技能目录复制进去，并保留目录名不变。

示例：

```text
~/.claude/skills/git-commit-conventional-commits/SKILL.md
```

### Windows PowerShell 示例

如果你当前目录是仓库根目录，可以执行：

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force ".\skills\git-commit-conventional-commits" "$HOME\.claude\skills\"
```

### macOS / Linux 示例

```bash
mkdir -p ~/.claude/skills
cp -R ./skills/git-commit-conventional-commits ~/.claude/skills/
```

## 安装后如何使用

在 Claude Code 中，当你准备提交 Git 变更时，可以直接提到这个技能或描述其适用场景，例如：

```text
使用 git-commit-conventional-commits skill，帮我为这次改动生成提交信息
```

或者：

```text
按 Conventional Commits 规范生成中文提交信息，只提交本次改动，并补充 description
```

Claude Code 在识别到触发条件后，会读取 `SKILL.md` 并按其中规则执行。

## 建议验证

安装完成后，重点确认以下几点：
- 技能目录名为 `git-commit-conventional-commits`
- 目录内主文件名为 `SKILL.md`
- `SKILL.md` 的 YAML frontmatter 包含 `name` 和 `description`

## 卸载

删除技能目录即可。

Windows 示例：

```powershell
Remove-Item -Recurse -Force "$HOME\.claude\skills\git-commit-conventional-commits"
```

macOS / Linux 示例：

```bash
rm -rf ~/.claude/skills/git-commit-conventional-commits
```
