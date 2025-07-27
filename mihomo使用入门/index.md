# Mihomo使用入门

<!--more-->
{{< admonition type=info title="提示" open=true >}}
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

## 基本配置

大部分机场都提供一份默认配置文件，包括一些基本的分流规则，但是我推荐自己写一份，更适合自己。

Mihomo 和 Clash 配置基本通用的，下面 Clash / Mihomo 称呼都基本一样，主文件 Config.yaml 基本内容：

```yaml
port: 1090
socks-port: 1080
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090
secret: 'o4UJC!kwdjainfuaenf'

dns:
tun:

proxies:
proxy-groups:
proxy-providers:

rule-providers:

rules:
- DOMAIN-SUFFIX,google.com,DIRECT
- DOMAIN-KEYWORD,google,DIRECT
- DOMAIN,google.com,DIRECT
- DOMAIN-SUFFIX,ad.com,REJECT
- GEOIP,CN,DIRECT
- MATCH,DIRECT
```

这里特别注意一点，对外控制 API 接口如果用不到就别开，开也设置个复杂密码。

## DNS

这部分的功能非常实用，虽然最开始我完全看不懂，也不知道什么意思，在这里的配置也让我踩坑了好几次，首先来看一个示例配置：

```yaml {open=false title="DNS相关配置"}
# DNS 服务器配置(可选；若不配置，程序内置的 DNS 服务会被关闭)
dns:
  enable: true
  listen: 0.0.0.0:53
  ipv6: true # 当此选项为 false 时, AAAA 请求将返回空

  # 以下填写的 DNS 服务器将会被用来解析 DNS 服务的域名
  # 仅填写 DNS 服务器的 IP 地址
  default-nameserver:
    - 223.5.5.5
    - 119.29.29.29
    - 114.114.114.114
  enhanced-mode: fake-ip # 或 redir-host
  fake-ip-range: 198.18.0.1/16 # Fake IP 地址池 (CIDR 形式)
  # use-hosts: true # 查询 hosts 并返回 IP 记录

  # 在以下列表的域名将不会被解析为 fake ip，这些域名相关的解析请求将会返回它们真实的 IP 地址
  fake-ip-filter:
    # 以下域名列表参考自 vernesong/OpenClash 项目，并由 Hackl0us 整理补充
    # === LAN ===
    - '*.lan'
    # === Linksys Wireless Router ===
    - '*.linksys.com'
    - '*.linksyssmartwifi.com'
    # === Apple Software Update Service ===
    - 'swscan.apple.com'
    - 'mesu.apple.com'
    # === Windows 10 Connnect Detection ===
    - '*.msftconnecttest.com'
    - '*.msftncsi.com'
    # === NTP Service ===
    - 'time.*.com'
    - 'time.*.gov'
    - 'time.*.edu.cn'
    - 'time.*.apple.com'
    - 'time1.*.com'
    - 'time2.*.com'
    - 'time3.*.com'
    - 'time4.*.com'
    - 'time5.*.com'
    - 'time6.*.com'
    - 'time7.*.com'
    - 'ntp.*.com'
    - 'ntp.*.com'
    - 'ntp1.*.com'
    - 'ntp2.*.com'
    - 'ntp3.*.com'
    - 'ntp4.*.com'
    - 'ntp5.*.com'
    - 'ntp6.*.com'
    - 'ntp7.*.com'
    - '*.time.edu.cn'
    - '*.ntp.org.cn'
    - '+.pool.ntp.org'

    - 'time1.cloud.tencent.com'
    # === Music Service ===
    ## NetEase
    - '+.music.163.com'
    - '*.126.net'
    ## Baidu
    - 'musicapi.taihe.com'
    - 'music.taihe.com'
    ## Kugou
    - 'songsearch.kugou.com'
    - 'trackercdn.kugou.com'
    ## Kuwo
    - '*.kuwo.cn'
    ## JOOX
    - 'api-jooxtt.sanook.com'
    - 'api.joox.com'
    - 'joox.com'
    ## QQ
    - '+.y.qq.com'
    - '+.music.tc.qq.com'
    - 'aqqmusic.tc.qq.com'
    - '+.stream.qqmusic.qq.com'
    ## Xiami
    - '*.xiami.com'
    ## Migu
    - '+.music.migu.cn'
    # === Game Service ===
    ## Nintendo Switch
    - '+.srv.nintendo.net'
    ## Sony PlayStation
    - '+.stun.playstation.net'
    ## Microsoft Xbox
    - 'xbox.*.microsoft.com'
    - '+.xboxlive.com'
    # === Other ===
    ## QQ Quick Login
    - 'localhost.ptlogin2.qq.com'
    ## Golang
    - 'proxy.golang.org'
    ## STUN Server
	  - 'stun.*.*'
    - 'stun.*.*.*'
    - '+.stun.*.*'
    - '+.stun.*.*.*'
    - '+.stun.*.*.*.*'
    
    - 'heartbeat.belkin.com'
    - '*.linksys.com'
    - '*.linksyssmartwifi.com'
    - '*.router.asus.com'
    - 'mesu.apple.com'
    - 'swscan.apple.com'
    - 'swquery.apple.com'
    - 'swdownload.apple.com'
    - 'swcdn.apple.com'
    - 'swdist.apple.com'
    - 'lens.l.google.com'
    - 'stun.l.google.com'
    - '+.nflxvideo.net'
    - '*.square-enix.com'
    - '*.finalfantasyxiv.com'
    - '*.ffxiv.com'
    - '*.mcdn.bilivideo.cn'

  # 支持 UDP / TCP / DoT / DoH 协议的 DNS 服务，可以指明具体的连接端口号。
  # 所有 DNS 请求将会直接发送到服务器，不经过任何代理。
  # Clash 会使用最先获得的解析记录回复 DNS 请求
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
     
  # 当 fallback 参数被配置时, DNS 请求将同时发送至上方 nameserver 列表和下方 fallback 列表中配置的所有 DNS 服务器.
  # 当解析得到的 IP 地址的地理位置不是 CN 时，clash 将会选用 fallback 中 DNS 服务器的解析结果。
  # fallback:
  #   - https://dns.google/dns-query

  # 如果使用 nameserver 列表中的服务器解析的 IP 地址在下方列表中的子网中，则它们被认为是无效的，
  # Clash 会选用 fallback 列表中配置 DNS 服务器解析得到的结果。
  #
  # 当 fallback-filter.geoip 为 true 且 IP 地址的地理位置为 CN 时，
  # Clash 会选用 nameserver 列表中配置 DNS 服务器解析得到的结果。
  #
  # 当 fallback-filter.geoip 为 false, 如果解析结果不在 fallback-filter.ipcidr 范围内，
  # Clash 总会选用 nameserver 列表中配置 DNS 服务器解析得到的结果。
  #
  # 采取以上逻辑进行域名解析是为了对抗 DNS 投毒攻击。
  # fallback-filter:
  #  geoip: false
  #  ipcidr:
  #    - 240.0.0.0/4
  #    - 0.0.0.0/32
    # domain:
    #   - '+.google.com'
    #   - '+.facebook.com'
    #   - '+.youtube.com'

tun:
  enable: true
  stack: gvisor # 或 system
  macOS-auto-route: true
  macOS-auto-detect-interface: true
  dns-hijack:
    - tcp://8.8.8.8:53
    - tcp://8.8.4.4:53
```

