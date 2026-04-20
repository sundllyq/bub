# Bub 框架设计解读书：全书目录与阅读建议

## 全书目录

### 前言

- 标题：《Bub 框架设计解读：一个共处型 Agent Runtime 的架构选择》
- 这部分要解决什么问题：定义全书边界，说明这本书写给谁看。它写给想理解 Bub 的架构、运行机制、扩展方式与适用边界的开发者；不写给只想安装、跑命令、照着 API 调用的读者。前言还要先回答三件事：Bub 不是聊天壳，它是一个以 turn pipeline 为核心的 runtime；它不是工作流编排器，它优先解决 agent 与人、与渠道、与工具共处的问题；它也不是强治理平台，它在扩展性和类型约束之间做了明显偏向。

### 第 1 篇 问题定义与总图

#### 第 1 章 Bub 在解决什么问题

- 核心问题：Bub 到底是什么，它要解决的核心矛盾是什么，它明确不想解决什么。
- 为什么值得单独成章：这是整本书的观察坐标，不先定性，后面所有设计判断都会失焦。
- 前置章节：无。
- 建议关注的源码入口：`README.md`、`src/bub/framework.py`、`src/bub/channels/manager.py`。

#### 第 2 章 为什么 Bub 的核心不是 Agent，而是一条 Turn Pipeline

- 核心问题：为什么框架把“一次消息处理链路”做成第一公民，而把 Agent 下沉为默认实现。
- 为什么值得单独成章：这是 Bub 与很多 agent 框架最根本的分野，直接决定它的主线、扩展点和宿主复用方式。
- 前置章节：第 1 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/builtin/hook_impl.py`、`tests/test_framework.py`。

#### 第 3 章 Core、Builtin、Host、Capability 的分层边界

- 核心问题：Bub 的系统分层如何组织，哪些职责属于 core，哪些故意留给 builtin、channel、tool、skill。
- 为什么值得单独成章：它回答“为什么代码看起来很散但主线并不乱”，也是后续所有章节的地图。
- 前置章节：第 1、2 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/builtin/agent.py`、`src/bub/tools.py`、`src/bub/skills.py`。

### 第 2 篇 运行时内核

#### 第 4 章 HookRuntime：Bub 的真实执行语义

- 核心问题：Bub 如何借 pluggy 做注册，但用自己的规则定义优先级、first-result、broadcast、sync/async 兼容和错误观察。
- 为什么值得单独成章：真正决定系统行为的不是 hookspec 名字，而是 HookRuntime 的调度语义；扩展 Bub 必须先懂这一层。
- 前置章节：第 2、3 章。
- 建议关注的源码入口：`src/bub/hookspecs.py`、`src/bub/hook_runtime.py`、`tests/test_hook_runtime.py`。

#### 第 5 章 一次 Inbound Turn 的生命周期

