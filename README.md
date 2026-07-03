# my-skills

我的自定义 Agent Skills 集合，可通过 `npx skills add` 安装。

## Skills

| Skill | 说明 |
|-------|------|
| [chain-analysis](skills/chain-analysis/) | 分析代码调用链路，生成可导入 CoderTools 插件的 JSON |
| [create-oc-cmd](skills/create-oc-cmd/) | 快速创建 opencode 启动 PowerShell 脚本 |
| [prd-quality-gate](skills/prd-quality-gate/) | PRD 质量闸门：6 维度评估需求文档质量，输出红黄绿分级报告 |

## 安装

```bash
# 安装单个 skill（全局）
npx skills add zhanglinlu/my-skills --skill chain-analysis -g
npx skills add zhanglinlu/my-skills --skill create-oc-cmd -g
npx skills add zhanglinlu/my-skills --skill prd-quality-gate -g

# 安装全部 skills
npx skills add zhanglinlu/my-skills -g --all

# 指定目标 agent
npx skills add zhanglinlu/my-skills --skill prd-quality-gate -g -a claude-code -y
```

支持的 agent：`claude-code`、`cursor`、`codex`、`opencode` 等。

## 目录结构

```
skills/
├── chain-analysis/
│   └── SKILL.md
├── create-oc-cmd/
│   └── SKILL.md
└── prd-quality-gate/
    └── SKILL.md
```
