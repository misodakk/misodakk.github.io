# Win11使用虚拟机遇到的一些问题


最近想用虚拟机装个 XP/7 怀旧一下老游戏，没想到遇到这么多问题。。。

首先我现在用的是 Win11 系统，除了家庭版，其他都是默认开启 Hyper-V 和 WSL2 的，既然 VM 现在免费了，那么我当然优先考虑使用 VM 来装。

话说现在博通收购 VM 以后，虽然免费了，但是下载是真难下，需要注册个账号，官网现在也乱的很，也许之后慢慢会正常吧，放一个[下载地址](https://support.broadcom.com/group/ecx/free-downloads)
<!--more-->

安装 VM 的时候默认它会出一个提示，反正意思就是和 Hyper-V 冲突，使用他们的兼容模式，如果没有，那说明你没有开启 Hyper-V。

> Hyper-V 是微软的虚拟化平台，也可以说是 Windows 自带的虚拟机，类似 Win7 时代的 Virtual PC，类比 ESXi 和 KVM 这些，本来是给服务器用的。
> 
> Hyper-V 它是 type-1 类型，直接在硬件层运行，可以理解为是你电脑上跑了个 Hyper-V 然后它虚拟出来了个 Win11 系统给你用，如果你新建一个虚拟机，那么这个虚拟机和宿主机 Win11 是同级别的，共享硬件，尤其是显卡共享就简单了；
> 
> 所以 Hyper-V 的性能会比较高；但是同时也有人指出这样宿主机的游戏性能会降低，但是一般人应该感觉不出来。

但是话说回来，Hyper-V 毕竟是微软搞的，如果想虚拟个 Linux 或者其他非 Win 的系统可能就不是那么好用了，易用性也不如 VM。

Hyper-V 我也测试过虚拟个 XP，效果也不是很好，可能微软已经放弃对老系统的支持了吧，并且确实不如 VM 顺手。
VMware Workstation Pro 以及 VirtualBox 这类软件是采用的 VT 虚拟化技术，某些场景下可能是个更好的选择。

VM 现在虽然可以和 Hyper-V 共存，但是我实际使用下来感觉不太行，尤其是虚拟 XP 更是非常的卡。

## 卡顿问题
装完 XP 后感觉非常卡顿，声音也断断续续，看了下 Win10 应该还好，Win11 尤其严重；

既然知道了 Hyper-V 的原理，我也有考虑是 VT 和 Hyper-V 的兼容性问题，当然也有人说和 VM 的版本有关系，我换过 VM16、VM15，效果是有的，但是不明显。

于是我决定关闭 Hyper-V 和 WSL，毕竟这俩我用的也不多，非要用还是直接 VM 虚拟一台 Linux 或者直接用家里的 PVE 上的 Ubuntu 吧。

折腾到后来才发现一个问题，Win 下的 Docker 现在是依赖 Hyper-V 的，如果把 Hyper-V 关了就没法用 Docker 了，于是来来回回折腾了好几次。

后来看到有网友说即使是 Win11，与 Docker 的兼容性还是不好，默认都会把数据、镜像啥的塞到 C 盘，性能占用也很大（有老哥说 Hyper-V 虚拟机不限制内存的话默认是有多少吃多少，这个我没有求证）

最后我还是决定关掉 Hyper-V，一来 Hyper-V 虚拟机我确实用着不顺手，即使使用三方 NanaBox 类似的软件管理也还是不习惯，性能对我来说倒是其次的。

Docker 的问题可以虚拟一个 Debian 来搞，或者家里有 PVE 的话这个其实也省了。

## 关闭 Hyper-V
首先我还是建议先去 Win 的安全中心里的防护里把内存隔离关了，我觉得用处不大，可能还影响性能；
听老哥说如果开着防护像国内腾讯、网易的游戏进行作弊扫描的时候也是改不了内存的，有的游戏可能会强制你关掉才能进；亦或者有的防作弊检测还会扫盘，这个真是有点恶心人，题外话了。

另外一个是使用各种安卓模拟器的时候可能也会和 Hyper-V 冲突，所以索性给关掉吧。

检查 VT 状态可以使用 [CPU-V](https://leomoon.com/downloads/desktop-apps/leomoon-cpu-v/) 来查看。

找到 控制面板→程序→启用或关闭 Windows 功能，然后关闭 “Hype-V”、“Windows 虚拟机监控程序平台”、“适用于 Linux 的 Windows 子系统” 和 “虚拟机平台” 这几项，然后重启。

这时候再看应该就没有 Hyper-V 选项了，然后右键此电脑→管理→服务和应用程序→服务，往下翻，将如图所示有 Hyper-V 字样的服务全部禁用。

如果效果还是不好，可以打开 PoweShell 执行
``` shell
bcdedit /set hypervisorlaunchtype off
```

还是不行的话就使用 bat 脚本强制卸载掉

``` bat
Dism /online /disable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
pushd &quot;%~dp0&quot;
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum &gt;hyper-v.txt
for /f %%i in (&#39;findstr /i . hyper-v.txt 2^&gt;nul&#39;) do dism /online /norestart /remove-package:&quot;%SystemRoot%\servicing\Packages\%%i&quot;
del hyper-v.txt
```

## 后记
关闭 Hyper-V 后，我下载了最新的 VM 进行重装，目前现在一切正常，感觉不到卡顿了，另外我又下载了一个 Debian 的镜像给装到了 VM 里面，并且安装了 Docker 和 Portainer，使用上也没什么问题，暂时就先这样。

如果效果再不好我还打算下个 VirtualBox 来试试的，目前看来是用不上了。
