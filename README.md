# my-skills

个人维护的 AI Agent Skills 集合，可通过 [`skills` CLI](https://github.com/vercel-labs/skills) 安装。

## 包含的 Skills

| Skill | 说明 |
|-------|------|
| [chain-analysis](skills/chain-analysis/SKILL.md) | 分析代码调用链路，生成可导入 CoderTools IntelliJ 插件的 JSON 文件 |
| [create-oc-cmd](skills/create-oc-cmd/SKILL.md) | 在用户主目录创建 PowerShell 启动脚本，快速启动 opencode |
| [prd-quality-gate](skills/prd-quality-gate/SKILL.md) | PRD 质量闸门，评估产品需求文档质量并输出红黄绿分级报告 |

## 安装

```bash
# 安装单个 skill（全局）
npx skills add zhanglinlu/my-skills --skill prd-quality-gate -g

# 安装单个 skill（项目级）
npx skills add zhanglinlu/my-skills --skill prd-quality-gate

# 安装全部 skills
npx skills add zhanglinlu/my-skills --skill '*' -g
```

支持的 agent（自动检测）：Claude Code、Cursor、Codex、OpenCode 等 30+ 种。

## 新增 Skill

1. 在 `skills/` 下创建以 skill 名称命名的目录
2. 目录内放 `SKILL.md`，frontmatter 的 `name` 字段与目录名一致
3. 提交推送即可
