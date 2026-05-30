# 理解 Tailwind CSS 和 shadcn


Tailwind CSS 从名字看就知道是和 css 相关的，它是一个 Utility-First（工具类优先）CSS 框架，用于快速构建网页 UI，更像是一个 css 的增强写法，在 AI Coding 流行的现在，建议是优先选择。

简单说，它不像 Bootstrap 那样给你现成的按钮、卡片、导航栏组件，而是提供大量低层级的 CSS 工具类，可以简单理解为每一类 css 属性封装成一个 class 名称，让我们更优雅的写 css。<!--more-->

例如：`class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"`，对应的是：
``` css
.your-element {
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  background-color: #3b82f6;
  color: #ffffff;
  border-radius: 0.25rem;
}
.your-element:hover {
  background-color: #2563eb;
}
```

这样看，Tailwind 和 Bootstrap 类似，给你定义了一些默认的 class 名，你可以直接用，只不过 Tailwind 每一个名字对应一项 css 定义，**是来替代基础 css 写法**，也就是直接通过 class 的名字就能直接知道对应的 css 具体是什么，体验上类似行内样式。样式跟着元素走，完全不需要再翻 css 文件找样式。

Bootstrap 更偏向于提供“现成组件样式”，给你一套做好的组件外观。

Tailwind 更偏向于提供“原子化/工具类样式”，给你一套拼样式的基础积木。

需要注意：这些 CSS 的顺序是**框架内部确定的**，不是随机的，**也不是简单按你 class 字符串里的顺序决定**。

## 为什么会有 Tailwind
一个暴论：大部分项目的 CSS 代码都是屎山，因为这玩意也没法测试（没法单独看），大部分就是看起来没问题，那它就是没问题。

改也不敢改，只敢往上覆盖，叠 class， `!important` 大法！！

它解决了什么问题：
- CSS 失控，无法维护的问题
- 减少频繁写 CSS 文件
- 避免命名困难
- 更容易保持设计一致性
- 不需要考虑样式覆盖、不需要考虑各种选择器
- 一定程度上不需要理解各种单位问题，适配友好
- 深色模式实现简单
- AI 友好

Tailwind 并不是“完全不存在样式冲突”，而是它通过一套设计方式，把传统 CSS 中最常见的全局冲突、选择器覆盖、命名污染、优先级大战 大幅减少了。

核心原因是：Tailwind 的样式大多是原子化工具类，每个类只做一件事，而且类名和 CSS 规则是框架统一生成的，不需要你自己到处写**选择器**。

因此有人比喻说就是全部使用行内样式，每个类只负责一个样式，不依赖 DOM 层级，没有嵌套选择器。

打包时 Tailwind 会扫描你的代码，只保留用到的类。

同时，它也有另一个非常让人喜欢的特点，如果你发现一个使用 Tailwind 做的组件或者页面，理论上你直接 F12 进行 copy 就能完美的复刻。

### 命名规则

基本规则：属性缩写 - 取值 - 修饰等级

例如 text-blue-500：

- text --> 作用的 CSS 属性类型，这里是文字颜色 color
- blue --> 颜色名称
- 500 --> 颜色深浅等级

关于这个颜色深度，例如：

``` css
.text-blue-500 {
  color: #3b82f6;
}
```

Tailwind 的颜色通常有一组色阶：

50、100、200、300...900、950。

数字越小，颜色越浅，500 通常是这个颜色的基础色。当然，如果你不满意，可以通过任意值语法进行随意修改：`bg-[#1da1f2]`

类似的 `hover:bg-blue-600`  意思就是鼠标悬停时背景变深。

其他的类名和颜色类似，有一组对应关系可以参考官网，这个需要稍微适应下，适应后靠代码补全就写的很快了。

### 区别组件库

Tailwind CSS 更接近“写样式的工具/框架”，不是传统意义上的前端 UI 组件库（Ant Design、Element Plus、Material UI）。

它解决的是：如何高效写 CSS、组织样式、保持设计一致性的问题；

而 UI 组件库解决的是：直接给你可用的 Button、Modal、Table、Select 等组件的问题。

组件库不仅有样式，还有：

- disabled 状态
- loading 状态
- hover/focus 样式
- 尺寸变体
- 图标位置
- 主题系统
- 交互行为
- ...

