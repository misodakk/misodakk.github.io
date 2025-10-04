# 我的软件清单&军火库


起因是最近重装了系统，虽然有备份还原可用，但是垃圾也会一起还原，所以直接从头开始，比较之前也装了很多乱七八糟的东西，但是基本就没用过。。。。

但是现在都已经全部弄完了，现在才想起来记录一下，emmmm，之后想到了就会更新的。<!--more-->

目前我使用的设备有 Windows11 台式机、一台 MBP、一台 MacMini，所以软件生态会分 Win 和 macOS，开发使用 macOS 居多，其他都是 Win。

当然有些云服务类就不限于平台了，内容比较杂，因为现在都是使用中状态，大多数也只能想起来再记录或者下次重装系统的时候再整理了。

一样是**不在多而在于精**，在可用的情况下，最多保留个备用的完事，再多同类的也大概率不会用。

{{< style "text-align:right; strong{color:#00b1ff;}" >}}
缓慢整理中 **也许会忘记也说不定呢**.
{{< /style >}}

## Windows

### 终端工具

还是因为习惯了 Xshell 了，但是貌似 WindTerm 用的人很多

1. Xshell

2. putty

3. WindTerm

4. Termius

   如果用，推荐[这个补丁](https://github.com/ArcSurge/Termius-Pro-zh_CN?tab=readme-ov-file)

Termius 之前在 macOS 上应该用过，怎么说呢，感觉卡卡的（Electron 你不要过来啊），外加没中文劝退了。

### 效率工具

装机必备，提升操作效率！

1. PixPin - 截图贴图

   Snipaste 也很好用，可惜不支持 OCR

2. Ditto - 剪切板管理

   作者貌似删库了，替代选择：CopyQ

3. FastKeys

   强大的自动化工具，最常用的是 Mac 上类似的"自定义短语"功能。

4. Notepad Next

   Notepad++ 的替代品，++ 的作者近些年感觉有点颠，推荐替换

### 补丁增强

可能有风险，谨慎使用（指被封号）。

1. [BetterWX](https://github.com/zetaloop/BetterWX)

   WX防撤回、多开等

2. WO Mic

   还没舍得买麦克风，用手机当作电脑麦克风

3. [Windows Update Disabler](https://github.com/tsgrgo/windows-update-disabler)

   最近的 Windows11 更新补丁导致 SSD 掉盘的风波，真的不太信任微软的这帮啊三了。

4. 主题美化 [niivu](https://www.deviantart.com/niivu)

5. [DevManView](https://www.nirsoft.net/utils/device_manager_view.html)

   驱动管理，IOBit 的 *Driver Booster PRO* 也很不错。

6. WizTree
   磁盘空间分析，另一个古老软件是 SpaceSniffer
### 快速启动

基本再用 uTools，最近看官方好像限制插件数量，引起不小的风波，如果没影响继续使用，有影响就换，我插件用的很少，基本就用个 截图 OCR + 翻译

1. uTools
2. Flow Launcher
3. Listary
4. Quicker

### 远程控制

Rustdesk 能自建最好，我最近也打算自建一个试一下；
1. Rustdesk
2. 网易 UU 远程
3. TeamViewer

除非必要，目前已不再推荐向日葵、ToDesk 之类，体验并不好。
## macOS

配置文件同步推荐使用 [Dotbot](https://github.com/anishathalye/dotbot) 初始化新系统，直接 git pull 下来 install 全部配置完毕的感觉太爽了。

brew 的备份使用 homebrew-bundle，例如：
``` sh
# 备份
brew bundle dump --describe --force --file="~/Desktop/Brewfile"

# 如果你使用 brew 安装了 MAS 的 App 需要提前安装
brew install mas
# 恢复
brew bundle --file="~/Desktop/Brewfile"
```

### 终端工具

参考 Windows 中的介绍，Xshell 没有 Mac 版，现在使用默认终端 + 食用；

1. Ghostty
2. FinalShell
3. Termius
4. kitty
5. WindTerm

Termius 感觉卡顿严重，也许我机器太老了。
## 通用&服务类

### 插件

沉浸式翻译删掉了，不光是因为最近的隐私事件，VIP 功能太杂了，我其实都用不上，换成简约翻译。

1. Kiss Translator

   配合 L 站的 DeepLX 效果更好

2. Auto-Group Tabs

3. uBlock Origin

4. uBlacklist

5. tampermonkey

6. Bitwarden

7. Checker Plus for Gmail

8. Elmo Chat

9. Hover Zoom+

10. JSON-handle

11. Octotree

12. SteamDB

13. SuperCopy

14. Wikiwand - Elevate Wikipedia with AI

15. Wikipedia Search

16. 划词翻译

17. 几枝

18. 為什麼你們就是不能加個空格呢

19. 扩展管理器（Extension Manager）

20. 终结内容农场

不太常用的一些：

1. Talend API Tester
2. WebRTC Control
3. Elmo Chat
4. Obsidian Web
5. Obsidian Web Clipper
6. Screenity
7. SmartProxy
8. ScriptSafe
9. Awesome Screenshot 截图录屏

### 脚本

1. LinkSwift

2. MiniblogImgPop - 微博浮图

   与 Hover Zoom+ 插件可以二选一

3. CSDNGreener

4. 中文维基百科优先简体中文

5. 自动无缝翻页

6. 网盘自动填写访问码

7. HTML5视频播放器增强脚本

   备用 TimerHooker

8. 去除博客导流公众号

9. AI 验证码自动识别填充

10. 万能验证码自动输入

### API 调试

Postman 用惯了，但是有点越来越恶心了，替代的也用了不少，还是找不到满意的。

1. Reqable
2. Apifox

Apifox 很多人推荐，但是个人还是不太喜欢，功能太多也有点烦，比较喜欢极简主义的；并且也是强制登录，强制云同步。

并且大概率也是（Electron 你不要过来啊）
### AI 开发辅助
目前用的还是较少，但是很多人在吹，老了折腾不动了，思想也落后了还是觉得 AI 的代码不如自己写的好，速度确实快，适合一些简单、要快速开发的项目吧。

1. [Roo Code](https://roocode.com/)

   具有智能代码生成和重构功能的高级 AI 驱动编码助手。
   应该就是 CC 这种的替代版，可以选择多种模型，比如本地部署的，以达到完全免费使用。
   VSC Only

2. [RunVSAgent](https://github.com/wecode-ai/RunVSAgent)

   使开发者能够在 JetBrains IDEs 或其他 IDE 平台中运行基于 VSCode 的编码代理和扩展。
   应该是基于 Roo Code 的。

3. Claude Code Router

   将 Claude Code 请求路由到不同的模型，并自定义任何请求。

看到有人发某 V 站的秀优越的帖子，以防有人不知道，像我这种穷鬼，Claude Code 是用不起的，别再说什么程序员一天的收入来买 $200 的 AI 也是物超所值的。

### 魔法相关

Win 优先 Spark，Mac 优先 Surrge 学习版（可惜新版本的 TUN 需要关 SIP）。

sing-box 还是有点望而却步。

裸内核运行也不是不可能，搭配 [zashboard](https://github.com/Zephyruso/zashboard) 面板，也未必会差。
### 影音聚合

除必要情况，使用 Emby 公费服，一个月几块钱换来的质量比 CMS 的那些资源要高的多。
主要是亲戚朋友让你给找资源，总不能给个网盘分享吧，不是 VIP 的体验也没多好，还是在线看的体验好一些，可以直接从这些资源站找，比一般的满屏广告的野站要好的多。

1. ~~[MoonTV](https://github.com/LunaTechLab/MoonTV)~~

   Web 服务聚合，对应的独立客户端选择 [LibreTV-App](https://github.com/KeyRotate/LibreTV-App)

   更新：存活了不到一个月，被人举报 DMCA 了，转生版本 [LunaTV](https://github.com/MoonTechLab/LunaTV)，现在也放弃维护了。

   替代先参考 [KatelyaTV](https://github.com/katelya77/KatelyaTV) 吧。

2. [爱看机器人](https://v.ikanbot.com/)

3. TV 直播源

   关注肥羊的 ALLINONE，可惜被人给恶心的退坑了，[备用关注 Mursor 大佬](https://github.com/Mursor/LIVE)
   其他的还真没发现有好用的。

4. 音乐聚合 [HE-Music](https://y.wjhe.top/#/discover?platform=kuwo)

   需要 L 站 lv2+，且用且珍惜


## 其他 Misc

**Linux 发行版选择？**

开发 GUI 的 Ubuntu，服务器 Debian，折腾 Arch；

像折腾又不想太折腾，基于 Arch Linux 的 CachyOS ？

- [ONLYOFFICE](https://www.onlyoffice.com/zh/)

  好用的 Office 替代，其他还有知名的 *LibreOffice*。

- CloudDrive2

  网盘聚合和本地挂载，其他的还有 AList 的分叉版本 OpenList。

