---
name: mcbe-addon-localization
description: >-
  Localizes Minecraft Bedrock Edition (MCBE) addons and mods: standardizes
  minecraft:display_name to lang keys, translates .lang files and languages.json,
  localizes behavior-pack script UI strings, handles hardcoded data-layer strings
  (enum/config constants, nameTags, logic-id vs display-label separation via
  localization.js display maps), applies translator credit suffixes, and optionally
  retextures in-image text. Use when localizing MCBE addons, fixing incomplete
  target lang where .lang is done but in-game text is still English, editing
  Script API forms, or ARC item colors.
license: MIT
compatibility: >-
  Works with any agent that can read/write local files. Optional: Node.js
  for JS syntax validation (node --check). Image localization needs a
  vision-capable model. No specific IDE required.
metadata:
  author: DEVILENMO
  version: "1.2.0"
  homepage: https://github.com/DEVILENMO/skill-mcbe-addon-localization
  tags: minecraft,bedrock,mcbe,addon,mod,localization,translation,i18n,lang,汉化
---

# MCBE 模组本地化

将 Minecraft **基岩版（Bedrock Edition）** Addon/模组本地化为任意 MCBE 官方支持语言（如 `zh_CN`、`ja_JP`、`de_DE`）。本 skill 描述经 ARC Auto Localizer 验证的生产流程，**任何 AI Agent 均可按此手动或半自动执行**。

## 何时使用 / 何时不用

**使用：**

- 汉化/翻译 MCBE 资源包（RP）或行为包（BP）
- 编辑 `texts/*.lang`、`languages.json`、`minecraft:display_name`
- 翻译行为包脚本中的玩家可见字符串（表单、聊天提示等）
- **`.lang` 已译但游戏内仍显示英文**（枚举名、配置 label、状态文案、NPC 名牌等）
- 区分 **logic id** 与 **display label**，建立显示名映射模块
- 为包名添加译者署名后缀
- 本地化贴图上的英文文字（guidebook 等）

**不使用：**

- Java 版模组（Forge/Fabric）——格式不同
- 仅修改游戏机制、不涉及玩家可见文案
- 需要翻译 `.mcstructure`、音频、视频等非文本资源

## 必需输入

向 Agent 提供以下信息（缺失则先询问用户）：

| 输入 | 说明 |
|------|------|
| `target_language` | MCBE 语言码，如 `zh_CN`（默认） |
| `resource_pack_path` | 资源包根目录（可选） |
| `behavior_pack_path` | 行为包根目录（可选） |
| `translator_credit` | 译者名，默认 `DEVILENMO` |
| 至少 RP 或 BP 之一 | 两者通常成对存在 |

## 输出要求

完成本地化后，Agent 应汇报：

1. 各步骤处理数量（标准化 display_name 数、翻译 lang 条数、脚本替换处数、硬编码映射条数等）
2. 修改的文件列表（相对包根目录）
3. **硬编码层**：显示名映射模块（如 `localization.js`）、已处理的常量类别（枚举/配置/NPC/状态…）
4. 跳过/回退项及原因（含「保留英文逻辑键」项）
5. 待用户手动处理的残留项（若有）

## 包结构

```
<pack>/
├── manifest.json
├── texts/
│   ├── languages.json       # ["en_US", "zh_CN", ...]
│   ├── en_US.lang           # 英文源（必须存在或可创建）
│   └── {target}.lang        # 目标语言
├── items/ entities/ blocks/ # JSON 定义
└── scripts/                 # BP 脚本 (.js/.ts)
```

`manifest.json` 可能含 JSONC 注释（`//`、`/* */`），解析时需支持。

典型安装路径（Windows）：

