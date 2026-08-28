# 脚本硬编码与数据层本地化

`.lang` 翻完 ≠ 游戏内全中文。带 **Script API** 的 MCBE 包常在 lang 体系之外还有一层玩家可见文案。本节描述 **可复用到任意模组** 的识别与处理模式。

## 两层（乃至三层）文案体系

| 层级 | 典型位置 | 形态 | 本地化方式 |
|------|----------|------|------------|
| **Lang 层** | `RP/BP/texts/*.lang` | `pack.name`、`item.ns:id.name`；表单 `{ translate: "ns:key" }` | 步骤 2 |
| **脚本字面量层** | `scripts/**/*.js` | `sendMessage("...")`、`.button("...")` | 步骤 3 |
| **数据/常量层** | `*constants*`, `*config*`, `*data*`, `variables.js` | 枚举数组、配置表 `name`/`label`/`title`、lookup 用 id | **步骤 3b** |

**核心原则：区分「逻辑标识」与「显示文案」。**

- **逻辑标识（logic id）**：参与 lookup、存档、比较、映射表键名、动态属性值 — **保持作者原始字符串（常为英文）**
- **显示文案（display label）**：仅给玩家看 — **译为目标语言**，或通过映射函数转换

误把 logic id 改成译文是最常见的「汉化后功能坏掉」原因。

---

## 何时进入步骤 3b

满足 **任一** 即应扫描数据层：

1. BP 存在 `scripts/`，且含 UI、配置表、枚举常量
2. `target.lang` 与 `en_US.lang` 键已对齐，游戏内仍有英文
3. 表单已用 `{ translate }`，但按钮/正文/插值仍显示英文
4. 用户反馈「某类名称（建筑、职业、等级、状态）未汉化」

**假完成信号：** target `.lang` 行数很多、步骤 3 已改大量 `sendMessage`，但 **constants / 配置 JSON / nameTag** 未动。

---

## 第一步：给字符串「贴标签」

对每条可疑英文字符串，先判断 **在代码里扮演什么角色**（可多选）：

| 角色 | 含义 | 能否直接改成译文 |
|------|------|------------------|
| **Logic id** | object key、Map 键、数组元素用于 `===` / 索引配置表 | ❌ 否 |
| **Stored value** | 写入 `setDynamicProperty`、NBT、scoreboard、数据库式字段 | ❌ 否（旧存档兼容） |
| **Display only** | 只出现在 UI、聊天、nameTag，不参与逻辑 | ✅ 可直改或进映射表 |
| **Dual-use** | 同一字符串既展示又当 id | ⚠️ 保留 id + 展示层映射 |
| **Lang-bound** | 已是 lang 键或 `{ translate }` | 只译 `.lang` |
| **Interpolated** | 出现在 `{ translate, with: [...] }` 或模板 `${var}` 的变量值 | 译 **变量来源**，不是译 translate 键 |

**追踪方法：** 对字符串做反向引用搜索 — 是否出现在 `obj[key]`、`switch`、`includes`、`setDynamicProperty`、JSON 配置的 **键名** 位置。

---

## 第二步：按来源扫描（BP）

### 高优先级文件

- `*constants*`, `*config*`, `*data*`, `variables.js`, `definitions.js`
- `scripts/ui/**`, `scripts/menus/**`
- 实体生成 / 管理：`spawn`, `summon`, `nameTag`
- 物品/方块 fallback：`getItemDisplayName`, `itemNameMap`, `displayNameMap`

### 通用 grep 模式

| 模式 | 通常是什么 |
|------|------------|
| `nameTag\s*=` | 实体头顶名、标记名 |
| `summon .+ "` | 生成实体时的显示名 |
| `setDynamicProperty\(` / `getDynamicProperty\(` | 存档字段（值可能是 id） |
| `sendMessage\(` / `setActionBar\(` | 未走 translate 的提示 |
| `\.button\(\s*[`'"]` | 硬编码表单（非 translate 对象） |
| `\.dropdown\([^)]*\[` | options 数组硬编码 |
| `export const \w+ = \[` + 英文字面量 | 枚举/分类列表 |
| `\w+:\s*\{[^}]*"(name|label|title|displayName)"` | 配置对象内嵌文案 |
| `get\w*DisplayName\|itemNameMap\|labelMap` | 显示名 fallback 字典 |
| `\{ translate:` 同文件内仍有英文字面量 | translate 与硬编码混用 |

