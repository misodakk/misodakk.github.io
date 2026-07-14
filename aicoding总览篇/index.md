# AI Coding总览篇


本文介绍下通过使用一些主流的 AI 编程工具后的一些心得，以及围绕编码方面的相关技术生态和工具。

我的整体态度是 AI 在编码方面算是进入可用阶段，还不能称之为成熟阶段，相比之下通用型 Agent 例如 Hermes 、各种 xxxWork 处于起步阶段，可用性没那么高。

对于老程序员，AI 是个利器，用的好可以提升很多的效率，不同智力的模型使用效果也差别很大，但是达到一定的智力程度，区别就没有那么夸张了，通过上下文管理、提示词优化对日常编码来说都可以胜任。<!--more-->

我目前所推崇的还是 AI Coding，并不是 Vibe Coding，但是很多人会把他们看作是一个东西；因为是新兴概念，我还是按我自己的理解来，即它们有相似之处，但不是一个概念；

我目前的态度是：想要真正用好 AI 进行 Coding 还是需要一些经验知识的，也需要知道如何提问和规划，要把整体方向把控在自己手里，对核心代码也需要 review。

当然如果是普通人只是为了解决某个小问题编写程序，那么 vieb coding 也是对的，完全不需要看代码，能跑就可以。

目前主流的工具有：

- Claude Code
- Codex
- OpenCode
- ~~Qoder~~
- ZCode
- Reasonix
- ~~Gemini Code~~

这些工具部分是有多种形态，例如 CLI、桌面客户端、IDE 插件，对我来说桌面环境还是选客户端，没有 GUI 环境就选 CLI，或者作为补充。

上面的工具本质还是一个 Agent，核心就是一个聊天框；编码方面还有一些做成 IDE 级别（大部分都是基于 VSC 魔改）：

- Cursor
- Antigravity
- 阿里的 Lingma（Qoder CN）
- 字节的 Trae
- Kiro

现在的 VSC 其实也加了一堆 AI 相关的东西其实有点越来越难用的意思了，我个人是不太喜欢使用 IDE 级别的，还不如配合插件使用；不过很多人都说已经完全替代 jetbrains 全家桶，但是我依然觉得 jetbrains 还是有不可替代的地方，我更喜欢两者一起用。

其他的一些正在上升期的工具（新一代轻量级 Agent）：

- Pi
- oh-my-pi
- Grok Build

综合性价比考虑，我目前使用的主要是 Codex。环境的不可抗力不可用的情况会使用 OpenCode、**Reasonix** 顶一下，目前国内各家模型厂也在推出类似 Codex 的 Agent，后面国产的话选择会很多。

**Reasonix** 是专为 DeepSeek 的 AI coding agent，目前已知问题在 CC 或者 Codex 之类的 Agent 使用 DeepSeek 会有很大的缓存命中问题，成本爆炸，完全体会不到 DeepSeek 的价格屠夫。

------

AI 可以帮我们高效的完成编码，甚至完全替代我们写代码、做设计，但是审查、架构方向还是无法做到我们理想的那样，毕竟架构设计、项目经验这一块是无法代替的，一个十年经验的程序员和一个非专业出身的普通人使用 AI 进行编程完全不是一回事。

![coding](/images/coding.jpg)

副作用：额度用光 = 天才程序员的陨落；重置日 = 我现在强的可怕！

## Vibe/AI Coding

氛围编程 (vibe coding) 是一种“靠感觉和意图驱动”的 AI 辅助编程方式；

你不再一行一行亲手写代码，而是把需求、想法、页面感觉、功能目标告诉 AI，让 AI 生成、修改、运行、调试代码；你主要负责判断结果是否符合预期，然后继续用自然语言调整。

我肤浅的认为，现在说的 vibe coding 就是纯自然语言描述，不亲自下场写代码，所以对没有编程基础的普通人是一种非常好的方式；另一方面因为完全不懂程序架构设计的人可以做出一个产品，对他来说这个程序其实是一个黑箱，用于生产我觉得是不妥的。

