# MCBE 官方语言码（29 种）

Microsoft Learn「Comprehensive Pack Contents」所列语言码。本地化时 `texts/{code}.lang` 与 `languages.json` 须一致。

> 署名后缀**不在本表维护**：统一见 [SKILL.md 步骤 4](../SKILL.md#步骤-4本地化者署名)。
> 本表只列语言码，避免两处模板打架。

| 代码 | 名称 |
|------|------|
| `zh_CN` | 简体中文 |
| `zh_TW` | 繁體中文 |
| `en_US` | 英语（美国） |
| `en_GB` | 英语（英国） |
| `ja_JP` | 日语 |
| `ko_KR` | 韩语 |
| `de_DE` | 德语 |
| `fr_FR` | 法语 |
| `fr_CA` | 法语（加拿大） |
| `es_ES` | 西班牙语 |
| `es_MX` | 西班牙语（墨西哥） |
| `pt_BR` | 葡萄牙语（巴西） |
| `pt_PT` | 葡萄牙语 |
| `it_IT` | 意大利语 |
| `nl_NL` | 荷兰语 |
| `ru_RU` | 俄语 |
| `uk_UA` | 乌克兰语 |
| `pl_PL` | 波兰语 |
| `cs_CZ` | 捷克语 |
| `sk_SK` | 斯洛伐克语 |
| `hu_HU` | 匈牙利语 |
| `tr_TR` | 土耳其语 |
| `sv_SE` | 瑞典语 |
| `da_DK` | 丹麦语 |
| `nb_NO` | 挪威语 |
| `fi_FI` | 芬兰语 |
| `el_GR` | 希腊语 |
| `bg_BG` | 保加利亚语 |
| `id_ID` | 印尼语 |

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