而 Tailwind 只负责样式。按钮的 loading、disabled、图标、事件逻辑、组件封装，要你自己处理。

如果觉得自己封装很麻烦，可以搭配 Headless UI 这种只提供交互逻辑、不强绑定样式的组件库。

## shadcn/ui
shadcn/ui 是现在很流行的一种组件方案。其实底层主要是 Radix UI + Tailwind CSS。它基于 React、Tailwind CSS、Radix UI、TypeScript，常见于 Next.js、Vite 项目。

它不是传统意义上的“组件库”，它更像是一个可复制、可定制的 UI 组件集合。


特点是：

- 组件不是安装在 node_modules 里当黑盒使用，而是直接**复制到你的项目代码中，成为你自己代码库的一部分**。
- 不是 npm 组件库，而是“代码分发”
- 不是那种“大而全的企业中后台视觉风格”，而是更偏现代 Web App，极简风格。
- 按需引用，非常轻量化。

也就标志着可定制性很强，例如  `npx shadcn@latest add button`，它会把 Button 组件源码直接生成到你的项目里，之后你可以随便改它。

对应的缺点或者不适合的场景：
- 不适合完全不想维护组件源码的团队
- 企业级复杂组件不如 Ant Design 大而全（超复杂表格、穿梭框、高级日期范围选择、复杂上传、企业复杂表单布局等）

另一方面，和 Tailwind 一样，对 AI coding 非常友好，这也是它近年来进一步流行的重要原因。
AI coding 工具最怕黑盒依赖。

如果组件在 node_modules 里，AI 通常只能猜 API，或者需要查文档。
而 shadcn/ui 相当于直接给代码，AI 可以直接读取，并且结构稳定、模式统一，利于模型学习；
在 class 定义中非常直观，不需要 AI 记住复杂组件 API，省上下文。


### Radix UI 
Radix UI 是一个面向 React 的 无样式 / 低样式 UI 基础组件库。

也就是说，它主要提供组件的 行为、交互、可访问性和状态管理，但基本不替你决定最终长什么样。

例如它提供：

- Dialog 弹窗
- Popover 浮层
- Dropdown Menu 下拉菜单
- Tooltip 提示
- Select 选择器
- Tabs 选项卡
- Accordion 折叠面板
- Switch 开关
- Checkbox 复选框
- Radio Group 单选组
- Slider 滑块
- Toast 通知
- Scroll Area 滚动区域
- ...

但这些组件默认样式很少，你需要自己写 CSS、Tailwind 或其他样式。

把这些复杂交互和无障碍能力封装好，你只负责样式和业务。

Radix UI 负责交互和可访问性，shadcn/ui 负责把它包装成好看的组件源码，它们搭配使用非常的流畅。


### Skill 和 MCP

AI coding 的时候，经常会遇到这些错误：

- import 路径写错
- 把 shadcn/ui 当 npm 包
- 乱写按钮、弹窗、表单
- 不使用已有组件
- 不遵守 Tailwind token
- 不符合项目目录结构
- 生成的 UI 风格不统一

Skill 就是来解决 AI 应该怎么写，类比操作手册，解决 AI 不懂 shadcn/ui 习惯、不按项目约束写。

即使 AI 按规范写了，还存在很多的幻觉错误：

- 不知道官方现在有哪些组件
- 不知道组件最新写法
- 不知道某个 block 的结构
- 不知道组件依赖哪些包
- 不知道团队自定义 registry 里有什么
- 编造不存在的组件 API

MCP 就是解决现在有哪些组件、组件源码是什么、应该怎么安装、有没有示例，这一类问题，即解决 AI 缺少最新、准确的组件信息。

它给 AI 提供一个可以查询 shadcn/ui registry、组件源码、示例、block、安装信息的工具服务。它更偏动态工具能力和实时上下文获取。

让 AI 查询 shadcn/ui 有哪些组件、获取组件的真实源码和依赖、使用最新的官方组件结构、帮助 AI 判断应该添加哪个组件。

### registry
默认情况下，你使用的就是 shadcn/ui 官方 registry，对应的就是 shadcn/ui 的源码库。本质上是一套组件元数据 + 源码文件描述。