门槛极低，极大缩短开发周期，让非程序员也能成为创作者。

vibe coding 真正有用的地方应该是做 MCP（Minimum Viable Product）最小可行产品，或者前期展示和设计使用，亦或者自己用或者小范围使用，不涉及敏感业务。

AI 帮写代码但是不会帮我们背锅，审查我们自己还是要上上心，毕竟传统的编程开发编码阶段其实占比并不高，安全、稳定、可维护等才是主线，不同场景要求也不一样，AI 默认的范式未必适合，说到底还是提示词工程，能高效和 AI 对话的人还是少数，提问的艺术很重要。

------

我这里更信仰的是 **AI Coding**（AI 辅助编码），让 AI 作为专业工程师的强力“副驾驶（Copilot）”，负责完成重复性高、繁琐的代码片段。

大幅提升专业开发者的效率，同时保留了人类对架构、性能及底层安全性的完全控制。

------

最近很流行一个概念 AI slop（AI 泔水、数字垃圾），简单说就是由生成式 AI 大量产出的低质量媒体，含意类似于“垃圾邮件”；有网友专门做了一个[网站](https://killaislop.com/)来进行展示对比，不过这个网站也是 AI 写的。

> Vibe Coding 做出来的产品，丑得千篇一律：蓝紫渐变、发光玻璃卡片、每行挂个 emoji、什么都贴个徽章。这是机器没品位、又想装高级时捡来的东西。因为满大街都是，你早就看麻木了。

本质还是功能实现毫无疑问，但是设计审美还是要需要有点东西上点心的。

不过如果是刚开始使用 AI 来 Coding，那么刚开始大概还是感觉不错的。

## 关于搜索

关于 Web 搜索的问题，主流的大模型一般都内置了 WebSearch 工具，OpenAI 的 WebSearch 和 WebFetch 是合并的；Anthropic 是拆开的。

> **WebSearch（网页搜索）**：一般指的是获取搜索引擎搜索结果，包含 URL、标题、部分内容/摘要等信息；
>
> **WebFetch（网页抓取）**：对于具体的一个 URL，抓取目标网页，提取原始HTML，过滤掉广告和无关格式，并将其转化为 Markdown 供大模型处理；相当于你从搜索列表打开看具体页面看内容；
>
> **DeepSearch（深度搜索）**：它会经历「搜索—阅读—推理」的循环，自动拆解子问题，并在几分钟内检索、分析数百个网页，最终生成如专业研究员般的结构化长篇报告。

WebFetch 在我们的环境下有个优点，因为是服务端逻辑，所以对于一些我们网络无法访问的网站，它也可以进行搜索总结。

也有一些相关的 MCP 服务（或者把 rest 接口转换为 MCP 服务的），如果用的到可以搜索下相关内容。

## Codex

总体上各家的 Agent 都差不多，这里我以 Codex Desktop 为例（现在已经更名 ChatGPT），个人感觉还是非常好用的。

Desktop 版肯定是比较容易上手的，CLI 也可以不过我认为更适合在没有 GUI 的服务器端，排查服务器端报错、日志文件、引用当前目录和文件、写配置还是挺方便的；IDE 插件对引用代码、文件、当前打开文件等操作更方便。

操作方面基本没什么可说的，按钮配置也不多，都是一眼就知道是什么的存在，权限给分了三类：

- 默认权限：读取、修改项目文件夹
- 自动审查：调用一个小模型来审查，低风险操作会放行，推荐。
- 完全访问权限：可自动控制电脑的操作无需审批。

项目外的文件操作、联网操作都需要人工批准，这是通过代码硬性控制的，算是一种 HarnessEngineering 的具体实现。

自带的多功能右侧栏中除了看效果还可以批注，非常好用。

Codex 的权限是围绕沙箱来展开的，执行基本都会在沙箱中，与 CC 不同，CC 一般需要手动开启，Codex 更像是把沙箱作为一种根基。

WorkTree 工作树功能是复制一份文件夹内容到新的文件夹，新的操作不影响主文件夹，也就是可以同时并行修改。
修改完毕后合并到主文件夹，多任务协同工作时使用。

闲聊和不涉及文件的聊天，直接使用对话，这样省的分析项目下的文件而浪费 token。

最后再提醒一下：根据我们之前了解的上下文管理工程，如果新动作不涉及上下文的内容，就新开一个对话！

GPT-5.5 具备原生的图像生成能力。OpenAI 为其配套推出了专用的 **ChatGPT Images 2.0** 模型，专门负责高质量的图像生成与编辑。

对于复杂任务，优先建议使用计划模式，这种模式下 Codex 会非常积极的询问我们一些问题，最后指定一份计划表。

对于希望无人值守让它完成某些长任务的场景，可以尝试 goal 目标模式。

### 基本范式

为了帮助 AI 理解项目，我们可以提前准备一些说明文件，这些文件你甚至可以让 AI 给你生成，为的是解决：

- 让 AI 每次进入项目都知道规矩和背景
- 不要把所有需求都塞进聊天上下文
- 需求、架构、任务、约束都有固定位置
- 切换对话后不会失忆
- 让 AI 可以自己查阅项目文档，而不是依赖你反复口述

其中 **AGENTS.md** 在 codex 中是一个特殊文件（Claude Code 中是 CLAUDE.md），AI 会优先读取它的内容加载到上下文中，每个会话都会携带，它不适合写成长篇需求，它更适合写：

- 简单描述项目是什么
- AI 开始工作前应该读哪些文件
- 编码规范/要求
- 禁止事项/注意事项
- 测试命令
- 提交前检查
- 文档更新要求

功能上和 README 文件差不多，只不过一个是给人看的，一个是给 AI 看的。它的内容通常更偏“操作规则”而不是项目介绍；建议只写“稳定、具体、能指导执行”的规则，不要写太泛的话（短、具体、可执行）。

完成较大的功能修改后要让 AI 更新对应的文档，这个很重要。

文件的推荐目录结构：

```
AGENTS.md            # 给 AI 的工作规则、阅读入口、约束
README.md            # 给人和 AI 的项目概览
docs/requirements.md # 产品需求，详细需求，可以写成产品需求文档，也就是 PRD
docs/architecture.md # 技术架构，架构说明
docs/tasks.md        # 当前任务拆分，任务列表
docs/status.md       # 当前进度、已完成、未完成、注意事项
docs/decisions/      # 架构决策记录 ADR
```

拆分多个文件是为了避免 `AGENTS.md` 过长，也避免上下文爆炸。
 如果需要所有的会话都生效，那可以把它放到用户目录下。

一个项目中，我认为比较重要的是 AGENTS 和 PRD 文件，至于如何写好这两个文件，可以问 AI 或者看看网友的经验，写的好不好完全决定你 AI 的智商。

### CLI 指令

最简单的执行 codex 进入的 TUI 界面，这个大部分一看就知道怎么使用；

使用 @ 来引入文件或文件夹，使用 / 来出发指令；

一次性任务使用 `codex exec "总结这个仓库的结构"`，可以使用 -C 来指定工作目录；也可以通过管道符来让它分析日志 `tail -n 300 app.log | codex exec "分析这些日志xxxx"`

常用命令：

```shell
/new	   开始新会话
/resume	   恢复历史会话
/compact   压缩上下文
/status	   显示会话状态
/clear	   清除屏幕
/review    代码审查
/goal      长任务的任务锚点
/mention   引用文件，比起 @ 更推荐
/init      初始化 AGENTS.md 指导文件
/keymap    快捷键
/agent     使用子代理

codex exec --sandbox read-only "检查当前代码中潜在的安全风险，不要修改文件"
```

CLI 我目前用的不算多，CC 应该也是类似。

### 配置

配置文件位置：`.codex/config.toml`，如果你使用 CC Switch，可以在它软件里改。一个示例配置：

```conf
# 表示默认使用名为 OpenAI 的模型提供方
# 这个名字要和后面的 [model_providers.OpenAI] 对应上
model_provider = "OpenAI"
model = "gpt-5.5"
review_model = "gpt-5.5"
# 推理强度-高
model_reasoning_effort = "xhigh"
# 关闭响应存储，少存或不存回答内容
disable_response_storage = true
# 允许网络访问
network_access = "enabled"
# 已经确认过 WSL 相关引导/提示，客户端一般不会再反复提醒
windows_wsl_setup_acknowledged = true

[model_providers.OpenAI]
# provider 的显示名
name = "OpenAI"
base_url = "https://xxxx.dev/v1"
# Responses API 风格
wire_api = "responses"
# 走 Codex 的“OpenAI 认证体系”
requires_openai_auth = true

[features]
# 启用 goals 相关能力
goals = true

[windows]
# 用“非提权”方式运行沙箱/命令环境
# 不以管理员权限执行，尽量把能力限制在普通用户权限内
sandbox = "unelevated"

[analytics]
# 关闭遥测数据上传
enabled = false
```

使用多个 AI coding agent 的情况下，还是推荐使用 CC Switch 进行统一管理，避免 MCP SKill 散落在各个地方。

配置文件中，我这里使用了 OpenAI 命名自定义 API，使用这个名称可以开启一些 OpenAI 的能力，如果使用的其他模型就不要叫这个名字。

例如 model_providers 中的 name 使用 OpenAI 应该就可以使用远程压缩。

> [!TIP]
> 注意 model_provider 的名字不要乱改，修改后你的历史回话可能会无法读取，不过通过 CC-Switch 可以管理

具体的配置可以直接问 AI 我也没用到几个。

### 插件&扩展

使用 cc switch，也就是使用 API 的方式，会发现 codex 的插件无法使用，因为确实有些插件是需要走 OpenAI 的后台，但是有一部分确实没有什么必要；
 以及 codex 的对话只能归档，不能删除，无法开启 fast 模式；

但是我发现是可以使用一些技术来绕过这些限制，例如[这篇文章](https://linux.do/t/topic/2112570)中，把操作方式直接总结成 Skill 的方式来绕过。

也有人直接写了一个可执行的补丁程序 [Codex++](https://github.com/BigPizzaV3/CodexPlusPlus) ，以及直接重新二开的三方客户端 [Codex Desktop Rebuild](https://github.com/Haleclipse/CodexDesktop-Rebuild)，最新版本的 cc switch 也对 codex 提供了一些增强功能。

现在有了 computer use 可以对我们的 PC 进行一定的控制，也有浏览器插件可以控制浏览器的一些行为，还可以连接浏览器的 DevTools，体验已经好了一大截，有点 OpenClaw 通用 Agent 的那味了。

最近的更新也可以通过手机端远程连接使用，感觉确实 codex 越来越好了，唯一的问题就是性能了。

一些可能有用的插件：

- FableCodex

   用来把 Fable 风格的工作习惯加入 Codex 流程：先检查，再跟踪目标，记录证据，关闭 review finding，并在声称完成前进行验证。

- Ponytail

   著名的马尾哥，精简生成的代码；确保自身安全的情况下避免 AI 过度写一堆兜底代码（防御性代码）

很多人说 GPT 有时候会降智，热心网友总结一条规则，在 AGENTS.md 中加入一句：`DO NOT send optional commentary`，会显著缓解降智的情况。

### Subagent

主 Codex 在一个任务里临时派出多个“子代理”，让它们并行做不同子任务，最后由主代理汇总结果。

Codex 可以生成多个专门化 agent 并行工作，再把结果收集成一个回答；适合代码库探索、多点 PR review、多步骤功能计划这类可并行任务。

```
主 Codex
  ├─ explorer：只读代码，找路径
  ├─ reviewer：审查 bug / 安全 / 测试风险
  ├─ docs_researcher：查官方文档
  └─ worker：做具体实现
```

Codex 不会随便自己开 subagent。官方文档说它只会在你明确要求时才会触发，例如你说 “spawn one agent per topic” 或者“请启动多个子代理并行分析当前分支”。

根据网友观察，使用最新的 gpt-5.6 Sol 的 Ultra 强度 Agent 会非常倾向于使用大量的 Subagent

### Hook钩子

和我们之前常用的 WebHook 是一样的概念，大概理解就是当触发什么行为的时候进行回调，主要是用来限制行为，例如当删除文件的时候执行一个你的什么函数、脚本之类。

至于支持哪些 Hook 这个要看具体的工具，看它们的官方文档，或者让 AI 直接给你写就完事了。

## 设计

现在阶段的 AI coding agent 对设计问题还是偏弱的，编码能力基本是可用了，但是像 UI 设计 UX 之类的还是比较弱；

目前比较好用的工具有 Claude Design 和 Google Stitch；开源的也有 [Open Design](https://github.com/nexu-io/open-design)。

配合一些 MCP、SKill（例如 Figma 提供了比较全面的 MCP 和 SKill）也有不错的体验；我目前看到了一些网友的推荐，后续可能会单独整理一个资源篇，这部分迭代速度还是很快的，因为目前还没一个共识的方向。

## 范式

我这里把它解释为你想要你的 Agent 遵循什么流程进行编码，也就是工作流吧，本质还是提问的艺术，都是提示词工程。

目前主流的有两种：

**SDD（Spec-Driven Development）**

即：根据需求写出规范文档，然后根据规范文档生成代码。

**TDD（Test-Driven Development）**

先写测试用例，再写代码让测试用例通过（这是一个循环）

本质是为了让代码更加容易维护，上面两个还都是理论概念，具体的实践工具对应有：

- OpenSpec：SDD --> 讨论-需求-规范-实现
- Superpowers：TDD --> 讨论-需求-测试-实现

以 OpenSpec 为例，它的使用非常简单，主要就是三条命令：

`/opsx:explore`：交互式提问理清需求

`/opsx:propose`：将需求转换为文档

`/opsx:apply`：文档转换为代码

对于要写一些比较复杂的业务，建议可以先让 AI 画一个 PlantUML，这样更加清晰，比文字描述也更加精确，对于 AI 来说，PlantUML 也非常的适合，还会省 token。

一般情况下，因为是 LLM 所以天然适合输出文字，MD 格式成了首选，但是在交付时，你可能需要的是 word、pdf 等等；

目前发现一个比较火的网站 [docu.md](https://docu.md/)，它可以把 AI 生成的 Markdown 变成可以提交、发送、发布或演示的成品。

## 其他补充

Agent 这一类工具核心还是提示词，有网友整理了主流 AI Coding 工具的[系统提示词设计资料](https://github.com/CreatorEdition/system-prompts-and-models-of-ai-tools-chinese/tree/main)可以参考下是怎么说的。

还有一个问题是中转的上下文注入投毒，因为不可抗力，有一大批会选择使用中转的方式来使用，如果中转站被黑或者心怀不轨，投数据卖数据都是次要的，如果给上下文注入一些执行危险代码的指令，而你开了完全的访问权限，那可能就容易变成肉鸡；一定要注意这个问题。

------

其他的一些参考资料/工具，大概率是用不上：

- [CodexGuide](https://github.com/freestylefly/CodexGuide)

   面向全球初学者、创作者、开发者与团队的 Codex 实践指南

- [ClaudeCode指南](https://claude.nagdy.me/)

- Desktop CC GUI

   把 Claude Code、Codex CLI、OpenCode 这些命令行 AI 编程工具，装进了一个好看好用的图形界面里

- Kun

   把需求澄清、设计稿、计划和 Agent 编码串成完整闭环，使用多个不同的大模型

- TOKENICODE

   Claude Code 三方桌面客户端

- Nezha

   Claude Code + Codex 并发编程

在视觉方面，Grok 值得一试。

国产模型里，GLM 5.2 都说表现非常不错，DeepSeek 是省钱的不二之选。

