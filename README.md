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
npx skills add DEVILENMO/skill-mcbe-addon-localization@v1.4.0 -g -y
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
| **隐式键补全（1b）** | 游戏**按 id 自己拼键**查的 lang：实体 / 物品 / 方块 / 刷怪蛋 / 离船提示 / 创造栏分组 |
| 语言文件翻译 | `en_US.lang` → 目标语言 |
| 脚本 UI 字符串 | 表单、聊天提示等 |
| 硬编码数据层（3b） | 枚举/nameTag/logic id 与 display label 分离 |
| 译者署名 | `pack.name` 后缀：译者名（每次确认）+ 工具署名 |
| 图片本地化 | 路线 A 图生图重绘 / 路线 B OCR 擦除写回（可选） |

**`.lang` 覆盖率 100% ≠ 游戏里没有原始键。** 游戏还会按 id 自己拼键去查 lang，
这一族漏掉时覆盖率照样 100%，玩家看到的却是原始 ID。步骤 1b 必须逐族反查，
详见 [`mcbe-addon-localization/references/implicit-keys.md`](mcbe-addon-localization/references/implicit-keys.md)。

图片路线的提示词按**能力**描述、不写死模型名，执行前先探测本机环境，
换机器无需改文档。详见 [`mcbe-addon-localization/references/image-localization.md`](mcbe-addon-localization/references/image-localization.md)。

详细说明见 [`mcbe-addon-localization/SKILL.md`](mcbe-addon-localization/SKILL.md)。

## 仓库结构

```
skill-mcbe-addon-localization/     ← 本仓库（Agent Skill 发布仓）
├── CHANGELOG.md
└── mcbe-addon-localization/       ← skill 本体（name 字段与此文件夹一致）
    ├── SKILL.md
    ├── README.md
    ├── LICENSE
    └── references/
        ├── file-formats.md        # .lang / manifest 格式
        ├── hardcoded-strings.md   # 脚本硬编码与 localization.js 模式
        ├── implicit-keys.md        # 游戏按 id 隐式查的 lang 键 + 官方措辞基准
        ├── image-localization.md  # 贴图文字：重绘路线 + OCR 写回路线 + 环境探测
        ├── translation-prompts.md  # LLM 提示词模板
        └── language-codes.md       # MCBE 29 语言码
```

## 更新记录

见 [CHANGELOG.md](CHANGELOG.md)。

## 作者

- **DEVILENMO** — [GitHub](https://github.com/DEVILENMO) · [Bilibili](https://space.bilibili.com/37202522)
- [ARC Minecraft](https://github.com/ARC-Minecraft) / ARCStudio

## 许可

MIT — 见 [mcbe-addon-localization/LICENSE](mcbe-addon-localization/LICENSE)