也检查 BP 内嵌 JSON（loot table 描述、trade table 等）及 RP 中带 `"text":` 的 UI JSON（较少见）。

### 英文残留复核（启发式）

在 `scripts/` 搜 **Title Case 短语、状态词、占位符**（按模组语境调整）：

```
None|Not Set|Unknown|Default
Level|Tier|Rank|Grade
Manager|Master|Chief|Admin
Progress:|Cost:|Remaining
```

不要用固定「商店/工厂」词表代替实际上游扫描；上表只是 **常见漏网形态**。

---

## 第三步：选择处理策略

### 策略 A — 纯显示常量

**适用：** 字符串从不参与 lookup / 存档 / 比较。

示例形态：随机名池、纯描述数组、`description` 字段仅用于 UI。

→ 直接改为目标语言，或写入 `displayNames` 映射（便于多语扩展）。

### 策略 B — 双用途：logic id + 显示名（最常见）

**适用：** 枚举值既在 UI 展示，又是配置表键或存档字段。

```javascript
// ❌ 不要改枚举/键
const TYPES = ["none", "warrior", "mage"];
const recipes = { warrior: { ... }, mage: { ... } };

// ✅ 展示时转换
form.button(getDisplayName("class", typeId), icon);
world.setDynamicProperty("player_class", typeId); // 仍存 id
```

**做法：** 新增 `localization.js`（或 `i18n/displayNames.js`），集中维护 **id → 译文** 映射；**不改** 源码中的 id 字符串与 object key。

### 策略 C — 多级映射（A → B → 显示名）

**适用：** 作者用两套 id（如「职业 id → 建筑 id → 玩家可见名」）。

每一层 **logic id 保留**；每一层 **展示** 各建映射或统一 `getDisplayName(category, id)`：

```javascript
export function getDisplayName(category, id) {
  const table = displayNameTables[category];
  return table?.[id] ?? id;
}
// category 示例: "enum", "building", "status", "item_fallback"
```

两套 id 不要混在一个 map 里键名冲突。

### 策略 D — 已走 translate，仍见英文

```javascript
form.title({ translate: "myaddon:ui.title" });
form.button({ translate: "myaddon:ui.btn", with: [rawLabel] });
```

1. `translate` 键 → 步骤 2 译 `.lang`
2. **`with` / 模板插值** → 追溯 `rawLabel` 来源；若在 constants 里，走策略 B

### 策略 E — 物品/实体显示名 fallback

脚本内自建 `itemId → 英文名` 字典时（常见于自定义 UI 列表、托盘、奖励预览）：

- 优先：RP/BP `.lang` 补 `item.*` / `entity.*` 键（步骤 1–2）
- 并行：`itemDisplayNames` / `entityDisplayNames` 映射作 fallback（步骤 3b）
- 字典 **键** 用 identifier（`namespace:id`），**值** 用译文

### 策略 F — 实体 nameTag / summon 字符串

```javascript
entity.nameTag = "§6Quest Giver";
// 或 execute ... summon ... "§6Quest Giver"
```

→ 改显示字符串或 `getDisplayName("npc_role", id)`；**不要**改实体 type id / 事件 id。

---

## 推荐：`localization.js` 模块结构

单语汉化包可只维护目标语言；多语包可按 `locale` 扩展。

