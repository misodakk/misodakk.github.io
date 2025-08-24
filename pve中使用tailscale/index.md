# PVE中使用Tailscale


Tailscale 是一种基于 WireGuard 的虚拟组网工具，简单说就是把异地的设备连接成同一个局域网，让我出门后也能像在家里一样访问我的局域网服务；

最开始是想用 docker 来部署，但是回头一想，直接创建个 CT 容器在 Linux 环境下安装不是更好吗？<!--more-->

ZeroTier 也尝试过，纯是感觉不如 Tailscale 的 Web 管理界面好看，以及 ZeroTier 国内应该无法直连访问了。

Tailscale 是基于 WireGuard 做的，如果纯手撸 **WireGuard** 的话倒是也可以。

## 创建 CT / LXC 环境

使用 debian 模板创建一个 LXC 虚拟环境，保持非特权容器默认配置即可；防火墙肯定要关掉，其他的配置看着选就行；像我这种轻度使用 5G 硬盘，512 内存也够了。

因为 Tailscale 是基于 WireGuard 的，所以我们需要配置 LXC 在 Linux 容器中启用 TUN / TAP 网络设备。
``` sh
# 容器内执行查看主机设备信息
ls -al /dev/net/tun
*** root root 10, 200 *** /dev/net/tun
 
# 修改CT配置，PVE 主机内执行
vim /etc/pve/lxc/<ID>.conf
# 修改下面两个，10:200 和上面的要对应起来
# lxc.cgroup2.devices.allow: c 10:200 rwm
# lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

下一步就是开启 IP 转发，在 LXC 容器中执行：
``` sh
vim /etc/sysctl.conf
# 添加以下配置
# net.ipv4.ip_forward = 1
# net.ipv6.conf.all.forwarding = 1
# 或者执行
# echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
# echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf

# 使其生效
sysctl -p /etc/sysctl.conf
```

作用是可以作为 exit node  使用和开启子网路由功能，exit node 在客户端中选择出口节点就可以了，这样我们访问任何服务都会通过这个 exit node 来代理访问，有点像代理服务器。

启动命令设置子网路由 advertise-routes，作用就是只要连上 Tailscale 的客户端，可以像在家一样访问局域网 IP，不需要使用 Tailscale 虚拟的 LAN IP；

子网路由的最大作用就是只需要有一台设备运行 Tailscale（也就是我配置的这个 LXC 容器），就可以访问整个子网中的任何服务，不需要每个设备/服务都运行一个 Tailscale 来组建虚拟 LAN。

比如我家里路由器配置的 LAN 是 192.168.6.1 网段，设置 exit node 后，随便一台客户端连上，就可以直接使用 192.168.6.x 来访问家里局域网的设备，不需要再额外记 Tailscale 的虚拟 IP 段了，原理自然就是通过我们的这台 LXC 进行数据转发。

PS. 通过这种方式可以实现 Site-to-Site。

另外，如果不配置 UDP GRO 转发，Tailscale 启动的时候可能会有个提示：

> Warning: UDP GRO forwarding is suboptimally configured on eth0, UDP forwarding throughput capability will increase with a configuration change.
>
> See https://tailscale.com/s/ethtool-config-udp-gro

实际上就是让你执行下面的命令来提高性能：

``` sh
apt install ethtool
apt install networkd-dispatcher

# 临时生效，重启后失效
NETDEV=$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")
ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off

# 配置开机自动执行，依赖 networkd-dispatcher
# 检查：systemctl is-enabled networkd-dispatcher
printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")" | tee /etc/networkd-dispatcher/routable.d/50-tailscale
chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale

# 测试效果
/etc/networkd-dispatcher/routable.d/50-tailscale
test $? -eq 0 || echo 'An error occurred.'
```

然后我还看到网友给出了另外一种实现方式，如果使用不了 networkd-dispatcher 那可以尝试一下，我使用的是上面官方的命令。

``` sh
# 查看 UDP 转发及开启 UDP GRO转发
ethtool -k eth0 | grep -e rx-gro-list -e rx-udp-gro-forwarding
ethtool -K eth0 rx-udp-gro-forwarding on
 
# 配置开机自动开启 UDP GRO 转发
# vim /etc/systemd/system/ethtool-config.service
[Unit]
Description=Apply ethtool settings
 
