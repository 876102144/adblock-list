# BlockAds 拦截日志域名整理（2026-11-08）

**数据源**：`blockads_logs_1794143581751.csv`，45 小时（11-07 00:08 ~ 11-08 21:10），6175 条拦截记录，168 个唯一域名，全部 `Blocked=true`。

**核心结论**：广告拦得没错，但规则误伤了小米系统服务、网络定位和微信登录；同时 App 开了 VPN + Root 全局代理两层拦截，进程一挂就把整机流量黑洞 —— 这才是 WiFi “连不上/假连”的根因。

## v3 更新（2026-11-09 20:50）

**新数据**：12.99 万条记录（去重 6.17 万），5,600 个域名，59 个 App，20.5 小时。

**新结论**：视频被"屏蔽"不是内容 CDN 被拦，而是**播放调度与接入网关被拦**：

| 域名族 | 总请求 | 被拦 | 拦截率 | 实际身份 |
|---|---|---|---|---|
| `tnc*.zijieapi.com` / `tnc*.bytedance.com` | 562 | 556 | **99.0%** | 字节流量调度（TNC） |
| `frontier*.snssdk.com` / `frontier*.fqnovel.com` | 335 | 260 | **77.6%** | 字节统一接入网关 |
| `vcs.zijieapi.com` | — | 81 | — | 配置/版本服务 |
| `qznovelvod.com`（真正视频流） | 3,700 | **0** | 0% | 内容 CDN |
| `douyinpic.com` | 15,026 | **0** | 0% | 图片 CDN |

**v3 改动**：删除 17 条误拦（tnc/frontier/vcs/polaris5/saveu5/ndcpp/ad-sign/i-lq/webcast-hl），新增 35 条关键白名单；拦截 105 条，放行 57 条。

**DNS 核查**：`:853`、`100.64.100.1`、`fd00::1`、`dns.weixin.qq.com.cn` 全部 0 拦截 → **WiFi 断网与规则无关，是拦截器 VPN + Root 全局代理叠加导致**，需在 App 内关掉一层。

---

## 一、分类统计（2026-11-08 首次分析）

| 分类 | 域名数 | 拦截次数 | 处理 |
|---|---|---|---|
| 必须放行 | 20 | 1174（19.0%） | 加白名单 |
| 可选（看你要不要推送/画质） | 65 | 634（10.3%） | 自行决定 |
| 继续拦截 | 83 | 4367（70.7%） | 纯广告/埋点 |

---

## 二、必须放行（20 个）

| 域名 | 次数 | 归属 | 为什么不能拦 |
|---|---|---|---|
| `n98-v-readingvideo.qznovelvod.com` | 316 | 红果免费短剧 | 视频内容 CDN，拦了不能播 |
| `v5-gzb2-gddgtc-reading-video.qznovelvod.com` | 288 | 红果免费短剧 | 同上 |
| `v5-gzb-jltc-reading-video.qznovelvod.com` | 66 | 红果免费短剧 | 同上 |
| `metok.sys.miui.com` | 140 | **网络位置服务** | 小米设备认证/MiPush token 注册 |
| `cgicol.amap.com` | 121 | **网络位置服务** | 高德网络定位 |
| `apd-pcdnwxlogin.teg.tencent-cloud.net` | 102 | **微信** | 微信登录 CDN，拦了登录失败/反复重登 |
| `snowflake.qq.com` | 52 | QQ/输入法 | 腾讯设备指纹与风控，拦了可能触发验证 |
| `api.sec.miui.com` | 23 | 系统界面/权限管理 | 小米安全与权限校验 |
| `loc.map.baidu.com` | 22 | SystemHelper | 百度网络定位，**每小时整点心跳一直被拦** |
| `mcc.inf.miui.com` | 16 | 系统界面/相机 | MIUI 系统配置服务 |
| `id6.me` | 5 | 抖音/红果 | 本机号码一键登录 |
| `logupdate.avlyun.sec.miui.com` | 4 | 系统安全组件 | 安全引擎病毒库更新 |
| `data.sec.miui.com` | 4 | 系统界面/小爱 | 小米安全组件通道 |
| `baidu.com` / `www.baidu.com` | 5 | 系统界面 | 主域，拦了波及面大 |
| `auth.be.sec.miui.com` | 2 | 权限管理服务 | 小米安全认证 |
| `adv.sec.miui.com` | 2 | 搜狗输入法 | 安全模块（不是广告） |
| `r.browser.miui.com` | 2 | 小米浏览器 | 浏览器资源 |
| `api.map.baidu.com` | 2 | — | 百度地图 API |
| `mshield.baidu.com` | 2 | — | 百度安全组件 |

> ⚠️ `qznovelvod.com` 三个是**判断项**：从路径（reading-video）和归属 App 看是内容 CDN，建议先放行；如果放行后发现该 App 反而开始放贴片广告，再单独拦回。

---

## 三、可选（65 个，自行取舍）

- **PCDN 加速类**（`*.ndcpp.com`、`comfylink.com` 的 `cf-gl/ipip-gl1/sd-gl/seeds-open`）：拦了省上行带宽，但视频可能变慢。
- **个推推送类**（`*.getui.com`、`*.getui.net`，含一堆 `cm-10-*` / `cmyd-10-*`）：拦了收不到推送（米家、今日水印相机等）。
- `puncher-ipv6.dcdn.baidu.com`：百度 CDN，拦了偶发资源加载慢。
- `sdktt.data.comfylink.com`：SDK 遥测，可拦，但和 PCDN 同域。

