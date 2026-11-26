## ACL4SSR_Online_Full_Mannix.ini

自定义 Clash 配置模板 https://dd.al/1phZx

可作为 订阅转换 配置转换 规则转换 的远程配置

修改自 https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini

---

### V2（历史记录）

自带旗帜 emoji 添加逻辑，原名不包含旗帜 emoji 才添加，原名已包含旗帜 emoji 则不添加

**需去除订阅转换链接中的参数 `emoji=true/false` 才能生效**，参考例子：

`https://api.dler.io/sub?target=clash&udp=true&new_name=true&config=https://dd.al/config&url={原订阅链接}`

短链：`https://dd.al/dler?url={原订阅链接}`

---

移除
- 📢 谷歌FCM
- Ⓜ️ 微软云盘
- Ⓜ️ 微软服务
- 🍎 苹果服务
- 📲 电报消息
- 🎶 网易音乐
- 🎮 游戏平台
- 📹 油管视频
- 🎥 奈飞视频
- 🌏 国内媒体
- 🌍 国外媒体
- 📺 巴哈姆特
- 🇰🇷 韩国节点

重命名
- 🚀 节点选择 -> ✈️ 起飞
- 🚀 手动切换 -> 👆🏻 指定
- ♻️ 自动选择 -> ⚡ 低延迟
- 📺 哔哩哔哩 -> 📺 B站
- 🎯 全球直连 -> 🛩️ 墙内
- 🐟 漏网之鱼 -> 🌐 未知站点
- 🇭🇰 香港节点 -> 🇭🇰 香港
- 🇨🇳 台湾节点 -> 🇹🇼 台湾
- 🇸🇬 狮城节点 -> 🇸🇬 新加坡
- 🇯🇵 日本节点 -> 🇯🇵 日本
- 🇺🇲 美国节点 -> 🇺🇸 美国

合并
- 🛑 广告拦截 + 🍃 应用净化 -> 💩 广告

新增
- 🇨🇳 中国 (含 🇭🇰 香港 🇹🇼 台湾)
- 🎏 其他
- 🤖 ‍OpenAI

url-test
- 延迟测试链接 http://www.gstatic.com/generate_204 -> https://i.ytimg.com/generate_204
- 间隔时间 300秒 -> 30秒
- 容差 50毫秒

📺 ‍B站 默认选择 🇨🇳 ‍中国

正则匹配大小写、简繁体，更好的匹配中转、IPLC节点

LocalAreaNetwork.list 使用 DIRECT

---

### V3 — Google 登录 / Gemini 出口一致性修复（2026-11-21）

**症状**：Gemini macOS 客户端显示「无法登录」；网页版可打开，但对话响应 2~6 秒。

**根因**：OpenClash 的 Smart 内核会把配置中所有 `url-test` 分组转换成
`type: smart` + `uselightgbm: true`，即**每一个连接独立打分选路**。
叠加 `🚀 节点选择` 是 `select` 且默认指向 `♻️ 自动选择`（同样是 Smart），
导致同一次 Google 登录的各个请求落到不同国家的出口：

| 域名 | 用途 | 修复前出口 |
|---|---|---|
| `accounts.google.com` | 账号认证 | 🇺🇸 美国 Frontier |
| `oauth2.googleapis.com` | 换取 token | 🇸🇬 新加坡 lite-02 |
| `people.googleapis.com` | 拉取用户信息 | 🇸🇬 新加坡 lite-02 |
| `alkalimakersuite-pa.clients6.google.com` | Gemini 后端 | 🇺🇸 美国 Cox |
| `cloudcode-pa.googleapis.com` | Gemini 后端 | 🇹🇼 台湾 hinet |
| `gemini.google.com` | 对话主服务 | 🇸🇬 新加坡 直连-01 |

当时的路由日志显示登录链路使用了多个出口。V3 将相关请求改为固定出口（V4 已改名为 `🔍 谷歌服务`），以减少登录会话切换地址。

**改动**：

1. `固定出口` 由 `url-test` 改为 `select` —— `select` 不会被 Smart 化，手工钉一个节点后 IP 全程一致。
   这是关键：只要还是 `url-test`，无论规则怎么配，节点都会被内核逐连接重选。