[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -K eth0 rx-udp-gro-forwarding on
RemainAfterExit=yes
 
[Install]
WantedBy=multi-user.target
 
# 启动服务
systemctl enable --now ethtool-config
```

根据 AI 的分析，他们作用是一样的，实现方式不一样，一个是事件驱动（网络接口状态变化时触发），一个是服务驱动（系统启动时执行）。

使用服务驱动就需要硬编码网卡接口了，但是可能更好理解一点。
## 安装 Tailscale

详细介绍参考[官方文档](https://tailscale.com/kb/1031/install-linux)，Linux 下可以使用一键脚本也可以使用包管理，我这里直接使用官方提供的脚本了。

``` sh
# 安装
curl -fsSL https://tailscale.com/install.sh | sh

# 安装基础软件包
apt install ethtool net-tools chrony -y

vim /etc/systemd/system/tailscale.service
# 可以使用 --hostname 指定 hostname
[Unit]
Description=AutoStart tailscale
After=tailscale.service
Requires=tailscale.service
[Service]
Type=oneshot
ExecStart=/usr/bin/tailscale up --authkey=xxxx --accept-routes --accept-dns=false --advertise-exit-node --advertise-routes=192.168.6.0/24
ExecStop=/usr/bin/tailscale down
RemainAfterExit=yes
Restart=on-failure
[Install]
WantedBy=multi-user.target
   
# 设置自动升级
tailscale set --auto-update

# 测试使用 exit node
sudo tailscale set --advertise-exit-node
```

这里认证可以使用默认的 URL 登录，或者在官方 Web 管理台创建一个认证密钥，不过可惜的是好像没有永久有效的，最多也就 90 天，90 天后需要换新的 token；

{{< admonition type=info title="关于登录有效期" open=true >}}
中间机器重启过，导致 Tailscaled 掉了我都不知道。。。。用的时候发现连不上，尴尬。

这次我本打算使用 web 方式登录看看有效期有多久，发现只要登录过 Session 是一直有效的，再次可以不使用 authkey 直接完成登录，那我就测试下不用 authkey 的话是不是可以一直保持，毕竟后台已经禁用过期了。
{{< /admonition >}}

## 开始使用 Tailscaled

启动 Tailscaled 非常简单，完成上述的安装后，只需要执行 `tailscale up` 就可以启动，但是我们可能需要进行一些个性化设置。

tailscale 默认会使用自带的 DNS 设置，也就是 Web 管理台上 DNS 哪一个选项卡里的配置，有一个主要的作用就是配置自定义域名来便捷访问服务，这样就不需要记 tailscale 分配的需要 LAN IP 了，但是如果启用了子网路由后，这一个就显得不是特别重要了，可以使用 `--accept-dns=false` 来禁用。


一切就绪后可以使用下面的命令来检测运行状态：
``` sh
# 手动运行，可以不使用 authkey 使用 url 进行登录
tailscale up --authkey=xxxx --accept-routes --accept-dns=false --advertise-exit-node --advertise-routes=192.168.6.0/24

tailscale netcheck

tailscale status

Tailscale ping xxx

# stop
tailscale down
```

App 中点击设备右上角也可以查看网络连接类型和质量。
## 自建 derp

目前我的网络环境使用 Tailscaled 的官方服务器是可以正常连通的，当遇到极端的网络环境 UDP 打洞失败，需要走服务器转发的方式的时候，官方的服务器的体验并不是那么好，毕竟原因我们都懂。

如果体验不好，可以使用一些商用的 derp 服务器，例如[微林](https://www.vx.link/)就提供 derp 服务器，如果自己有闲置的服务器也可以自己搭建。

我目前可以正常打洞成功，暂时还用不到，但还是记录一下。

[DERP](https://tailscale.com/kb/1118/custom-derp-servers) 是用 Go 写的，所以安装 DERP 需要先配置 Go 编译环境。

准备工作，需要放行需要的端口，80 (HTTP)，443 (HTTPS), 3478 (UDP，用于 STUN)。

``` sh
# 安装 Go
# 如果你的服务器在境内，可以为Go配置代理加速  
# go env -w GOPROXY=https://goproxy.cn,direct
wget https://go.dev/dl/go1.25.0.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.25.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version

# 安装 derp
go install tailscale.com/cmd/derper@main

# derper 命令在~/go/bin 下，可以直接拷到任意目录
# sudo cp ~/go/bin/derper /usr/bin/

# 启动
sudo derper --hostname=example.com
# or 加入验证
sudo derper --hostname=derp.example.com --verify-clients

```

其他的也有使用 [Docker](https://github.com/fredliang44/derper-docker/pkgs/container/derper) 来完成部署 DERP 的，类似的作者也有很多，这里就不再细说了。

tailscale 走的 SSL 加密，主要是为了加密代理数据，所以我们其实可以自签一个 SSL 证书，hostname 直接使用 IP，让人如果有 SSL 的域名那更好了。
``` sh
# 修改自己的 IP
DERP_IP="12.12.12.12"
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout ${DERP_IP}.key -out ${DERP_IP}.crt -subj "/CN=${DERP_IP}" -addext "subjectAltName=IP:${DERP_IP}"

derper --hostname="12.12.12.12" -certmode manual -certdir ./
```

搭建完成后，在管理页面的 Access Controls 中，新增或者修改规则，最后加入：
``` json
{
  // ... other parts of tailnet policy file
  "derpMap": {
    "OmitDefaultRegions": true,
    "Regions": {
      "900": {
        "RegionID": 900,
        "RegionCode": "myderp",
        "Nodes": [
          {
            "Name": "1",
            "RegionID": 900,
            "HostName": "example.com",
            // IPv4 and IPv6 are optional, but recommended, to reduce
            // potential DERP connectivity issues if DNS is unavailable
            // or having issues. Addresses must be publicly routable
            // and not in private IP ranges.
            "IPv4": "203.0.113.15",
            "IPv6": "2001:db8::1"
          }
        ]
      }
    }
  }
}
```

RegionID 可以在 900-999 任选；OmitDefaultRegions 为 true 时，只会使用自建的，不会再使用[官方的 derp 服务器](https://controlplane.tailscale.com/derpmap/default)。

最后使用 `tailscale netcheck` 验证即可。

### 进一步优化

可以创建一个启动、停止脚本，方便进行操作，下面两个版本各取所需。

``` sh
# create /opt/derper/runderper

# open SSL ver
#!/bin/sh
cd /usr/local/gopath/bin
nohup ./derper -hostname <DOMAIN> -c=derper.conf -a :<PORT> -http-port -1 -certdir /usr/local/cert -certmode manual -verify-clients -stun > console.log 2>&1 &
echo $! > app.pid

# disabled SSL ver
#!/bin/sh
cd /usr/local/gopath/bin
nohup ./derper -hostname <DOMAIN> -c=derper.conf -a :<PORT> manual -verify-clients -stun > console.log 2>&1 &
echo $! > app.pid

# 停止脚本 /opt/derper/stopderper
#!/bin/sh
kill `cat app.pid`
rm -rf app.pid
```

如果你需要 SSL，把 SSL 证书上传到 `/usr/local/cert` 文件夹，证书命名格式为 `<DOMAIN>.<SUFFIX>`，将证书文件后缀改为 `crt` ，密钥文件后缀改为 `key`。

创建服务（开机自启 `systemctl enable --now derper.service`）

``` ini
# /etc/systemd/system/derper.service
[Unit]
Description=Derper服务
After=network.target
 
[Service]
Type=forking
ExecStart=/opt/derper/runderper
ExecStop=/opt/derper/stopderper
 
[Install]
WantedBy=multi-user.target
```

参考：[自建 Tailscale DERP 服务教程](https://blog.1l1.icu/2024/04/07/zi-jian-tailscale-derp-fu-wu-jiao-cheng/)
## 自建 Headscale

[Headscale](https://github.com/juanfont/headscale) 可以理解为是开源的 Tailscale，因为官方的 Tailscale 控制端并不是开源的。

headscale 是用 Go 写的，通过源码编译的话需要准备 Go 环境，也可以通过 docker 来部署。

按照[官方文档](https://headscale.net/stable/setup/install/official/)的方式部署非常简单，一路执行即可，我也收集了一下网友的最佳实践，博主自己目前还没有部署，使用的官方的控制端。

``` sh
# 下载二进制版本
VER=0.24.1
wget https://github.com/juanfont/headscale/releases/download/v${VER}/headscale_${VER}_linux_amd64

# 下载同版本的示例配置文件  
mkdir /etc/headscale/
wget -O /etc/headscale/config.yaml https://raw.githubusercontent.com/juanfont/headscale/v${VER}/config-example.yaml
  
chmod a+x /usr/local/bin/headscale
# 创建配置目录
mkdir -p /etc/headscale
# 创建证书和数据目录
mkdir -p /var/lib/headscale
# 创建空数据库
touch /var/lib/headscale/db.sqlite


# 创建 sock，创建文件夹
mkdir /var/run/headscale
# 创建文件
touch /var/run/headscale/headscale.sock

# adduser --no-create-home --disabled-login --shell /sbin/nologin --disabled-password headscale
# 修改 Owner
# chown -R headscale:headscale /var/lib/headscale

# 测试运行
headscale serve

# 启动服务
systemctl start headscale
# 关闭服务
systemctl stop headscale
# 开机自启
systemctl enable headscale
# 查看状态
systemctl status headscale
```

Docker 部署参考：

``` yaml
services:
  headscale:
    image: headscale/headscale:0.23.0-alpha8
    container_name: headscale
    restart: always
    command: serve
    volumes:
      - ./config:/etc/headscale
    ports:
      - 8080:8080
      - 9090:9090
```

配置文件是通用的，之后就可以使用 headscale 命令来创建用户之类的操作了。

需要注意的是 headscale 默认是没有 UI 的，不过有一些第三方 UI 可供选择，例如 [headscale-ui](https://github.com/gurucomputing/headscale-ui)、[headscale-admin](https://github.com/GoodiesHQ/headscale-admin)

客户端使用：

``` sh
# 浏览器打开 apple/windows
http://你的域名或ip:8080/apple
# 执行页面中的命令行
tailscale login --login-server http://你的域名或ip:8080
# 获取返回的命令
headscale -n 命名空间 nodes register --key nodekey:上面这行命令返回结果的key
# 到Headscale服务器上执行返回的命令

### linux
tailscale up --login-server=http://你的域名或ip:8080 --accept-routes=true --accept-dns=false
# 获取返回的命令
headscale -n 命名空间 nodes register --key nodekey:上面这行命令返回结果的key
# 到Headscale服务器上执行返回的命令
```

### 配置文件

headscale 的主要配置文件：

``` yml
# /etc/headscale/config.yaml

# 这里填写你的实际外网地址,域名或ip都可以
server_url: http://XXX.XXX.XXX.XXX:8080  
# Headscale 要绑定/监听的 Port
listen_addr: 0.0.0.0:8080
metrics_listen_addr: 0.0.0.0:9090
# 为客户端分配的 IP 段
ip_prefixes:
  - fd7a:115c:a1e0::/48
  - 10.1.0.0/16
randomize_client_port: true
# 修改对自己来说方便的DNS，可以保持默认
dns_config:
  # 使用本地 DNS
  override_local_dns: true
  nameservers:
    - 223.5.5.5

# 如果启用，必须配置其他部分，如 https、TLS
derp:
  server:
    enabled: false
# 建议关闭Magic DNS，否则有可能造成客户端无法正常上网
magic_dns: false
# 修改Socket存储位置
unix_socket: /var/run/headscale/headscale.sock
```

要使用 DERP [参考文档](https://blog.goalonez.site/blog/Tailscale%E8%87%AA%E5%BB%BA(Headscale%E5%8F%8ADerp).html#%E5%9C%A8headscale%E9%85%8D%E7%BD%AE%E8%8A%82%E7%82%B9%E4%BF%A1%E6%81%AF)，注册服务实现自启动：

``` properties
# /etc/systemd/system/headscale.service
[Unit]
Description=headscale controller
After=syslog.target
After=network.target

[Service]
Type=simple
User=headscale
Group=headscale
ExecStart=/usr/local/bin/headscale serve
Restart=always
RestartSec=5

# 可选的权限和安全配置
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/headscale /var/run/headscale
AmbientCapabilities=CAP_NET_BIND_SERVICE
RuntimeDirectory=headscale

[Install]
WantedBy=multi-user.target
```

常用命令：

``` sh
# 启动服务
systemctl start headscale
# 关闭服务
systemctl stop headscale
# 开机自启
systemctl enable headscale
# 查看状态
systemctl status headscale

# 显示节点列表
headscale nodes ls
# 删除节点
headscale nodes delete -i  <id>
# 启动服务
systemctl start headscale
# 开机自启
systemctl enable headscale
# 查看状态
systemctl status headscle
# 创建命名空间
headscale namespaces create <namespace>
# 查看命名空间列表
headscale namespaces list
# mac ping
/Applications/Tailscale.app/Contents/MacOS/Tailscale ping 100.64.0.2
```

参考：[Tailscale 自建 (Headscale 及 Derp)](https://blog.goalonez.site/blog/Tailscale%E8%87%AA%E5%BB%BA(Headscale%E5%8F%8ADerp).html)

## 与魔法软件共存

虽然可以使用 exit node 节点代理所有流量，但是家用带宽的上行一般是不够用的，而 Tailscaled 默认也会创建一个 Tun 和魔法工具一样的套路（但是正常的 OpenVPN 之类的是可以共存的），同时运行他们一般是冲突的。

有一种解决方案是将 Tailscaled 作为一个 socks5 代理的方式运行，但是我感觉不妥。感兴趣可以搜索一下。

移动端就直接建议魔法软件使用系统代理模式，不要用 Tun 了，把 VPNService 留给 Tailscaled。

PC 端可以在魔法软件尝试配置排除 Tailscaled 的 100.x 网段（tun 下的 exclude-interface 和 route-exclude-address 配置），然后关闭本地 MagicDNS（取消勾选 Use Tailscale DNS settings），但是一般情况应该也没啥影响。

``` yaml
# 参考配置
tun:
  enable: true
  exclude-interface:
    - Tailscale  # macOS 名字可能是 utun 固定前缀的
  auto-route: true
  auto-redirect: true

route-exclude-address:
  - 100.64.0.0/10
```

或者考虑曲线救国，搭建一个 OpenVPN 连过去。