- 核心问题：一条 inbound 消息如何完成 `resolve_session -> load_state -> build_prompt -> run_model -> save_state -> render_outbound -> dispatch_outbound`。
- 为什么值得单独成章：这是全框架最重要的控制流，也是 CLI、Telegram、后续宿主共用的一条主链。
- 前置章节：第 2、4 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/builtin/hook_impl.py`、`tests/test_builtin_hook_impl.py`。

#### 第 6 章 流式优先的模型执行

- 核心问题：为什么 `run_model_stream` 是主接口，`run_model` 只是兼容层；流式输出如何贯通到宿主。
- 为什么值得单独成章：这不仅是接口差异，而是对交互体验、宿主渲染和错误传播的统一选择。
- 前置章节：第 4、5 章。
- 建议关注的源码入口：`src/bub/hookspecs.py`、`src/bub/framework.py`、`src/bub/channels/cli/__init__.py`。

### 第 3 篇 默认 Agent 与装配机制

#### 第 7 章 默认 Agent 的执行循环

- 核心问题：默认 Agent 如何在命令快路径、LLM 调用、工具执行、自动 continue 之间推进任务。
- 为什么值得单独成章：Bub 的 core 不等于 Agent，但默认 Agent 代表了 Bub 对“可工作的 agent loop”给出的具体答案。
- 前置章节：第 5、6 章。
- 建议关注的源码入口：`src/bub/builtin/agent.py`、`tests/test_builtin_agent.py`。

#### 第 8 章 Prompt 不是一段字符串：system、tools、skills、context 的装配

- 核心问题：最终送进模型的 prompt 到底由哪些层拼出来，它们的顺序、来源和覆盖关系是什么。
- 为什么值得单独成章：Bub 的 prompt 不是单点模板，而是系统装配结果；这直接决定模型能力边界和扩展方式。
- 前置章节：第 5、7 章。
- 建议关注的源码入口：`src/bub/builtin/agent.py`、`src/bub/builtin/hook_impl.py`、`src/bub/tools.py`、`src/bub/skills.py`。

#### 第 9 章 配置、Provider 与运行环境绑定

- 核心问题：模型名、凭证、fallback、timeout、workspace、AGENTS.md 如何进入运行时。
- 为什么值得单独成章：它解释 Bub 为什么能保持 core 很薄，同时又把模型/环境差异注入到默认运行时。
- 前置章节：第 3、7、8 章。
- 建议关注的源码入口：`src/bub/builtin/settings.py`、`src/bub/builtin/hook_impl.py`、`tests/test_settings.py`。

### 第 4 篇 会话、状态与记忆

#### 第 10 章 Session 与 State：弱约束共享状态的代价与收益

- 核心问题：Bub 如何定义 session identity，为什么 `Envelope` 和 `State` 被刻意保持为弱类型边界。
- 为什么值得单独成章：这是很多实现选择的根源，也是 Bub 为扩展性付出的主要工程代价。
- 前置章节：第 3、5 章。
- 建议关注的源码入口：`src/bub/types.py`、`src/bub/envelope.py`、`src/bub/channels/message.py`。

#### 第 11 章 Tape 不是日志：上下文、记忆与审计的统一底座

- 核心问题：为什么 Bub 不做 session 累积，而要把上下文重建、anchor、search、reset、handoff 都放进 Tape。
- 为什么值得单独成章：这是 Bub 最稳定、最非主流、也最能解释其长期任务能力的抽象。
- 前置章节：第 7、10 章。
- 建议关注的源码入口：`src/bub/builtin/tape.py`、`src/bub/builtin/context.py`、`src/bub/builtin/store.py`。

#### 第 12 章 Fork、Handoff 与 Merge-Back：长任务和子代理的隔离策略

- 核心问题：主会话、临时子代理、上下文溢出恢复为什么都依赖 fork tape，而不是复制对话历史。
- 为什么值得单独成章：它把 agent loop、memory 模型和子代理语义连成一个完整系统。
- 前置章节：第 7、11 章。
- 建议关注的源码入口：`src/bub/builtin/agent.py`、`src/bub/builtin/store.py`、`tests/test_fork_store_merge_back.py`、`tests/test_subagent_tool.py`。

### 第 5 篇 能力层设计

#### 第 13 章 Tool 注册表与模型可见名

- 核心问题：为什么运行时工具名和模型看到的工具名要分离，为什么 Bub 要维护一个中心注册表。
- 为什么值得单独成章：这不是装饰器语法问题，而是模型调用契约、日志和扩展兼容性的汇合点。
- 前置章节：第 7、8 章。
- 建议关注的源码入口：`src/bub/tools.py`、`tests/test_tools.py`。

#### 第 14 章 操作员直达路径：逗号命令、Shell 与文件工具

- 核心问题：为什么 Bub 明确保留一条绕过自然语言推理的操作员通道。
- 为什么值得单独成章：这体现 Bub 不是纯聊天代理，而是面向真实协作场景的操控型 runtime。
- 前置章节：第 7、13 章。
- 建议关注的源码入口：`src/bub/builtin/agent.py`、`src/bub/builtin/tools.py`、`src/bub/channels/cli/__init__.py`。

#### 第 15 章 能力边界与风险暴露

- 核心问题：Bub 的工具系统负责什么，不负责什么；权限、路径、宿主约束为什么没有被框架统一收口。
- 为什么值得单独成章：它直接回答 Bub 解决了什么，也明确回答它放弃了什么。
- 前置章节：第 10、13、14 章。
- 建议关注的源码入口：`src/bub/builtin/tools.py`、`docs/features.md`、`src/bub/types.py`。

### 第 6 篇 扩展机制

#### 第 16 章 Hook、Tool、Skill、Subagent：四层扩展而不是一种插件

- 核心问题：为什么 Bub 不把所有扩展压成统一插件接口，而是保留四个不同层次。
- 为什么值得单独成章：这章最能解释 Bub 的扩展哲学，也最能帮助读者判断“该从哪一层改”。
- 前置章节：第 4、8、13 章。
- 建议关注的源码入口：`src/bub/hookspecs.py`、`src/bub/tools.py`、`src/bub/skills.py`、`src/bub/builtin/tools.py`。

#### 第 17 章 为什么 Skill 是 `SKILL.md`，而不是 Python 代码

- 核心问题：为什么技能走“文档发现 + frontmatter 校验 + 按需展开”这条路，而不是插件类注册。
- 为什么值得单独成章：这是 Bub 很独特的能力扩展面，既影响 prompt 设计，也影响分发与覆盖策略。
- 前置章节：第 8、16 章。
- 建议关注的源码入口：`src/bub/skills.py`、`tests/test_skills.py`。

#### 第 18 章 内置优先级、覆盖规则与分发方式

- 核心问题：builtin 为什么要先注册、外部插件为什么后注册反而优先，技能与插件又如何一起打包分发。
- 为什么值得单独成章：它回答“Bub 怎样在可替换默认值和稳定运行之间取得平衡”。
- 前置章节：第 4、16、17 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/skills.py`、`docs/extension-guide.md`。

