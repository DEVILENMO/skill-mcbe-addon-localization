# MCBE 官方语言码（29 种）

Microsoft Learn「Comprehensive Pack Contents」所列语言码。本地化时 `texts/{code}.lang` 与 `languages.json` 须一致。

| 代码 | 名称 | 署名后缀模板 |
|------|------|-------------|
| `zh_CN` | 简体中文 | `（{credit}汉化）` |
| `zh_TW` | 繁體中文 | `（{credit}漢化）` |
| `en_US` | 英语（美国） | `({credit} Localization)` |
| `en_GB` | 英语（英国） | `({credit} Localization)` |
| `ja_JP` | 日语 | `（{credit}翻訳）` |
| `ko_KR` | 韩语 | `({credit} 번역)` |
| `de_DE` | 德语 | `({credit} Localization)` |
| `fr_FR` | 法语 | `({credit} Localization)` |
| `fr_CA` | 法语（加拿大） | `({credit} Localization)` |
| `es_ES` | 西班牙语 | `({credit} Localization)` |
| `es_MX` | 西班牙语（墨西哥） | `({credit} Localization)` |
| `pt_BR` | 葡萄牙语（巴西） | `({credit} Localization)` |
| `pt_PT` | 葡萄牙语 | `({credit} Localization)` |
| `it_IT` | 意大利语 | `({credit} Localization)` |
| `nl_NL` | 荷兰语 | `({credit} Localization)` |
| `ru_RU` | 俄语 | `({credit} Localization)` |
| `uk_UA` | 乌克兰语 | `({credit} Localization)` |
| `pl_PL` | 波兰语 | `({credit} Localization)` |
| `cs_CZ` | 捷克语 | `({credit} Localization)` |
| `sk_SK` | 斯洛伐克语 | `({credit} Localization)` |
| `hu_HU` | 匈牙利语 | `({credit} Localization)` |
| `tr_TR` | 土耳其语 | `({credit} Localization)` |
| `sv_SE` | 瑞典语 | `({credit} Localization)` |
| `da_DK` | 丹麦语 | `({credit} Localization)` |
| `nb_NO` | 挪威语 | `({credit} Localization)` |
| `fi_FI` | 芬兰语 | `({credit} Localization)` |
| `el_GR` | 希腊语 | `({credit} Localization)` |
| `bg_BG` | 保加利亚语 | `({credit} Localization)` |
| `id_ID` | 印尼语 | `({credit} Localization)` |

默认目标语言：`zh_CN`

## 文字系统（待译检测用）

| script | 语言示例 |
|--------|----------|
| `han` | zh_CN, zh_TW |
| `kana` | ja_JP |
| `hangul` | ko_KR |
| `cyrillic` | ru_RU, uk_UA, bg_BG |
| `greek` | el_GR |
| `latin` | en_*, de_DE, fr_FR, es_ES, … |

拉丁语系之间无法靠字符集区分，翻译时以 `en_US.lang` 对照为准。