2. 新增 `GoogleLogin.list`，把登录全链路域名（认证 / token / 用户信息 / Gemini 后端 /
   静态资源）统一收敛到 `固定出口`，并置于 `ProxyGFWlist` **之前**，
   避免被 `DOMAIN-KEYWORD,google` 抢先匹配到 `🚀 节点选择`。模板中排列更靠前的 Gemini 和 FCM 规则仍可能先匹配，V4 合并了这些入口。

**使用**：订阅更新后在策略组里把 Google 分组（V3 为 `固定出口`，V4 为 `🔍 谷歌服务`）手工选成一个固定节点，并实际验证 Google 首页、登录与 Gemini。

### V4 · Google 和 AI 分流整理

| 策略 | 流量 | 默认选择 |
| --- | --- | --- |
| `🔍 谷歌服务` | Google 搜索、账户、Workspace、Play、FCM、Gemini、Antigravity（反重力）、YouTube，及 Google 服务目的 IP | 优先显示家宽节点，其次常用地区节点，也可从手动切换中选取具体节点 |
| `🤖 AI` | ChatGPT、Codex、Claude、Grok、Copilot、Cursor、OpenRouter 等 | 独立选择具体节点 |
| `🚀 节点选择` | 通用海外站点、Telegram | 自动选择 |
| `🎯 全球直连` | 国内通用、普通微软与苹果服务 | DIRECT |
| `🛑 广告拦截` | 原广告拦截和应用净化规则 | REJECT |

模板从 39 个策略组减到 20 个。保留了组网、公司地址、交易所、游戏、国内外媒体、Netflix、兜底及常用地区选项。局域网始终直连，组网和公司地址在公共规则前匹配。Google 应用域名在通用代理规则之前匹配，广告规则仍可先拦截广告域名。Google 和其他 AI 组均为 `select`，各自选择具体出口。单条节点链路故障时，需要改选节点。原 `固定出口` 分组改名为 `🔍 谷歌服务`，Google 相关规则只引用新名称。

`GoogleLogin.list` 现在是 Google 域名集合，保留文件名供原规则链接继续引用。Antigravity 的独立更新主机按完整域名匹配。Google 的 IPv4 和 IPv6 使用 `GoogleIP.list`，另保留上游 FCM 和 Google Play 专用 IP。目的 IP 规则使用 `no-resolve` 并放在服务域名规则之后。网段按 Google 官方说明从 `goog.json` 扣除客户使用的 `cloud.json` 得出，更新命令为 `python3 update_google_ips.py`。 [Google 网段说明](https://knowledge.workspace.google.com/admin/security/obtain-google-ip-address-ranges) · [FCM 网络说明](https://firebase.google.com/docs/cloud-messaging/network-configuration) · [Antigravity 网络主机](https://docs.cloud.google.com/docs/security/compliance/deploy-antigravity-us-public-sector)

`OpenAI.list` 与配套的 `OpenAI.yaml` 用于非 Google AI 分流。按服务域名匹配，普通 Bing、整段第三方云厂商 ASN 和通用支付站点沿用正常分流。 [ChatGPT 和 Codex 网络说明](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)

#### OpenClash 导入

- 主入口为 `ACL4SSR_Online_Full_Mannix.ini`。远程 INI 和配套规则链接应指向同一版本，更新订阅前确认发布目标已包含改动。本地验证可将仓库内规则 URL 指向同一个本地 HTTP 服务。订阅转换使用 `target=clash&new_name=true` 生成 Mihomo 支持的字段。移除旧链接中的 `scv=true`，让 TLS 节点沿用自身的证书校验设置。
- 已使用旧版分流的路由器，应检查 OpenClash 的 `chatgpt_inject.rb` 和 `gemini_egress_inject.rb` 覆写，以及自定义规则里对旧策略组的引用。`gemini_egress_inject.rb` 把分组名写死为 `固定出口`，切换到本模板后需改为 `🔍 谷歌服务` 或停用，否则会重新生成旧分组并把登录域名改写回旧出口。它们可能优先匹配并再次拆分 ChatGPT 或 Google 请求。
- 透明代理依赖 DNS 与防火墙将流量交给内核。若 MosDNS 给 Google 静态资源返回了国内 CDN 地址，或内核前的国内 IP 绕过规则直接放行了目标，单靠订阅分流无法接管该连接。
