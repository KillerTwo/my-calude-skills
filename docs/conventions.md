# Repository Conventions

## Directory Rules

- 所有技能统一存放在 `skills/`
- 每个技能使用独立目录：`skills/<skill-name>/`
- 每个技能目录至少包含 `SKILL.md`
- 可选增加 `README.md`、示例文件、参考文档或辅助脚本

## Naming Rules

- 技能目录名使用短横线风格
- `SKILL.md` frontmatter 中的 `name` 与目录名保持一致
- `description` 使用 `Use when...` 风格，描述触发条件，而不是流程摘要

## Documentation Rules

- 仓库根 `README.md` 负责总览、技能列表和快速安装
- 技能目录内 `README.md` 负责该技能的独立说明
- 跨技能安装和维护说明统一放在 `docs/`

## Recommended Skill Layout

```text
skills/
└─ example-skill/
   ├─ SKILL.md
   ├─ README.md
   └─ examples/
```

## Publishing Guidance

- 根目录保持简洁，不直接堆放多个技能目录
- 新增技能时同步更新根 `README.md` 的技能列表
- 尽量保证每个技能都能被单独复制安装
