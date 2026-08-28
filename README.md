# skill-mcbe-addon-localization

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](mcbe-addon-localization/LICENSE)

Minecraft **基岩版（MCBE）** Addon/模组本地化 [Agent Skill](https://agentskills.io)。

基于 [ARC MCBE Addon Auto Localizer](https://github.com/ARC-Minecraft/ARC-MCBE-Addon-Auto-Localizer) 验证的生产流程，适用于 Cursor、Claude Code、Codex、OpenClaw 等支持 Agent Skills 的 AI Agent。

## 安装

```bash
npx skills add DEVILENMO/skill-mcbe-addon-localization -g -y
```

固定版本：

```bash
npx skills add DEVILENMO/skill-mcbe-addon-localization@v1.2.0 -g -y
```

GitHub CLI：

```bash
gh skill install DEVILENMO/skill-mcbe-addon-localization
```

手动安装：将 [`mcbe-addon-localization/`](mcbe-addon-localization/) 复制到 Agent 的 skills 目录（如 `~/.cursor/skills/`）。

## Skill 内容

| 能力 | 说明 |
|------|------|
| Display Name 标准化 | `minecraft:display_name` → lang 键 |
| 语言文件翻译 | `en_US.lang` → 目标语言 |
| 脚本 UI 字符串 | 表单、聊天提示等 |
| 硬编码数据层（3b） | 枚举/nameTag/logic id 与 display label 分离 |
| 译者署名 | `pack.name` 后缀 |
| 图片本地化 | guidebook 贴图内文字（可选） |

详细说明见 [`mcbe-addon-localization/SKILL.md`](mcbe-addon-localization/SKILL.md)。

## 仓库结构

```
skill-mcbe-addon-localization/     ← 本仓库（Agent Skill 发布仓）
└── mcbe-addon-localization/       ← skill 本体（name 字段与此文件夹一致）
    ├── SKILL.md
    ├── LICENSE
    └── references/
```

## 作者

- **DEVILENMO** — [GitHub](https://github.com/DEVILENMO) · [Bilibili](https://space.bilibili.com/37202522)
- [ARC Minecraft](https://github.com/ARC-Minecraft) / ARCStudio

## 许可

MIT — 见 [mcbe-addon-localization/LICENSE](mcbe-addon-localization/LICENSE)
