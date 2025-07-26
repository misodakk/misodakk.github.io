# 开发中的环境版本管理工具


如果经常需要在各种开发项目之间切换，而这些项目又各自需要不同的运行环境，尤其是不同的运行时版本或依赖库，那么没有一个环境管理工具就会变得特别混乱。

<!--more-->

作为一个主业 Java 方向的，看到 Node.js/Python 的包管理，真的是头大，处理版本问题那是小心翼翼，轻易不敢升级，一不小心就跑不起来了。。。

后来接触到有一些管理软件可以快速切换版本，但是每一种语言装一个还是有点麻烦，毕竟像我这样同时有多个语言环境需求的肯定不在少数，每一个工具都是学习成本；
后来我发现有一种多语言环境管理工具，可以把各个语言统一管理，并且他们之间的配置文件大多能兼容，这个就是我推荐的，放在最后。

## Node.js

前端（当然也能做后端了）我只能说发展太快了，我这种非专业的根本追不上，各种新工具层出不穷，JS 的高度灵活性真是能玩出花，node 的各个版本也有一定的差异，官方包管理 npm 也有一些历史问题被人疯狂吐槽，现在我知道的比较出名的代替是：

1. Yarn

   并行下载安装，在大型项目中使用更可靠的锁定文件.

2. pnpm

   通过内容寻址来去除重复依赖，节省磁盘空间，加速安装.

当然这玩意发展太快，现在又出现了什么新工具我就不了解了。

而 node 版本的管理，我认为比较有名的有这几个：

- nvm

  基于 shell 脚本的 Node.js 版本管理工具，老牌，广为人知和功能全面

- n

  基于 npm 的 Node.js 版本管理工具，以其简洁的命令行界面和快速的安装、切换速度著称

- fnm

  基于 Rust 编写的 Node.js 版本管理工具，以其极快的速度和跨平台支持而闻名

这些工具的具体使用方法就不展开说了，基本都差不多，毕竟我觉得现在有了更好的方案。
让我最难受的大概是每个项目依赖都要在当前文件夹安装一遍，嵌套可能还深不见底，Windows 下压缩打包都不一定能打包的了；到现在还能看到开发一个 app 100kb，安装的依赖 10G 的地狱笑话 haha

## Python

个人感觉它的包管理和 Node.js 的 npm 真的挺类似的，py 官方的包管理是 pip，也经常被吐槽慢，也同样存在依赖库版本兼容问题，并且还有 python2/python3 的兼容问题，虽然现在基本都是 py3 的天下了。

在介绍版本环境控制之前，先介绍两个好用的工具，我使用的是 virtualenv 来做环境管理隔离，如果用的是 fish 的话要用 VirtualFish；而 uv 则是解决 pip 依赖安装速度等问题；再搭配版本管理工具基本就能满足大部分需求了。

VirtualFish 的一些基本使用方法：

```sh
# 创建新环境
vf new name
# 激活环境
vf activate name
# 退出环境
vf deactivate
# 删除环境
vf rm name
# 查看环境
vf ls
# 启用自动激活
vf auto enable
```

uv 的一些基本使用方法：

```sh
# 安装单个包（比 pip 快的多）
uv pip install requests
uv pip list
uv pip list --outdated # 查看更新包

# 全局生效
uv pip install --system pandas
```

当然如果是非编程方向，例如数据分析方向，uv 未必合适，可能还是 conda 更合适一些，python 的生态真的太割裂了，conda 也被吐槽的很厉害，或者可以尝试一下 miniconda 或者 pixi

VirtualFish 和 uv 这类工具完全可以并且通常推荐和版本管理工具搭配使用，毕竟他们解决的不是一类问题。

## Java

Java 生态我感觉相比之下还是要好得多，maven/gradle 下大部分的库都是可以向下兼容的，从 JDK8 之后 JDK 的向下兼容会差一些，毕竟放弃历史包袱也是为了更快的适应时代潮流。

