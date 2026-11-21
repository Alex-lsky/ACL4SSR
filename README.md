## ACL4SSR_Online_Full_Mannix.ini

自定义 Clash 配置模板 https://dd.al/1phZx

可作为 订阅转换 配置转换 规则转换 的远程配置

修改自 https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini

---

### V2

自带旗帜 emoji 添加逻辑，原名不包含旗帜 emoji 才添加，原名已包含旗帜 emoji 则不添加

**需去除订阅转换链接中的参数 `emoji=true/false` 才能生效**，参考例子：

`https://api.dler.io/sub?target=clash&udp=true&scv=true&config=https://dd.al/config&url={原订阅链接}`

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

Google OAuth 要求认证链路的出口 IP 全程一致，否则风控判定会话被劫持 → 拒绝登录。

**改动**：

1. `固定出口` 由 `url-test` 改为 `select` —— `select` 不会被 Smart 化，手工钉一个节点后 IP 全程一致。
   这是关键：只要还是 `url-test`，无论规则怎么配，节点都会被内核逐连接重选。
2. 新增 `GoogleLogin.list`，把登录全链路域名（认证 / token / 用户信息 / Gemini 后端 /
   静态资源）统一收敛到 `固定出口`，并置于 `ProxyGFWlist` **之前**，
   避免被 `DOMAIN-KEYWORD,google` 抢先匹配到 `🚀 节点选择`。

**使用**：订阅更新后在策略组里把 `固定出口` 手工选成一个固定节点
（推荐美/日/欧的家庭宽带或直连线路，香港出口对 Gemini 一律 403），
之后不要再切换到「自动选择」类分组。

