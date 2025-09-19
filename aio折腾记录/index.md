# 一台小主机的 AIO 之旅


All in boom 来啦，PVE + OpenWrt + 黑群晖 DSM + Windwos10 + Ubuntu + Docker + HAOS。
亡命之徒直接冲 RAID0 ！！
硬件：GMK M6 + D2-320 磁盘阵列 2x4T。<!--more-->
Q："文章可以多配些步骤图吗，更直观"

A："不行，我懒人，太麻烦，省电模式中，再说吧"

## 安装 PVE

去官网下载 ISO 镜像，我使用 Ventoy 来进行引导，也可以通过写盘工具直接把 ISO 写到 U 盘，这里最开始我卡 loading，原因是 Ventoy 的版本太老了，升级一下就正常了（不需要格盘）。
安装 PVE 的教程很多，也没什么注意的点，IP 只要写对就没啥问题，安装的时候 Ventoy 引导就保持默认第一个。

参考视频: [利用PVE虚拟机，来打造属于自己的All In One系统吧！](https://www.bilibili.com/video/BV1bc411v7A3/?share_source=copy_web&vd_source=bb09006a95944f3a8aec376ed6eeb2e7)

{{< bilibili BV1bc411v7A3 >}}

PVE 装完后的两个比较好用的工具：

- [pveTools](https://github.com/ivanhao/pvetools)
- [pve_source](https://bbs.x86pi.com/thread?topicId=20)

就是改个源，去除个订阅提示，硬盘直通可以使用 pvetools。

关于标记：点数据中心 - 选项 - 标记样式设定 - 完整；然后去每个节点上最上面一栏加标记即可。

## OpenWrt

软路由还是建议安装一下，但是不推荐主力使用，做一个旁路网关就好，例如安装魔法后解决后面 Docker 拉不到镜像等问题。
软路由推荐不良林的视频，说的很清楚：[视频链接](https://youtu.be/JfSJmPFiL_s?si=09dZqcEKDu1anurs)

{{< youtube JfSJmPFiL_s >}}

由于官方固件太简洁，这里选择 immortalwrt，首先去[下载固件](https://firmware-selector.immortalwrt.org/?version=23.05.4&target=x86%2F64&id=generic)，建议选择 COMBINED-EFI (SQUASHFS-COMBINED-EFI.IMG.GZ)版本，因为是 X86 架构直接选。

上传镜像，然后使用命令:`qm importdisk 100 /var/lib/vz/template/iso/openwrt.img local-lvm` 导入到虚拟机。

安装魔法三件套，openClash、homeProxy、passwall

网卡优先半虚拟化不行的话就切回 E1000，CPU 能 Host 就 Host。

> 经过测速，起码 PVE8 是不需要 img2kvm 这个工具了，直接 qm 命令导入即可，一般情况下 2 核 1G 的配置够用了。

不过到现在我也还是没搞明白 OpenWrt 的配置，还是挺复杂的，理解为啥有那么多人用爱快了，后面再研究吧，现在网卡都让我删没了。。。但是能用！

---

安装 DDNS-Go 配合路由器的端口转发可以实现外网访问，需要一个域名、光猫是桥接，有公网 IP。

## DSM

采用的是 [GXNAS](https://wp.gxnas.com/11849.html) 的引导文件，或者可以看看著名的原生 RR 引导，复活版的哦： https://github.com/RROrg/rr

GXNAS 大佬的镜像直接是支持虚拟网卡的，所以直接选半虚拟化就行，性能更好。

作为 All in boom，我这里当然选的是 RAID0，直接拉满，数据安全什么的全靠我定期冷备份。

启用 MacOS 的时间机器支持：[官方文档](https://kb.synology.cn/zh-cn/DSM/tutorial/How_to_back_up_files_from_Mac_to_Synology_NAS_with_Time_Machine)，总结就是新建好共享文件夹，给新建个用户(配额还是建议写一下 300-500G 够用了)，开启 **SMB** 服务，高级设置里**启用 Bonjour 服务发现**和**启用通过 SMB 进行 Bonjour Time Machine 播送**，设置下文件夹完成。

备份 PVE：和时间机器差不多，新建个文件夹，开启共享的 NFS 服务，然后在 NFS 权限里添加 PVE 的 IP 权限，映射为 admin，所有能勾的都勾上。然后去 PVE 存储里添加 NFS 即可，内容选 VZDump备份（或者你还想存其他东西）。
PS：不要备份 DSM 本身，会出问题，快照最快最小，需要本体才能恢复，停止和挂起基本一致，可以 PVE Boom 了进行恢复，建议停止模式，备份时虚拟机会暂停几分钟直到备份后启动。

## Windows/Linux

我这里装的是 Win10 LTSC，Win11 有点复杂，要开 UEFI + TPM，还要登录，就不想搞的那么复杂。

PS：目前最新的 Win11 优化的也很到位了 2G 内存跑轻度任务都是很流畅的。

VirtIO 驱动镜像可以通过访问页面找到[download the latest stable](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso) 点击下载，详情移步 https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers

Win 一般肯定是通过远程桌面使用，在计算机右键属性里打开即可，就可以使用 mstsc 进行连接了；Win10 的 LTSC 刚装上可能会导致风扇狂转，这是一个 bug，更新下系统重启后就好了。

---

Linux 没啥可以说的很简单，一路下一步和实体机没区别，我装了个 Ubuntu Server。
Docker 安装可以使用一键脚本 `bash <(curl -sSL https://linuxmirrors.cn/docker.sh)` ，换源也有类似的一键换源脚本 `bash <(curl -sSL https://linuxmirrors.cn/main.sh)`。

需要允许远程 ssh 登陆的话修改 `/etc/ssh/sshd_config` 将 `PermitRootLogin prohibit-password` 改为 `PermitRootLogin yes` 然后重启服务 `service ssh restart`

## Docker

这里我选择使用 LXC 来创建一个 Docker，首先要下载一个 CT 模板，我使用的是 Ubuntu 的，因为要使用 DockerHub 这里网关使用前面 OpenWrt 的；一定要去掉五特权容器的勾；完成后去选项里面把功能里的嵌套之类全勾上。

需要注意的是，LXC 虚拟虽然效率高但是不是完全的虚拟，还是会共用宿主机的一些东西或者有限制，所以 Linux 我没用 LXC，Docker 的话感觉还是可以的。

为了保证 Docker 可以启动，需要在 PVE 修改一下对应 LXC 的配置，在 `/etc/pve/lxc/{CTID}.conf` 中加入：

```
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```

大部分情况下 Docker 是可以正常运行了。

使用快捷指令 `bash <(curl -sSL https://linuxmirrors.cn/docker.sh)` 安装 Docker，源就不换了。

安装完毕后先装一个 portainer：

```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data outlovecn/portainer-cn:latest
```

服务推荐：跑一个 [speedtest](https://github.com/librespeed/speedtest/blob/master/doc_docker.md) 测速下局域网的网速，或者使用 iperf3 也可以。

## HAOS

建议使用[冬瓜 HAOS](https://bbs.hassbian.com/thread-24065-1-1.html)，直接使用 PE 版本的 ISO，一键安装。

还没时间仔细研究，一些插件：

- 米家 - Xiaomi MIoT
- 美的美居 - Midea AC LAN
- 海尔智家 - [haier](https://github.com/banto6/haier)
- 格力 - [Gree Climate](https://www.home-assistant.io/integrations/gree/)
- 石头 - [homeassistant-roborock](https://github.com/humbertogontijo/homeassistant-roborock)
- HomeKit Bridge - 设备接入苹果家庭

还没仔细研究，貌似有点复杂。

## Tailscale

最开始是想装到 Docker 里的，没发现特别好的文章，也不想自己搞了，后来看很多都是装 OpenWrt，有大部分是爱快里装的，比较简单，但是显然不适合我，但是有两个项目感觉不错：

https://github.com/CH3NGYZ/tailscale-openwrt

https://github.com/adyanth/openwrt-tailscale-enabler

不过很可惜的是装完我能登录，但是配置的子网路由无效，大概可能是和我 OpenWrt 前面折腾删掉网口有关，最后还是放弃；

最终采用了[这位大佬](https://acchw.top/posts/3940/index.html) 的方案，使用 LXC 来安装，和之前配置 Docker 一样，需要在 `/etc/pve/lxc/{CTID}.conf` 文件中加入开启 TUN：

```conf
# 这个 10 200 可以通过在 pve 上执行 ls -al /dev/net/tun 获得，一般都是 10 200
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

还需要在 LXC 中的 `/etc/sysctl.conf` 开启转发：

```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1

# 执行生效
# sysctl -p /etc/sysctl.conf
```

然后使用官方的脚本安装：`curl -fsSL https://tailscale.com/install.sh | sh`
执行 `tailscale up --authkey=xxxxx --accept-routes --advertise-routes=192.168.0.0/24` 开启，记得换成自己的网段和 authkey。
配置自动启动：
使用 systemd 来进行开启自启。在 `/etc/systemd/system` 下新建一个配置文件: `tailscale.service`, 文件内容如下：

```
Description=AutoStart tailscale
   After=tailscale.service
Requires=tailscale.service
   [Service]
   Type=oneshot
   ExecStart=/usr/bin/tailscale up --authkey=你的authKey --accept-routes --advertise-routes=你的转发范围
   ExecStop=/usr/bin/tailscale down
   RemainAfterExit=yes
   Restart=on-failure
   [Install]
   WantedBy=multi-user.target
```

然后执行如下命令：

```shell
systemctl enable tailscale.service
systemctl start tailscale.service
```

登录后首先去 Web 控制台重命名设备名字，关掉登录过期，开启自定义路由转发，之后就可以像访问内网那样在外面访问内网服务了。

## LXC

修改 LXC 的主机名：

```
/var/lib/lxc/容器的id编号/config
/etc/pve/lxc/容器的id编号.conf
```

只修改后面这个文件也可以，建议都改。

串流，未完待续。