### 第 7 篇 多宿主复用

#### 第 19 章 Channel 抽象与 ChannelManager

- 核心问题：Bub 如何把不同消息宿主接进同一条 runtime，又如何把输出安全地路由回去。
- 为什么值得单独成章：这是 Bub 能跨 CLI、Telegram 复用的关键中介层。
- 前置章节：第 3、5 章。
- 建议关注的源码入口：`src/bub/channels/base.py`、`src/bub/channels/manager.py`、`tests/test_channels.py`。

#### 第 20 章 会话并发控制：Debounce、Active Window 与 Quit

- 核心问题：为什么并发和会话治理主要落在 channel 层，而不是 agent core。
- 为什么值得单独成章：Bub 的问题场景从一开始就不是单用户串行对话，这一章直接体现它的现实约束。
- 前置章节：第 19 章。
- 建议关注的源码入口：`src/bub/channels/handler.py`、`src/bub/channels/manager.py`、`tests/test_channels.py`。

#### 第 21 章 CLI 与 Telegram：两种宿主，两种交互表面

- 核心问题：同一个 runtime 为什么能同时表现为交互式终端和聊天机器人，它们各自向内核暴露了什么。
- 为什么值得单独成章：这章是“宿主复用”最具体的落点，也能说明 Bub 当前为何不是 API-first。
- 前置章节：第 19、20 章。
- 建议关注的源码入口：`src/bub/channels/cli/__init__.py`、`src/bub/channels/telegram.py`、`src/bub/builtin/cli.py`。

#### 第 22 章 流包装与出站路由：Runtime 与 UI/Transport 的边界