clash DNS 请求逻辑：

1. 当访问一个域名时，nameserver 与 fallback 列表内的所有服务器并发请求，得到域名对应的 IP 地址。
2. clash 将选取 nameserver 列表内，解析最快的结果。
3. 若解析结果中，IP 地址属于国外，那么 clash 将选择 fallback 列表内，解析最快的结果。

因此，在 nameserver 和 fallback 内都放置无污染、解析速度较快的国内 DNS 服务器，以达到最快的解析速度。
但是 fallback 列表内服务器会用在解析境外网站，为了结果绝对无污染，尽量使用支持 DoT/DoH 的服务器。

DNS 配置注意事项：

1. 如果您为了确保 DNS 解析结果无污染，请仅保留列表内以 tls:// 或 https:// 开头的 DNS 服务器，但是通常对于国内域名没有必要。
2. 如果您不在乎可能解析到污染的结果，更加追求速度。请将 nameserver 列表的服务器插入至 fallback 列表内，并移除重复项。

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

需要注意的是，在 [v1.9](https://github.com/Dreamacro/clash/releases/tag/v1.9.0) 版本后，作者调整了配置的格式（改动很小），下面使用的是最新的个数，详情可以看官方的说明。

```yaml {open=false title="节点 proxies 配置"}
proxies:
  # shadowsocks
  # 支持加密方式：
  #   aes-128-gcm aes-192-gcm aes-256-gcm
  #   aes-128-cfb aes-192-cfb aes-256-cfb
  #   aes-128-ctr aes-192-ctr aes-256-ctr
  #   rc4-md5 chacha20 chacha20-ietf xchacha20
  #   chacha20-ietf-poly1305 xchacha20-ietf-poly1305

  - name: "ss1"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    # udp: true

  # vmess
  # 支持加密方式：auto / aes-128-gcm / chacha20-poly1305 / none
  - name: "vmess"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # tls: true
    # skip-cert-verify: true
    # servername: example.com # 优先级高于 wss host
    # network: ws
    # ws-opts:
    #   path: /path
    #   headers:
    #     Host: v2ray.com
    #   max-early-data: 2048
    #   early-data-header-name: Sec-WebSocket-Protocol
    
  - name: "vmess-http"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # network: http
    # http-opts:
    #   # method: "GET"
    #   # path:
    #   #   - '/'
    #   #   - '/video'
    #   # headers:
    #   #   Connection:
    #   #     - keep-alive
    
  - name: vmess-grpc
    server: server
    port: 443
    type: vmess
    uuid: uuid
    alterId: 32
    cipher: auto
    network: grpc
    tls: true
    servername: example.com
    # skip-cert-verify: true
    grpc-opts:
      grpc-service-name: "example"

  # socks5
  - name: "socks"
    type: socks5
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true
    # skip-cert-verify: true
    # udp: true

  # http
  - name: "http"
    type: http
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true # https
    # skip-cert-verify: true
  
  # Trojan
  - name: "trojan"
    type: trojan
    server: server
    port: 443  - name: "trojan"
    type: trojan
    server: server
    port: 443
    password: yourpsk
    # udp: true
    # sni: example.com # aka server name
    # alpn:
    #   - h2
    #   - http/1.1
    # skip-cert-verify: true
    
  - name: trojan-grpc
    server: server
    port: 443
    type: trojan
    password: "example"
    network: grpc
    sni: example.com
    # skip-cert-verify: true
    udp: true
    grpc-opts:
      grpc-service-name: "example"

  - name: trojan-ws
    server: server
    port: 443
    type: trojan
    password: "example"
    network: ws
    sni: example.com
    # skip-cert-verify: true
    udp: true
    # ws-opts:
      # path: /path
      # headers:
      #   Host: example.com
```

现在我也不太清楚流行什么协议，Trojan 好像很牛逼，vmess 如果效果还是不理想可以切换试试看，我暂时还没用过。
最流行可能还是 vmess，而订阅模式使用 proxy-providers 来定义，体验更好，使用的时候通过 use 关键字。

```yaml {open=false title="订阅配置"}
proxy-groups:
  - name: Proxy
    type: url-test
    use:
      - provider1

proxy-providers:
  provider1:
    type: http
    # 使用 url 在线订阅
    url: "url"
    interval: 3600
    path: ./conf/provider1.yaml
    health-check:
      enable: true
      interval: 600
      url: http://cp.cloudflare.com/generate_204
  test:
    type: file
    # 从文件中读取
    path: /test.yaml
    # 可以使用正则来过滤节点
    filter: '(香港|台湾|美国).*'
    health-check:
      enable: true
      interval: 36000
      url: http://www.gstatic.com/generate_204
```

使用 proxy-providers 省去了我们自己维护 proxy 节点，直接从在线或者本地文件读取 proxy 节点信息，其他规则还是我们自己定义，顺便提一嘴，如果你没机场只是偶尔临时用，可以看看 [proxypool](https://github.com/zu1k/proxypool) 这个项目，从互联网爬取免费的节点，还有好心人提供了 proxy-providers 的在线地址，可以临时顶一顶，不过毕竟免费风险还是有的，这个自己取舍。

订阅模式需要注意的是拉取的不一定只有节点，包括代理组、规则集可能都有，这样就可能面临一个问题，如果你使用远程订阅，你自定义的一些规则等配置在下一次更新订阅后可能会丢失；

所以自定义配置部分，节点使用 proxy-providers 是一个很不错的解决方案，我目前就是使用的这种方式来订阅多个机场，并且统一使用我自定义的配置，或者也可以尝试使用 Parser 规则解决。

如果你的机场不提供 clash 订阅连接，可以使用在线服务进行转换，这个一搜一大堆不多说，找个靠谱点的就像，或者自建。

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
      - ss2

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
      - ss2
      - vmess1
      - auto
```

这里注意 Proxy 这个关键组，GUI 默认使用这个（其实是后面的规则配的是这个），它的类型是 select 可以允许我们在 GUI 中手动选择一个节点或者组，默认我使用 auto，也就是 url-test 模式的。

如果你在配置文件中设置了 tolerance，Clash 将会计算所有代理服务器的延迟时间，然后以最快的代理服务器的延迟时间为基准，根据 tolerance 的值来筛选其他的代理服务器。只有当一个代理服务器的延迟时间小于基准延迟时间加上 tolerance 时，它才会被选择作为请求的代理服务器，换句话说就是只要在这个范围就不会自动切换，以避免频繁切换带来的体验差。

## 规则集

简单说就是一组规则的集合，只不过可以在线获取，定时更新，也就是可以直接用别人写好的分流规则，非常爽啊，这里我用过好多，最终选择了 [Sukka](https://skk.moe/) 大佬的，感觉数量不多，但是够用，很精简。

``` yaml {open=false title="规则集订阅"}
rule-providers:
  reject_non_ip_drop:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/reject-drop.txt
    path: ./ruleset/reject_non_ip_drop.txt
  reject_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/reject.txt
    path: ./ruleset/reject_non_ip.txt
  reject_domainset:
    type: http
    behavior: domain
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/domainset/reject.txt
    path: ./ruleset/reject_domainset.txt
  reject_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/ip/reject.txt
    path: ./ruleset/reject_ip.txt
  sogouinput:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/sogouinput.txt
    path: ./ruleset/sogouinput.txt
  stream_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/stream.txt
    path: ./ruleset/stream_non_ip.txt
  stream_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/ip/stream.txt
    path: ./ruleset/stream_ip.txt
  ai_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/ai.txt
    path: ./ruleset/ai_non_ip.txt
  telegram_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/telegram.txt
    path: ./ruleset/telegram_non_ip.txt
  telegram_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/ip/telegram.txt
    path: ./ruleset/telegram_ip.txt
  apple_cdn:
    type: http
    behavior: domain
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/domainset/apple_cdn.txt
    path: ./ruleset/apple_cdn.txt
  apple_services:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/apple_services.txt
    path: ./ruleset/apple_services.txt
  apple_cn_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/apple_cn.txt
    path: ./ruleset/apple_cn_non_ip.txt
  lan_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/lan.txt
    path: ./ruleset/lan_non_ip.txt
  lan_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/ip/lan.txt
    path: ./ruleset/lan_ip.txt
  domestic_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/domestic.txt
    path: ./ruleset/domestic_non_ip.txt
  direct_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/direct.txt
    path: ./ruleset/direct_non_ip.txt
  global_non_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: https://ruleset.skk.moe/Clash/non_ip/global.txt
    path: ./ruleset/global_non_ip.txt
  domestic_ip:
    type: http
    behavior: classical
    format: text
    interval: 43200
    url: 'https://ruleset.skk.moe/Clash/ip/domestic.txt'
    path: ./ruleset/domestic_ip.txt
  spotify:
    type: http
    behavior: classical
    interval: 43200
    url: 'https://ghp.ci/https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Spotify/Spotify.yaml'
    path: ./ruleset/Spotify.yaml
  speedtest:
    type: http
    behavior: classical
    interval: 43200
    url: https://ghp.ci/https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Speedtest/Speedtest.yaml
    path: ./ruleset/Speedtest.yaml
```

然后配置下最终规则，整个配置就算完成了：

``` yaml {open=false title="最终规则"}
rules:
  # 自定义规则 示例
  - DOMAIN-SUFFIX,todesk.com,DIRECT

  # ---内置规则集---
  # > Safari 防跳转
  - DOMAIN,app-site-association.cdn-apple.com,REJECT
  # ban UDP on Youtube
  - AND,(AND,(DST-PORT,443),(NETWORK,UDP)),(NOT,((GEOSITE,cn))),REJECT
  # ban National Anti-fraud Center
  - DOMAIN,prpr.96110.cn.com,DIRECT
  - DOMAIN-KEYWORD,96110,REJECT
  - DOMAIN-SUFFIX,gjfzpt.cn,REJECT
  # > 🆕 拒绝国家反诈中心请求
  - DOMAIN-SUFFIX,gjfzpt.cn,REJECT
  # SYSTEM
  - DOMAIN,ess.apple.com,DIRECT
  - PROCESS-NAME,maps,DIRECT
  - DOMAIN-KEYWORD,courier.push.apple.com,DIRECT
  - DOMAIN-KEYWORD,apac-china-courier,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT

  # > KEYWORD
  - DOMAIN-KEYWORD,docker,Proxy
  - DOMAIN-KEYWORD,mastodon,Proxy
  - DOMAIN-KEYWORD,techhub.social,Proxy
  - DOMAIN-KEYWORD,macked,Proxy
  - DOMAIN-KEYWORD,leensasf,Proxy
  # > SUFFIX
  - DOMAIN-SUFFIX,nssurge.com,Proxy
  - DOMAIN-SUFFIX,surgee.me,Proxy
  - DOMAIN-SUFFIX,st7eve.me,Proxy
  - DOMAIN-SUFFIX,www.torrentmac.net,Proxy
  - DOMAIN-SUFFIX,cloudflare.com,Proxy
  - DOMAIN-SUFFIX,appstorrent.ru,DIRECT
  # > DOMAIN
  - DOMAIN,jable.tv,Proxy
  - DOMAIN,api.amplitude.com,Proxy
  - DOMAIN,api.revenuecat.com,REJECT
  - DOMAIN,m.cmx.im,Proxy

  - DOMAIN-SUFFIX,lan,DIRECT
  - DOMAIN-SUFFIX,home.arpa,DIRECT
  - DOMAIN-SUFFIX,localhost,DIRECT
  - DOMAIN-SUFFIX,localdomain,DIRECT
  - DOMAIN-SUFFIX,10.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,254.169.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,16.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,17.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,18.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,19.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,30.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,31.172.in-addr.arpa,DIRECT
  - DOMAIN-SUFFIX,168.192.in-addr.arpa,DIRECT

  - RULE-SET,reject_non_ip,REJECT
  - RULE-SET,reject_domainset,REJECT
  - RULE-SET,sogouinput,REJECT
  - RULE-SET,spotify,Spotify
  - RULE-SET,stream_non_ip,Netflix
  - RULE-SET,ai_non_ip,AI
  - RULE-SET,telegram_non_ip,Telegram
  - RULE-SET,apple_cdn,DIRECT
  - RULE-SET,apple_services,DIRECT
  - RULE-SET,apple_cn_non_ip,DIRECT
  - RULE-SET,lan_non_ip,DIRECT
  - RULE-SET,domestic_non_ip,DIRECT
  - RULE-SET,direct_non_ip,DIRECT
  - RULE-SET,speedtest,Speedtest
  - RULE-SET,global_non_ip,Proxy

  - RULE-SET,reject_ip,REJECT
  - RULE-SET,reject_non_ip_drop,REJECT
  - RULE-SET,stream_ip,Netflix
  - RULE-SET,telegram_ip,Telegram
  - RULE-SET,lan_ip,DIRECT
  - RULE-SET,domestic_ip,DIRECT

  - GEOIP,LAN,DIRECT,no-resolve
  # 不关注 DNS 泄露不加 no-resolve
  - GEOIP,CN,DIRECT,no-resolve
  - MATCH,NoMatch

script:
  shortcuts:
    quic: network == 'udp' and dst_port == 443

# - RULE-SET,apple,DIRECT,no-resolve
```

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

## 脚本

同样，这也是 Pro 的专有功能，除了全局、直连、规则，还增加了一个更灵活的脚本模式，来应对日益增多的 Rule 规则。但是目前用的人很少，我也没这个需求，暂不关注。
如果有更个性化的节点处理需求，可以尝试使用 parsers。

## 参考规则

这里推荐几个开箱即用的规则：

[Profiles](https://github.com/DivineEngine/Profiles)

[clash-rules](https://github.com/Loyalsoldier/clash-rules)

[SS-Rule-Snippet](https://github.com/Hackl0us/SS-Rule-Snippet)

[GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN)

