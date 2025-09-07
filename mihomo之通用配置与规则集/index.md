# Mihomo 之通用配置与规则集


最近陆陆续续改了好几版的配置都不太满意，总是出现一些小问题，最讨厌的就是 ERR_CONNECTION_CLOSED，然后过几秒或者你手动刷新网页就又好了。

同时还会出现魔法软件测速慢、延迟高（相比移动端）等奇奇怪怪的问题。

这次更新一版配置文件来解决这个问题，同时也对分流规则进行了大改，目前使用还没出现什么问题，如果有问题会继续更新。<!--more-->

扫盲篇请看：[Mihomo 使用入门](https://moe.sakanoy.com/mihomo%E4%BD%BF%E7%94%A8%E5%85%A5%E9%97%A8/)

以及推荐一个[指南手册](https://www.haitunt.org/app.html)

## 配置模板

也是我基于 L 站网友修改而来，适用于多机场用户，关于配置，真的就是如果不关注 DNS 泄露，尽可能的精简配置，配置越多越容易出问题；

而关于 DNS 泄露，扫盲可以去看不良林的视频，我的观点是日常使用不需要关注，除非是你做一些敏感的东西，这个玩意是要牺牲一定性能的，大部分人其实没必要。

``` yaml {open=false title="通用配置V2.0"}
subComm: &subComm
  type: http
  udp: true
  interval: 86400
  proxy: DIRECTLY
  lazy: true
  health-check:
    enable: true
    # https://cp.cloudflare.com/generate_204
    # http://latency-test.skk.moe/endpoint
    url: http://www.gstatic.com/generate_204
    interval: 600
    timeout: 5 # 秒
    lazy: true
    expected-status: '204'
    method: HEAD

subAll: &subAll # **所有订阅组引用**
  use:
    - s1
    - s2

subMain: &subMain # **主用订阅组引用**
  use:
    - s1

# **各种代理提供者（机场）的订阅配置**
proxy-providers:
  s1:
    url: ''
    path: ./proxy_provider/s1.yaml
    lazy: true
    <<: *subComm
    override:
      additional-prefix: 's1 »'

  s2:
    url: ''
    path: ./proxy_provider/s2.yaml
    <<: *subComm
    override:
      additional-prefix: 's2 »'

# **=============================== 节点信息 ===============================**
proxies:
  - { name: DIRECTLY, type: direct, udp: true }

# **=============================== DNS 配置 ===============================**
dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  prefer-h3: true # 如果DNS服务器支持DoH3会优先使用h3，提升性能
  respect-rules: true # 让 DNS 解析遵循 Clash 的路由规则
  cache-algorithm: arc # 使用性能更优的 ARC 缓存算法，默认 lru
  cache-size: 2048 # 限制缓存大小，避免占用过多内存

  use-hosts: true # 使用hosts
  use-system-hosts: true # 使用系统hosts

  # 启用 Fake-IP 模式，这是强制劫持所有 DNS 请求的关键。
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  # Fake-IP 过滤器：确保国内域名不被 Fake-IP 转换。
  fake-ip-filter-mode: blacklist
  fake-ip-filter:
    - 'geosite:connectivity-check'
    - 'geosite:private'
    - 'rule-set:fake_ip_filter_DustinWin'

  default-nameserver:
    - 223.5.5.5 # 阿里（国内）
    - 119.29.29.29 # 腾讯（国内）

  # 默认 DNS, 供所有请求使用
  nameserver:
    - https://223.5.5.5/dns-query#h3=true
    - https://dns.alidns.com/dns-query
    - https://223.5.5.5/dns-query
    - https://doh.pub/dns-query
    # - 223.5.5.5
    # - 119.29.29.29

  nameserver-policy:
    'geosite:cn,private': # 国内域名和私有域名强制走国内 DNS
      - https://223.5.5.5/dns-query#h3=true
      - https://dns.alidns.com/dns-query
      - https://223.5.5.5/dns-query
    'geo:cn':
      - https://223.5.5.5/dns-query#h3=true
      - https://dns.alidns.com/dns-query
      - https://223.5.5.5/dns-query

  # 用于代理服务器自身的 DNS 解析
  proxy-server-nameserver:
    - https://223.5.5.5/dns-query
    - 223.5.5.5
    - 119.29.29.29

# **控制面板**
external-controller: 127.0.0.1:9090
secret: '123456.'
external-ui: './ui'
external-ui-url: 'https://github.com/Zephyruso/zashboard/releases/latest/download/dist.zip'

# **=============================== 全局设置 ===============================**
# 影响全局网络和系统的配置
# 设置代理监听的端口、系统参数等
# 控制代理如何与系统交互
port: 7890
socks-port: 7891 # SOCKS5 代理端口
mixed-port: 7892 # HTTP(S) 和 SOCKS 代理混合端口
allow-lan: true # 允许局域网连接
mode: rule
bind-address: '*' # 绑定 IP 地址，仅作用于 allow-lan 为 true，'*'表示所有地址
ipv6: true
unified-delay: true # 更换延迟计算方式，去除握手等额外延迟
tcp-concurrent: true # 启用 TCP 并发连接。这允许 Clash 同时建立多个 TCP 连接，可以提高网络性能和连接速度
log-level: warning # 查东西时改成info
find-process-mode: 'strict' # 设置进程查找模式为严格模式，Clash 会更精确地识别和匹配网络流量来源的进程
global-client-fingerprint: chrome # 设置全局客户端指纹为 Chrome，使 Clash 在建立连接时模拟 Chrome 浏览器的 TLS 指纹，增强隐私性和绕过某些网站的指纹检测
keep-alive-idle: 600
keep-alive-interval: 15
disable-keep-alive: false

profile:
  # 记忆选择
  store-selected: true
  store-fake-ip: true

# **=============================== TUN 配置 ===============================**
# 配置 TUN 模式，适用于透明代理和高效流量转发
tun:
  enable: false
  stack: mixed # system/gvisor/mixed
  auto-route: true
  auto-redirect: true
  auto-detect-interface: true
  strict-route: true # 避免冗余路由污染系统表
  dns-hijack:
    - any:53
    - tcp://any:53
  mtu: 1500 # 减少 MTU 问题，如网页无法打开、测速慢
  gso: true # 启用通用分段卸载（提升性能）
  gso-max-size: 65536
  udp-timeout: 300 # UDP 会话保持时间

sniffer:
  enable: true
  force-dns-mapping: true
  parse-pure-ip: true
  override-destination: true
  sniff:
    HTTP:
      ports:
        - 80
        - 8080-8880
      override-destination: true
    TLS:
      ports:
        - 443
        - 8443
    QUIC:
      ports:
        - 443
        - 8443
  force-domain:
    - +.v2ex.com
  skip-domain:
    - Mijia Cloud
    - +.push.apple.com
  skip-src-address:
    - 192.168.1.1/24
    - 192.168.66.1/24
  skip-dst-address:
    - 91.105.192.0/23
    - 91.108.4.0/22
    - 91.108.8.0/21
    - 91.108.16.0/21
    - 91.108.56.0/22
    - 95.161.64.0/20
    - 149.154.160.0/20
    - 185.76.151.0/24
    - 2001:67c:4e8::/48
    - 2001:b28:f23c::/47
    - 2001:b28:f23f::/48
    - 2a0a:f280:203::/48

# **=============================== GEO 数据库配置 ===============================**
# 该配置用于加载地理位置信息数据库，包括 IP 和域名的地理位置数据
# 在不同的规则配置中，可以使用地理信息进行更精确的流量路由

# GEO 数据库模式配置，启用该功能以获取和使用地理位置信息
geodata-mode: true

# GEO 数据库加载模式，选择合适的内存占用策略
# 'memconservative' 适用于较低内存占用，保证系统性能
geodata-loader: standard

# 是否自动更新 GEO 数据库，默认每48小时更新一次
geo-auto-update: true
# GEO 数据库更新间隔时间，单位为小时
geo-update-interval: 48

# GEO 数据库文件下载地址配置，提供各类 GEO 数据文件的 URL，确保实时更新
geox-url:
  geoip: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.dat'
  geosite: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat'
  mmdb: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.metadb'
  asn: https://github.com/xishang0128/geoip/releases/download/latest/GeoLite2-ASN.mmdb

# **=============================== 代理组设置 ===============================**
# 代理组用于管理不同代理的分类和调度策略
# 注意锚点必须放在引用的上方，可以集中把锚点全部放yaml的顶部。

FilterHK: &FilterHK '(?=.*(🇭🇰|港|HK|(?i)Hong))^((?!(日|坡|狮|美)).)*$'
FilterJP: &FilterJP '(?=.*(🇯🇵|(?<!尼)日|JP|(?i)Japan))^((?!(港|坡|狮|美)).)*$'
FilterSG: &FilterSG '(?=.*(🇸🇬|坡|狮|獅|SG|(?i)Singapore))^((?!(港|日|美)).)*$'
FilterUS: &FilterUS '(?=.*(🇺🇸|美|US|(?i)States|American))^((?!(港|日|坡|狮)).)*$'

UrlTest: &urltest
  type: url-test
  interval: 6
  tolerance: 20
  lazy: true
  # url: http://latency-test.skk.moe/endpoint
  url: http://www.gstatic.com/generate_204
  disable-udp: false
  timeout: 2000
  max-failed-times: 3
  hidden: false
FallBack: &fallback
  type: fallback
  interval: 6
  lazy: true
  url: http://www.gstatic.com/generate_204
  disable-udp: false
  timeout: 2000
  max-failed-times: 3
  hidden: false
LoadBalance:
  type: load-balance
  interval: 6
  lazy: true
  url: http://www.gstatic.com/generate_204
  disable-udp: false
  strategy: consistent-hashing
  timeout: 2000
  max-failed-times: 3
  hidden: false

# **代理组**
proxy-groups:
  - name: Proxy
    type: select
    proxies:
      - 台湾故转
      - 日本故转
      - 香港自动
      - 日本自动
      - 日本节点
      - 全部节点
    icon: 'https://raw.githubusercontent.com/Mithcell-Ma/icon/refs/heads/main/Manual_Test_Log.png'

  # AI 主策略组：手动选择入口
  - name: AI
    type: select
    proxies:
      - 美国节点
    icon: 'https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ai.png'

  - name: 台湾故转
    <<: *subMain
    type: fallback
    interval: 6
    lazy: true
    url: http://www.gstatic.com/generate_204
    disable-udp: false
    timeout: 2000
    max-failed-times: 3
    hidden: false
    filter: '(?i)(台湾|TW)'
    icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png'
  - name: 日本故转
    <<: *subMain
    type: fallback
    interval: 6
    lazy: true
    url: http://www.gstatic.com/generate_204
    disable-udp: false
    timeout: 2000
    max-failed-times: 3
    hidden: false
    filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)'
    icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png'

  - name: 香港自动
    <<: *subMain
    type: url-test
    interval: 6
    tolerance: 20
    lazy: true
    # url: http://latency-test.skk.moe/endpoint
    url: http://www.gstatic.com/generate_204
    disable-udp: false
    timeout: 2000
    max-failed-times: 3
    hidden: false
    filter: "(?i)(香港|hk|🇭🇰|hong\\s?kong)"
    icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/release/icon/color/auto.png'
  - name: 日本自动
    <<: *subMain
    type: url-test
    interval: 6
    tolerance: 20
    lazy: true
    # url: http://latency-test.skk.moe/endpoint
    url: http://www.gstatic.com/generate_204
    disable-udp: false
    timeout: 2000
    max-failed-times: 3
    hidden: false
    filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)'
    icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/release/icon/color/auto.png'

  # 📺 媒体平台
  - {
      name: YouTube,
      hidden: true,
      type: select,
      <<: *subMain,
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/YouTube.png',
    }

  - {
      name: Speedtest,
      hidden: true,
      type: select,
      proxies: [DIRECTLY, Proxy],
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/Speedtest.png',
    }
  - {
      name: Trackerslist,
      hidden: true,
      type: select,
      proxies: [DIRECTLY, Proxy],
      icon: 'https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trackerslist.png',
    }

  # 🌍 区域节点直选（供 UI 手动切换）
  - {
      name: 美国节点,
      type: select,
      <<: *subAll,
      skip-cert-verify: true,
      include-all: true,
      filter: "(?i)(🇺🇸|美国|US|united\\s?states|america|usa|洛杉矶|达拉斯|New\\s?York|西雅图)",
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/United_States.png',
    }
  - {
      name: 日本节点,
      type: select,
      <<: *subAll,
      skip-cert-verify: true,
      include-all: true,
      filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)',
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/Japan.png',
    }

  - {
      name: 全部节点,
      type: select,
      <<: *subAll,
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/World_Map.png',
      include-all: true,
      filter: '(?i)^((?!(DIRECTLY|DIRECT|Proxy|Traffic|Expire|Expired|过期|剩余|流量|官网|超时|timeout|失效|Invalid|Test|测速|本地|Local)).)*$',
    }
  - {
      name: 漏网之鱼,
      type: select,
      proxies: [DIRECTLY, Proxy],
      icon: 'https://testingcf.jsdelivr.net/gh/aihdde/Rules@master/icon/fish.png',
    }

# **=============================== 规则设置 ===============================**
rules:
  # 屏蔽 QUIC
  - AND,(AND,(DST-PORT,443),(NETWORK,UDP)),(NOT,((GEOSITE,cn))),REJECT
  - AND,(AND,(DST-PORT,443),(NETWORK,UDP)),(NOT,((GEOIP,CN))),REJECT
  - DOMAIN-SUFFIX,2022822.xyz,DIRECTLY
  - DOMAIN-SUFFIX,download.nvidia.com,DIRECTLY
  # --- 拒绝/阻止类规则 (最高优先级，直接拦截) ---
  # 这些规则不涉及 DNS 解析，直接拒绝，应放在最前面，以阻止不必要的流量。
  - RULE-SET,AWAvenue_Ads_Rule,REJECT # 秋风广告过滤

  - RULE-SET,telegram_ip,Proxy,no-resolve
  - RULE-SET,Telegram_No_Resolve,Proxy,no-resolve
  - PROCESS-NAME-REGEX,.*telegram.*,Proxy
  - RULE-SET,telegram_domain,Proxy

  - GEOSITE,private,DIRECT
  - GEOSITE,category-public-tracker,DIRECT
  - GEOSITE,category-games@cn,DIRECT
  - GEOSITE,apple@cn,DIRECT
  - GEOSITE,icloud,DIRECT
  - GEOSITE,microsoft@cn,DIRECT
  - GEOSITE,onedrive,DIRECT
  - GEOSITE,microsoft,Proxy
  - GEOSITE,telegram,Proxy
  - GEOSITE,category-ai-!cn,AI
  # ====== IP =======
  - GEOIP,telegram,Proxy,no-resolve
  # - GEOIP,netflix,Netflix,no-resolve
  - GEOIP,private,DIRECT,no-resolve

  - RULE-SET,ai,AI
  - RULE-SET,youtube_domain,YouTube
  - RULE-SET,speedtest_domain,Speedtest

  # Sukka 的规则集
  - RULE-SET,global_non_ip,Proxy
  - RULE-SET,skk_direct,DIRECT
  - RULE-SET,domestic_non_ip,DIRECT
  - RULE-SET,domestic_ip,DIRECT,no-resolve
  - RULE-SET,china_ip,DIRECT,no-resolve
  - RULE-SET,china_ip_ipv6,DIRECT,no-resolve
  # - RULE-SET,cdn_domainset,Proxy
  # - RULE-SET,cdn_non_ip,Proxy

  - RULE-SET,proxy,Proxy
  - RULE-SET,gfw_domain,Proxy

  - DOMAIN-SUFFIX,cn,DIRECT
  - RULE-SET,blackmatrix7_direct,DIRECTLY
  # 苹果服务直连 (macOS)
  # - DOMAIN-SUFFIX,apple.com,DIRECTLY
  # - DOMAIN-SUFFIX,icloud.com,DIRECTLY
  # - DOMAIN-SUFFIX,cdn-apple.com,DIRECTLY # 针对苹果CDN加速
  # - DOMAIN-SUFFIX,ls.apple.com,DIRECTLY # 针对 Apple 定位，直连规则

  # --- Tracker / BT 下载 (特殊流量，优先级较低) ---
  - RULE-SET,trackerslist,Trackerslist

  # --- 最终回退规则 (最低优先级，捕获所有未匹配流量) ---
  - GEOIP,CN,DIRECT
  - MATCH,漏网之鱼

# **=============================== 规则提供者 ===============================**
# **rule引用设置**
Classical: &classical
  type: http
  behavior: classical
  interval: 86400
Domain: &domain
  type: http
  behavior: domain
  interval: 86400
IP: &ip
  type: http
  behavior: ipcidr
  interval: 86400

# **动态加载规则配置，提供实时更新的规则集合**
rule-providers:
  # 广告过滤
  AWAvenue_Ads_Rule:
    {
      <<: *domain,
      format: yaml,
      path: ./ruleset/AWAvenue_Ads_Rule_Clash.yaml,
      url: 'https://raw.githubusercontent.com/TG-Twilight/AWAvenue-Ads-Rule/main//Filters/AWAvenue-Ads-Rule-Clash.yaml',
    }
  # fake-ip 过滤
  fake_ip_filter_DustinWin:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/fake_ip_filter_DustinWin.mrs,
      url: 'https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/fakeip-filter.mrs',
    }

  # skk 规则
  skk_direct:
    <<: *classical
    format: text
    path: ./rule-providers/direct.txt
    url: https://ruleset.skk.moe/Clash/non_ip/direct.txt
  global_non_ip:
    <<: *classical
    format: text
    path: ./rule-providers/global_non_ip.txt
    url: https://ruleset.skk.moe/Clash/non_ip/global.txt
  cdn_domainset:
    <<: *domain
    format: text
    path: ./rule-providers/cdn_domainset.txt
    url: https://ruleset.skk.moe/Clash/domainset/cdn.txt
  cdn_non_ip:
    <<: *classical
    format: text
    path: ./rule-providers/cdn_non_ip.txt
    url: https://ruleset.skk.moe/Clash/non_ip/cdn.txt
  domestic_non_ip:
    <<: *classical
    format: text
    path: ./rule-providers/domestic_non_ip.txt
    url: https://ruleset.skk.moe/Clash/non_ip/domestic.txt
  domestic_ip:
    <<: *classical
    format: text
    path: ./rule-providers/domestic_ip.txt
    url: https://ruleset.skk.moe/Clash/ip/domestic.txt
  china_ip:
    <<: *ip
    format: text
    path: ./rule-providers/china_ip.txt
    url: https://ruleset.skk.moe/Clash/ip/china_ip.txt
  china_ip_ipv6:
    <<: *ip
    format: text
    path: ./rule-providers/china_ip_ipv6.txt
    url: https://ruleset.skk.moe/Clash/ip/china_ip_ipv6.txt
  download_domainset:
    <<: *domain
    format: text
    url: https://ruleset.skk.moe/Clash/domainset/download.txt
    path: ./rule_set/sukkaw_ruleset/download_domainset.txt
  download_non_ip:
    <<: *domain
    format: text
    url: https://ruleset.skk.moe/Clash/non_ip/download.txt
    path: ./rule_set/sukkaw_ruleset/download_non_ip.txt

  # 直连类规则
  blackmatrix7_direct:
    {
      <<: *domain,
      format: yaml,
      path: ./ruleset/blackmatrix7_direct.yaml,
      url: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Direct/Direct.yaml',
    }
  private_domain:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/private_domain.mrs,
      url: 'https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/private.mrs',
    }
  cn_domain:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/cn_domain.mrs,
      url: 'https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/cn.mrs',
    }
  # BT 下载隐私类
  trackerslist:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/trackerslist.mrs,
      url: 'https://github.com/DustinWin/ruleset_geodata/raw/refs/heads/mihomo-ruleset/trackerslist.mrs',
    }

  # 代理类规则
  proxy:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/proxy.mrs,
      url: 'https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/proxy.mrs',
    }
  gfw_domain:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/gfw_domain.mrs,
      url: 'https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/gfw.mrs',
    }
  ai:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/ai,
      url: 'https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ai.mrs',
    }
  youtube_domain:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/youtube_domain.mrs,
      url: 'https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/youtube.mrs',
    }
  # Telegram
  telegram_domain:
    {
      <<: *classical,
      format: yaml,
      path: ./ruleset/telegram_domain.yaml,
      url: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Telegram/Telegram.yaml',
    }
  telegram_ip:
    {
      <<: *ip,
      format: mrs,
      path: ./ruleset/telegram_ip.mrs,
      url: 'https://github.com/DustinWin/ruleset_geodata/raw/refs/heads/mihomo-ruleset/telegramip.mrs',
    }
  Telegram_No_Resolve:
    {
      <<: *classical,
      format: yaml,
      path: ./ruleset/Telegram_No_Resolve.yaml,
      url: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/rule/Clash/Telegram/Telegram_No_Resolve.yaml',
    }
  speedtest_domain:
    {
      <<: *domain,
      format: mrs,
      path: ./ruleset/speedtest_domain.mrs,
      url: 'https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/ookla-speedtest.mrs',
    }
```

mihomo / clash 支持的 ruleset 根据 behavior 不同分为 **domain、ipcidr 和 classical 三种**。

在引入第三方规则时，尽量**避免使用** behavior 为 classical 的规则，尤其是在软路由等资源受限环境中，不要搞特别多的规则。

domain / ipcidr 专门进行域名 / IP 的匹配，在 mihomo 内部是高度优化的；但 classical 规则支持完整的 mihomo 匹配语法，资源消耗更大，大部分规则集会按类型进行拆分以此优化性能。

format 可选 **yaml / text / mrs**，默认 yaml，其中 mrs 是二进制格式，如果有，尽量就选 mrs，但是仅支持 domain / ipcidr。

根据这些分流规则我也手撸了个 Surge 的，相比之下 Surge 就简单多了。

## DNS 与 QUIC

回到文章开始提到的 ERR_CONNECTION_CLOSED 问题，经过我观察日志，可以确定是 QUIC 的问题。

具体关于 QUIC 的相关解释，我单独写了一篇文章：[基于 UDP 的新世界：QUIC、DNS、WebRTC](https://moe.sakanoy.com/%E5%9F%BA%E4%BA%8Eudp%E7%9A%84%E6%96%B0%E4%B8%96%E7%95%8Cquicdnswebrtc/)

在知道了以上原因后，大概也就知道为什么会 ERR_CONNECTION_CLOSED 了，这个确实可自己的网络、魔法机器有很大关系，我换另一家机场就不太容易出现这个问题，所以我猜测这和入口节点对 QUIC 的支持有关系。

并且如果你的 DNS 只配了 DoH3 或者 DoT、DoH，那么有概率也会出问题，例如我的网络环境下，晚高峰就直接没法进行 DNS 解析。

Google 和 Cloudflare 目前对 http/3 支持最积极，所以使用 chrome 访问网页尤其是 cloudflare 代理的网站会触发 http/3 尝试，根据咱们的环境，这大概率会失败，所以我在配置的分流中屏蔽了国外的 QUIC，确实好了很多，基本没看到 ERR_CONNECTION_CLOSED 了。

当然也可以进 `chrome://flags/#enable-quic` 禁用 Chrome 的 QUIC，这样应该也是可以的（如果你在意 DNS 泄露，还需要禁用 WebRTC，或者使用相关插件禁用）。

我的 DNS 配置中，基本都是用了 DoH，如果打开网页速度不理想，可以退回 UDP 试试。

如果你使用 GUI 来管理 Mihomo 内核，那么大概率 DNS、TUN、嗅探这些配置默认是 GUI 接管的，配置中的并不会生效，可以从设置中看看关掉接管，TUN 这种肯定是没法关掉，可以按照配置文件再配置一次。

推荐阅读：[为什么 Surge 进行代理转发时屏蔽了 QUIC 流量](https://kb.nssurge.com/surge-knowledge-base/zh/faq/common-faqs#wei-shen-me-surge-jin-xing-dai-li-zhuan-fa-shi-ping-bi-le-quic-liu-liang)

简单理解就是，TCP 协议和 QUIC 协议都是可靠的传输协议（各种魔法协议的基础），这表示他们在传输数据时，如果发现某数据包丢包，会自动将该数据包重传。

当发生丢包时，在双重可靠传输协议的嵌套下，产生了双倍的重发包，并且由于 QUIC 使用 UDP 的特点，这个速度和数量是非常恐怖的，然后那就非常容易引起 ISP 的 QoS，所以使用魔法最好就禁用 QUIC。
## GeoSite

很多分流规则中会出现 GeoSite、GEOIP，这个我在之前的文章中提到过，实际可以理解是内置的一些常用规则，分别对应域名、IP 规则，还可以按服务进行过滤，这里我找到了一篇介绍 GeoSite 比较详细的文章，感兴趣的可以看看：[Clash 中 GeoSite 分流的正确使用方式](https://www.aloxaf.com/2025/04/how_to_use_geosite/)

对于轻量级用户，实际上只靠 GeoSite 基本就可以满足需求，不需要再加其他规则了。
## 使用裸核

在折腾的过程中，我突然发现还有裸核的打开方式，这个我之前以为裸核只能通过命令来控制，会比较复杂，但是发现现在已经有一些好用的 Web 面板了，使用上完全不是问题，我的配置文件中使用了 zashboard 这个面板，感觉还是非常漂亮好用的。

Mihomo 核心下载可以从[文档](https://wiki.metacubex.one/startup/)这里，或者去官方 Github 也是可以的，这里我仅仅介绍 Windows 下的使用，Linux 也是同理，Linux 可以使用 systemctl 自定义服务控制反而更容易。

``` conf
# 在 /etc/systemd/system 目录创建 mihomo.service 文件
[Unit]
Description=Mihomo Proxy Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/mihomo -f /etc/mihomo/config.yaml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Windows 的话，建议还是通过”任务计划程序“来配置开机启动（另一种方案是使用 WinSW 创建一个服务），然后配合 PowerShell 脚本来实现对核心的控制，TUN 模式是需要管理员权限的。具体配置参考[这篇文章](https://senzyo.net/2023-7/)。

当然你选择使用终端手动来跑也是可以的，命令：
``` sh
# -d 参数代表设置配置文件的目录，-f 参数指定配置文件。
mihomo.exe -d ./conf -f mihomo.yml
```

第一次启动会下载规则集和 GeoSite 文件，如果网络不好可以提前下载好放到配置文件目录中，不使用 `-d` 参数的话，默认目录是用户目录的 `.config/mihomo` 文件夹。

> [!TIP]
> 像 Spark 之类的 GUI 程序，也提供“轻量模式”运行，这个其实就是裸核了，不过省去了你自己配置服务或者敲命令运行。

运行没问题的话，就可以使用配置文件里配置的 external-controller 地址访问面板了。

## 知名规则

GeoIP / GeoSite： [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)

Mihomo 官方维护： [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat/releases/tag/latest)

知名第三方规则：[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule)

可莉：[Loon 模块库](https://github.com/luestr/ProxyResource)

墨鱼：[Quantumult X](https://github.com/ddgksf2013)

**其他备用：**

- [DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata/releases/tag/mihomo-ruleset)

- [SukkaW](https://github.com/SukkaW/Surge/)

- [family](https://whatshub.top/)

再次提醒，规则不在于多，适合自己就好，我自己反而更在意性能，尤其是在软路由这类机器上。

