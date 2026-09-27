# Pi Agent Harness


在开始前，先统一下 Harness 和 Agent 的概念；

Agent（智能体）是一个**概念层**的词，指"能感知环境、自主决策、采取行动来达成目标"的实体。Agent 描述的是一种**行为模式**，不特指某个产品。

在 LLM 场景下，Harness 指**把模型套起来、变成可用工具的整套工程框架**，通常包括 Agent Loop、系统提示词、工具集、上下文管理、权限与审批、界面。<!--more-->

简单说就是说 Harness 的时候一般指程序本身，是工程框架；而说 Agent 的时候多是指的这个能干活的整体，是一种行为实体。

所以，多数时候对于 Codex / Claude Code / Pi 这一类你既可以叫它 Agent 也可以说是 Harness，只是关注的角度不同。

------

Pi Agent 是一个开源的、极简主义的 AI 编程代理（Coding Agent）工具包和 SDK，主要在终端环境中运行；与市面上许多功能臃肿的 AI 工具不同，Pi Agent 的核心设计理念是 “**极简主义**”。

可以理解是通用/编码 Agent 的毛坯房，只包含最基本的东西；

其他更复杂的功能，可以通过插件的方式引入，官方的扩展包市场：https://pi.dev/packages

我还是建议直接用套壳的 GUI 一劳永逸。

## 为什么关注

只提供基本的文件读取、写入、编辑，和一些执行命令的工具，再加上核心的 Agent Loop，此外就没有什么东西了。

随着大模型迭代越来越聪明，对于大部分的日常工作，其实不需要特别引导，基本都可以完成的差不多，意思就是不需要每次都内置一大串的 system prop，Pi 在上下文方面就非常的精简，你会明显感觉到 token 消耗的少了（感知节省一半左右）；

当然，这带来的问题就是复杂问题确实可能容易过于发散，比不上 codex 这种全能选手。

默认情况下，它没有 MCP、计划模式、权限管理、目标模式、子 Agent；当然你可以通过扩展包来进行支持，和我理解的一致，大部分情况下这些功能你可能并不会去用。

最后，如果你想开发一个自己的 Agent，那么用 Pi 作为基础 SDK 是一个非常好的选择。

当下，闭源 Agent 接连出现后台偷偷上传代码隐私问题，开源的通用实现越来越流行，不同场景下选择合适的 Agent 更有性价比。

## 扩展

可以说就是 Hook，pi 基本在主体框架的每一步都设置了 Hook，例如会话启动时、输入任何文本后、进入 agent 循环之前、工具执行完毕后、循环的一轮完成时等等，pi 中有非常多的这种扩展点，你可以在这些扩展点做你想做的事。

大部分情况，你可能只需要描述下你想要什么能力，直接让 pi 参考自己的文档给你写一个扩展即可，下面有一些比较热门的网友写好的扩展：

1. pi-observability：统计一下 Token 的各种指标
2. pi-web-access：用于支持 WebSearch 和 WebFetch
3. pi-mcp-adapter： 支持 MCP 接口
4. @pi-lab/input-history：实现类似 Codex 那种按照项目的方式，按上键可以恢复输入历史
5. pi-powerline-footer： Pi Agent 底部会显示一个状态栏
6. pi-rewind：每次 AI 修改会建立一个 git refs 的 checkout point, 避免 AI 把代码改坏了同时又没有 commit 的时候
7. pi-lens：LSP、lint、类型检查、实时反馈
8. @juicesharp/rpiv-ask-user-question：用结构化问卷澄清需求
9. @juicesharp/rpiv-todo：实时任务清单
10. pi-simplify：审查代码是否清晰简洁
11. pi-agent-browser-native：浏览器自动化和网页测试

实际上，你可能完全不需要这些工具，真正需要的时候你可能直接就去用 Codex 之类的工具了。

因为对于复杂任务需要模型高智力的时候，现在的趋势就是模型和客户端绑定才能发挥出真正的实力，即例如 GPT 系列 + Codex 才是完全体。

## DeepSeek Harness

DeepSeek Harness（DSH）最近很火，但是我感觉大概率都是图一个新鲜，真正证明优秀还需要时间的沉淀。

对于 pi 来说，可以理解成骨架是固定的，例如 Agent Loop、模型、工具、上下文（当然也支持注册新工具、命令、模型、界面）；扩展性主要就来自这套骨架沿途开放的大量接缝，再每一个接缝中你都有机会去干点什么；它的执行流很清晰简单，沿着这条执行逻辑很容易分析那个阶段发生了什么，有什么问题。

dsh 的理念和 pi 有一些相似，都是走极简路线；它的口号是“一切皆插件”，简单说在 pi 的基础上，它考虑骨架能不能换，Agent Loop、模型、会话等等都是插件，都可以替换，只需要遵循 dsh 的“语言”或者说规范、约束，那么什么都能换。

把这些插件搭配组装起来的机制是 cordis，非常薄，有点模块化的那个感觉，不同的预设模式就相当于用不同的插件搭起来的结构。

这套机制有个能力就是甚至可以在运行时进行插件的替换，当然 Agent Loop 这种承重作用的插件是不太可能，其他的边缘插件就未必了，所以还需要考虑残留、注册和依赖关系。

所以说 dsh 从设计层面要远比 pi 复杂，要考虑的东西更多，这种复杂的东西初始阶段问题应该不会少，在某些特定场景应该非常有竞争力。

dsh 和 pi 是两种技术路线，现在来看并不能说谁是正确的，还需要时间去检验，只是在当下来说，pi 还是一个更稳定的选择。

## 相关资源

专有桌面端方面，目前有很多，但是特别火的没有，国内网友维护的 [PiDeck](https://github.com/ayuayue/PiDeck)、[pi-app](https://github.com/justhil/pi-app) 看着是比较活跃的。

其他的 [pi-gui](https://github.com/minghinmatthewlam/pi-gui) 比 [Pi Desktop](https://github.com/gustavonline/pi-desktop) 的维护要更加积极。

但是我还是推荐直接上集成的套壳 Agent，例如 Cherry Studio、Cindy 之类。

- [Pi Manager](https://github.com/scp3500/pi-manager)
- [中文文档](https://pi-doc.com/)