`%APPDATA%\Minecraft Bedrock\Users\Shared\games\com.mojang\`

## 标准流程（顺序固定）

```
- [ ] 0. 规范化 JSON / JS（可选但推荐）
- [ ] 1. Display Name 标准化
- [ ] 2. 语言文件翻译
- [ ] 3. 脚本提示本地化（仅 BP）
- [ ] 3b. 硬编码数据层本地化（Script 包必查）
- [ ] 4. 本地化者署名
- [ ] 5. 同步包文件夹名（可选）
- [ ] 6. 图片本地化（独立，按需）
- [ ] 7. 打包导出 .mcaddon / .mcpack（分发）
```

**核心五步** = 1 → 2 → 3 → **3b** → 4。步骤 1 会写入 lang 键，步骤 2 统一翻译；**不要**先翻脚本再改 display_name。

**常见误区：** 步骤 2 完成后 target `.lang` 很长，但只做了步骤 3 的字面量替换，漏掉 **constants / 配置表 / nameTag / translate 插值变量** —— 玩家仍会反馈「汉化不全」。凡 BP 含 `scripts/` 且玩家可见文案来自数据层，**必须执行 3b**（详见 [references/hardcoded-strings.md](references/hardcoded-strings.md)）。

---

## 步骤 0：规范化 JSON / JS

- 格式化 RP + BP 内 JSON（indent 2 或 4，与包内现有风格一致）与 JS
- 内容无变化则不写盘
- 后续步骤应在此步之后执行，避免 diff 混乱

---

## 步骤 1：Display Name 标准化

扫描所有 JSON 中的 `minecraft:display_name`：

| 情况 | 处理 |
|------|------|
| 可解析根组件 + `description.identifier` | value → 语言键；英文原文写入 `en_US.lang` 与目标 `.lang` |
| 已是语言键（`item.ns:id.name` 形态） | 跳过改写；确保 lang 有条目 |
| 无法推导 key | **直译**写回 JSON value（不建 lang 键） |

**组件 → 键前缀：**

| 根组件 | 前缀 | 示例键 |
|--------|------|--------|
| `minecraft:item` | `item` | `item.arc:potion.name` |
| `minecraft:entity` | `entity` | `entity.arc:boss.name` |
| `minecraft:block` | `tile` | `tile.arc:machine.name` |

identifier 格式：`namespace:id`（须含 `:`）。

---

## 步骤 2：语言文件翻译

对 RP 与 BP **分别**执行：

1. **Manifest 字面量**：若 `header.name` / `header.description` 是硬编码字符串（非 `pack.name` / `pack.description`），改为语言键，原文写入 lang
2. `languages.json` 追加目标语言码（若缺失）
3. 若无 `{target}.lang`，从 `en_US.lang` 复制
4. 批量翻译待译条目（建议每批 30–50 条）

**待译判定：**

- 目标值与 `en_US` 相同 → 待译（目标为 `en_US`/`en_GB` 时跳过）
- 无 `en_US` 对照，或文本字符集不像目标语言 → 待译
- 拉丁语系互译：主要靠与 `en_US` 对比

**写回规则：**

- 只改等号右边；键名、注释（`##`）、空行保留
- 保留 `§` 颜色码、`%s`/`%1$s`、`\\n` 转义
- 已有译文不覆盖（只追加缺失 key 或更新仍等于英文的条目）

格式细节见 [references/file-formats.md](references/file-formats.md)。  
LLM 提示词见 [references/translation-prompts.md](references/translation-prompts.md)。

---

## 步骤 3：脚本提示本地化（仅行为包）

翻译玩家可见字符串，**不改代码结构、不碰逻辑**。

**处理：**

- `sendMessage`、`setActionBar`、`setTitle`、`setSubtitle`
- 表单：`.title()` `.button()` `.label()` `.body()` `.toggle()` `.slider()` `.dropdown()`（**含 options 数组**）`.textField()` 等
- 属性：`tooltip`、`placeholderText`、`placeholder`

**跳过：**

- 语言键（`foo.bar.baz`，无空格、无 `§`）
- 路径/ID（`textures/`、`minecraft:`、`http://`）
- 多行字符串、嵌套反引号模板
- `${}` 内含 `.map` `.join` 箭头函数等复杂表达式

**可保留的简单插值：** `${obj.prop}`、`${Math.floor(x)}`

**安全：**

- 写盘前用 `node --check` 校验 JS；失败则回滚
- 译文 `${` 数量必须与原文一致

**与步骤 3b 的分工：**

| 步骤 3 | 步骤 3b |
|--------|---------|
| 脚本里零散的字符串字面量 | `*constants*` / `*config*` / `*data*` 等 **数据常量** |
| `sendMessage("...")` | 枚举数组、配置表 `name`/`label`/`title` |
| 简单 `.button("文本")` | id → label 映射表、多级 id 映射 |
| — | 实体 `nameTag`、`summon` 显示名 |
| — | 物品/实体 displayName fallback 字典 |
| — | 已用 `{ translate }` 但 **插值 / with 变量** 仍为英文 |

表单已用 `{ translate: "warning.xxx" }` 的条目在步骤 2 处理；若 UI 仍英文，查传入 `with:` 的值是否来自未译的数据常量。

---

## 步骤 3b：硬编码数据层本地化

**完整指南：** [references/hardcoded-strings.md](references/hardcoded-strings.md)

### 何时必须做

- BP 含 `scripts/`，且存在 UI、配置表、枚举常量
- 用户反馈「某类名称/状态仍是英文」
- target `.lang` 与 `en_US.lang` 键已对齐，但游戏内仍有英文

### 核心原则

**logic id 不变，display label 本地化。** 凡参与 lookup、存档、比较、配置表键名的 id **不要改成译文**；仅在 UI、nameTag、聊天展示层转换。

