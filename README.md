# adblock-list

BlockAds（小米 15 / HyperOS）自定义规则订阅。基于 `abclq/home-dns-adblock` 的 AdGuard DNS 列表修正而来。

## 订阅地址（BlockAds → 过滤器 → 自定义 → 添加）

```
https://cdn.jsdelivr.net/gh/876102144/adblock-list@main/adguard-dns-custom.txt
```

## 白名单（BlockAds → 设置 → 自定义规则 / 用户规则）

```
https://cdn.jsdelivr.net/gh/876102144/adblock-list@main/whitelist-adguard.txt
```

> 白名单必须放在「用户规则」，优先级高于过滤器；不要当普通订阅列表用。

## 文件说明

| 文件 | 条数 | 用途 |
|---|---|---|
| `adguard-dns-custom.txt` | 72 | 广告/埋点拦截（订阅用） |
| `whitelist-adguard.txt` | 22 | `@@` 放行规则（用户规则用） |

## 相对上游的改动

- 删除 `||apd-pcdnwxlogin.teg.tencent-cloud.net^` —— 微信登录 CDN，误伤会导致反复重登
- 删除 `||dns.alidns.com^`、`||doh.pub^` —— DNS 服务器自身，叠加 BlockAds 的 `blockDoT` 后会导致断网
- `||comfylink.com^` 收窄为 `||sdktt.data.comfylink.com^` —— 保留埋点拦截，放行 PCDN 以免视频卡顿

## 白名单覆盖的关键域名

系统服务：`metok.sys.miui.com`、`api.sec.miui.com`、`data.sec.miui.com`、`auth.be.sec.miui.com`、`mcc.inf.miui.com`、`logupdate.avlyun.sec.miui.com`

网络定位：`cgicol.amap.com`、`loc.map.baidu.com`、`api.map.baidu.com`

登录/风控：`apd-pcdnwxlogin.teg.tencent-cloud.net`、`snowflake.qq.com`、`id6.me`

内容 CDN：`*.qznovelvod.com`（红果正片，非广告）

DNS：`dns.alidns.com`、`doh.pub`

## 更新后刷新 CDN 缓存

jsDelivr 对 `@main` 有缓存，改完访问一次即可立即生效：

```
https://purge.jsdelivr.net/gh/876102144/adblock-list@main/adguard-dns-custom.txt
```

## 注意

- 仓库必须保持 **public**，jsDelivr 不读私有库
- GitHub raw 域名在国内不稳定，务必使用上面的 `cdn.jsdelivr.net` 地址