执行 add button 时，CLI 会去官方 registry 查找 button 这个条目，然后把相关源码复制到你的项目里。

shadcn/ui 支持添加来自其他 registry 的组件。如果团队要长期用，最好自己建 registry

### TailwindToken
Tailwind token 不是一个特别严格的官方术语，更多是大家在使用 Tailwind & shadcn/ui 时的一种说法。

它大概指：在 Tailwind 里被统一定义和复用的设计值，比如颜色、间距、圆角、字体、阴影等。

也可以理解为：Tailwind 里的 design tokens。在设计系统里，design token 指的是把设计里的基础值抽象成名字。

比如不要在代码里到处写，其实就相当于定义变量。

- primary = 品牌主色
- background = 页面背景色
- foreground = 正文文字色
- border = 边框色
- radius = 默认圆角

Tailwind 本身提供了一套 utility class。shadcn/ui 通常会在全局 CSS 里定义 CSS variables。

``` css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 240 10% 3.9%;

    --primary: 240 5.9% 10%;
    --primary-foreground: 0 0% 98%;

    --border: 240 5.9% 90%;
    --input: 240 5.9% 90%;
    --ring: 240 10% 3.9%;

    --radius: 0.5rem;
  }

  .dark {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;

    --primary: 0 0% 98%;
    --primary-foreground: 240 5.9% 10%;

    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 240 4.9% 83.9%;
  }
}
```

例如 bg-primary 的含义就是背景使用主品牌色；AI 很容易生成硬编码颜色，所以 AI coding 中经常强调 design token


官方 shadcn/ui 的主版本是基于 React 的，Vue 中也有对应的复刻项目，只是目前看还不太火，不过 Vue 生态的 Vite 是非常流行的。


## 关于 Next.js
Node.js 是一个让 JavaScript 可以运行在服务器端的运行环境。Next.js 是基于 React 的**全栈** Web 框架，支持：

- 页面路由
- 服务端渲染 SSR
- 静态生成 SSG
- API Routes
- 图片优化
- 前后端一体化开发
- 部署优化

经常用的 npm run dev 这个命令实际上是通过 Node.js 启动 Next.js 开发服务器。

React 不像我们常说的 HeroUI 这种组件库，它主要提供的是**一套构建 UI 的基础机制**，比如：

- 组件模型
- JSX
- 状态管理：useState
- 副作用处理：useEffect
- 组件复用
- 虚拟 DOM / 更新机制
- Hooks
- 单向数据流

React 是用于构建用户界面的 JavaScript 库, 是现代前端页面开发的基础框架/基础层。

官方更喜欢称它为 library，不叫 framework，因为 React 只管 UI 层，不强制规定路由、数据请求、构建、服务端渲染、项目结构等内容。

Next.js 是基于 React 的 Web 应用框架，它在 React 之上补齐了很多实际开发网站需要的能力，例如 API 接口定义，这个层面来说类似一个后端服务。

正如它定义中的全栈二字：在一个 Next.js 项目里，可以同时写前端页面和后端 Web 逻辑。


从 2016 年之后，Next.js 就逐渐成为 React 做网站、尤其是需要 SSR/SEO 场景时的主流选择；

简单理解为 React + Next.js 相当于 React + React Router + Vite/Webpack + SSR 方案 + API 框架 + 部署配置。

其他的一些常见组合：

- React + Vite + React Router 
  纯前端 SPA、后台管理系统
- Electron + React
  桌面应用
- React + Gatsby
  静态网站、内容站
- React Native
  移动 App

React Router 作用：在 React 单页应用 SPA 里，根据 URL 显示不同页面组件。

Next.js 内置了类似 React Router 的功能，但实现方式不同。

React Router 只管路由。Vite 主要管开发服务器和打包。Next.js 可以说啥都管。

通过是否需要服务端渲染、SEO、公开访问性能优化、内容预生成、前后端一体化等能力来决定时候适合单页面 SPA 应用，例如后台管理系统就非常适合 SPA。


Vite 目前作为纯静态前端打包工具，优点是：

- 架构简单
- 开发快
- 前后端边界清晰
- 部署简单，打包后就是静态文件
- 不需要维护 Node.js SSR 服务器


当然 Next.js 可以打包成纯静态资源文件，就是没有 Vite 那么的轻量级。

