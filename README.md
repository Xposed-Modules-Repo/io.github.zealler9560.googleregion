# 谷歌区域伪装（Google Region Spoof）

一个 LSPosed 模块，把**谷歌应用读到的客户端区域信号**伪装成选定的目标地区：
SIM/网络国家码、运营商（MCC+MNC 与名称）、语言（Locale）、时区、Resources 配置的 locale、`gsm.*` 系统属性，以及可选的 GPS 坐标。

> **边界先说清**：本模块只改应用在本地读到的区域信号，**不改变 IP 出口**。很多谷歌服务是「IP + 客户端信号」一起看的，请配合代理（IP 出口与所选区域一致）使用，否则信号互相矛盾，效果会打折。

## 作用域（推荐）

- `com.google.android.gms`（Google Play 服务）
- `com.android.vending`（Play 商店）
- `com.google.android.googlequicksearchbox`（Google 应用）
- `com.google.android.apps.bard`（Gemini）
- `com.google.android.youtube`（YouTube）

## 功能

- **国家 → 运营商二级联动**：先选国家（带国旗、可搜索的列表），运营商卡片随之切换为该国运营商，点选即保存；切换国家时运营商自动重置为该国第一家。
- **当前伪装值面板**：SIM 国家码、网络国家码、SIM 运营商、语言/地区、时区、GPS 位置一览，随时核对。
- **GPS 伪装（可选，默认关）**：按所选城市（默认首都，可换同国其他主要城市）伪装定位坐标；在 GMS 等系统应用里开启可能影响依赖定位的谷歌服务，不需要时请保持关闭。
- **外观设置**：浅色 / 深色 / 跟随系统三档 + 主题颜色（橙/蓝/绿/紫/红）。

## 国家与运营商（14 国，每国 2–3 家真实运营商）

| 国家 | 运营商（MCC+MNC） | 语言 | 时区 |
|---|---|---|---|
| 美国 us | T-Mobile 310260 / AT&T 310410 / Verizon 311480 | en-US | America/New_York |
| 日本 jp | NTT docomo 44010 / SoftBank 44020 / au 44007 | ja-JP | Asia/Tokyo |
| 新加坡 sg | Singtel 52501 / StarHub 52505 / M1 52503 | en-SG | Asia/Singapore |
| 香港 hk | CMHK 45400 / CSL 45419 / SmarTone 45406 | zh-HK | Asia/Hong_Kong |
| 台湾 tw | 中华电信 46692 / 台湾大哥大 46697 / 远传电信 46601 | zh-TW | Asia/Taipei |
| 英国 gb | EE 23430 / O2 23410 / Vodafone 23415 | en-GB | Europe/London |
| 德国 de | Telekom.de 26201 / O2 26207 / Vodafone 26202 | de-DE | Europe/Berlin |
| 韩国 kr | SK Telecom 45005 / KT 45008 / LG U+ 45006 | ko-KR | Asia/Seoul |
| 泰国 th | AIS 52001 / TrueMove H 52004 / dtac 52005 | th-TH | Asia/Bangkok |
| 澳大利亚 au | Telstra 50501 / Optus 50502 / Vodafone 50503 | en-AU | Australia/Sydney |
| 马来西亚 my | Maxis 50212 / Celcom 50219 / Digi 50216 | ms-MY | Asia/Kuala_Lumpur |
| 菲律宾 ph | Globe 51502 / Smart 51503 | en-PH | Asia/Manila |
| 越南 vn | Viettel 45204 / Vinaphone 45202 / Mobifone 45201 | vi-VN | Asia/Ho_Chi_Minh |
| 巴西 br | Vivo 72410 / Claro 72405 / TIM 72402 | pt-BR | America/Sao_Paulo |

默认：美国 / T-Mobile。

## 使用

1. 安装 APK，在 LSPosed 中启用本模块并勾选上面的作用域，重启目标应用。
2. 打开模块界面，选择国家与运营商。
3. 切换国家/运营商后：**强停**对应的谷歌应用；Play 商店还需**清一次缓存**才生效。
4. Play 商店的账号国家还受账号设置与支付资料影响，本模块改不了平台侧规则。

## 实现说明

基于经典 Xposed API 82。每个 hook 独立 try/catch，失败静默放行原结果：

- `TelephonyManager`：getSimCountryIso / getNetworkCountryIso / getSimOperator / getNetworkOperator / getSimOperatorName / getNetworkOperatorName（含带 int 参数的隐藏重载）
- `Locale.getDefault()`、`TimeZone.getDefault()`
- `Resources.getConfiguration()`（返回改过 locale 的副本）
- `SystemProperties.get`：仅拦截 `gsm.*.operator.*` 只读键
- GPS（开关打开时）：`Location.getLatitude/getLongitude`、`LocationManager.getLastKnownLocation`

配置经 XSharedPreferences 读取，读不到时回退默认（美国 / T-Mobile），不会因读不到配置崩溃。

## 源码

https://github.com/zealler9560/GoogleRegionSpoof

## 致谢

功能面参考了 GlobalKit（com.tiktok.unlocker）的公开说明；预设组织方式借鉴了 Nrfr（Ackites/Nrfr）。本模块为独立编写。
