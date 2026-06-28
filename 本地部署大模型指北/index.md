# 本地部署大模型指北


距离上次了解这部分已经很久了，目前的生态发生了非常多的变化，主要是发现之前发文章的网站倒闭了，也有必要写一篇新的与时俱进。

先说 LLM 方向，上一次还是 Ollama + OpenWebUI 搭建运行环境，目前的环境下 Ollama 已经跌落神坛，效率不行，伪开源商业化严重，已经是非必要不选择的状态。关于它的罪恶，可以看网友的[这篇文章](https://sleepingrobots.com/dreams/stop-using-ollama/)。<!--more-->

Ollama 运行环境的核心其实是 [llama.cpp](https://github.com/ggml-org/llama.cpp) 这个开源项目，所以我们其实是直接跑 llama.cpp 没有中间商赚差价，效率更高，只不过需要懂一些命令行。

当然也有上手难度低的易用版本：

- [jan](https://github.com/janhq/jan)
- [LM Studio](https://lmstudio.ai/link)
- [Unsloth Studio](https://unsloth.ai/docs/zh)

前面俩都是不错的客户端，第三个除了作为 Chat 还可以进行训练，Unsloth 出品值得一试吧，如果想训练的话可以用。

当然，就算直接跑 llama.cpp 也有一些启动器可以方便我们修改配置而不需要手动去写命令。

- [Sakura](https://github.com/PiDanShouRouZhouXD/Sakura_Launcher_GUI)

- [LlamaOptGUI_V5](https://www.bilibili.com/video/BV1VL9wBbEbR)

- [llama.cpp-hub](https://github.com/IIIIIllllIIIIIlllll/llama.cpp-hub)

  Web 壳，用图形化的 Web 页面来操作模型和 llama.cpp。内置了很多花里胡哨的功能。

- [llama-swap](https://github.com/mostlygeek/llama-swap)

当然，如果不想写启动脚本或者打命令，我还是更推荐使用启动器，本质就是替你写参数，把参数用 GUI 的方式体现出来，自己让 AI 给写个启动脚本也没啥问题。

## 环境准备

本地运行 LLM 都知道需要显卡，模型的大小和显存直接关联，根据上面的介绍，选择适合自己的模型。

这里以最常用的 N 卡举例，可以安装下 **CUDA**，先查看驱动支持的版本，然后去下载 CUDA Toolkit，安装完后验证。

```cmd
# 查看驱动支持的 cuda 版本
nvidia-smi
# 查看 cuda 版本
nvcc -V
```

当然，跑 也不一定需要 CUDA Toolkit，官方的发布页会提供一个类似 CUDA 13.1 DLLs 这样的下载包，会包含相关 dll 依赖，大概率是可以直接跑起来。

下载地址：<br/>
https://developer.nvidia.com/cuda-downloads<br/>
https://developer.nvidia.com/cuda-toolkit-archive

如果你不写 CUDA C++，不微调编译，那么理论上只装 Runtime 就可以；但是 nvcc 命令可能就用不了了；甚至不需要 Runtime，因为 llama.cpp、PyTorch 这类都会自带运行库。

接下来需要的环境：

- **Python**

  不用多说，AI 多少也需要它

- **PyTorch**

  深度学习 / 机器学习框架，主要用来构建、训练和部署神经网络模型。

  对于我们这种纯跑模型的，只需要知道它可以加速模型推理就可以了。

  对于低版本可能还需要 xformers 可以进一步加速，高版本一般不需要了，主要是为了在图像生成方面加速。

- **uv**

  py 环境、版本、依赖管理流行工具

- **modelscope**

  用来从魔搭下载模型，国内满速，可以配置 MODELSCOPE_CACHE 环境变量来控制下载目录；

  如果你 Hugging Face 速度很快，那倒是也不需要。

- **Node.js**

上面的环境作为开发者不难搭建，甚至本来的电脑多多少少就有，准备好后就可以下模型跑了。

## GGUF

前置知识：需要先了解[量化、MoE、Dense 等概念](https://moe.sakanoy.com/大模型之densemoe与量化/)才能更好的知道自己需要什么格式的模型。

GGUF 是一种文件格式，可以把它理解成：**大模型的“打包格式”**。
 GGUF 全称 **GPT-Generated Unified Format**。它是以前 **GGML / GGMF / GGJT** 这些格式路线后面发展出来的，它更统一，也更适合后续扩展。

**它可以保存 AI 模型权重、配置信息，以及一些附加元数据的一种格式**，主要常见在 **本地运行大语言模型** 的场景里，让你开箱即用，例如我们常用的 LM Studio、llama.cpp 都是使用 GGUF 加载。

比如一个模型原本可能是 Hugging Face 上的 PyTorch 权重的 safetensors 文件，文件很大、没有进行量化、结构也偏训练框架化。但你可能只是需要量化后的省内存、省显存能跑在个人电脑的模型；那就常常会把模型转换成 **GGUF** 格式。

你可以把 GGUF 理解成类似这样几个比喻：

- **像压缩包**：把模型相关内容装进去
- **像安装包**：方便推理程序直接读取
- **像通用容器**：里面不只是权重，还有 tokenizer、上下文长度、量化信息等

不过它**不等于普通压缩文件**，重点是它是为**高效推理加载**设计的。

所以它很适合“**下载下来就跑**”的本地模型体验，但是一些大模型官方很少提供 GGUF，多数还是 safetensors；这时候就可以找一下比较热门可信赖的机构发布的 GGUF 文件。

## 模型选择

实际的显存占用是模型权重、KV cache、上下文长度、运行后端、多模态模块、批大小共同决定。

而 GGUF 文件体积一般就是**量化后的模型权重大小**，所以一个简单的判断就是模型文件 +2G（低于 8K 上下文）要小于你的显存才能完整的载入的显存中。

如果模型比显存大很多要慎重了，不是跑不起来（放不下的需要 offload 到 CPU），而是 token 速度是无法用的状态。

| 显存 | 比较合适的选择                                  |
| ---- | ----------------------------------------------- |
| 8GB  | 27B / 35B - A3B 的 2-bit 极限尝试，质量风险较高 |
| 12GB | 27B Q2/Q3，35B-A3B Q2/Q3 短上下文               |
| 16GB | 27B Q3/Q4，35B-A3B Q3/IQ4_XS                    |
| 24GB | 27B Q4/Q5/Q6，35B-A3B Q4                        |
| 32GB | 27B Q8，35B-A3B Q5/Q6                           |

以我 16G 显存来说，选择 Q3_K_M、IQ4_XS、Q4_K_M（理论 20G 显存）根据实际情况尝试。

- 12GB 显存：尝试 `27B UD-IQ2_M` 或 `35B-A3B UD-IQ2_M`，上下文要短。
- 16GB 显存：尝试 `27B Q3_K_M` 或 `35B-A3B UD-IQ3_XXS`。
- 24GB 显存：优先看 `27B Q4_K_M`、`35B-A3B UD-IQ4_NL`、`35B-A3B UD-Q4_K_M`。

对于现在热门的 Qwen3.6，我找到的一些不错量化版本：

- **unsloth/Qwen3.6-35B-A3B-GGUF**

  对于我 16G 显存来说，选择的 Qwen3.6-35B-A3B-UD-IQ4_XS，需要搭配 offload

- **mudler/Carnice-Qwen3.6-MoE-35B-A3B-APEX-GGUF**

  APEX 量化，优先选择，我自己情况选择的 Carnice-Qwen3.6-MoE-35B-A3B-APEX-I-Quality；或者选 I-Balanced 版本。

- **unsloth/Qwen3.6-27B-GGUF**

  稠密模型，根据我的内存选择 Qwen3.6-27B-IQ4_XS.gguf / Qwen3.6-27B-Q3_K_M

需要注意：如果下载使用多模态的大模型，一般是需要下载同目录下的 mmproj-BF16 文件（mmproj 文件通常保持全精度，因为视觉编码器对量化比语言模型更敏感，量化后质量下降明显），显卡不支持的情况下换用 F16 吧，要不然多模态可能无法使用。

也有网友反馈以我的这个硬件标准，跑 3.6 的 35B-A3B 量化 Q4_KM 也不错，参数需要先测试出最优参数。

目前的 Qwen3.6 是比较全能的模型，如果有其他特定场景下的需求，可以看看特化模型，对硬件要求要低的多。

具体选择上，可以参考这两个网站，根据你 PC 配置进行建议：

- [Can I Run AI locally](https://www.canirun.ai/)

- [趋近智](https://apxml.com/zh/tools/vram-calculator)

  显存计算和热度排名

当然，以上也只是理论参考，实际情况和使用环境的差别影响也很大。

根据之前的 MoE 原理文章，一定要记住，除非是特殊需求，不要使用 MoE 的蒸馏模型，绝大多数训练的真把握不住。

### 参数选择

4B “智商”的模型已经可以执行一些简单的逻辑了，例如当视觉识别、执行器、判断器。显存大约 3-5G。

9B 的可以拿来做一些文本总结、日常对话、OCR，需要大概 6-8G 显存。

一些特化的翻译模型就是基于 4B/7B 来做的。

27B 以上基本上就属于通用模型了，各方面都比较好了。

## WebSearch

为大模型提供网络搜索能力，搜索是很重要的功能，尤其是在 Agent 中，它可以通过搜索文档或者相关知识给你更高质量的输出。

最开始像 Google Bing 这类搜索引擎供应商是提供 api 的，但是后续都关了，现在这条路不要想了。

目前还存活的几类：

- 爬取搜索引擎结果的 SERP(search engine result page) API 的供应商，如 serpAPI、serper、searchAPI 等。
- 自建搜索 API 的，现在也大都和 AI 集成的比较丝滑，EXA、Tavily、Brave、Linkup 等。
- 用爬虫技术发起搜索引擎请求的工具，如 Duckduckgo MCP、orz-mcp 等，适用于反爬虫比较弱的搜索引擎。
- AI 模型自带搜索功能 OpenAI、Anthropic、perplexity 等。

或者你可以使用 **SearXNG** 来本地搭建一个搜索引擎，它可以同时向 Google、Bing、DuckDuckGo 等近百个独立的搜索引擎发送查询请求，聚合整理出结果，有大规模使用需求的同学可以试试。

这些工具有一定的差异，不过我们关注的还是性价比；

结论就是：首先推荐 exa，因为可以纯免费的体验使用，但是会限流拉黑 IP，不要并发使用（可能 1 qps）；

限制少功能多一些选 tavily，每月 1000 次额度，1 qps，不需要绑卡，很良心；

linkup 也可以考虑，同样每月 1000 次额度，qps 会高一些。

参考：[Websearch工具对比](https://www.xiaogenban1993.com/blog/26.03/WebSearch工具对比)

## 特化模型

这一些模型的特点是在一些垂直领域进行特化训练的，所以在一些固定场景是非常强的，并且性能消耗很低，所以如果有固定场景，这一类模型的性价比非常高。

### 通用翻译

腾讯的 **Hy-MT2-1.8B** 反馈是不错的；1.8B 量化版本甚至只需要不到 1G 内存. 随便找个 MacMini 都能跑. 直接能搭配使用翻译网页, 游戏 PDF, 电子书啥的都能搞定.

其中 Hy-MT2-30B-A3B 在 DomainMTBench (这是个专门测试特定领域翻译能力的 benchmark, 包含金融, 法律, 医疗, 技术等) 测试中全面超越了 DeepSeek-V4-Pro.

翻译这个领域其实还可以细分，例如对于特点语言，日语轻小说、视觉游戏现在不需要等汉化组，直接上 SakuraLLM 配合 LunaTranslator 可以无缝翻译。

### 同声传译

看到谷歌最近发布了 Gemini 3.5 Live Translate，支持 70 多种语言的实时语音到语音翻译；具体还需要进一步了解其效果，不过可以预见，以后不同语言的交流应该都不是问题。

### 编码

目前看到千问有单独的特化模型：Qwen 3 Coder；但是在能跑的情况下，不少人依然还是选择通用模型 3.6，实际效果需要验证。

### 角色扮演

可以尝试 Gemma 系列的，比较有活人感，例如 Gemma4-31B-it，只不过需要24+ 的显存，也比较容易破限。

说起破限，Mistral 系列（Mistral-small-3.2-24B）也比较容易，并且它有 14B 无限制版本，显存需求更小（12 - 16G），所以不少拿来做微调。

## 应用推荐

一些你可能用的上的应用：

- AnythingLLM

  类似 OpenWebUI，不过是 RAG 方向，管理自己的知识库，提供桌面端

- LocalAI

  提供 docker 部署，除了提供一个 Web 的皮，还具有下载跑模型的能力，并且提供统一的 API。

- OpenWebUI

  本地 LLM 获取中转站的 Web 皮，如果是简单聊天有点重，docker 镜像要 1G+

- ChatBox

  客户端应用，如果不喜欢 WebUI 可以尝试，轻量级

我本来是想找一个 OpenWebUI 差不多页面效果但是资源消耗很低的，还没有找到。

其他的一些模型测试工具：

- [EvalScope](https://github.com/modelscope/evalscope)
- [LLM API Benchmark Tool](https://github.com/Yoosu-L/llmapibenchmark)

最后一个暴论：以现在某些大模型的价格，要比自己部署便宜的多，不光是机器的成本，单是从电费角度考虑都比我们自己搞便宜太多。真就是算力的尽头是电力？

