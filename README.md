# Shadowrocket Rules / lazy.conf

一个用于 [Shadowrocket](https://apps.apple.com/app/shadowrocket/id932747118) 的细致分流规则配置文件，面向日常使用场景优化。

## 文件结构

配置分为六个章节：

| 章节 | 作用 |
|------|------|
| `[General]` | 基础运行参数：DNS、IPv6、TUN、绕过列表、更新地址 |
| `[Proxy]` | 节点声明（本地自填，文件中仅作占位） |
| `[Proxy Group]` | 按地区分组的策略组：香港 / 台湾 / 日本 / 新加坡 / 韩国 / 美国 |
| `[Rule]` | 核心分流规则，按优先级从高到低共 12 层 |
| `[Host]` | Apple / iCloud 系统 DNS 映射及 localhost 绑定 |
| `[URL Rewrite]` | `g.cn` / `google.cn` → `google.com` 302 重写 |
| `[MITM]` | 仅声明 `*.google.cn`，配合 URL Rewrite 使用 |

## General 关键参数

- **bypass-system = true** — 系统请求不走代理，避免推送/通知延迟
- **dns-server** — 国内并发 DNS：腾讯 doh.pub、阿里 dns.alidns.com、223.5.5.5、119.29.29.29
- **ipv6 = true / prefer-ipv6 = false** — 开启 IPv6 但不优先
- **always-real-ip** — fake-ip 模式下白名单域名走真实 DNS 直连
- **hijack-dns** — 劫持 8.8.8.8:53 / 8.8.4.4:53 的 DNS 请求
- **udp-policy-not-supported-behaviour = REJECT** — 不支持 UDP 的策略直接拒绝，不回退
- **update-url** — 配置远程更新地址

## Proxy Group 设计

按地区分组，每个组通过 `policy-regex-filter` 正则在订阅节点池中自动匹配对应节点：

| 组名 | 策略类型 | 匹配规则 |
|------|----------|----------|
| 香港 | `select`（手动选择） | 🇭🇰 / HK / Hong / 香港 / 沪港 / 京港 … |
| 台湾 | `url-test`（自动优选） | 🇹🇼 / TW / Taiwan / 台湾 / 台北 / 台中 … |
| 日本 | `select`（手动选择） | 🇯🇵 / JP / Japan / 东京 / 大阪 / 京日 … |
| 新加坡 | `select`（手动选择） | 🇸🇬 / SG / Sing / 新加坡 / 狮城 … |
| 韩国 | `select`（手动选择） | 🇰🇷 / KR / Korea / 首尔 / 春川 … |
| 美国 | `select`（手动选择） | 🇺🇸 / US / USA / 洛杉矶 / 西雅图 / 芝加哥 … |
| 自动选择 | `url-test`（自动优选） | 排除台湾节点的其余所有节点 |

- **台湾 / 自动选择** 使用 `url-test` 自动测速选优（间隔 600s，容差 50ms）
- 其他地区使用 `select` 手动指定（select=0 表示默认选中列表第一个符合条件的节点）

## Rule 分流优先级（12 层）

规则从上到下逐条匹配，**先命中的生效**：

### 第 1 层：局域网直连
```
RULE-SET Lan.list → DIRECT
```
局域网网段直接放行，确保最高优先级。

### 第 2 层：自定义高阶分流
```
DOMAIN-SUFFIX jinnll.xyz → 美国
DOMAIN wiki.l23.im → 美国
DOMAIN mspp.abchina.com.cn → REJECT（屏蔽农行风控检测）
DOMAIN-KEYWORD binance → 日本
DOMAIN-KEYWORD saasexch → 日本
DOMAIN-KEYWORD yingwangtech → 日本
DOMAIN-SUFFIX push.apple.com → 美国（APNS 推送可靠路由）
```
个人定制走这里，银行风控域名直接拒绝，加密货币平台走日本路线。

### 第 3 层：广告屏蔽与第三方路由
```
RULE-SET TalkatoneAntiAds → REJECT
RULE-SET Crypto → 日本
RULE-SET apns → 美国
RULE-SET Binance → 日本
RULE-SET TalkatoneProxy → 美国
```
广告拦截 + 小众应用的定向路由。

### 第 4 层：流媒体与社交网络
```
YouTube → PROXY
Netflix → 日本
Disney+ → PROXY
HBO / Spotify / litix.io / discomax.com / brightline.tv → PROXY
Telegram → 美国
PayPal / Twitter / Facebook / TikTok → 美国
GitHub → PROXY
```
流媒体走代理不锁区，社交网络固定美国节点。

### 第 5 层：AI 服务（美国专线）
```
x.ai / Claude / Gemini / OpenAI → 美国
```
所有主流 AI 服务锁定美国节点。

### 第 6 层：国内应用直连
```
WeChat / BiliBili / NetEaseMusic / Baidu / DouBan / DouYin
Sina / Zhihu / XiaoHongShu / JingDong / Pinduoduo → DIRECT
```
国内主流应用强制直连，不走代理浪费流量。

### 第 7 层：游戏与商店平台
```
Sony / Nintendo / Epic / SteamCN / Steam / Game → DIRECT
```
游戏下载走本地 CDN 加速。

### 第 8 层：China-List 国内域名兜底
```
RULE-SET China.list → DIRECT
```
大中华区域名全集直连。

> **为什么 China-List 放在 Apple/Microsoft/Google 之前？**
>
> 国内 CDN 域名（如 `cn-cdn.apple.com`）会被 China-List 先匹配为直连，后面的 Apple.list 只命中真正的跨国服务，避免大厂中国 CDN 被误送代理。

### 第 9 层：跨国巨头基础设施
```
Apple / Microsoft / Google / Amazon → 美国
```
这些公司的跨国服务走美国节点，国内 CDN 已在第 8 层被截获直连。

### 第 10 层：Global 剩余域名
```
RULE-SET Global.list → PROXY
```
不在上述所有列表中的国外域名，统一走代理。

### 第 11 层：GEOIP 中国大陆 IP 直连
```
GEOIP,CN → DIRECT
```
IP 属于中国大陆的直连。

### 第 12 层：FINAL 兜底
```
FINAL → PROXY
```
以上全部未匹配的流量，全部走代理。

## 流量走向示意图

```
客户端请求
  │
  ├─ 局域网/国内 IP → DIRECT
  ├─ 自定义屏蔽 → REJECT
  ├─ 广告列表 → REJECT
  ├─ 流媒体/社交 → PROXY / 指定地区
  ├─ AI 服务 → 美国
  ├─ 国内应用/China-List → DIRECT
  ├─ Apple/MS/Google/Amazon → 美国
  ├─ Global 列表 → PROXY
  ├─ GEOIP CN → DIRECT
  └─ 未匹配 → FINAL,PROXY
```

## 快速使用

1. 在 Shadowrocket 中添加节点订阅
2. 导入 `lazy.conf` 作为配置
3. 确保节点名称包含地区标识（如 `🇭🇰`、`🇯🇵`），策略组会自动匹配
4. 也可通过远程更新地址获取最新配置：
   ```
   https://gh-proxy.com/https://raw.githubusercontent.com/Jinn-Y/shadowrocket-rule/refs/heads/main/lazy.conf
   ```

## 参考上游

- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) — 大部分 `RULE-SET` 来源
- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)
- [LOWERTOP/Shadowrocket-First](https://github.com/LOWERTOP/Shadowrocket-First)
- [iab0x00/ProxyRules](https://github.com/iab0x00/ProxyRules)