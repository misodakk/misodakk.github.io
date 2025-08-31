# Mihomo使用入门

<!--more-->
{{< admonition type=info title="更新记录" open=true >}}

**2025-08-24：**

经过一段时间的使用，发现这套分流配置还是有点问题，于是又魔改了一版，之前的配置是从 Clash 继承而来的，现在 Clash 已经没了，Mihomo （ClashMeta）目前还在维护，虽说 Mihomo 可以兼容 Clash，这次更新一版 Mihomo 专有的配置文件吧。

**2025-07-26：**

截止目前，距离我首次写这篇文章的时候，环境已经发生了比较大的变化，对部分内容进行了重写，原版可以去看[旧博客](https://sakanoy.com/2021/06/23/ClashX%E4%BD%BF%E7%94%A8%E5%85%A5%E9%97%A8/)。

由于 clash for windows（CFW） 这个 GUI 的作者退坑，不明真相的吃瓜群众都以为是内核 Clash 跑路了，由于人实在是太多，引起了比较大的影响，这种软件主打就是闷声发大财，动静太大作者可不想被喝茶。

紧接着一系列 Clash 系软件删库或者归档，包括内核 Clash、ClashX、ClashForAndroid、clash-verge、ClashMeta 等等。

目前有一些项目进行了复活，例如 ClashMeta 转生 Mihomo，是个二次开发的内核（为了不再次发生上次的情况，Mihomo 要求下游 GUI 软件名字不得使用 Mihomo）；

目前基于 Mihomo 也有一些不错的项目，例如 Spark、Clash Verge Rev、FlClash、clash nyanpasu、MihomoParty（还未改名，原作者已弃坑，暂不推荐）
{{< /admonition >}}

首先说明一点，我认为目前对于小白用户，尤其是机场用户，Clash / Mihomo 仍然是最佳选择，上手简单、分流优秀。只要理解了分流、Tun、FakeIP 使用上不是问题。

{{< admonition warning "警告" >}}
以我个人的看法，毕竟 Clash 已经没了，Clash 系今后的发展并不明朗，有能力的朋友还是尽量转换到一些新流行的内核项目，例如 Sing-box，但是目前问题比较多，也没有特别成熟的 GUI，我目前当作备用，GUI 的备用可以选择 v2rayN。
{{< /admonition >}}

Mac 用户有更好的选择，例如 Surge 或者 Loon、Stash 等，囊中羞涩也可以用学习版嘛~

软路由用户除了 OpenClash 也还有 passwall、homeproxy 可选。

## 机场节点

最近进行了一场大规模的中转入口打击，并且还没有停止的意思，其中打击方式有很多骚操作，各位自行了解，不确定之后会不会无中转可用，悲。

- 直连

  您的设备直接传输数据到国外的代理服务器，路线图可以表示为 **用户 ----- 代理服务器**

- 普通中转

  从您的设备传输数据到国内的数据中心，再由数据中心发送至国外的代理服务器，线图可以表示为 **用户 ----- 国内中转机 ------ 代理服务器**

- 隧道中转

  从您的设备传输数据到国内的数据中心，再由数据中心传输至国外的数据中心，国外的数据中心再将数据发送至代理服务器。 线路图可以表示为 **用户 ---- 国内中转机 ----- 国外中转机 ---- 代理服务器**

  这个应该是目前主流的方式

- 专线

  目前常见的专线有 IPLC 和 IEPL，这种线路一般服务于大型的跨国企业，银行等，特点是延迟低，稳定性高，不受防火墙的审查。缺点是价格特别贵。大部分其实都是噱头，实际并不是。

目前稍微好一些的基本都是全中转，所以很多软件都出了入口、落地节点查询，中转下单靠 TCP 测延迟并没有那么有效果，url-test 才能反应实际连接情况。

所以也就理解为什么中转都是 SS 协议，这个协议抗干扰很弱，但是速度非常快，损耗低，用在中转的环境很合适，甚至可以作为游戏节点，如果你订阅到的协议不是 SS 是 VMess 之类，那大概率就是直连，成本很低。

## 配置模板

大部分机场都提供一份默认配置文件，包括一些基本的分流规则，但是我推荐自己写一份，更适合自己。

Mihomo 可以兼容 Clash 的配置，但是 Mihomo 支持一些特殊的语法，下面我参考了 L 站网友的配置，适用于机场用户：

```yaml
proComm: &proComm
  type: http
  udp: true
  interval: 86400
  proxy: DIRECTLY
  lazy: true
  health-check:
    enable: true
    url: https://cp.cloudflare.com/generate_204
    interval: 600
    timeout: 5 # 秒
    lazy: true
    expected-status: "204"
    method: HEAD
  # ⚠️ smux 多路复用：部分机场不支持，如连接失败请注释本段
  # 稳定性出现问题，可以尝试切换为 smux4 或禁用 padding。
  smux: # smux（连接复用，提升多路复用效率，减少握手）
    enabled: true # 开启 smux 多路复用
    padding: true # 增加随机数据，防探测（推荐开）
    protocol: smux # 或 smux4

u: &u # **所有订阅组引用**
  use:
    - 1.p1
    - 2.p2

u_s: &u_s # **主用订阅组引用**
  use:
    - 1.p1

# **各种代理提供者（机场）的订阅配置**
proxy-providers:
  1.p1:
    url: ""
    path: ./proxy_provider/p1.yaml
    lazy: true
    <<: *proComm
    override:
      additional-prefix: "p1 »"

  2.p2:
    url: ""
    path: ./proxy_provider/p2.yaml
    <<: *proComm
    override:
      additional-prefix: "p2 »"


# **=============================== 节点信息 ===============================**
proxies:
  - { name: DIRECTLY, type: direct, udp: true }

# **=============================== DNS 配置 ===============================**
# DNS 解析配置，决定了域名解析方式和缓存策略
dns:
  enable: true # 启用 DNS 功能
  ipv6: true 
  listen: 0.0.0.0:1053 # 监听地址和端口
  prefer-h3: false     # 如果DNS服务器支持DoH3会优先使用h3，提升性能
  respect-rules: true  # 让 DNS 解析遵循 Clash 的路由规则
  cache-algorithm: arc # 使用性能更优的 ARC 缓存算法
  cache-size: 2048     # 限制缓存大小，避免占用过多内存

  use-hosts: false        # 使用hosts
  use-system-hosts: false # 使用系统hosts
  
  # 启用 Fake-IP 模式，这是强制劫持所有 DNS 请求的关键。
  enhanced-mode: fake-ip       # 设置增强模式为 fake-ip 模式，提高解析速度和连接性能
  fake-ip-range: 198.18.0.1/16 # fake-ip 地址范围
  # Fake-IP 过滤器：确保国内域名不被 Fake-IP 转换。
  fake-ip-filter-mode: blacklist
  fake-ip-filter:
    - "rule-set:private_domain,cn_domain"
    - "geosite:connectivity-check"
    - "geosite:private"
    - "rule-set:fake_ip_filter_DustinWin"

  default-nameserver:
    - 1.1.1.1                    # Cloudflare Public DNS (UDP)
    - 8.8.8.8                    # Google Public DNS (UDP)
#    - 223.5.5.5                 # 阿里（国内）
#    - 119.29.29.29              # 腾讯（国内）
#    - system # 系统 DNS (保留以防万一)

  #`nameserver-policy` 精准分流与严格兜底。**
  # 确保国内域名走国内 DNS，境外域名走境外 DNS。这是解决问题的关键。
  # 这是 Clash 进行主要 DNS 查询时使用的服务器列表。
  nameserver: # 默认 DNS，供所有请求使用，支持 DoH3 的在前面
    - https://1.1.1.1/dns-query    # Cloudflare（支持 H3）
    - https://dns.google/dns-query # Google（支持 H3）
    - 1.1.1.1                      # Cloudflare Public DNS (UDP)
    - 8.8.8.8                      # Google Public DNS (UDP) 
#    - https://dns.alidns.com/dns-query # 阿里（国内稳定）
#    - https://doh.pub/dns-query # 腾讯 (境内，DoH，可作为备选)

  nameserver-policy:
    "geosite:cn,private": # 国内域名和私有域名强制走国内 DNS
      - https://223.5.5.5/dns-query # 阿里
      - https://doh.pub/dns-query   # 腾讯
      - 223.5.5.5                   # 阿里 UDP
      - 119.29.29.29                # 腾讯 UDP
    "geo:cn":                       # 也可以用 geo:cn 匹配 IP
      - https://223.5.5.5/dns-query
      - https://doh.pub/dns-query
      - 223.5.5.5                   # 阿里 UDP
      - 119.29.29.29                # 腾讯 UDP
    "geosite:gfw":                  # 新增：GFW 列表域名强制走国外 DNS
      - https://1.1.1.1/dns-query
      - https://dns.google/dns-query
      - 1.1.1.1
      - 8.8.8.8
    "geosite:geolocation-!cn":      # 新增：非中国大陆域名强制走国外 DNS
      - https://1.1.1.1/dns-query
      - https://dns.google/dns-query
      - 1.1.1.1
      - 8.8.8.8
    "full-nameserver":              # 新增：最终兜底，所有未匹配的域名查询强制走国外 DNS
      - https://1.1.1.1/dns-query
      - https://dns.google/dns-query
      - 1.1.1.1
      - 8.8.8.8

  # 当 `nameserver` 中的 DNS 服务器解析失败时，Clash 会尝试这里的 DNS。
  fallback:
    - 1.1.1.1 # Cloudflare DNS备用
    - 8.8.8.8 # Google DNS备用

  # 用于代理服务器自身的 DNS 解析，仅包含国外 DNS。
  proxy-server-nameserver:          # 当请求通过代理（即国外站）时使用
    - https://1.1.1.1/dns-query     # Cloudflare，DoH3
    - https://dns.google/dns-query  # Google，DoH3
    - 1.1.1.1
    - 8.8.8.8

# **控制面板**
external-controller: 127.0.0.1:9090
secret: "123465."
external-ui: "./ui"
external-ui-url: "https://github.com/Zephyruso/zashboard/releases/latest/download/dist.zip"

# **=============================== 全局设置 ===============================**
# 影响全局网络和系统的配置
# 设置代理监听的端口、系统参数等
# 控制代理如何与系统交互
port: 7890
socks-port: 7891 # SOCKS5 代理端口
redir-port: 7892 # 透明代理端口，用于 Linux 和 MacOS
mixed-port: 7893 # HTTP(S) 和 SOCKS 代理混合端口
tproxy-port: 7894
allow-lan: true  # 允许局域网连接
mode: rule
bind-address: "*" # 绑定 IP 地址，仅作用于 allow-lan 为 true，'*'表示所有地址
ipv6: true
unified-delay: true  # 更换延迟计算方式，去除握手等额外延迟
tcp-concurrent: true # 启用 TCP 并发连接。这允许 Clash 同时建立多个 TCP 连接，可以提高网络性能和连接速度
log-level: warning   # 查东西时改成info
find-process-mode: "strict"       # 设置进程查找模式为严格模式，Clash 会更精确地识别和匹配网络流量来源的进程
global-client-fingerprint: chrome # 设置全局客户端指纹为 Chrome，使 Clash 在建立连接时模拟 Chrome 浏览器的 TLS 指纹，增强隐私性和绕过某些网站的指纹检测
keep-alive-idle: 600
keep-alive-interval: 15
disable-keep-alive: false

profile:
  store-selected: true # 记忆选择
  store-fake-ip: true

# **=============================== 流量嗅探配置 ===============================**
# 启用流量嗅探功能，用于监控和分析网络流量
sniffer:
  enable: true
  sniff:
    HTTP:
      ports: [80, 8080-8880]
      override-destination: true
    TLS:
      ports: [443, 8443]
    QUIC:
      ports: [443, 8443]
  skip-domain:
    - "+.baidu.com"
    - "+.bilibili.com"

# **=============================== TUN 配置 ===============================**
# 配置 TUN 模式，适用于透明代理和高效流量转发
tun:
  enable: true
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

# **=============================== GEO 数据库配置 ===============================**
# 该配置用于加载地理位置信息数据库，包括 IP 和域名的地理位置数据
# 在不同的规则配置中，可以使用地理信息进行更精确的流量路由

# GEO 数据库模式配置，启用该功能以获取和使用地理位置信息
geodata-mode: true

# GEO 数据库加载模式，选择合适的内存占用策略
# 'memconservative' 适用于较低内存占用，保证系统性能
geodata-loader: memconservative

# 是否自动更新 GEO 数据库，默认每48小时更新一次
geo-auto-update: true
# GEO 数据库更新间隔时间，单位为小时
geo-update-interval: 48

# GEO 数据库文件下载地址配置，提供各类 GEO 数据文件的 URL，确保实时更新
geox-url:
  geoip: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.dat"
  geosite: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat"
  mmdb: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.metadb"


# **=============================== 代理组设置 ===============================**
# 代理组用于管理不同代理的分类和调度策略
# 注意锚点必须放在引用的上方，可以集中把锚点全部放yaml的顶部。

# **整体调用模版**
pg: &pg
  type: select
  proxies:
    - Proxy
    - 香港故转
    - 台湾故转
    - 新加坡故转
    - 日本节点
    - 全部节点
    - DIRECTLY

# **代理组**
proxy-groups:
  - name: Proxy
    type: select
    proxies:
      - 台湾故转
      - 日本故转
      - 香港故转
      - 新加坡故转
      - 香港自动
      - 日本自动
      - 新加坡自动
      - 自动选择
      - 台湾节点
      - 美国节点
      - 日本节点
      - 全部节点
    icon: 'https://raw.githubusercontent.com/Mithcell-Ma/icon/refs/heads/main/Manual_Test_Log.png'

  # AI 主策略组：手动选择入口
  - name: AI
    type: select
    proxies:
      - AI_稳定节点
      - AI_自动优选
    icon: 'https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ai.png'

  # --- AI_稳定节点：针对 AI 服务优化，优先保持IP稳定 ---
  - name: AI_稳定节点
    # fallback：故障转移，优先使用列表中的第一个可用节点
    type: select
    <<: *u_s
    url: https://cp.cloudflare.com/generate_204
    interval: 7200 # 测速间隔：2 小时 (7200 秒)，降低节点切换频率，有助于保持IP稳定性
    strategy: consistent-hashing # 一致性哈希,减少IP波动。
    max-failed-times: 1 # 失效阈值：节点测试失败 1 次即认为失效，切换到下一个节点
    tolerance: 100 # RTT 容忍度：减少因网络微小波动导致的频繁切换
    lazy: true
    # 节点筛选器，使用 filter 精确筛选出您想要的 AI 节点，包括其前缀
    # 格式：使用您客户端中显示的完整节点名称，包括 "p2 »" 前缀
    filter: '(?i)(🇺🇸|美国|台湾|TW)'
    icon: 'https://testingcf.jsdelivr.net/gh/aihdde/Rules@master/icon/ai.png'

  # --- AI_自动优选：AI 稳定节点失效时的备用方案，追求最佳速度 ---
  - name: AI_自动优选
    type: url-test
    <<: *u_s
    url: https://cp.cloudflare.com/generate_204 # 节点健康检查URL (或更针对 AI 服务的测速URL)
    interval: 3600 # 测速间隔延长至 1 小时 (3600 秒)，平衡速度与资源消耗,降低切换频率
    tolerance: 50 # RTT 容忍度：适度允许延迟波动，避免过于频繁的切换
    lazy: false
    filter: '(?i)(🇺🇸|美国|US)'
    # preferred 参数：指定希望优先选择的节点列表
    # 如果您发现某些节点，虽然测速不总是第一，但实际体验更佳，可以将其名称加入
    # preferred:
    #   - '机场自定义前缀 »指定节点名称' # 假设这是您认为在自动优选中也较稳定的节点
    #   - 's_500 »🇺🇸 美国-01'
    icon: 'https://raw.githubusercontent.com/Mithcell-Ma/icon/refs/heads/main/ai_backup.png'

  - {
      name: 香港故转,
      type: fallback,
      <<: *u_s,
      filter: "(?i)(香港|hk|🇭🇰|hong\\s?kong)",
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png',
    }
  - {
      name: 台湾故转,
      type: fallback,
      <<: *u_s,
      filter: '(?i)(🇺🇸|台湾|TW)',
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png',
    }
  - {
      name: 日本故转,
      type: fallback,
      <<: *u_s,
      filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)',
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png',
    }
  - {
      name: 新加坡故转,
      type: fallback,
      <<: *u_s,
      filter: '(?i)(🇸🇬|新加坡|狮城|SG|Singapore)',
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/asn.png',
    }

  - {
      name: 香港自动,
      type: url-test,
      <<: *u_s,
      include-all: false,
      tolerance: 100,
      interval: 600,
      lazy: true,
      filter: "(?i)(香港|hk|🇭🇰|hong\\s?kong)",
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/release/icon/color/auto.png',
    }
  - {
      name: 日本自动,
      type: url-test,
      <<: *u_s,
      include-all: false,
      tolerance: 100,
      interval: 600,
      lazy: true,
      filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)',
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/release/icon/color/auto.png',
    }
  - {
      name: 新加坡自动,
      type: url-test,
      <<: *u_s,
      include-all: false,
      tolerance: 100,
      interval: 600,
      lazy: true,
      filter: '(?i)(🇸🇬|新加坡|狮城|SG|Singapore)',
      icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/release/icon/color/auto.png',
    }

  - name: 自动选择
    type: url-test
    <<: *u_s
    url: https://cp.cloudflare.com/generate_204
    include-all: false
    tolerance: 50
    interval: 900 #安卓可用1800，也就是半小时
    lazy: false
    health-check:
      enable: true
      interval: 900
      url: https://cp.cloudflare.com/generate_204
      method: HEAD
      timeout: 5
      expected-status: '204'
    filter: '(?i)^((?!(DIRECTLY|DIRECT|Proxy|Traffic|Expire|Expired|过期|剩余|流量|官网|超时|timeout|失效|Invalid|Test|测速|本地|Local)).)*$'
    icon: 'https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/icon/color/urltest.png'

  # 📺 媒体平台
  - {
      name: TikTok,
      hidden: true,
      type: select,
      proxies: [台湾故转, 新加坡故转, 美国节点, 新加坡节点, 日本节点],
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/TikTok.png',
    }
  - {
      name: YouTube,
      hidden: true,
      type: select,
      <<: *pg,
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
      name: OneDrive,
      hidden: true,
      type: select,
      proxies: [DIRECTLY, Proxy],
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/OneDrive.png',
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
      name: 香港节点,
      type: select,
      <<: *u,
      skip-cert-verify: true,
      include-all: true,
      filter: "(?i)(香港|hk|🇭🇰|hong\\s?kong)",
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/Hong_Kong.png',
    }
  - {
      name: 美国节点,
      type: select,
      <<: *u,
      skip-cert-verify: true,
      include-all: true,
      filter: "(?i)(🇺🇸|美国|US|united\\s?states|america|usa|洛杉矶|达拉斯|New\\s?York|西雅图)",
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/United_States.png',
    }
  - {
      name: 台湾节点,
      type: select,
      <<: *u,
      skip-cert-verify: true,
      include-all: true,
      filter: '(?i)(🇺🇸|台湾|TW)',
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/United_States.png',
    }
  - {
      name: 新加坡节点,
      type: select,
      <<: *u,
      skip-cert-verify: true,
      include-all: true,
      filter: '(?i)(🇸🇬|新加坡|狮城|SG|Singapore)',
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/Singapore.png',
    }
  - {
      name: 日本节点,
      type: select,
      <<: *u,
      skip-cert-verify: true,
      include-all: true,
      filter: '(?i)(🇯🇵|日本|東京|东京|JP|japan|tokyo|osaka)',
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/Japan.png',
    }

  - {
      name: 全部节点,
      type: select,
      <<: *u,
      icon: 'https://testingcf.jsdelivr.net/gh/Koolson/Qure@master/IconSet/Color/World_Map.png',
      include-all: true,
      filter: '(?i)^((?!(DIRECTLY|DIRECT|Proxy|Traffic|Expire|Expired|过期|剩余|流量|官网|超时|timeout|失效|Invalid|Test|测速|本地|Local)).)*$',
    }
  - {
      name: 漏网之鱼,
      type: select,
      <<: *pg,
      icon: 'https://testingcf.jsdelivr.net/gh/aihdde/Rules@master/icon/fish.png',
    }

# **=============================== 规则设置 ===============================**
# 用于配置规则引擎，决定哪些流量走哪些代理
rules:
  # --- 拒绝/阻止类规则 (最高优先级，直接拦截) ---
  # 这些规则不涉及 DNS 解析，直接拒绝，应放在最前面，以阻止不必要的流量。
  - RULE-SET,AWAvenue_Ads_Rule,REJECT     # 秋风广告过滤
  # - RULE-SET,blackmatrix7_ad,REJECT     # 广告过滤
  # - RULE-SET,porn,REJECT                # 色情类拦截

  # --- 精确的 IP / ASN 规则 (强制 no-resolve，防DNS泄漏) ---
  # 这些规则直接根据 IP 地址匹配，不进行 DNS 解析。确保它们在依赖域名解析的规则之前。
  - RULE-SET,cn_ip,DIRECTLY,no-resolve            # 国内 IP 优先直连
  - RULE-SET,geoip_cloudfront,DIRECTLY,no-resolve # Cloudfront IP 直连

  # 明确的代理IP规则，确保相关服务流量不泄漏
  - RULE-SET,telegram_ip,Proxy,no-resolve         # Telegram IP 强制走代理，不加no-resolve会dns泄漏。
  - RULE-SET,Telegram_No_Resolve,Proxy,no-resolve # Telegram 无需解析的规则
  - RULE-SET,geoip_cloudflare,AI,no-resolve       # Cloudflare IP 导向 AI

  # --- 精确的直连域名规则 (确保国内流量直连) ---
  - RULE-SET,blackmatrix7_direct,DIRECTLY
  - RULE-SET,private_domain,DIRECTLY  # 自定义私有域名（如内网域名、不希望代理的特定域名）
  - RULE-SET,cn_domain,DIRECTLY       # 中国大陆域名（如百度、QQ等，通常包含在国内域名规则集）

  # 苹果服务直连 (macOS)
  - DOMAIN-SUFFIX,apple.com,DIRECTLY
  - DOMAIN-SUFFIX,icloud.com,DIRECTLY
  - DOMAIN-SUFFIX,cdn-apple.com,DIRECTLY # 针对苹果CDN加速
  - RULE-SET,apple_cn_domain,DIRECTLY    # 针对 Apple 国内域名，确保直连
  - DOMAIN-SUFFIX,ls.apple.com,DIRECTLY  # 针对 Apple 定位，直连规则

  # --- 特定服务/流媒体/AI 规则 (强制代理到指定代理组) ---
  # PROCESS-NAME 规则对于 macOS 和安卓（如果客户端支持）尤其有用，可以直接指定某个应用程序的流量走代理。
  - PROCESS-NAME-REGEX,.*telegram.*,Proxy

  # 学术类
  - DOMAIN-SUFFIX,lingq.com,AI
  - DOMAIN-SUFFIX,youglish.com,AI
  - DOMAIN-SUFFIX,deepl.com,AI
  - DOMAIN-SUFFIX,chat.openai.com,AI
  - DOMAIN-SUFFIX,grammarly.com,AI
  - DOMAIN-KEYWORD,sci-hub,AI
  - RULE-SET,ai,AI # AI 规则集

  # 媒体服务
  - RULE-SET,youtube_domain,YouTube # YouTube 域名
  - RULE-SET,tiktok_domain,TikTok   # TikTok 域名

  # 服务类
  - RULE-SET,onedrive_domain,OneDrive   # OneDrive 域名
  - RULE-SET,speedtest_domain,Speedtest # Speedtest 域名
  - RULE-SET,telegram_domain,Proxy      # Telegram 域名 (注意，Telegram IP 已在前面处理)

  # --- 通用代理规则 (捕获大部分需代理流量) ---
  - RULE-SET,gfw_domain,Proxy # 被GFW阻断的域名，走代理（非常重要）
  - RULE-SET,geolocation-!cn,Proxy # 非中国大陆的地理位置流量，走代理
  - RULE-SET,proxy,Proxy # 最后的通用代理规则，捕获所有未被明确分流的代理流量

  # --- Tracker / BT 下载 (特殊流量，优先级较低) ---
  - RULE-SET,trackerslist,Trackerslist

  # --- 最终回退规则 (最低优先级，捕获所有未匹配流量) ---
  - MATCH,漏网之鱼

# **=============================== 规则提供者 ===============================**
# **rule引用设置**
rule-anchor:
  ip: &ip { 
      type: http,
      interval: 86400, 
      behavior: ipcidr, 
      format: mrs 
    }
  domain: &domain { 
      type: http, 
      interval: 86400, 
      behavior: domain, 
      format: mrs 
    }
  class: &class { 
      type: http, 
      interval: 86400, 
      behavior: classical, 
      format: text 
    }
  yaml: &yaml { 
      type: http, 
      interval: 86400, 
      behavior: domain, 
      format: yaml 
    }
  classical_yaml: &classical_yaml {
      type: http,
      interval: 86400,
      behavior: classical,
      format: yaml,
    }

# **动态加载规则配置，提供实时更新的规则集合**
rule-providers:
  # 广告过滤
  AWAvenue_Ads_Rule:
    {
      <<: *yaml,
      path: ./ruleset/AWAvenue_Ads_Rule_Clash.yaml,
      url: "https://raw.githubusercontent.com/TG-Twilight/AWAvenue-Ads-Rule/main//Filters/AWAvenue-Ads-Rule-Clash.yaml",
    }
  # fake-ip 过滤
  fake_ip_filter_DustinWin:
    {
      <<: *domain,
      path: ./ruleset/fake_ip_filter_DustinWin.mrs,
      url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/fakeip-filter.mrs",
    }

  # 直连类规则
  blackmatrix7_direct:
    {
      <<: *yaml,
      path: ./ruleset/blackmatrix7_direct.yaml,
      url: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Direct/Direct.yaml",
    }
  private_domain:
    {
      <<: *domain,
      path: ./ruleset/private_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/private.mrs",
    }
  cn_domain:
    {
      <<: *domain,
      path: ./ruleset/cn_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/cn.mrs",
    }
  cn_ip:
    {
      <<: *ip,
      path: ./ruleset/cn_ip.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geoip/cn.mrs",
    }
  # BT 下载隐私类
  trackerslist:
    {
      <<: *domain,
      path: ./ruleset/trackerslist.mrs,
      url: "https://github.com/DustinWin/ruleset_geodata/raw/refs/heads/mihomo-ruleset/trackerslist.mrs",
    }

  # 代理类规则
  proxy:
    {
      <<: *domain,
      path: ./ruleset/proxy.mrs,
      url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/proxy.mrs",
    }
  gfw_domain:
    {
      <<: *domain,
      path: ./ruleset/gfw_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/gfw.mrs",
    }
  geolocation-!cn:
    {
      <<: *domain,
      path: ./ruleset/geolocation-!cn.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/geolocation-!cn.mrs",
    }
  # 平台类规则
  # 学术类
  ai:
    {
      <<: *domain,
      path: ./ruleset/ai,
      url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ai.mrs",
    }
  # cloudflare
  cloudflare:
    <<: *domain
    path: ./ruleset/cloudflare.mrs
    url: https://github.com/MetaCubeX/meta-rules-dat/raw/refs/heads/meta/geo/geosite/cloudflare.mrs

  geoip_cloudflare: {
      <<: *ip,
      path: ./ruleset/geoip_cloudflare.mrs,
      url: "https://github.com/MetaCubeX/meta-rules-dat/raw/refs/heads/meta/geo/geoip/cloudflare.mrs",
    } # 可 no-resolve
  # 国外媒体
  youtube_domain:
    {
      <<: *domain,
      path: ./ruleset/youtube_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/youtube.mrs",
    }
  tiktok_domain:
    {
      <<: *domain,
      path: ./ruleset/tiktok_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/tiktok.mrs",
    }
  # Telegram
  telegram_domain:
    {
      <<: *yaml,
      path: ./ruleset/telegram_domain.yaml,
      url: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Telegram/Telegram.yaml",
    }
  telegram_ip:
    {
      <<: *ip,
      path: ./ruleset/telegram_ip.mrs,
      url: "https://github.com/DustinWin/ruleset_geodata/raw/refs/heads/mihomo-ruleset/telegramip.mrs",
    }
  Telegram_No_Resolve:
    {
      <<: *classical_yaml,
      path: ./ruleset/Telegram_No_Resolve.yaml,
      url: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/refs/heads/master/rule/Clash/Telegram/Telegram_No_Resolve.yaml",
    }
  # Apple
  apple_cn_domain:
    {
      <<: *domain,
      path: ./ruleset/apple_cn_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/apple-cn.mrs",
    }
  # 微软
  onedrive_domain:
    {
      <<: *domain,
      path: ./ruleset/onedrive_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/onedrive.mrs",
    }
  speedtest_domain:
    {
      <<: *domain,
      path: ./ruleset/speedtest_domain.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/ookla-speedtest.mrs",
    }
  # 可选添加的 geoip 分类（如需细分 IP 层处理）
  geoip_cloudfront: {
      <<: *ip,
      path: ./ruleset/geoip_cloudfront.mrs,
      url: "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geoip/cloudfront.mrs",
    } # 可 no-resolve
```

这里特别注意一点，对外控制 API 接口如果用不到就别开，开也设置个复杂密码。

## DNS

这部分的功能非常实用，虽然最开始我完全看不懂，也不知道什么意思，在这里的配置也让我踩坑了好几次，模板配置中几个不容易理解的地方：

```yaml {open=false title="DNS相关配置"}
dns:
  listen: 0.0.0.0:1053 # 监听地址和端口
  # 启用 Fake-IP 模式，这是强制劫持所有 DNS 请求的关键。
  enhanced-mode: fake-ip

  fake-ip-filter-mode: blacklist
  fake-ip-filter:
  
  default-nameserver:

  nameserver:
  nameserver-policy:
  fallback:

  proxy-server-nameserver:
```

clash DNS 请求逻辑：

1. 当访问一个域名时，nameserver 与 fallback 列表内的所有服务器并发请求，得到域名对应的 IP 地址。

2. clash 将选取 nameserver 列表内，解析最快的结果。

3. 若解析结果中，IP 地址属于国外，那么 clash 将选择 fallback 列表内，解析最快的结果。

4. 【Mihomo】当配置了 nameserver-policy，会优先使用 nameserver-policy 配置的规则。

   geosite 一般是软件内置的基于服务或者地理位置的规则，例如 `geosite:cn` 包含了所有在中国大陆常用的网站域名，而 `geo:cn` 是 IP 匹配规则。

因此，在 nameserver 和 fallback 内都放置无污染、解析速度较快的国内 DNS 服务器，以达到最快的解析速度。
但是 fallback 列表内服务器会用在解析境外网站，为了结果绝对无污染，可以使用支持 DoT/DoH 的服务器。

> [!TIP]
> 模板配置中，我在 nameserver、proxy-server-nameserver、default-nameserver 配置了国外的 DNS（防止 DNS 泄露），实际测试中部分中转机场使用国外 DNS 会解析到"错误的 IP"，导致速度上不去；并且对于某些国内冷门网站，CDN 匹配效果不好，访问速度也很慢，如果有和我相同的问题，可以把这三个都换成注释的国内 DNS。

fake-ip-filter 的作用就是让一些请求不走 fake-ip，因为有些功能 dns 返回私有 IP 是会有问题的，再说国内的网站也没必要。

特殊的两个：default-nameserver 是默认的 DNS，必须是 IP，用于解析 DNS 服务器的域名；而 proxy-server-nameserver 仅用于解析代理节点的域名，如果不填则遵循 nameserver-policy、nameserver 和 fallback 的配置。

DNS 配置注意事项：

1. 如果您为了确保 DNS 解析结果无污染，请仅保留列表内以 tls:// 或 https:// 开头的 DNS 服务器，但是通常对于国内域名没有必要。
2. 如果您不在乎可能解析到污染的结果，更加追求速度。请将 nameserver 列表的服务器插入至 fallback 列表内，并移除重复项。

fake-ip 一般是要和 dns 配置搭配使用，dns 中我们配置了监听 1053 端口，在 tun 的配置中我们使用 dns-hijack 劫持了原本的 53 的 dns 请求到 dns 模块，也就是将 53 重定向到了 1053，这也是 fake-ip 的基础。

---

关于 DNS over HTTPS (DoH) 和 DNS over TLS (DoT) 的选择：

对于两项技术双方各执一词，而且会无休止的争论，各有利弊。各位请根据具体需求自行选择，但是配置文件内默认启用 DoT，~~因为目前国内没有封锁或管制~~。

**DoH:** 以 https:// 开头的 DNS 服务器。拥有更好的伪装性，且几乎不可能被运营商或网络管理封锁，但查询效率和安全性可能略低。

**DoT:** 以 tls:// 开头的 DNS 服务器。拥有更高的安全性和查询效率，但端口有可能被管制或封锁。

> 这里有个坑，或许是我网络的问题，fallback 中的地址我全部连不上，这就导致一个很严重的后果，国外的网站全部无法命中，甚至 AppStore 都挂了。。。
>
> 后面通过调整日志等级到 DEBUG 才知道是 DNS 解析的问题。
>
> 除非你所在的地区 DNS 污染特别严重，否则非常不建议使用 fallback，拖慢速度还不稳定，一般的网络情况配置下 nameserver 普通 DNS 足够了，例如阿里的 223.5.5.5。

### fake-ip

下面补充说明下这个 fake-ip 是什么东西；

> 虽然 Fake IP 这个概念早在 2001 年就被提出来了，但是到 Clash 提供 fake-ip 增强模式以后，依然有很多人对 Fake IP 这个概念以及其作用知之甚少。
>
> 参考：https://blog.skk.moe/post/what-happend-to-dns-in-proxy/

当 TCP 连接建立时，Clash DNS 会直接返回一个保留地址的 IP（即 Fake IP；Clash 默认使用 198.18.0.0/16），同时 Clash 继续解析域名规则和 IP 规则。

PS：开启增强模式后可以尝试 nslookup 看一下解析情况。

由于 TCP/IP 的协议特性，在应用发起 TCP 连接时，会先发出一个 DNS question（发一个 IP Packet），获取要连接的服务器的 IP 地址，然后直接向这个 IP 地址发起连接。

在不使用代理的情况下，DNS 查询流程想必大家很熟悉了，如果使用了代理，直连模式下以使用 SOCKS5 代理的浏览器为例：

1. 浏览器不再需要从自己的 DNS 缓存中寻找域名对应的 ip，因为已经有了 SOCKS5 代理，浏览器可以直接将域名封装在 SOCKS5 流量之中发往代理客户端（clash）
2. 代理客户端从 SOCKS5 流量中抽出域名并设法获得解析结果
3. 代理客户端将你的 SOCKS5 流量还原成标准的 TCP 请求
4. 代理客户端将这个 TCP 连接建立起来，TCP 连接可以承载的是 HTTPS

传统上，大部分浏览器等应用都会调用系统的内置方法去解析域名，这时候如果你想做一些魔法操作，那么就是在系统这一层上；你可以在本地或者其他地方搭建一个黑魔法 DNS 服务器，然后设置系统的 DNS 为它，就可以实现一些黑魔法效果。

例如，在上面的 2 和 3 之间，可以插入一步：代理客户端使用 **某种协议** 将浏览器发出的 SOCKS5 的流量重组并发给远端服务器；

远端服务器使用相同的协议还原，然后拿到域名，进行解析；这样就实现了域名在远端进行解析。

这种就是非直连的方式代理，也就是走转发的部分，可以避免本地 DNS 的污染，但是延迟会高一些，各有优劣，各自体会。

---

然后再说说分流；分流是一个麻烦事。一般情况下，你可能会需要使用域名进行分流（不论是白名单还是黑名单）。不过更多情况下你会使用到基于 IP 的规则来进行分流。

这里可以通过 GUI 的界面来观察连接，如果 TUN 显示一个域名使用了大量端口占用了大量的 ip 池资源，可以考虑将它放到 fake-ip-filter 中不使用 fake-ip。

在域名规则下，如果判断是直连，那么代理客户端没必要进行 DNS 解析，交给原本的流程使用系统接口即可。

在 ip 规则下，当然需要先解析域名，然后匹配规则，但是由于某些协议可以封装域名，因此最终还是会将域名发给远端，由远端进行解析，也就是你本地匹配规则的 ip 与最终代理请求的 ip 可能并不是一个，也避免了影响 CDN 效果。

### 全局流量代理TUN

全局流量代理可能会出现在路由器上或者 TUN/TAP 型的支持全局代理客户端上。用户不再主动为每个应用程序设置代理。**此时应用程序是不会感知到代理客户端的存在，它们会正常的发起 TCP 连接**，Clash 的增强模式或者说 TUN 模式，会接管设备的 TCP 协议栈，并且由于 TCP/IP 协议在拿到 DNS 解析结果之前，连接是不能建立的。

这时候配合上面说的 DNS 解析过程就比较有趣了，前半部分不变，最终会调用系统接口进行解析域名，这时候会向系统配置的 DNS 服务器发起请求；

如果我们在系统的网络设置之中有设置上游 DNS 地址，例如代理客户端可能会修改系统设置中的 DNS 到 127.0.0.1 或者别的内网 IP、也可能保留用户之前的设置，这无所谓，因为… **操作系统发出的 DNS 解析请求会经过代理客户端并最终被截获**；这其实就是配置中的 dns-hijack，截获配置的 DNS 服务器的请求。

代理客户端可以将这个解析请求原样发出去、或者用自己的黑魔法，总之都会拿到一个解析结果；

代理客户端将这个解析结果（下面说的 FakeIP 就是返回一个假的内网 IP）返回回去，操作系统拿到了这个解析结果并返回给浏览器 浏览器对这个解析结果的 IP 建立一个 TCP 连接并发送出去，**这个 TCP 连接被代理客户端截获**。

由于之前代理客户端进行的 DNS 解析请求这一动作，代理客户端可以找到这个只包含目标 IP 的 TCP 连接原来的目标域名；

如果是支持 redir 的代理客户端，那么代理客户端就会直接将域名和 TCP 连接中的其它数据封装成 **某种魔法协议** 发给远端服务器；或者封装成 SOCKS5 后交给支持 SOCKS5 的代理客户端。

和应用程序直接将流量封装成 SOCKS5 大有不同，在类似于透明代理的环境下浏览器和其它应用程序是正常地发起 TCP 连接。因此除非得到一个 DNS 解析结果，否则 TCP 连接不会建立；代理客户端也会需要通过这个 DNS 查询动作，才能找到之后的 TCP 连接的域名。 

你大概能够发现，浏览器、应用程序直接设置 SOCKS5 代理的话，可以不在代理客户端发起 DNS 解析请求就能将流量发送给远端服务器；

而**在透明代理模式下，不论是否需要 IP 规则分流都需要先进行一次 DNS 解析才能建立连接**。 有没有办法能像直接设置 SOCKS5 代理一样省掉一次 DNS 解析呢？

有，就是代理客户端自己不先执行查询动作，丢一个 Fake IP 回去让浏览器、应用程序立刻建立 TCP 连接。

有了 Fake IP，代理客户端无需进行 DNS 解析。最后不论是浏览器、代理客户端还是远端服务器都不会去和 Fake IP 进行连接，因为在代理客户端这里就已经完成了截获、重新封装。 

即使按照域名规则分流，代理客户端都没有进行 DNS 解析的需要。只有在遇到了按照 IP 进行分流的规则时，代理客户端才需要进行一次解析拿到一个 IP 用于判断。即便如此，这个 IP 只用于分流规则的匹配，不会被用于实际的连接。

PS：Clash 的增强模式既有 redir-host 也有 Fake IP，目前流行的是 Fake IP 模式。

---

这里有个很有意思的问题，如果操作系统或者浏览器缓存了 Fake IP，但是代理客户端中 Fake IP 和域名的映射表丢失以后，会出现什么状况？可能会出现什么错误信息？

你应该大概意识到 Clash 在 Fake IP 模式下偶发的无法上网的原因了。

在使用 Fake-ip 模式后，Application 拿到的是 Clash DNS 返回的 Fake IP，所以也不会出现某些应用程序拒绝连接一些 IP 的情况；和 redir-host 模式一样，在大部分情况下 fake-ip 模式下也可以完全无视 DNS 污染。

## 节点

这部分可参考 SS-Rule，写的很好，或者可以看看官方推荐的[文档](https://lancellc.gitbook.io/clash/clash-config-file/proxies/config-a-vmess-proxy)或者 wiki，基本是给自建的人用的，订阅的方式是机场维护。

本次更新（2025-08-24）删掉了示例内容，对订阅用户的意义不大，可参考我的老文章。

订阅模式需要注意的是拉取的不一定只有节点，包括代理组、规则集可能都有，是一份完整的配置；这样就可能面临一个问题，如果你使用远程订阅，并在配置文件内自定义或者修改了一些规则，等配置在下一次更新订阅后可能会丢失；

所以我们自己新建一个配置文件，自定义分流、DNS、等配置部分，节点使用 proxy-providers 引入远程订阅中的节点信息，其他配置会被自己的配置文件覆盖，这是一个很不错的解决方案，我目前就是使用的这种方式来订阅多个机场，并且统一使用我自定义的配置（参考上文的模板配置）。

如果你的机场不提供 clash 订阅连接，可以使用在线服务进行转换，这个一搜一大堆不多说，找个靠谱点的就像，或者自建一个。

顺便提一嘴，如果你没机场只是偶尔临时用，可以看看 [proxypool](https://github.com/zu1k/proxypool) 这个项目，从互联网爬取免费的节点，还有好心人提供了 proxy-providers 的在线地址，可以临时顶一顶，不过毕竟免费风险还是有的，这个自己取舍。

## 代理组

这部分也很重要，配合上门的节点分组，避免一个节点挂掉还得手动换，选择最佳的节点连接。

```yaml {open=true}
proxy-groups:
  # 代理的转发链, 在 proxies 中不应该包含 relay. 不支持 UDP.
  # 流量: clash <-> http <-> vmess <-> ss1 <-> ss2 <-> 互联网
  - name: "relay"
    type: relay
    proxies:
      - http
      - vmess
      - ss1

  # url-test 可以自动选择与指定 URL 测速后，延迟最短的服务器
  - name: "auto"
    type: url-test
    proxies:
      - ss1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
    
  - name: "auto"
    type: url-test
    # 使用订阅节点
    use:
      - provider1
    tolerance: 300

  # fallback 可以尽量按照用户书写的服务器顺序，在确保服务器可用的情况下，自动选择服务器
  - name: "fallback-auto"
    type: fallback
    proxies:
      - ss1
    url: 'http://cp.cloudflare.com/generate_204'
    interval: 300

  # load-balance 可以使相同 eTLD 请求在同一条代理线路上
  - name: "load-balance"
    type: load-balance
    proxies:
      - vmess1
    url: 'http://www.youtube.com/generate_204'
    interval: 300

  # select 用来允许用户手动选择 代理服务器 或 服务器组
  # 您也可以使用 RESTful API 去切换服务器，这种方式推荐在 GUI 中使用
  - name: Proxy
    type: select
    proxies:
      - ss1
      - vmess1
      - auto
```

这里注意 Proxy 这个关键组，GUI 默认使用这个（其实是后面的规则配的是这个），它的类型是 select 可以允许我们在 GUI 中手动选择一个节点或者组，默认我使用 auto，也就是 url-test 模式的。

如果你在配置文件中设置了 tolerance，Clash 将会计算所有代理服务器的延迟时间，然后以最快的代理服务器的延迟时间为基准，根据 tolerance 的值来筛选其他的代理服务器。只有当一个代理服务器的延迟时间小于基准延迟时间加上 tolerance 时，它才会被选择作为请求的代理服务器，换句话说就是只要在这个范围就不会自动切换，以避免频繁切换带来的体验差。

## 规则集

简单说就是一组规则的集合，只不过可以在线获取，定时更新，也就是可以直接用别人写好的分流规则，非常爽啊，这里我用过好多，~~最终选择了 [Sukka](https://skk.moe/) 大佬的，感觉数量不多，但是够用，很精简。~~

Mihomo 的分流参考上面的模板配置，Surge 的分流目前还在使用 Sukka 佬的配置。

这里说下 Clash 支持的几种规则；

- DOMAIN-SUFFIX：域名后缀匹配
- DOMAIN：域名匹配
- DOMAIN-KEYWORD：域名关键字匹配
- IP-CIDR：IP段匹配
- SRC-IP-CIDR：源IP段匹配
- GEOIP：GEOIP 数据库（国家代码）匹配
- DST-PORT：目标端口匹配
- SRC-PORT：源端口匹配
- PROCESS-NAME：源进程名匹配
- RULE-SET：Rule Provider 规则匹配
- MATCH：全匹配

写法上面已经有示例了，特别的情况，如果你不想让 Clash 进行 DNS 解析，可以在后面加上 no-resolve。

自带的两种规则是 REJECT 和 DIRECT，很好理解，广告相关的一般直接使用 REJECT，不需要代理的就使用 DIRECT，剩下的就需要指定我们配的代理组，例如本例的 Proxy。

`MATCH`需要位于规则列表末尾，除了那些漏网之鱼。

## 参考规则

这里推荐几个开箱即用的规则：

[ruleset_geodata](https://github.com/DustinWin/ruleset_geodata)

[MetaCubeX官方](https://github.com/MetaCubeX/meta-rules-dat)

[clash-rules](https://github.com/Loyalsoldier/clash-rules)

[ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)

[GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN)

