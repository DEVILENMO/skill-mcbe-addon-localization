# 隐式查键与官方措辞基准

MCBE 会**按 id 拼键**去 lang 里查名字。除了 JSON 里显式写着的 `minecraft:display_name`，
还有一整族「游戏自己拼出来的键」。**只翻 `display_name` 会漏掉这一族**，
表现就是玩家看到 `item.spawn_egg.entity.xxx.name` 这种原始键。

> 本文的用例来自《华丽深渊 1.0.1》（Nearbear）实测：玩家连续三次反馈
> 「游戏里还是英文 / 印着原始键」，三次全部来自这一族。

## 隐式键全表

| 键 | 触发条件 | 缺了会怎样 |
|---|---|---|
| `entity.<ns>:<id>.name` | BP 实体（无 `display_name` 组件时） | 实体名显示原始 ID |
| `item.<ns>:<id>.name` | BP 物品 | 物品名显示原始 ID |
| `tile.<ns>:<id>.name` | BP 方块 | 方块名显示原始 ID |
| `item.spawn_egg.entity.<ns>:<id>.name` | BP 实体 `is_spawnable: true` | 刷怪蛋印原始键 |
| `action.hint.exit.<ns>:<id>` | BP 实体有 `minecraft:rideable` | 离船提示印原始键 |
| `action.hint.exit.console.<ns>:<id>` | 同上（手柄版变体） | 手柄用户看到原始键 |
| `<ns>:itemGroup.name.<group>` | 方块/物品 `menu_category.group` 指向自定义组 | 创造栏显示原始键 |

### 三个最容易搞错的判定依据

1. **刷怪蛋看 `is_spawnable`，不看 RP 的 `spawn_egg` 块。**
   RP 的 `minecraft:client_entity.description.spawn_egg` **只决定蛋的图标**，
   不决定有没有蛋 —— 游戏会对任何 `is_spawnable: true` 的实体自动生成刷怪蛋物品。
   - 只翻 RP 的 client_entity 会漏掉用**默认蛋图标**的实体。
   - 刷怪蛋**没有 BP 物品文件**，按 items 目录或文件名扫都找不到。
2. **离船提示看 `minecraft:rideable` 组件**，不看别的。
3. **命名空间写法两套并存：**
   - 原版用**短名**：`entity.pig.name`、`item.spawn_egg.entity.pig.name`、`action.hint.exit.boat`
   - 自定义内容用**完整 `ns:id`**：`entity.glammed:gleamer.name`、`action.hint.exit.glammed:azalea_boat`
   - 查原版措辞时别把短名直接套到自定义内容上。

## 官方措辞基准：原版语言文件

任何译名有疑问，**不要凭印象翻**，去原版官方语言文件里查同一条键：

```
<游戏安装目录>\data\resource_packs\vanilla\texts\zh_CN.lang
<游戏安装目录>\data\resource_packs\vanilla\texts\en_US.lang
```

Windows 实际路径示例：

- LeviLauncher 实例：`%LOCALAPPDATA%\LeviLauncher\Instances\<版本>\data\resource_packs\vanilla\texts\`
- Microsoft Store 版：`<安装目录>\minecraftpe_Data\resources\...`，或在
  `C:\Program Files\WindowsApps\Microsoft.MinecraftUWP_*` 包内找 `vanilla/texts/`

原版 zh_CN 有 **13000+ 条键**，是官方简中用词的权威来源。

### 实测用例（可直接抄）

| 键 | 官方中文 |
|---|---|
| `action.hint.exit.boat` | 点击“跳跃”离开船 |
| `action.hint.exit.console.boat` | 按下 :_input_key.jump: 离开船 |
| `item.spawn_egg.entity.pig.name` | 猪生成蛋 |
| `item.boat.oak.name` | 橡木船 |
| `item.chest_boat.oak.name` | 橡木运输船 |
| `entity.boat.name` | 船 |
| `entity.chest_boat.name` | 运输船 |

三条要注意的坑：

1. **船只有本体 + console 两个变体，没有 scheme 变体。**（原版 `action.hint.exit.scheme.*`
   里没有 boat/chest_boat/raft）
2. **生成蛋格式是「<实体名>生成蛋」**，不是「<实体名>的蛋」。
3. ⚠ **原版 13420 条中文键里「小船」出现 0 次** —— 「小船」是民间叫法，不是官方译名。
   自定义木种船按原版结构应译「XX木船」（木板叫「XX木板」→ 木种即「XX木」），
   不是「XX小船」。

### 命名判据：看图标是什么就叫什么

- 船的刷怪蛋**图标就是船本身**、物品即船 → 沿用船名（「杜鹃木船」）
- 用**默认蛋图标** → 沿用原版生成蛋格式（「闪耀者生成蛋」，同「猪生成蛋」）

实体名与刷怪蛋名必须一致。

## 审计脚本：解析失败不许静默跳过

作者包的 JSON 常带 `//` 注释、`/* */` 块注释、**尾逗号**。MCBE 自己的解析器容忍这些，
`JSON.parse` 不容忍。审计脚本若写成 `catch { continue }`，**该文件会被所有规则跳过**，
变成静默盲区 —— 实测漏掉过一个实体的 `entity.*.name` 与它的刷怪蛋键，
玩家实机截图才发现。

```js
function parseLooseJson(file) {
  let s = fs.readFileSync(file, 'utf8')
    .replace(/\/\*[\s\S]*?\*\//g, '')
    .replace(/^\s*\/\/.*$/gm, '');
  try { return JSON.parse(s); } catch {}
  s = s.replace(/,(\s*[}\]])/g, '$1');          // 去尾逗号
  try { return JSON.parse(s); } catch {}
  return null;                                  // ← 必须列进报告，不能默默跳过
}
```

**通用判据**：审计脚本里「解析失败就跳过」= 一个静默盲区。
要么换成宽松解析，要么把解析失败的文件**列进报告**（至少要能看见）。

## 验收：别用「覆盖率 100%」当完成标准

**lang 覆盖率 100% ≠ 游戏里没有原始键。** 验收要过三遍：

1. **显式键** —— JSON 里的 `minecraft:display_name` 在两个 lang 里都存在
2. **隐式键** —— 按上表逐族反查：实体、物品、方块、刷怪蛋、离船提示、创造栏分组
3. **兜底** —— 把「应有键集合」与 lang 实际键集合做差集，必须为空

产出报告里要点名列出：**实体 N 个 / 物品 N 个 / 方块 N 个 / 刷怪蛋 N 个 / 可骑乘 N 个**，
每个都带 id，便于人工过一眼。构建日志打印这些清单，比只报「通过」有用得多。

### 反向测试：确认规则不是摆设

每加一条校验规则，都要验证它真的能拦住漏译：

1. 从译文表里删掉一条目标键
2. 跑构建 → 必须非 0 退出，且报出那条键
3. 原样还原 → 构建必须恢复通过

只加规则不测，等于没加。
