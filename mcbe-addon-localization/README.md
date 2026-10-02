# mcbe-addon-localization

Minecraft 基岩版（MCBE）Addon/模组本地化 Agent Skill。

基于 [ARC MCBE Addon Auto Localizer](https://github.com/ARC-Minecraft/ARC-MCBE-Addon-Auto-Localizer) 验证的生产流程，适用于 Cursor、Claude Code、Codex、OpenClaw 等支持 [Agent Skills](https://agentskills.io) 格式的 AI Agent。

## 目录结构

```
mcbe-addon-localization/
├── SKILL.md                      # 入口（Agent 读取）
├── README.md                     # 本文件（人类阅读）
├── LICENSE
└── references/
    ├── file-formats.md           # .lang / manifest 格式
    ├── hardcoded-strings.md      # 脚本硬编码与 localization.js 模式
    ├── image-localization.md     # 贴图文字：重绘路线 + OCR 写回路线 + 环境探测
    ├── translation-prompts.md    # LLM 提示词模板
    └── language-codes.md         # MCBE 29 语言码
```

## 核心能力

1. **Display Name 标准化** — JSON 内 `minecraft:display_name` → lang 键
2. **语言文件翻译** — `en_US.lang` → `zh_CN.lang` 等
3. **脚本 UI 字符串** — 表单、聊天提示等字面量
4. **硬编码数据层（3b）** — 枚举/配置常量、nameTag、logic id 与 display label 分离、`getDisplayName()` 映射
5. **译者署名** — `pack.name` 后缀（译者名每次向使用者确认 + 工具署名）
6. **图片本地化**（可选）— 路线 A 图生图重绘 / 路线 B OCR 擦除写回

带 Script API 的包：**`.lang` 翻完不等于汉化完成**。须识别 logic id 与 display label，执行步骤 3b，详见 [references/hardcoded-strings.md](references/hardcoded-strings.md)。

## 跨机器可移植性

本 skill **不假设**使用者装了某个本地模型或工具。图片汉化按能力描述（"支持参考图输入的模型"），
执行前先探测环境，缺什么就换路线或如实告知——换机器不需要改文档。详见
[references/image-localization.md](references/image-localization.md#环境探测)。

## 安装

推荐（从 GitHub 安装整个 skill 仓库）：

```bash
npx skills add DEVILENMO/skill-mcbe-addon-localization -g -y
```

手动复制本文件夹到 Agent skills 目录：

| Agent | 路径 |
|-------|------|
| Cursor | `~/.cursor/skills/mcbe-addon-localization/` |
| Claude Code | `~/.claude/skills/mcbe-addon-localization/` |
| 项目内 | `.cursor/skills/mcbe-addon-localization/` |

规范见 https://agentskills.io/specification

## 作者

- **DEVILENMO** — [GitHub](https://github.com/DEVILENMO) · [Bilibili](https://space.bilibili.com/37202522)
- [ARC Minecraft](https://github.com/ARC-Minecraft) / ARCStudio

## 许可

MIT — 见 [LICENSE](LICENSE)
