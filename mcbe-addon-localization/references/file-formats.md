# MCBE 本地化文件格式参考

## .lang 文件

```lang
## 注释以 ## 开头
pack.name=Arc Survival
pack.description=An addon for Bedrock Edition.
item.arc:health_potion.name=§9Health Potion
entity.arc:guard.name=§fGuard
tile.arc:machine.name=Crafting Machine
```

规则：

- 一行一条：`key=value`
- 键名不翻译
- 值可含 `§` 颜色码、`\n`（文件中为字面 `\n` 或转义）、`%s` / `%1$s` 占位符
- 编码 UTF-8（可带 BOM）

## languages.json

```json
[
  "en_US",
  "zh_CN"
]
```

目标语言码必须出现在此数组中，游戏才会加载对应 `.lang`。

## manifest.json 包名语言键

推荐写法：

```json
{
  "header": {
    "name": "pack.name",
    "description": "pack.description"
  }
}
```

若原为字面量 `"My Cool Addon"`：

1. 写入 `pack.name=My Cool Addon` 到 `en_US.lang` 与目标 lang
2. 将 header 改为 `"pack.name"`

## Display Name 标准化示例

**标准化前：**

```json
{
  "minecraft:item": {
    "description": { "identifier": "arc:health_potion" },
    "components": {
      "minecraft:display_name": "Health Potion"
    }
  }
}
```

**标准化后：**

```json
"minecraft:display_name": "item.arc:health_potion.name"
```

**lang 追加：**

```lang
item.arc:health_potion.name=Health Potion
```

## 语言键形态识别

视为已是语言键（跳过 JSON 改写）：

- 匹配 `^[A-Za-z][\w]*(\.[\w:]+)+$`
- 示例：`item.arc:potion.name`、`entity.minecraft:cow.name`

## 待译检测（启发式）

对非拉丁目标语言（如 `zh_CN`）：

- 统计文本中目标文字系统字符占比（汉字、假名等）
- 占比 < 20% 且含字母 → 视为待译

对拉丁语系目标（`de_DE`、`fr_FR` 等）：

- 无法靠字符集区分，主要用「是否与 `en_US` 完全相同」判断

## 导出包结构

`.mcaddon` = zip，内含 RP 与 BP 文件夹（或 `.mcpack` 单包）。  
压缩包根目录应直接是包内容（含 `manifest.json`），不要多套一层无关目录。

## Script 包：lang 与数据层

带 Script API 的包常见 **两套并行文案**：

| 体系 | 位置 | 示例 |
|------|------|------|
| Lang | `texts/*.lang` | `item.ns:id.name`、表单 `{ translate: "ns:key" }` |
| 数据/常量 | `scripts/**` 内枚举、配置表、nameTag | 枚举 id、配置 `label`、`sendMessage("...")` |

**原则：** 参与 lookup / 存档 / 比较的 **logic id 不改**；玩家可见 **display label** 在展示层翻译或映射。

处理模式见 [hardcoded-strings.md](hardcoded-strings.md)。

**反例：** 把配置表 object key 改成译文 → `config[selectedId]` 查找失败。

**正例：** 保留 id；UI 调用 `getDisplayName("category", selectedId)`。
