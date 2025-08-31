# 我的 Cloudflare 食用手册


Cloudflare 不愧是互联网的赛博菩萨，看了下免费用了它家好多服务，用起来也是非常舒服，但是也只是它全部功能的一小部分，开个文章整理下，以防后面忘记怎么用....<!--more-->

目前我的域名也在 Cloudflare 续费了，除开其他家第一年的优惠，Cloudflare 续费稳定的价格是真的很有性价比。

其实 Github 上有人整理过，并且 Star 非常高：[Awesome Cloudflare](https://github.com/zhuima/awesome-cloudflare) 但是这么多我也不可能全部都用到，所以整理下我实际用的功能。

## DNS 管理

最基础的功能，这个没什么好说的，最特色的就是橙色云朵，开启代理后把自己的服务隐藏在 Cloudflare 后面可以避免很多麻烦的不安全的东西，缺点就是速度对大陆用户不太友好。

Cloudflare 提供的 API 对于实现 DDNS 也非常的友好，目前我在使用 ddns-go 来使用 DDNS。

## SSL/TLS

> 选择 Cloudflare 如何加密访问者与 Cloudflare 之间以及 Cloudflare 与源服务器之间的流量，以阻止数据盗窃和其他篡改行为。

则其中最重要的就是选择什么加密模式，当开启了橙色小云，那么我们的服务会隐藏在 Cloudflare 后面，这同时也省去了麻烦的 SSL 证书问题，这样用户可以直接使用 HTTPS 访问我们的服务，毕竟有些是强制 HTTPS 的（HSTS），浏览器也会对 HTTP 进行警告。

Cloudflare 用于连接到您的源服务器的加密模式：

- 严格（仅 SSL 源服务器拉取）
  
  在 Cloudflare 与您的源服务器之间强制进行加密。使用此模式可以确保与您的源服务器的连接始终是加密的，不考虑您的访问者的请求。
  
- 完全（严格）
  
  启用端到端加密，对源服务器证书**强制执行验证**。使用 Cloudflare 的源服务器 CA 为您的源服务器生成证书（可以通过 Cloudflare 给你生成一个超长时间的证书）。
  
- 完全
  
  启用加密端到端。当您的源服务器支持 SSL 认证但**未使用有效的公开可信的证书时**，使用此模式。
  
- 灵活
  
  仅在访问者与 Cloudflare 之间启用加密。这可以避免浏览器发出安全警告，但 Cloudflare 与您的源服务器之间的所有连接均通过 HTTP 建立。
  
- 关闭（不安全）
  
  未应用加密。关闭 SSL 将禁用 HTTPS，浏览器同时会显示警告，指出您的网站不安全。

{{< admonition type=tip title="端口问题" open=false >}}
使用 Cloudflare 进行代理后，Cloudflare 对服务访问端口有限制，当时我也踩坑搞了好久；

我用 Docker 跑了两个服务，一个映射在 8443 端口仅支持 https；另一个跑在 8002 端口使用了普通的 http，不支持 SSL。

配置好 DNS 后，开始使用域名以 https 访问 8443 端口发现超时；然后访问 8002 时也超时；

- 当使用**灵活模式**时，无论你使用 http 还是 https 访问，CF 都会以 http 的方式访问源站，此时源服务器不需要 SSL 证书；

- 使用**完全模式**时，会使用访问者请求的方案连接到源。也就是如果访问者使用 `http`，则 Cloudflare 使用 http 连接到源，https 亦然。
  
  这种模式**不会校验源站的 SSL 证书**，所以可以使用自签的证书或者 CF 签发的证书。

我当时配置的是灵活模式，根据描述会以 http 的方式访问源站，那确实不会通，但是无法解释 https 访问 8002 为何会超时；

最后我在官方文档的灵活模式解释部分找到了原因：

> Flexible mode is only supported for HTTPS connections on port 443 (default port). Other ports using HTTPS will fall back to Full mode.  
>
> https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/flexible/#limitations

也就是**灵活模式这种方式有一个限制，就是仅支持 443 端口的 https 连接，如果使用了其他端口会回退到 Full 完全模式。**

也就是如果想用 CF 的 https，就不要自定义端口，把服务端口从 8002 切换到 80 后确实正常了。

---

改为完全模式后，测试 8443 可以正常访问了，但是 8002 我还是想通过 https 访问，于是我通过 Nginx 做了一个支持，让 8002 支持 https，但是访问依然超时。

现在的情况就有点棘手：

Cloudflare 当前处于 Full 模式，使用 https 访问的情况下，8443 正常；8002 超时；  
这两个服务都支持了 SSL，通过 IP 访问都正常；

后来都快放弃了，决定翻翻 Cloudflare 的文档（当时用 LLM 问，都是瞎说，浪费了不少时间），最终发现文档有这么一篇：

> HTTP ports supported by Cloudflare
>
> - 80
> - 8080
> - 8880
> - 2052
> - 2082
> - 2086
> - 2095
>
> HTTPS ports supported by Cloudflare
>
> - 443
> - 2053
> - 2083
> - 2087
> - 2096
> - 8443
>
> [Network ports](https://developers.cloudflare.com/fundamentals/get-started/reference/network-ports/)

看到这我也是挺无语的，就那么凑巧碰到了 8443 这个白名单。。。。

那既然有端口限制，那估计没戏了，但是！可以通过规则里 **Origin Rules** 来进行端口重写，实现曲线救国！

原文我发布在这里了：[Cloudflare 代理后访问超时](https://nyan.sakanoy.com/CloudflareTimeOut)
{{< /admonition >}}

由于一个域名只能配置一种加密类型（或者通过 Configuration Rules 来实现精细控制），所以我觉得应该至少有两个域名才能比较舒服，一个配置灵活，一个配置完全/完全（严格）；差不多就可以应对所有场景了。

还可以在设置中进行更多的例如 HSTS、TLS 版本、HTTPS 重写等配置。

## 规则

> 修改 HTTP 请求和响应，执行 URL 重定向，配置 Cloudflare 设置，以及触发匹配请求的操作。
>
> 规则按照从第一个到最后一个的顺序进行评估和执行。如果有多个规则进行同一项修改，最后执行的规则有效。

我比较常用的规则有重定向规则、URL 重写、Origin Rules，非常实用的功能；

例如 301 / 302 重定向规则，这个在国内的 DNS 平台好像需要备案才能用，301 永久重定向浏览器 URL 会刷新，302 临时重定向地址栏 URL 不会变。

Origin Rules 这个规则可真是非常好用，尤其是如果服务部署在家里或者一些不能使用 80 / 443 端口的机器上，可以通过 Origin Rules 匹配对应的主机域名进行重写端口，这样访问的时候就不需要输那烦人的端口啦！

## Workers 和 Pages

Workers 和 Pages 我感觉非常像，CF 中也是放在一个管理里面，Pages 感觉更倾向于放静态资源，不涉及计算逻辑的那种，Workers 则是有一定的处理逻辑，例如对请求进行一些个性化处理。

> Workers 最初是拓展 CDN 的一种方式，后来成为高度可配置的通用计算平台。
>
> Pages 最初是静态 web 托管，后来拓展到 Jamstack 领域。
>
> 渐渐地，Pages 开始拥有更多 Workers 的强大计算功能，而 Workers 也开始加入 Pages 丰富的开发者功能。
>
> 所以，这两项产品的界限变得模糊，让用户很难辨别其中的差别，从而很难根据自己的应用程序需求来选择正确的产品。
>
> 我们将努力实现对 Pages 和 Workers 合二为一，同时将产品、工程和设计团队进行融合，而不仅仅是融合产品本身。

我在 Pages 就是放了几个静态页面做门户，或者仍几个 LLM 的 WebUI，都是很简单逻辑的界面，有很多项目都是同时提供 Worker 和 Pages 两种部署方式，所以我感觉可能也总体上差不太多，但是 Pages 在国内的网络环境下可能会更难。

Workers 可以看作是个 Serverless / 边缘计算平台，在 Cloudflare 的全球网络上构建**无服务器函数和应用**，无需配置或维护基础设施。

Worker 既可以在线编辑运行的 JS 代码，也可以使用官方提供的 CLI 工具（wrangler，更加灵活，支持更全面）进行部署。

一般来说，Worker 都是和 K-V 数据库搭配使用的，毕竟即使无服务应用，完全不存储数据的服务也很少。

目前有大量开源的 Worker 项目，我部署了图床、笔记服务，还有一些乱七八糟例如订阅转换，TG Bot 之类。

## Tunnels 和 Access

现在 Tunnels 合并到 Zero Trust 里了，Zero Trust 里有很多功能，但是目前我也就是只用到了 Tunnels。

Tunnel 可以做什么？简单理解就是一种内网穿透，把内网的服务发布到公网上，但是相比一般的内网穿透，限制比较多，一般就是用作 Web 服务。

优点是不需要公网云服务器，同时自带域名解析，无需 DDNS 和公网 IP。当然就是国内的速度也就那样，算是勉强能用。

按照 Web 界面，创建一个隧道，一步步按照 Cloudflare 的提示走即可。

{{< admonition type=Warning title="QUIC 问题" open=false >}}
Tunnels 默认会尝试使用 QUIC，都知道经典的我大\*自有国情在此，所以这玩意可不兴开啊，所以启动的时候建议强制使用 http2, 以防出现各种奇奇怪怪的问题。
``` sh
cloudflared tunnel run --protocol http2
```
{{< /admonition >}}

如果选择使用的是命令行，关闭命令：`sudo systemctl stop cloudflared`

---

另外我再推荐一个 [DockFlare](https://dockflare.app/docs) 项目，来管理我们的 Tunnels。它可以根据 Docker 的标签自动映射 Tunnels，对于服务比较多的很合适。

准备好一个配置文件，按照文档新建一个自定义 API 密钥，和账户 ID 一起填进去。
``` properties
# === REQUIRED CLOUDFLARE CREDENTIALS ===
CF_API_TOKEN=vJWdiP9N_e9Ow6Ebqiwi3xC4m0ZX_ESz26Hy5GiS
CF_ACCOUNT_ID=9390330d72b32cb9cc413b3db2e6a7b0
CF_ZONE_ID=0bf4bdb5562cf4b89d69a751969a5beb

# === TUNNEL CONFIGURATION ===
TUNNEL_NAME=DockFlare-Tunnel
CLOUDFLARED_NETWORK_NAME=cloudflare-net
```

然后使用使用 docker-compose 启动服务：

``` yml
version: '3.8'

services:
  dockflare:
    image: alplat/dockflare:stable # Or :unstable for the latest features
    container_name: dockflare
    restart: unless-stopped
    ports:
      - "5000:5000"
    env_file:
      - .env
    environment:
      - STATE_FILE_PATH=/app/data/state.json
      - TZ=Asia/Shanghai # Set your timezone
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - dockflare_data:/app/data
    networks:
      - cloudflare-net
    # Optional labels to expose DockFlare itself via DockFlare
    # labels:
    #   - "dockflare.enable=true"
    #   - "dockflare.hostname=dockflare.yourdomain.tld"
    #   - "dockflare.service=http://dockflare:5000"
    #   - "dockflare.access.policy=authenticate" # Example: require login

volumes:
  dockflare_data:

networks:
  cloudflare-net:
    name: cloudflare-net
```

如果你像我一样使用 Portainer 部署，那么 env 的引用记得改为 `stack.env` 才能正确引用。

之后对需要穿透的服务打上标签，加入到 cloudflare-net 网络即可

``` yml
labels:
  - "dockflare.enable=true"
  - "dockflare.hostname=my-app.example.com"
  - "dockflare.service=http://my-app-name:80"
```

同时，DockFlare 的管理页面也可以进行简单的权限控制，还是很方便的。

---

有些服务不方便给所有人看到，可以建一个认证界面，就是 Zero Trust 的 Access 功能，进入 Access 选择应用程序，选择一个自托管类型，根据服务的域名创建一个验证模式即可，例如输入邮箱发送验证码的方式。
## R2 对象存储

官方表示这是分布式对象存储，也就是可以作为静态资源 CDN 使用，尤其是图床场景，许多对象存储的服务商都提供了「免费」的额度，看起来很爽，但是万一被刷之后面对高额的账单就很不爽了：

R2 是 Cloudflare 推出的对象存储服务，主打零出口费用（也就是免流量费）和与 Amazon S3 兼容的 API，适合存储大量数据且需频繁访问的场景，完美解决传统图床的痛点。

- 读取操作（例如下载请求） 1000 万次/月免费，超限后仅 $0.36/千万次
- 改变操作（例如上传）100 万次 / 月，超出 4.50 美元 / 百万次
- 存储免费 10 GB / 月，超出后 0.015 美元 / GB 存储
- 出口流量全免

很多人配合 Workers 使用实现免服务器实现某些服务，例如图床，我使用 [CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed) 部署了一个图床服务。
## AI Gateway

> Cloudflare AI Gateway 为你的 AI 应用提供集中管理和控制。只需一行代码即可连接您的应用，监控使用情况、成本和错误。通过缓存、速率限制、请求重试和模型回退来降低风险和成本。轻松确保可靠性、可扩展性和生产力。

既然叫网关，那就会有传统网关的一些功能，例如：监控，日志，限速，反向代理，请求或响应改写，集成用户系统等。这些功能其实和 AI 关系不大就是把 LLM 的 API 当成了一个普通的 API 进行接入。

针对 AI 进行的特殊优化：
例如限速功能增加基于 Token 的限速，缓存功能增加基于 Prompt 的缓存，防火墙基于 prompt 和 LLM 返回进行过滤，多个 LLM API Key 之间的负载均衡，多个 LLM Provider 的 API 转换。

这些功能在原有的 API 网关就存在类似的概念，不过在 AI 场景下又有了相应的扩展。

Cloudflare 的这款 AI Gateway 主要功能其实就是一个反向代理。

如果你原来用的是 OpenAI 的 API 那么现在你要做的就是把 SDK 里的 baseURL 换成 `https://gateway.ai.cloudflare.com/v1/${accountId}/${gatewayId}/openai` 就可以了。

在这个过程中由于流量进出都是过 Cloudflare 的，Cloudflare 平台上就可以提供对应的监控，日志，缓存等功能。

我最初的目的是借助 Cloudflare 的全球网络可以一定程度隐藏掉源 IP，实现对于一些 OpenAI API 访问受限的区域用这个可以绕过去；但是后面发现有点不对，不知道是不是因为 OpenAI 也是用的 CF 的原因还是 Gateway 转发的时候会携带原始请求的一些信息，导致还是会被 OpenAI 封，后面就弃用了。

