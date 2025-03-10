# [ShellCrash](https://github.com/juewuy/ShellCrash) 本地配置自定义出站和规则-ruleset 方案
- 注：此方案适用于 [sing-box](https://github.com/SagerNet/sing-box)，采用 `rule_set` 规则
# 前言：
1. 本教程只适用于 ShellCrash
2. 自定义规则参考 [DustinWin/ruleset_geodata/ruleset](https://github.com/DustinWin/ruleset_geodata#%E4%BA%8C-ruleset-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
3. 本教程**仅适合白名单模式**（没有命中规则的网络流量统统使用代理，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户）
4. 本教程最终效果媲美《[生成带有自定义出站和规则的 sing-box 配置文件直链-ruleset 方案](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/sing-box/%E5%9F%BA%E7%A1%80%E7%AF%87/%E7%94%9F%E6%88%90%E5%B8%A6%E6%9C%89%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BA%E7%AB%99%E5%92%8C%E8%A7%84%E5%88%99%E7%9A%84%20sing-box%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE-ruleset%20%E6%96%B9%E6%A1%88.md)》（出站分组更直观，操作更方便），但不依赖于网络
5. 所有步骤完成后，请连接 SSH 执行命令 `$CRASHDIR/start.sh restart` 后生效
6. 推荐使用 [Visual Studio Code](https://code.visualstudio.com/Download) 等专业编辑器来修改配置文件
---
# 一、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box)
可参考《[ShellCrash 配置-ruleset 方案/导入 sing-box PuerNya 版内核](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/sing-box/%E5%9F%BA%E7%A1%80%E7%AF%87/ShellCrash%20%E9%85%8D%E7%BD%AE-ruleset%20%E6%96%B9%E6%A1%88.md#%E4%B8%80-%E5%AF%BC%E5%85%A5-sing-box-puernya-%E7%89%88%E5%86%85%E6%A0%B8)》里的步骤进行操作
# 二、 导入配置文件
1. 进入 ShellCrash->6 导入配置文件->1 在线生成 singbox 配置文件->4 选取在线配置规则模版，选择 4 [ACL4SSR](https://acl4ssr-sub.github.io) 极简版（适合自建节点）  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/70e60a4d-6819-43aa-9f6c-0a0ab4384db6" width="60%"/>

2. 进入 ShellCrash->6 导入配置文件->1 在线生成 singbox 配置文件，输入订阅链接后回车，再输入“1”并回车即可
# 三、 自定义出站和规则
## 1. 自定义代理集合 outbound_providers.json
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/outbound_providers.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：
```
{
  // 代理集合（获取机场订阅链接内的所有节点）
  "outbound_providers": [
    {
      "tag": "🛫 我的机场 1",
      "type": "http",
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m",
      // 机场订阅链接，使用 Clash 链接
      "download_url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport1.yaml",
      "download_ua": "clash.meta",
      "download_interval": "24h",
      "download_detour": "DIRECT",
      // 若机场节点支持 IPv6，可添加此参数
      "override_dialer": { "domain_strategy": "prefer_ipv6" }
    },
    {
      "tag": "🛫 我的机场 2",
      "type": "http",
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m",
      "download_url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport2.yaml",
      "download_ua": "clash.meta",
      "download_interval": "24h",
      "download_detour": "DIRECT",
      "override_dialer": { "domain_strategy": "prefer_ipv6" }
    }
  ]
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
## 2. 自定义出站 outbounds.json
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/outbounds.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：
```
{
  // 出站
  "outbounds": [
    // 手动选择国家或地区节点；根据“国家或地区出站”的名称对 `outbounds` 值进行增删改，须一一对应
    { "tag": "🈯 节点指定", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🆓 免费节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    // 选择`🎯 全球直连`为测试本地网络（运营商网络速度和 IPv6 支持情况），可选择其它节点用于测试机场节点速度和 IPv6 支持情况
    { "tag": "📈 网络测试", "type": "selector", "outbounds": [ "🎯 全球直连", "🇭🇰 香港节点", "🆓 免费节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🔗 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🪜 代理域名", "type": "selector", "outbounds": [ "🈯 节点指定", "🎯 全球直连" ] },
    { "tag": "🎮 游戏平台", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "Ⓜ️ 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "📢 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🇨🇳 国内 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🈯 节点指定" ] },
    { "tag": "🔒 私有网络", "type": "selector", "outbounds": [ "🎯 全球直连" ] },
    { "tag": "🛑 广告拦截", "type": "selector", "outbounds": [ "REJECT" ] },

    // 单个出站节点（以 vless 为例）
    {
      "tag": "🆓 免费节点",
      "type": "vless",
      "server": "example.com",
      "server_port": 443,
      "uuid": "{uuid}",
      "network": "tcp",
      "tls": { "enabled": true, "server_name": "example.com", "insecure": false },
      "transport": { "type": "ws", "path": "/?ed=2048", "headers": { "Host": "example.com" } },
      "domain_strategy": "prefer_ipv6"
    },

    // -------------------- 国家或地区出站 --------------------
    // 自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 100ms 才会切换到延迟低的那个节点；筛选出“香港”节点，支持正则表达式
    { "tag": "🇭🇰 香港节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)港|hk|hongkong|hong kong" ] },
    { "tag": "🇹🇼 台湾节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)台|tw|taiwan" ] },
    { "tag": "🇯🇵 日本节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)日本|jp|japan" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)新|sg|singapore" ] },
    { "tag": "🇺🇸 美国节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)美|us|unitedstates|united states" ] }
  ]
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
## 3. 自定义规则 route.yaml
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：
```
{
  // 路由
  "route": {
    // 规则
    "rules": [
      // 自定义规则优先放前面
      { "rule_set": [ "ads" ], "outbound": "🛑 广告拦截" },
      { "rule_set": [ "private" ], "outbound": "🔒 私有网络" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "Ⓜ️ 微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "🍎 苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "📢 谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "🎮 游戏平台" },
      { "rule_set": [ "networktest" ], "outbound": "📈 网络测速" },
      { "rule_set": [ "proxy" ], "outbound": "🪜 代理域名" },
      { "rule_set": [ "cn" ], "outbound": "🔗 直连域名" },
      { "rule_set": [ "telegramip" ], "outbound": "📲 电报消息" },
      { "rule_set": [ "privateip" ],  "outbound": "🔒 私有网络" },
      { "rule_set": [ "cnip" ], "outbound": "🇨🇳 国内 IP" }
    ],
    // 规则集（binary 文件每天自动更新）
    "rule_set": [
      {
        "tag": "ads",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/ads.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "private",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/private.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "microsoft-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/microsoft-cn.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "apple-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/apple-cn.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "google-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/google-cn.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "games-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/games-cn.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "networktest",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/networktest.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/proxy.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/cn.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "telegramip",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/telegramip.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "privateip",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/privateip.srs",
        "download_detour": "DIRECT"
      },
      {
        "tag": "cnip",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/cnip.srs",
        "download_detour": "DIRECT"
      }
    ]
  }
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车  
**贴一张面板效果图（举个例子：我手动选择 `🇹🇼 台湾节点` 出站，而该出站是将机场内所有台湾节点按照 url 测试结果自动选择延迟最低的台湾节点）：**  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/e04a650c-5c88-4852-bbaa-38cc85ec5036" width="60%"/>

## 4. 自定义 DNS dns.json
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：
```
{
  // DNS
  "dns": {
    // DNS 服务器
    "servers": [
      // 广告 DNS
      { "tag": "dns_block", "address": "rcode://success" },
      // 国内 DNS
      { "tag": "dns_direct", "address": "h3://dns.alidns.com/dns-query", "address_resolver": "dns_ip", "detour": "DIRECT" },
      // IP 格式的 DNS
      { "tag": "dns_ip", "address": "https://223.5.5.5/dns-query", "detour": "DIRECT" },
      // FakeIP
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    // DNS 规则
    "rules": [
      { "outbound": "any", "server": "dns_ip" },
      { "clash_mode": "Direct", "server": "dns_direct" },
      { "clash_mode": "Global", "server": "dns_fakeip", "rewrite_ttl": 1 },
      // `rule_set` 规则集中必须包含 `ads` 规则
      { "rule_set": [ "ads" ], "server": "dns_block" },
      // `rule_set` 规则集中必须包含以下规则
      { "rule_set": [ "microsoft-cn", "apple-cn", "google-cn", "games-cn", "cn", "private" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      // `rule_set` 规则集中必须包含 `proxy` 规则
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 }
    ],
    // 默认 DNS 服务器，即上述 DNS 规则外的域名使用该 DNS 解析
    "final": "dns_direct",
    //本地网络有 IPv6 时可配置为 `prefer_ipv6`
    "strategy": "prefer_ipv6",
    "independent_cache": true,
    "reverse_mapping": true,
    "fakeip": { "enabled": true, "inet4_range": "198.18.0.0/15", "inet6_range": "fc00::/18" }
  }
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
# 四、 修改出站或规则
**举例：我想添加一个规则，使奈飞走日本和新加坡节点**  
① 进入 [MetaCubeX/meta-rules-dat/sing/bm7](https://github.com/MetaCubeX/meta-rules-dat/tree/sing/bm7) 后按 Ctrl+F 组合键搜索“netflix”  
② 可以精确搜索到“netflix”  
③ 选择 .srs 文件，然后点击“Raw”获取下载地址，并将下载地址[转换为 CDN 链接](https://www.jsdelivr.com/github)，那么就可以这样编写：
注：
- 1. **一定要保证缩进对齐！一定要保证缩进对齐！一定要保证缩进对齐！**
- 2. 以下只是节选，请酌情套用

## 1. 修改 outbounds.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/outbounds.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：
```
{
  // 出站
  "outbounds": [
    // 打开奈飞后自动选择延迟最低的日本或新加坡节点；容差大于 100ms 才会切换到延迟低的那个节点
    { "tag": "🎥 奈飞视频", "type": "urltest", "tolerance": 100, "providers": [ "🛫 我的机场 1", "🛫 我的机场 2" ], "includes": [ "(?i)日本|jp|japan|新|sg|singapore" ] }
  ]
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
## 2. 修改 route.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），优先在最上方编辑如下内容并粘贴：
```
{
  // 路由
  "route": {
    // 规则
    "rules": [
      // 自定义规则优先放前面
      { "rule_set": [ "netflix" ], "outbound": "🎥 奈飞视频" }
    ],
    // 规则集（binary 文件每天自动更新）
    "rule_set": [
      {
        "tag": "netflix",
        "type": "remote",
        "format": "binary",
        "url": "https://cdn.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@sing/bm7/Netflix.srs",
        "download_detour": "DIRECT"
      }
    ]
  }
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
# 五、 添加小规则
仅添加特定网址走直连或走代理，连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），在**最上方**粘贴如下内容：  
注：
- 1. 以下内容只是举例，请根据自身需要进行增删改
- 2. 其它规则请参考《[sing-box Wiki](https://sing-box.sagernet.org/zh/configuration/route/rule)》

```
{
  // 路由
  "route": {
    // 规则
    "rules": [
      // 以 googleapis.cn 为后缀的所有域名走代理
      { "domain_suffix": [ "googleapis.cn" ], "outbound": "🈯 节点指定" },
      // 与哔哩哔哩相关的所有域名走直连
      { "geosite": [ "bilibili" ], "outbound": "DIRECT" },
      // 含有 ipv6 关键字的所有域名走直连
      { "domain_keyword": [ "ipv6" ], "outbound": "DIRECT" }
    ]
  }
}
```
按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
