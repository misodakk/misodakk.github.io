# Android Root 技术派系


目前各家的定制 Android 系统已经非常完善，一般人正常使用下 Root 的必要性很低，并且灰产、外挂的原因，现在手机厂商都在收紧 BL 锁的权限，所以我觉得大部分人是完全用不到的。

首先定个调，只是个人观点，目前（起码国内的）玩机圈已经快烂完了，好多大佬已经退圈，用户也是各种神操作、神发言，大部分都是伸手党，现在的年轻一代，已经很少有 90 及之前的那种折腾精神；不久的将来 Root 可能成为历史，iOS 越狱也一样。

如果你当前可以拿到 Root，操作之前务必备份，推荐 [DataBackup](https://github.com/XayahSuSuSu/Android-DataBackup)

## 背景知识

在 Android 12 及之前，大多是 boot.img 同时打包了 Kernel (内核二进制) 和 Ramdisk (根文件系统/init)。

从 Android 13 开始，Google 推行通用内核镜像（**GKI 2.0**），为了把 Google 官方维护的内核与 OEM 厂商定制的 ramdisk 分离，进行了拆分：

- **boot.img**：只存放纯粹的通用 Linux 内核二进制 (kernel)，无 ramdisk。
- **init_boot.img**：专门存放全局的 ramdisk（包含第一阶段 init 引导脚本）。

再说手机的内核版本，可以理解为手机上先跑了一个 Linux 系统，用来处理和具体硬件之间的交互，在这个系统上跑了 Android 系统；

内核指的就是这个定制的 Linux 版本，一般**出厂就固定了，不会再更新**，毕竟之间和硬件交互，出问题会非常麻烦。

### LKM

Linux 原生机制，Loadable Kernel Module（可加载内核模块，即 `.ko` 文件）。

它允许你不在一开始就把驱动编译死在内核里，而是在系统运行中动态地把驱动（`.ko`）“插”进内核。

可以认为，只要是跑 Linux 的，比如 Android 系统手机，就支持这个特性。

### GKI

Generic Kernel Image（通用内核镜像）。Google 为了解决 Android 碎片化推行的一项**工程规范**（从 Android 12 试行，Android 13 强制要求 5.10+ 内核）

以前每个芯片厂商、每个手机品牌都魔改内核，互不通用。GKI 规定：手机底层的 Linux 内核（boot.img）必须使用 Google 官方编译的标准内核镜像，OEM 厂商一律不准魔改内核本体！

厂商自己的硬件驱动，全部编译成 `.ko` 模块，放在 `vendor_boot` 分区，开机时挂载加载。

在 GKI 1.0 时代，Google 初次尝试推行通用内核镜像（GKI），管控并不是很严格，芯片商、手机商依然可以进行一定程度魔改；

到 GKI 2.0 时代，也就是**内核 5.10 及以上 / Android 12 及之后**，技术完全成熟，Google 开始 **“硬强制” 执行**。

------

如果你使用 KSU 分支，那么如果你是 GKI 2.0 体验会好很多，老机器需要自己编译内核。

### Zygisk

Zygote 是 Android 系统中用来启动所有应用的核心进程，而 Zygisk 是集成在 Magisk 中的一项功能，用于将代码注入到这个核心进程中。

KernelSU 和 APatch 因为本身没有常驻的用户态特权守护进程，因此它们将 Zygote 的注入能力解耦成了独立模块——**ZygiskNext**，稳定性已经和 Magisk 的 Zygisk 没有什么区别。

## 现代 Android Root 工具

目前比较流行的就是 **Magisk、KernelSU、APatch** 及其下游生态

- 上古时代： SuperSU，各类一键 Root 工具，时代的眼泪

- 现代用户态时代 (2016~)：Magisk

  兼容性无敌

- 现代内核态时代 (2022~)：KernelSU、APatch

  原生内核级挂载，系统开销低，挂载更干净

特殊的，例如 SKRoot 主要强化隐蔽性，在安全逆向、灰产与游戏外挂界极火。

如果你是 GKI 2.0 设备（内核版本 5.10 / 5.15 / 6.1+）： 首选 KernelSU-Next / KernelSU。

如果不想换内核，首选 APatch。

剩下的所有情况，例如老机器非 GKI 内核、内核版本低，无脑选择 Magisk。

### Magisk

现在除了老手机或者特别难搞的机器，一般不是第一选择了；因为运行在用户态，**特征极其明显**，非常容易被检测，当然也有成熟的一键隐藏包，但还是过于折腾

不过因为成熟稳定、设备通用性极高，基本就没有不能刷的设备。门槛最低，不需要考虑内核版本，一键刷入 init_boot / boot 即可使用

衍生版本：

- Magisk Alpha

  前沿灰度测试分支

- Kitsune Mask（原 Magisk Delta / 狐狸面具）

  恢复并强化了类似旧 MagiskHide 的机制，支持挂载层隐藏与独立隔离，防检测能力优于官方 Magisk。

原理简述：

Magisk 的生命线是 /init 进程。它必须把自己的 magiskinit 替换或注入进 ramdisk 中。

在 Android 13+ 机型上，ramdisk 移到了 init_boot.img，因此 Magisk 必须去修补 init_boot.img。

### KernelSU

俗称 KSU 运行在内核态，隐蔽性极高，用户态完全感知不到 su 二进制的存在；未授权 App 即使扫盘也找不到痕迹。

依赖编译进内核或通过 LKM (加载型内核模块) 加载，对内核版本有严苛要求（最好是 GKI）。

衍生版本：

- KernelSU-Next

  集成了官方拒绝合并但社区刚需的功能，例如深度整合 SUSFS、非 GKI 设备补丁、扩展 Root 授权规则与提权接口。

- SukiSU

  针对旧内核（Non-GKI 4.14 / 4.19 设备）做了大量的 Backport 移植支持；深度内置对各种环境检测，追求开箱即用的极高隐蔽性。

- ReSukiSU

  SukiSU 争议不断下的产物

- [MKSU - Magic KernelSU](https://github.com/5ec1cff/KernelSU)

KSU 有两个工作模式，网络是很多都按 GKI 和 LKM 来区分，其实这不太对，GKI 其实就是指**内核源码内置 (Built-in)** 技术，沿用这个说法那就可以认为：

- GKI 刷机（**编译进内核**）：

  **KSU 根本不需要修补任何镜像**，直接用包含 KSU 的自定义内核**替换整个 boot.img**。

- LKM（**内核可加载模块**）：

  KSU 被编译成一个独立的内核模块文件 ksu.ko；内核自己不会主动加载外部模块，必须在根文件系统挂载阶段（**Ramdisk**）植入引导挂载脚本，Android 13+ 版本对应 init_boot.img

KSU 作者主张最小权限、安全第一，对非 GKI (旧内核 4.x/3.x) 官方不予支持，也不原生集成防检测黑科技。

如果没有特殊需求，优先官方版本 KSU。

------

高通、联发科、三星各个厂商早期对 Linux 内核做了极其严重的面目全非的魔改。系统调用路径、结构体偏移各不相同，所以官方默认不支持非 GKI。

但是，如果你会自己编译，可以自己根据内核源码集成 KSU 进行编译；从理论上讲，GKI 肯定是更稳定兼容性更好，这是对 KSU 来说的，但是大部分手机厂商都会对内核进行特定优化，刷了 GKI 大概率会出现耗电增加或者一些奇奇怪怪的问题。

除非你有明确的目的，否则不要轻易刷 GKI。

### APatch

运行在内核 + 用户态；兼具 KernelSU 的隐蔽性与 Magisk 的易安装性，**不需要专门编译内核，直接修补 boot.img**。

利用 **KernelPatch** 技术（它是一个纯粹的**二进制反编译与修补工具**）。在不重新编译内核的情况下，反汇编解析并静态/动态 patch 预编译的 kernel 二进制文件。同样在内核提供调用门（SuperCall）。

因为是不需要内核源码，直接注入的方式，部分特殊内核可能修补失败导致卡第一屏。

因为只，所以只需要修补 boot.img 即可。

衍生版本：

- FolkPatch

  预置了更多的社区补丁集；针对部分小众 SOC/特异机型的 boot.img 解包做兼容性修补；部分版本内置对 KPM (KernelPatch Module) 的增强调度支持。

- WebAPatch

  针对免装终端或不想装繁琐工具箱的用户，提供纯 Web 端的 boot.img 静态修补器。

原理简述：

APatch 的核心是 KernelPatch，它的靶标是 Linux Kernel 二进制本身，而非 ramdisk，在汇编指令级别寻找内核函数的挂钩点（Hook），把自己的代码静态写死进 kernel 镜像。

无论什么版本，kernel 永远存放在 boot.img 里。因此 APatch 永远只需要你提取 boot.img 进行修补。

### 注意事项

Magisk 运行在用户态，所以不支持内核级模块 (KPM)，不过由于是最早一批 Root 工具，对于各种模块来说可以说是规则制定者，兼容性 100%。

对于 KPM 内核级模块，除了 Magisk 原理上不支持，KSU 需要重新编译内核，所以也可以认为不支持，只有 APatch 系支持。

Magisk 内置原生 Zygisk，剩下的那俩，基本都需要 ZygiskNext 模块来支持运行时注入。

口碑来说，KernelSU-Next 有一定的争议，被说是缝合怪，虽然不够优雅，但是默认 SUSFS、老内核支持等扩展对小白来说还是非常有用的。

SukiSU 则争议更大一些，早期被指是代码屎山，有一部分原因也是过度宣传，但是对纯小白用户确实也友好；甚至还诞生了 ReSukiSU 重写版本。

## 模块

简单说，模块可分为三类，一般我们指的都是应用级模块，即 Xposed / LSPosed 模块。

> 在“模块”层面，LSPosed 模块本质上就是 Xposed 模块，两者用的是同一套标准；区别不在模块本身，而在它们背后的“运行框架

1. 内核级模块 KPM

   修补内核功能（如 SUSFS 隐匿、BBR 加速、驱动挂载）

2. 系统级无痕挂载模块（Magisk 规范模块）

   替换系统文件、改字体、改音效、Hosts 屏蔽、温控调度、欺骗设备机型，格式常见为 zip

3. 虚拟机进程级 Hook 模块（应用层 Xposed 模块）

   改微信功能、去开屏广告、Hook 应用内部方法、修改状态栏图标行为，一般为 app 的形式出现

这里重点说第三个，也就是 Xposed / LSPosed 模块，它的原理是**注入 Android 应用的母进程（Zygote），动态修改 Java 运行时的字节码/函数**

Xposed 大约有十几年的历史了，可以认为是模块的一个接口规范，而 **LSPosed** 是现代开发者为了在 Android 8 ~ 15+ 上继续运行 Xposed 模块，而全新重构开发出的**新一代 Xposed 运行时框架**。

相比原版，LSPosed 是白名单机制，即由原来的：
 手机里启动的任何一个 App（包括计算器、日历、银行）都会被强行塞进 Xposed 运行环境。
 优化为：
 模块想改谁，就只注入谁。未勾选的 App 进程绝对干干净净，完全不加载任何 Hook 代码。

它必须依赖 Zygisk 实现即改即生效的热加载机制

24 年 1 月，LSPosed 团队因为社区环境恶化，大量小白用户将各种不兼容、三方魔改系统（如澎湃 OS、ColorOS）导致的闪退发 issue 辱骂开发者，加上某些灰色黑产团队对其代码的盗用与攻击，核心开发团队心灰意冷，决定公开停更。

目前大部分使用的是 [JingMatrix](https://github.com/JingMatrix/Vector) 版（原 LSPosed 团队成员接盘），或者 [Irena 分支](https://github.com/re-zero001/LSPosed-Irena)

------

常见模块：

- Play Integrity Fix (PIF)

  解决 play 商店无法使用

- Scene

- Shamiko

- LuckyTool

- Chimi / Cemiuiler / HyperCeiler

- Fake Location

- 隐藏应用列表 (HMA - Hide My Applist)

- Surfing

- Zygisk Next

- 自动墓碑后台/Frozen

- Thanox

其他的还有很多用于隐藏的模块，这里不列举了，效果会实时变化，根据自己需求网上搜吧。

关键词：Tricky Store (TS)、Tricky Addon Enhanced、TEESimulator

墓碑后台机制，收费有 Freezer / NoActive。

App 推荐：MT 管理器、爱玩机工具箱

------

如果仅仅是想对一两个特定应用使用 Xposed 模块（不想给整台手机 Root）： 社区继续演进着基于 LSPosed 核心的 LSPatch 以及下游 NPatch。

它们通过直接将 Xposed 核心注入并重新打包目标 APK，实现免 Root 运行 Xposed 模块。

------

基于目前的环境，大部分模块作者为了避免麻烦，都走私域传播，例如各种群、频道等。

一些仓库索引：

- MMRL (Modern Module Repository Lab) 
- Xposed Module Repository (GitHub 官方索引)
- KernelSU-Modules-Repo

## 临时 Root

所谓“不解锁 BL 获取临时 Root”（通常被称为 **Temp Root** 或 **运行时提权**），并不是通过正常刷机手段装上了 Magisk/KSU，而是**利用了系统未修复的底层安全漏洞，在内存中强行劫持了特权进程**。

它一般是纯内存提权，用一句话总结它的核心：**“只要一重启，手机立马恢复原状，不留任何痕迹，也无法实现开机自启。”**

早期各种一键 Root 时代，本质上就是把当时公开的 Linux 内核提权漏洞打包成 App。手机点一下提权，在不解 BL 的情况下拿到临时 Root，然后再把 su 强行写入当时的 system 分区。

现在为什么不行了？

不过当发现漏洞时，通过内核洞拿到临时权限后，直接对底层 `devinfo` 分区改写字节，实现重启后永久解锁 BL。参考小米的解锁节。

## OTA

任何现代 Root 方案，如果“直接放任系统点更新并重启”，OTA 之后 100% 会丢 Root。

现代 Android 全部采用 **A/B 分区无缝更新（Seamless Updates）**：

1. 比如你现在在 **A 槽位**（里面刷了有 Root 的 boot/init_boot）。
2. 系统收到官方推送，会自动在后台把新系统的全部镜像写入闲置的 **B 槽位**。
3. 官方更新包写入 B 槽位的是**厂商纯净原厂镜像**，没有任何 Root 代码。
4. 你点击“重启立即生效”，手机直接切换到 **B 槽位开机** → **Root 自然荡然无存**。

为了在 OTA 后不丢 Root，必须在**系统写入 B 槽位完成、但千万还没点重启的那一刻**，把 Root 补丁“隔空安装到另一个槽位”。

Magisk 有成熟的机制保持 OTA 后可用，KSU 的 LKM 模式也大多都可以支持，GKI 方式比较难，APatch 也有对应的解决方案，就是需要更加小心。

但是无论那种方案，都是有一定风险的，必须使用全量 OTA 包，增量容易校验不过；必须在重启之前操作，最后是一定要备份。

------

需要注意的是，OTA 不会回锁 BL，官方更新包写入的都是**常规系统软件分区**，BL 的锁状态（Locked / Unlocked）记录在极其底层的**专用保护分区/安全硬件区**中；

Google 规范要求 Bootloader 的锁定/解锁只能由用户在 **Fastboot 模式下物理介入确认**，绝不允许系统在后台自动化执行。

## 旧设备

我使用我的一台旧 [K40 内核](https://github.com/AlirezaParsi/pocof3) 4.19 的设备测试，KSU 官方需要自己 Fork 仓库编译内核，不过也不是多难；

反而 KSU Next、Suki SU 这些官方文档写着支持低内核设备的分支文档做的真的太差了；反而 ReSukiSU 相对还好一些。

FolkPatch 文档是最详细的，也不需要编译内核，我是非常推荐的。