- 核心问题：流式输出、错误消息、host-specific 渲染如何被隔离在运行时之外。
- 为什么值得单独成章：它回答“为什么 framework 不需要知道自己运行在终端还是聊天窗口里”。
- 前置章节：第 6、19、21 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/channels/manager.py`、`src/bub/channels/cli/__init__.py`。

### 第 8 篇 产品化案例与设计边界

#### 第 23 章 案例一：把 Bub 当成交互式协作终端

- 核心问题：当 Bub 运行在 CLI 中时，哪些设计让它像“协作者”，而不是一层 shell 包装。
- 为什么值得单独成章：这是最容易落地的产品化形态，也是理解 operator-centric 设计的最佳案例。
- 前置章节：第 7、14、21、22 章。
- 建议关注的源码入口：`src/bub/channels/cli/__init__.py`、`src/bub/builtin/tools.py`。

#### 第 24 章 案例二：把 Bub 放进群聊与异步会话

- 核心问题：当 Bub 进入 Telegram 群聊时，哪些机制保证它能共处而不是打扰。
- 为什么值得单独成章：这正是 `README.md` 中最反复强调的原生场景，也是 Bub 设计哲学最直接的来源。
- 前置章节：第 19、20、21 章。
- 建议关注的源码入口：`src/bub/channels/telegram.py`、`src/bub/channels/manager.py`。

#### 第 25 章 案例三：把 Bub 作为可分发的扩展底座

- 核心问题：当团队要分发一个带 hooks、tools、skills 的扩展包时，Bub 提供了什么，缺少什么。
- 为什么值得单独成章：它把“框架扩展”从源码概念落到真实工程交付。
- 前置章节：第 16、17、18 章。
- 建议关注的源码入口：`src/bub/framework.py`、`src/bub/tools.py`、`src/bub/skills.py`、`docs/extension-guide.md`。

#### 第 26 章 反主流选择、适用边界与不该使用 Bub 的场景

- 核心问题：Bub 通过 Hook-first、Tape-first、Skill-as-doc、弱类型边界换来了什么，又放弃了什么。
- 为什么值得单独成章：如果不正面回答边界，整本书就会变成替框架圆场。
- 前置章节：全书主体。
- 建议关注的源码入口：`README.md`、`docs/features.md`、`src/bub/types.py`。

### 第 9 篇 附录与阅读路线

#### 第 27 章 源码阅读地图：按设计问题而不是按目录树读

- 核心问题：第一次进仓库，怎样沿着设计问题读源码，而不是被文件树带偏。
- 为什么值得单独成章：它直接服务本书的阅读目的，也能避免读者掉进“逐文件导读”的低效路径。
- 前置章节：第 1、3 章。
- 建议关注的源码入口：`docs/read-v2/README.md`、`src/bub/framework.py`、`src/bub/builtin/agent.py`、`src/bub/channels/manager.py`。

#### 第 28 章 二次开发切口：新增宿主、新增插件、新增技能、新增工具

- 核心问题：要改 Bub，第一刀应该落在哪里，什么应该继承，什么应该替换。
- 为什么值得单独成章：这章把全书收束到工程实践，给二次开发一个可执行入口。
- 前置章节：第 16 至 22 章。
- 建议关注的源码入口：`docs/extension-guide.md`、`src/bub/hookspecs.py`、`src/bub/tools.py`、`src/bub/skills.py`。

## 阅读建议

### 架构理解路线

- 第 1 章 -> 第 2 章 -> 第 4 章 -> 第 5 章 -> 第 7 章 -> 第 11 章 -> 第 16 章 -> 第 19 章 -> 第 26 章。
- 这条路线先抓框架形状，再看运行时语义、默认 agent、记忆模型、扩展机制和边界判断。

### 源码阅读路线

- 第 3 章 -> 第 5 章 -> 第 4 章 -> 第 7 章 -> 第 8 章 -> 第 11 章 -> 第 12 章 -> 第 19 章 -> 第 21 章 -> 第 27 章。
- 这条路线优先对应核心文件，适合边读书边开源码。

### 二次开发路线

- 第 16 章 -> 第 18 章 -> 第 13 章 -> 第 17 章 -> 第 19 章 -> 第 22 章 -> 第 28 章。
- 这条路线直接服务“我要扩展 Bub”，先判断该用哪种扩展层，再落到工具、技能、宿主和出站边界。

## 章节筛选说明

- 这些章节被保留，因为它们满足三个条件：是稳定抽象，不是偶然实现；背后有明确取舍，不是纯机械拼装；横跨多个模块，能解释系统级行为。
- 没有把每个模块单独成章，因为很多文件只是薄包装、具体适配或实现细节。`__main__.py`、`auth.py`、单个内置工具、Telegram parser、JSONL 搜索细节都重要，但不足以单独解释 Bub 的架构。
- 这套目录刻意围绕“设计问题”而不是“代码位置”组织。读者读完应该能回答“为什么这里是 hook、那里是 tool、为什么上下文来自 tape、为什么宿主层负责并发控制”，而不是只记住文件名。
- 最能代表 Bub 设计哲学的章节是第 2 章、第 4 章、第 11 章、第 16 章、第 26 章。它们分别对应 Bub 的五个核心判断：以 turn 为核心、以 HookRuntime 定义语义、以 Tape 定义记忆、以多层扩展而非单一插件扩展能力、明确承认自己的适用边界。