### 推荐产出

在 BP `scripts/` 下建集中映射模块（如 `localization.js`）：

- `displayNameTables` 按 **category** 分表（`enum`、`building`、`status`、`item_fallback` 等）
- `getDisplayName(category, id)` 统一展示入口
- 可选：`getDisplayNameFromKey()` 处理蛇形/ kebab 配置键

在 **所有 UI 渲染点** 接入：表单按钮/标签、列表项、实体 nameTag、含 id 的模板字符串。

### 快速扫描（BP 根目录）

对 `scripts/**/*.js` 搜索：

- `nameTag\s*=`、`summon .+"`
- `setDynamicProperty|getDynamicProperty`
- `export const \w+ = \[` 且含英文字面量
- `get\w*DisplayName|itemNameMap|labelMap`
- `\{ translate:` 与同文件硬编码英文并存处

完整模式与策略见 [references/hardcoded-strings.md](references/hardcoded-strings.md)。

### 同步补 lang

有 identifier 的物品/实体/方块优先走步骤 1–2；脚本 fallback 映射与 lang 键 **双保险**（键用 `namespace:id`）。

### 验收

使用 [references/hardcoded-strings.md](references/hardcoded-strings.md) 验收清单；改完脚本后 `node --check` 全部通过；游戏内抽测选单、名牌、状态行。

---

## 步骤 4：本地化者署名

在目标语言 `.lang` 的 `pack.name` 后追加后缀（先剥旧后缀）：

| 语言码 | 后缀模板 |
|--------|----------|
| `zh_CN` | `（{credit}汉化）` |
| `zh_TW` | `（{credit}漢化）` |
| `ja_JP` | `（{credit}翻訳）` |
| `ko_KR` | `({credit} 번역)` |
| 其它 | `({credit} Localization)` |

若 manifest `header.name` 已是 `pack.name`，署名仅改 lang 即可在游戏内生效。

完整语言码列表见 [references/language-codes.md](references/language-codes.md)。

---

## 步骤 5：同步文件夹名（可选）

读取目标语言 `pack.name`（含署名），将 RP/BP 文件夹重命名为可读包名。重命名后更新工作路径引用。

---

## 步骤 6：图片本地化（独立）

- 只处理用户指定的图片；原图备份到同目录 `_backup/`
- 写回**必须**保持原像素宽高（含非正方形）
- 优先处理路径含 `book`/`guide` 或文件名含 `page` 的目录（guidebook 贴图）
- 提示：只改图上文字，不改画幅与图案

---

## ARC 模组：物品品级颜色

ARC 系列物品 `.lang` 名称前必须带品级颜色码（RP/BP 的 `texts/*.lang` 同步）：

| 品级 | 代码 |
|------|------|
| 普通 | `§f` |
| 稀有 | `§9` |
| 史诗 | `§5` |
| 传奇 | `§6` |
| 神秘 | `§c` |

```lang
item.arc:example.name=§9生命药水
```

所有语言文件保留相同颜色码，只翻译码后文字。

定级参考：基础素材 `§f`；常规补剂 `§f`–`§9`；综合补给 `§9`–`§5`；顶级 `§6`；神秘向 `§c`。

---

## Agent 执行检查清单

开始编辑前确认：

1. **顺序**：Display Name → lang → 脚本 → **硬编码数据层 (3b)** → 署名
2. **键一致**：JSON 中的 lang key 在 `en_US.lang` 与目标 `.lang` 均存在
3. **manifest**：`header.name` 应为 `pack.name`，不是字面量
4. **脚本**：不破坏引号/括号；`${}` 计数一致
5. **logic id**：配置表 key、枚举值、动态属性存盘值 **未改成译文**
6. **display label**：UI / nameTag / 插值 已走 `getDisplayName()` 或等价映射
7. **颜色码**：ARC 物品检查 `§` 前缀
8. **双包同步**：RP 与 BP 的 `texts/` 键对齐；lang 与脚本 fallback 映射一致

## 辅助操作（非核心）

| 操作 | 说明 |
|------|------|
| PBR 灵动视效 | RP `manifest.json` → `capabilities: ["pbr"]` |
| 成就兼容 | RP/BP → `metadata.product_type: "addon"` |
| 导出 | 将 RP+BP 打成 `.mcaddon` 或单包 `.mcpack` |

## 参考实现

开源桌面工具 [ARC MCBE Addon Auto Localizer](https://github.com/ARC-Minecraft/ARC-MCBE-Addon-Auto-Localizer) 实现了上述全流程；本 skill 与其逻辑对齐，Agent 无需安装该工具即可按相同规范手工操作。
