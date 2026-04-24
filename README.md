# my-calude-skills

个人常用 Claude Code 技能仓库。

这个仓库按“多技能仓库”组织，后续可以持续新增、维护和分发常用技能。所有技能统一放在 `skills/` 目录下，每个技能一个独立子目录，目录内至少包含 `SKILL.md`。

## Repository Layout

```text
my-calude-skills/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ skills/
│  └─ git-commit-conventional-commits/
│     ├─ SKILL.md
│     └─ README.md
└─ docs/
   ├─ install.md
   └─ conventions.md
```

## Skills

| Skill | Purpose |
|------|---------|
| `git-commit-conventional-commits` | 约束 Git 提交只包含当前任务改动，使用中文 Conventional Commits，并补充具体 `description` |

## Quick Install

把需要的技能目录复制到 Claude Code 本地技能目录：

```text
~/.claude/skills/
```

Windows PowerShell 示例：

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force ".\skills\git-commit-conventional-commits" "$HOME\.claude\skills\"
```

macOS / Linux 示例：

```bash
mkdir -p ~/.claude/skills
cp -R ./skills/git-commit-conventional-commits ~/.claude/skills/
```

更完整的安装说明见 [docs/install.md](/D:/documents/test1/my-calude-skills/docs/install.md)。

## Adding New Skills

新增技能时，推荐遵循以下约定：
- 路径固定为 `skills/<skill-name>/`
- 主文件名固定为 `SKILL.md`
- 目录名与 frontmatter 里的 `name` 保持一致
- 技能目录内可选增加 `README.md`、示例文件或辅助参考文档

仓库约定见 [docs/conventions.md](/D:/documents/test1/my-calude-skills/docs/conventions.md)。

## Usage

在 Claude Code 中直接提及技能名，或描述与技能 `description` 相符的触发场景即可。

示例：

```text
使用 git-commit-conventional-commits skill，帮我为这次改动生成提交信息
```

## License

本仓库使用仓库根目录下的 `LICENSE`。