---

## 四、继续拦截（83 个，占 70.7%）

拦截量最大的都是真广告/埋点，可以放心留黑：

- 字节系：`dig.bdurl.net`(487)、`*.zijieapi.com`(约 2000+)、`p3-ad-sign.byteimg.com`(450)、`*.bytedance.com`(241)、`*.snssdk.com`、`*.fqnovel.com`、穿山甲 `*.pangolin-sdk-toutiao*.com`
- 腾讯系：`aeventlog.beacon.qq.com`(278)、`*.gdt.qq.com`（广点通）、`rmonitor.qq.com`、`h.trace.qq.com`
- 小米广告/遥测：`api.ad.xiaomi.com`(58)、`tracking.miui.com`(90)、`om.tracking.miui.com`(32)、`tracker.xiaomixiaoai.com`(48)
- 百度广告：`mobads.baidu.com`、`cpro.baidustatic.com`、`afdconf.baidu.com`
- 第三方监测：`*.statisticslinks.com`、`*.metricslinks.com`、`*.fancydsp.com`、`*.miaozhen.com`、`*.umeng.com`、`*.sensorsdata.cn`
- 随机域名广告 CDN：`*.jspcdn.cn`、`*.qrstuvwxyzab.com`（含 IPv6 形式）

---

## 五、导入后怎么验证

1. 导入 `白名单_必须放行.txt`（纯域名，一行一个，通用格式）。
2. **关掉 Root/全局代理，只保留 DNS 过滤一层。**
3. 重启手机，让 VPN/tun 与 iptables 规则彻底重建。
4. 观察 24 小时：
   - 微信不再掉登录
   - 红果短剧能正常播放
   - WiFi 不再出现“已连接但无法上网”
5. 若 WiFi 仍偶发，记录当时的 `BlockAds` 状态（是否 VPN 已断但图标还在），基本可确认是 VPN 层残留。

---

## 六、附带建议

- 取消 BlockAds 的**开机自启**，避免它在 WiFi 起来前抢占 VPN 槽。
- 不要把 DNS 改成 `127.0.0.1`；如已改，恢复“自动”或系统私有 DNS。
- 日志重复率：5615/6175 条是重复记录，且出现 `RootProxy` 条目 —— 说明**同时开了两层拦截**，这是最大的稳定性隐患。


---

## 七、订阅链接复核（2026-11-08 21:25 拉取）

**订阅地址**（2026-10-10 记录，国内镜像）：
`https://cdn.jsdelivr.net/gh/abclq/home-dns-adblock@main/dist/adguard-dns.txt`
添加位置：BlockAds →「过滤器」→「自定义」→「添加」（填名称 + URL）→ 保存并打开开关。

**现状**：已从 10-10 的 205/206 条缩到 **75 条**；当时的两条致命规则
`p3-novel-sign.byteimg.com`、`api5-normal-sinfonlinea.fqnovel.com` **已删除**（作者修了）。

**但仍有 3 处会出事**：

| 订阅规则 | 问题 | 本次日志证据 |
|---|---|---|
| `||apd-pcdnwxlogin.teg.tencent-cloud.net^` | 微信登录 CDN | 被拦 **102 次** |
| `||dns.alidns.com^` | 阿里 DNS 服务器本身 | 若手机私人 DNS 填了它 → DNS 全灭 |
| `||doh.pub^` | 腾讯 DoH 服务器本身 | 同上，DoH 被掐 |
| `||ndcpp.com^`、`||comfylink.com^` | PCDN 视频加速整域 | 红果/抖音可能变慢或播不动 |

> BlockAds 的 dex 里存在 `blockDoT` / `blockDohBypass` 开关，本身就会阻断 DoT/DoH。
> 订阅再拦 `dns.alidns.com` / `doh.pub`，等于把私人 DNS 的最后退路也堵死 ——
> 这是「WiFi 已连接但无法上网」最合理的机制。

**结论**：订阅链接保留，不用删；用 `白名单_adguard语法.txt`（22 条 `@@||…^`）覆盖即可。


---

## 八、v2 扩充（2026-11-08 22:05）

在 72 条基础上补入 **50 条**，总数 **122 条**：
- 旧日志里被内置列表拦过、但自建列表没覆盖的广告/埋点（`*.gdt.qq.com`、`*.beacon.qq.com`、`tracking.miui.com`、`api.ad.xiaomi.com`、`*.umeng.com`、`*.sensorsdata.cn`、`*.statisticslinks.com` 等）
- 新日志里发现的 `-lq` 变体埋点：`polaris5-normal-lq.zijieapi.com`、`log5-applog-lq.fqnovel.com`、`mon11-misc-lq.fqnovel.com`、`saveu5-normal-lq.zijieapi.com`、`i-lq.snssdk.com`、`frontier*-lq.fqnovel.com`

**刻意没有拦的**（怕破坏功能）：`imapi-oth.zijieapi.com`、`api5-normal-lq.fqnovel.com`、`*.ecombdapi.com`、`p*-novel-sign.byteimg.com`、`lf3-reading.fqnovelpic.com`、`*.mijia.tech`、`*.iot.mi.com`、`*.getui.*`（推送）。