```javascript
/** category → { logicId: displayLabel } */
const displayNameTables = {
  enum: { none: "无", warrior: "战士" },
  building: { smelter: "冶炼厂" },
  status: { poor: "较差", good: "良好" },
  item_fallback: { "minecraft:iron_ingot": "铁锭" },
};

export function getDisplayName(category, id, fallback = "") {
  if (id == null || id === "") return fallback || "—";
  const label = displayNameTables[category]?.[id];
  return label ?? id; // 未知 id 回退原文，避免 undefined
}

/** 蛇形/ kebab 配置键 → 显示名（如 treasure_hunter） */
export function getDisplayNameFromKey(category, key) {
  const normalized = String(key).replace(/[-\s]+/g, "_").toLowerCase();
  return getDisplayName(category, normalized) ?? key;
}
```

命名不必叫 `localization.js`；关键是 **集中、可 grep、展示层统一入口**。

### 接入点（按优先级）

1. **所有 UI 入口** — 表单按钮/标签/下拉 options 在渲染前过一遍 `getDisplayName`
2. **列表/详情页** — 配置表 `name`/`title` 字段展示处
3. **实体 nameTag** — 生成、刷新、状态变更时
4. **聊天/ActionBar** — 含 id 的模板字符串
5. **fallback 字典** — `getItemDisplayNameText` 等改为读映射表
6. **translate 插值** — 传入 `with:` 前转换

### 禁止事项

- 不用译文替换 **object key**、**Map 键**、**动态属性存盘值**
- 不改 **事件名 / 函数名 / 命令参数 id**
- 不造成 **循环 import**（constants ↔ localization）：映射表放独立模块，constants 不 import UI
- 不假设「数组下标 = 显示顺序」而改数组长度或顺序

---

## 与步骤 2 的协同

| 内容 | 优先路径 |
|------|----------|
| 有稳定 identifier 的物品/实体/方块 | 步骤 1–2：lang 键 |
| 脚本 UI 固定文案 | 步骤 3 或 translate + lang |
| 枚举 id、配置表 label、nameTag | 步骤 3b 映射 |
| 托盘/预览用物品名 | lang + `item_fallback` 双保险 |

---

## 验收清单（通用）

```
- [ ] target.lang 与 en_US.lang 键集合对齐
- [ ] 所有 { translate } 键在 target.lang 存在
- [ ] 配置表/枚举：logic id 未改成译文
- [ ] UI/nameTag/聊天：展示处已走 getDisplayName 或等价映射
- [ ] translate 的 with/插值来源已本地化
- [ ] 物品/实体：lang 键与 fallback 映射一致（若两者并存）
- [ ] 动态属性/存档字段：仍写原始 id（抽样进游戏验证旧档）
- [ ] scripts 下无新增英文 Title Case 漏网（grep 复核）
- [ ] node --check 通过（修改过的 .js）
- [ ] 游戏内抽测：选单、实体名牌、状态行、奖励列表
```

---

## 决策树

```
该字符串是否玩家可见？
├─ 否 → 不译
└─ 是 → 是否已通过 lang / { translate } 完整覆盖？
    ├─ 是 → 仅查插值变量来源
    └─ 否 → 是否作为 logic id / 存档值 / object key？
        ├─ 是 → 保留 id + 展示层 getDisplayName()
        └─ 否 → 直译字面量 或 写入 displayNameTables
```

---

## 附录：典型场景 → 策略对照

抽象场景，不绑定特定模组文件名。

| 玩家所见问题 | 常见根因 | 策略 |
|--------------|----------|------|
| 选单里是英文 id | `.button(enumValue)` 直接用枚举 | B |
| 教程/UI 已中文，列表仍英文 | lang 已译，constants 未译 | B + 扫描 UI |
| 两种名称体系（类型 vs 实例） | 两套 id 映射 | C |
| 表单标题中文、按钮英文 | translate + 硬编码混用 | D |
| 物品名在自定义 UI 里英文 | 脚本 fallback 字典 | E |
| 实体头顶英文 | nameTag / summon 字符串 | F |
| 状态行「Poor / None」 | 格式化函数内硬编码 | A 或 B |
| 汉化后配方/建筑失效 | 误改 object key 或枚举值 | **回滚 id**，仅改展示层 |