对于 JDK 版本管理问题，推荐 sdkman，它在 MacOS/Linux 下体验还是不错的，是一个 CLI 工具，可以比较方便的切换、安装不同发行版的 OpenJDK 版本。

也支持安装一些 Java 生态的常用工具，感兴趣的可以看看[使用手册](https://sdkman.io/usage)。

```sh
# 安装 cli，查看版本
curl -s "https://get.sdkman.io" | bash
sdk version

# 安装 JDK，默认 tem
sdk install java
# 指定版本
sdk install scala 3.4.2
# 卸载
sdk uninstall scala 3.4.2
# 搜索查看可安装的发行版
sdk list java

# 使用
sdk use scala 3.4.2
sdk default scala 3.4.2
sdk current java
```

### OpenJDK 发行版选择

自从 Java8 之后 Oracle 的骚操作，官方版本的 JDK 肯定是不推荐的，能用得起的企业也没多少吧。
主要的发行版有：

- Oracle OpenJDK
- Adoptium (Eclipse Temurin)
- Amazon Corretto
- Red Hat build of OpenJDK
- BellSoft Liberica JDK
- Azul Zulu
- Alibaba Dragonwell
- ...

其他的华为、腾讯都有自己的发行版，他们这些云计算厂商都是根据自己的平台优化过的，所以除非绑定平台可以尝试，要不然还是用一些通用的吧。

说结论，Liberica JDK 与 AdoptOpenJDK（现已更名为 Eclipse Temurin）可能是最佳的选择。

Liberica 是 Spring 官方推荐的发行版，我了解了下 BellSoft 这家公司，是一家专注于 Java 技术的公司，是 OpenJDK 的主要贡献者之一，根据场景有不同的版本，例如 Lite 版本是容器优化，Standard 日常开发，Full 可以 Fx 等 GUI 的开发。
它家我看还有对 Java App 特别优化的 Linux 容器 Alpaquita，有空我也可以尝试下。

而 Eclipse Temurin 原名 AdoptOpenJDK 就不用说了，Eclipse 社区出品不会有太大问题，优点和缺点都是一个，社区驱动嘛，有舍有得，sdkman 默认的就是 Temurin。

我之前一直使用的是 AdoptOpenJDK，现在打算切换到 Liberica 试试。

## 通用版本管理

推荐上，个人的排序是 mise > vfox > asdf

这一类工具一般都会把版本信息保存在项目目录下的一个配置文件中，这样除了工具可以识别当前环境信息，还可以在项目中共享这些信息，确保团队中的每个人都使用相同的工具版本。

```sh
# mise exec/x 虽然非常适合运行一次性命令，但激活 mise 会更加方便。
mise exec node@22 -- node -v
# mise node@22.14.0 ✓ installed
# v22.14.0
mise x python@3.12 -- ./myscript.py

# 没有特殊说明，配置仅对当前目录生效
mise use --global node@lts
mise ls
mise use node@23 pnpm@10
# 仅安装，不激活
mise install node

mise cache clear
mise ls-remote node

# 查看所有可用的插件
mise plugins list-all

# 安装插件（比如要使用 node）
mise plugins add node

# 查看已安装的插件
mise plugins ls

# 查看已安装的版本
mise ls node
mise ls
mise use node@lts

# 设置环境变量
mise set NODE_ENV=development
# 查看当前配置
mise settings
```

mise 的安装参考[文档](https://mise.jdx.dev/getting-started.html)

mise 兼容 asdf 等环境管理软件的配置，并且因为 mise 的设计灵感源自 asdf，老版本兼容 asdf 的插件系统，但是现在并不推荐；速度比 asdf 快的多。

他们两个的配置文件包括下文的 vfox 都是可以互相识别的。

vfox 和 asdf 差不多，也是通过安装插件来实现各种功能。

```sh
# 查看所有插件
vfox available
# 添加插件
vfox add nodejs
# 安装
vfox install nodejs@latest
vfox install nodejs@21.5.0

vfox search nodejs
# 全局生效
vfox use -g nodejs
# 临时 session/当前终端生效
vfox use -s nodejs
```

