# 第一章 Mastra 概述与快速上手

> 本章对齐官方文档：[Quickstart](https://mastra.ai/guides/getting-started/quickstart)、[Manual Install](https://mastra.ai/docs/getting-started/manual-install)、[Project Structure](https://mastra.ai/reference/project-structure)、[Studio](https://mastra.ai/docs/studio/overview)、[Agents Overview](https://mastra.ai/docs/agents/overview)、[create-mastra CLI](https://mastra.ai/reference/cli/create-mastra)、[GitHub](https://github.com/mastra-ai/mastra)。  
> 跟着做完，你将拥有：一个可运行的 Mastra 项目 + Studio 调试环境 + 一个可对话的 Agent。

---

## 1.1 Mastra 是什么

[Mastra](https://mastra.ai) 是一个面向现代 TypeScript 技术栈的 AI 应用与智能体（Agent）框架，由 Gatsby 团队打造，开源仓库见 [mastra-ai/mastra](https://github.com/mastra-ai/mastra)。

官方定位可以概括为：

> 用 TypeScript 从早期原型一路做到可上线的 AI 产品；既能嵌进 React / Next.js / Node，也能作为独立服务部署。

它帮你做的事包括：

- 用几行代码创建能调用工具、具备记忆** 的 Agent
- 用图引擎 + 链式 API（`.then()` / `.branch()` / `.parallel()`）编排多步骤 Workflow
- 通过 **MCP** 暴露或接入外部工具与服务
- 内置 **Studio** 做本地调试，无需先写前端 UI
- 配套 Evals、Observability 等生产向能力

### 为什么用 Mastra，而不是 Deep Agents / AgentScope / Hermes Agent / OpenClaw？

> **对比口径（截至 2026 年 7 月）：** 这里的 Harness Agent 指的不只是一个 ReAct 循环，而是围绕模型补齐 workspace、文件与 Shell 工具、长任务状态、记忆、上下文压缩、子 Agent、权限与沙箱、调试观测、部署和产品接入的一整套运行环境。对比以各项目已经公开的官方文档与开源实现为准，不把路线图当作现有能力。

先区分两类产品，否则很容易把「框架」和「成品 Agent」混在一起：

- **可嵌入的 Agent 框架 / 运行时：** [Mastra](https://mastra.ai)、[Deep Agents](https://github.com/langchain-ai/deepagents)、[AgentScope](https://github.com/agentscope-ai/agentscope)。适合开发自己的 SaaS、企业应用或垂直 Agent，并控制 API、UI、数据模型和租户体系。
- **可直接运行的通用 Agent 产品：** [Hermes Agent](https://github.com/NousResearch/hermes-agent)、[OpenClaw](https://github.com/openclaw/openclaw)。它们已经带 CLI、消息渠道、技能和长期运行方式，最快得到的是「一个能工作的助手」，但要改造成深度白标、多租户的产品，通常需要拆改其既有产品假设。

#### 面向“快速做出 Harness Agent 产品”的能力对比

| 维度 | Mastra | LangChain Deep Agents | AgentScope | Hermes Agent | OpenClaw |
|---|---|---|---|---|---|
| **主要定位** | TS AI 应用框架 + 官方 Agent Harness 模板 | 基于 LangGraph 的 batteries-included Agent harness | Python/Java Agent 框架 + Runtime；Java 2.0 提供 `HarnessAgent` | 自改进、可长期运行的个人 Agent 成品 | 消息渠道优先、可自托管的个人 Agent 平台 |
| **最快起点** | 一条 CLI 从模板生成 workspace、Shell、memory、tasks、web、schedule、storage、observability | `create_deep_agent()` 即得规划、文件系统、子 Agent 与上下文管理 | Python `ReActAgent` 起步快；Java `HarnessAgent` 一次装配 workspace、memory、session、subagent、sandbox | 安装后即可通过 CLI / Desktop / 消息平台使用，内置大量工具 | 安装并配置 Gateway、模型和 channel 后即可作为常驻助手使用 |
| **主要语言 / 产品栈** | **TypeScript 原生**，与 Node、React、Next.js 同栈 | Python 与 TypeScript 均有官方 SDK；依托 LangChain / LangGraph | Python 与 Java；Java 2.0 的 Harness 能力尤其完整 | Python 为主，偏运行产品与插件扩展 | TypeScript 为主，偏 Gateway、channels 与插件扩展 |
| **Agent loop 与编排** | Agent + 显式 Workflow 图；开放任务和确定性流程可放在同一项目 | 长任务默认值成熟；底层 LangGraph 支持持久化图、流式与中断 | ReAct、planning、message hub、多 Agent workflow、HITL；Java 有 middleware/toolkit 扩展点 | 内置自治循环、todo/plan、并行 delegation、定时任务 | 内置 Agent runtime、会话队列、cron、Agent / subagent 与多 Agent 路由 |
| **Workspace / 文件 / Shell** | 官方 Harness 模板开箱即用；支持审批门 | 一等公民；filesystem backend 可换成本地、沙箱或远端 | Runtime + sandbox + workspace；Java 可持久化运行态与 sandbox snapshot | file、terminal、process、checkpoint/rollback；可切本地、Docker、SSH、Daytona、Modal 等 backend | 每 Agent 独立 workspace/state/session；文件、Shell 和工具策略配置成熟 |
| **沙箱与权限** | 模板有文件修改、删除和 Shell 审批；更强隔离需自行接 sandbox/runtime | backend 抽象清晰，可接 sandbox；生产部署通常与 LangSmith 或自建基础设施组合 | **强项**：AgentScope Runtime 提供 Docker / 远端 sandbox，Java Harness 直接集成；支持 HITL | 命令审批、授权与容器隔离；多种本地/远端 terminal backend | **强项**：per-agent sandbox、tool allow/deny、容器作用域和 channel 权限可细配 |
| **记忆与长任务状态** | 多种 memory、storage、thread、working memory 与 workflow state | LangGraph checkpoint/persistence + 文件型长期上下文；上下文压缩和长任务处理是核心设计 | 短期/长期 memory、压缩、SQLite session；Java Harness 自动保存跨调用状态 | **强项**：`MEMORY.md` / `USER.md`、会话检索、外部 memory provider；能从经验生成和改进 skills | 文件型 memory、session history、per-agent 状态；适合长期个人助理 |
| **Skills / MCP / 协议** | Skills、MCP client/server、Tools；同一 TS 工程内易二次封装 | Skills、tools、subagents；MCP 通过 LangChain adapter 生态接入 | Anthropic Agent Skills、MCP、A2A；多 Agent 互操作覆盖较全 | agentskills.io-compatible skills，Agent 可管理 skills；MCP client | Agent Skills、MCP、丰富插件；skills 可按 Agent 与 workspace 隔离/白名单 |
| **Web / Browser / Computer use** | 官方 Harness 带搜索和网页抓取；Browser/Computer 工具可组合，具体供应商需配置 | 不是默认全家桶，通常把浏览器或 computer-use 作为自定义 tool/backend 接入 | 有 browser-use Agent 与 browser sandbox 镜像 | 内置 web、browser、vision、computer-use 等 toolsets | 浏览器、channels 和设备能力是核心使用场景之一，生态覆盖广 |
| **开发调试与可观测性** | **强项**：本地 Studio + logs/traces/evals/scorers；对 TS 开发者反馈链路短 | LangSmith tracing/eval/deploy 配套最成熟，但完整体验含托管产品 | Studio、本地 tracing、OTel、evaluation、finetuning，偏完整 Agent 生命周期 | CLI / Desktop / dashboard、trajectory export；更偏“运行和使用 Agent” | Gateway / Web UI / CLI、会话和运行日志；更偏运维一个常驻助手 |
| **API、前端与白标产品** | **强项**：Agent 可嵌入现有 Node 服务，官方 server/client 与 React/Next.js 路径直接 | 后端 Agent 服务能力强；前端、鉴权和业务 API 需按 LangGraph/LangSmith 体系或自行搭建 | code-first deploy，可本地、serverless、K8s；适合 Python/Java 企业后端 | 自带交互入口最快；要变成完全自定义的多租户 SaaS，需改造 gateway、身份和数据边界 | 自带多渠道产品壳最快；深度白标和业务数据模型不是其首要目标 |
| **部署与规模化** | Node 服务、自托管或 Mastra Cloud；storage 可替换，适合渐进式产品化 | LangGraph runtime + LangSmith Deployment 路径成熟，也可自托管 | **强项**：AgentScope Runtime、serverless/K8s、OTel；适合企业部署 | VPS、本机、容器与 serverless terminal backend，适合单用户/小团队常驻 Agent | 自托管 Gateway、多 channel、多 Agent 路由成熟；需认真做网络与工具权限加固 |
| **最明显代价** | TS-only 团队最顺；超大规模 sandbox 调度与消息渠道不是核心开箱项 | Python/LangGraph 概念和中间件层较多；Studio/部署的最佳体验与 LangSmith 绑定更深 | Python、Java、Runtime、Studio 等子项目较分散；版本线与文档面较宽 | 产品意见很强，Agent 会写 memory/skills，治理与审批策略必须先设计 | 系统面大、权限面宽；更像可运营的个人 Agent 平台，而非轻量 SDK |

上表所依据的关键官方入口包括：[Mastra Agent Harness 模板](https://mastra.ai/templates/agent-harness)、[Deep Agents Python](https://github.com/langchain-ai/deepagents) 与 [TypeScript 文档](https://docs.langchain.com/oss/javascript/deepagents/overview)、[AgentScope README](https://github.com/agentscope-ai/agentscope)、[AgentScope Java Harness 架构](https://java.agentscope.io/v2/en/docs/harness/architecture.html)、[AgentScope Runtime Sandbox](https://runtime.agentscope.io/en/sandbox/sandbox.html)、[Hermes Agent 文档](https://hermes-agent.nousresearch.com/docs/)、[OpenClaw Agent Runtime](https://docs.openclaw.ai/concepts/agent) 与 [OpenClaw Multi-Agent](https://docs.openclaw.ai/multi-agent)。

#### 结论：Mastra 不是全面胜出，而是这门教程场景下的最短路径

如果目标是用 **TypeScript / Next.js 快速做一个可嵌入、可白标、可继续产品化的 Harness Agent**，Mastra 的优势不是某一个 Agent 算法更强，而是：

1. **同一语言贯穿 Agent、工具、API 与 Web 前端。** 少一层 Python 服务边界，类型、流式事件和业务 SDK 更容易复用。
2. **默认 starter 已经是产品骨架。** workspace、Shell 审批、memory、任务追踪、web、schedule、storage、observability 不必从空白逐项拼装。
3. **Agent 与确定性 Workflow 共存。** 自由探索交给 Agent，支付、审批、发布等关键业务步骤放进可恢复、可观测的 Workflow。
4. **本地 Studio 缩短第一次验证。** 在写 UI 之前就能调 prompt、tools、memory 和 traces；验证后再嵌入现有 React / Next.js 产品。
5. **抽象层次更适合“做产品”。** 它比 Hermes/OpenClaw 更少预设最终交互形态，又比从裸 LangGraph/ReAct 循环开始补齐的工程项更少。

但以下情况不应强行选 Mastra：

- 团队以 **Python + LangGraph** 为主，长任务、checkpoint、可替换 filesystem backend 和 LangSmith 生产链路优先：选 **Deep Agents**。
- 团队以 **Java/Python** 为主，需要内建 sandbox runtime、K8s、OTel、A2A 或企业级部署：重点评估 **AgentScope（尤其 Java 2.0 `HarnessAgent`）**。
- 目标是今天就部署一个会从经验生成技能、跨消息平台工作的个人 Agent，而不是先开发 SaaS：选 **Hermes Agent**。
- 目标是自托管、多 channel、多 persona、常驻运行的“个人 Agent 操作系统”：选 **OpenClaw**，并优先完成 sandbox、tool allowlist 和网络边界加固。

同属 AgentScope 生态的 [CoPaw](https://github.com/agentscope-ai/CoPaw) 也值得关注：它更接近 Hermes Agent / OpenClaw 这一侧的个人 Agent 工作站，而不是 Mastra / Deep Agents 这一侧的嵌入式开发框架。若你的需求首先是多渠道个人助手，应把 CoPaw 纳入 PoC；若首先是自定义产品 API 与 UI，则仍从框架层比较更合适。

---

## 1.2 核心架构一览

```
┌─────────────────────────────────────────────┐
│                 Mastra 实例                   │
│                                              │
│  ┌─────────┐  ┌──────────┐  ┌────────────┐  │
│  │  Agent   │  │ Workflow  │  │   Tools    │  │
│  │ (智能体) │  │ (工作流)  │  │  (工具集)   │  │
│  └────┬─────┘  └────┬─────┘  └─────┬──────┘  │
│       │             │              │          │
│  ┌────┴─────────────┴──────────────┴──────┐  │
│  │            共享基础设施                    │  │
│  │  Memory · Storage · Vector · Logger    │  │
│  │  Observability · MCP · Scorers         │  │
│  │  Workspace · Browser · Channels …      │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

| 组件 | 作用 | 何时用 |
|------|------|--------|
| **Agent** | 接收目标、推理、调工具、迭代到给出答案 | 步骤事先不确定的开放任务 |
| **Workflow** | 显式控制每一步顺序与分支 | 流程固定、要精确编排时 |
| **Tool** | Agent 的「手」：调 API、查库、写文件等 | Agent 需要副作用或外部数据 |
| **Memory** | 对话历史、偏好、语义召回 | 多轮对话、个性化 |
| **MCP** | 与外部 Agent/工具互通的标准协议 | 接入或暴露工具生态 |
| **Storage** | 持久化记忆、工作流状态、观测数据 | 生产环境几乎必备 |
| **Scorers** | 评测输出质量 | 回归测试、持续改进 |
| **Studio** | 本地交互式调试 UI | 开发阶段默认入口 |

CLI 默认 starter（Agent Harness）还会带上 workspace 工具、任务追踪、网页访问、定时任务、存储与可观测性等能力；本章先把「创建 → 跑通 → 自己写一个 Agent」走通。

---

## 1.3 环境准备（动手前必做）

### 步骤 1：检查 Node.js 版本

Mastra **要求 Node.js ≥ 22.13.0**（开发与生产环境都一样）。版本不够会报 `EBADENGINE`。

```bash
node -v
# 期望类似：v22.13.0 或更高（如 v22.18.0、v24.x）
```

若版本偏低，用 [nvm](https://github.com/nvm-sh/nvm) 或 [fnm](https://github.com/Schniz/fnm) 升级，例如：

```bash
# nvm 示例
nvm install 22
nvm use 22
node -v
```

### 步骤 2：确认包管理器

以下任选其一即可：`npm` / `pnpm` / `yarn` / `bun`。本章以 **npm** 为主，并在关键处给出其它写法。

```bash
npm -v
```

### 步骤 3：准备至少一个模型 API Key

默认 starter 交互安装时可选：**OpenAI / Anthropic / Google Gemini / xAI**。也可以先跳过，之后写入 `.env`。

| 提供商 | 环境变量名 | 模型字符串示例 | 申请入口（自行打开） |
|--------|------------|----------------|----------------------|
| OpenAI | `OPENAI_API_KEY` | `openai/gpt-5.5`、`openai/gpt-5-mini` | platform.openai.com |
| Anthropic | `ANTHROPIC_API_KEY` | `anthropic/claude-sonnet-4-6` | console.anthropic.com |
| Google | `GOOGLE_API_KEY` | `google/gemini-2.5-flash` | aistudio.google.com |
| xAI | `XAI_API_KEY` | `xai/grok-4.3` | console.x.ai |

完整目录与更多提供商见 [Model Providers](https://mastra.ai/models)。

> **注意：** Google 使用 `GOOGLE_API_KEY`（不是旧文档里偶见的 `GOOGLE_GENERATIVE_AI_API_KEY`）。模型字符串格式是 `provider/model`，中间是 **斜杠 `/`**，不要写成 `openai:gpt-xxx`。

### 步骤 4（可选）：关闭 CLI 遥测

Mastra CLI 会收集匿名用量（OS、Mastra/Node 版本等）。若不想上报：

```bash
export MASTRA_TELEMETRY_DISABLED=1
```

可写入 shell 配置文件（`~/.zshrc` / `~/.bashrc`），或在单次命令前临时加：

```bash
MASTRA_TELEMETRY_DISABLED=1 npm create mastra@latest
```

---

## 1.4 创建第一个项目（推荐：CLI Quickstart）

官方推荐入口就是 `create-mastra` CLI。默认会生成带 Agent Harness 的 starter：含 workspace 工具、memory、任务追踪、web 访问、schedules、storage、observability 等。

下面提供三种**互相替代**的创建方式，任选一种即可，不需要依次执行：

1. **交互式创建（推荐新手）**：CLI 逐项询问项目名、模型提供商和 API Key。
2. **非交互式创建（推荐脚本与自动化）**：把选项直接写在命令中。
3. **让 AI 编码助手代为创建**：把预构建 Prompt 交给 Claude Code、Codex 或 Cursor。

### 方式一：交互式创建（本章采用）

先进入准备存放项目的父目录，再启动 CLI：

```bash
# 本章实际使用的父目录
cd /Users/fly/code/mastra-demo

# npm（推荐写法）
npm create mastra@latest

# 等价写法
npx create-mastra@latest

# 其它包管理器
pnpm create mastra
# 或：pnpm dlx create-mastra@latest
yarn create mastra
bunx create-mastra
```

然后按提示回答问题。本章的实际选择是：

| CLI 问题 | 本章选择 | 说明 |
|---|---|---|
| `What do you want to name your project?` | `demo-flywiki` | 最终目录为 `/Users/fly/code/mastra-demo/demo-flywiki` |
| `Select a default model provider` | `OpenAI` | starter 会安装 `@ai-sdk/openai` |
| `Enter your OpenAI API key?` | `Skip for now` | 稍后复制 `.env.example` 再填写 |
| `Enable Mastra platform observability?` | `No` | 本地仍使用 DuckDB observability，不连接 Mastra Platform |

![image-20260725124800784](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260725124800784.png)

CLI 随后会：

1. 克隆默认的 **Agent Harness** 模板。
2. 安装项目依赖。
3. 检测本机编码助手并安装 Mastra skills（本次检测到 Claude Code 和 Codex；可用 `--no-skills` 跳过）。
4. 在合适时初始化 Git；本项目位于已有 worktree 内，因此 CLI 正确地跳过了这一步。

### 方式二：使用参数非交互式创建（脚本 / CI）

需要在脚本、CI 或批量初始化场景中复现创建过程时，把项目名和模型提供商直接写进命令，避免交互问答：

```bash
# OpenAI
npm create mastra@latest my-mastra-app -- --llm openai

# Anthropic / Google / xAI
npm create mastra@latest my-mastra-app -- --llm anthropic
npm create mastra@latest my-mastra-app -- --llm google
npm create mastra@latest my-mastra-app -- --llm xai

# 创建时直接写入 API Key
npm create mastra@latest my-mastra-app -- --llm openai --llm-api-key sk-xxxx

# 最小空项目：无 Agent、无示例、无模型 SDK、无 .env
npm create mastra@latest my-empty-project -- --empty

# 从模板创建
npm create mastra@latest my-mastra-app -- --template agent-harness
# 或 GitHub URL
npm create mastra@latest my-mastra-app -- --template https://github.com/mastra-ai/template-agent-harness

# 跳过 skills / git
npm create mastra@latest my-mastra-app -- --llm openai --no-skills --no-git
```

> `--empty` 与 `--template` 不能同时用。`--llm` / `--llm-api-key` 仅适用于默认 starter，不适用于 template 模式。

完整参数见 [create-mastra 参考](https://mastra.ai/reference/cli/create-mastra)。

### 方式三：让 AI 编码助手代为创建

如果希望 Cursor、Claude Code 或 Codex 帮你确认选项、执行命令并启动服务，可以直接复制下面这段 Prompt。它本质上仍会调用“方式二”的非交互命令：

```text
Create a new Mastra project. Mastra is a framework for AI applications and agents on a modern TypeScript stack. Before running the command, ask these questions one at a time and wait for each answer unless it was already provided:

Project name? (default: "my-mastra-app")
Provider? (required; options: "openai", "anthropic", "google", "xai")
If the provider isn't supported, ask again and list the supported values.

Run: npm create mastra@latest <project-name> -- --llm <provider>

The command creates a default Mastra project, installs Mastra skills for detected coding assistants, and initializes Git when appropriate.

After creation, enter the project directory and start the dev server: npx bgproc start -n <project-name> -w -- npm run dev

Open Mastra Studio at http://localhost:4111. Studio is the interface for building, testing, and managing agents, workflows, and tools.

Also mention that the Mastra model router provides access to thousands of models: https://mastra.ai/models
```

这里的 `create-mastra` 是最快的起步方式；如果你想完全控制依赖与目录结构，跳到后面的手动安装；如果是给现有项目补 Mastra，使用 `mastra init`。

### 创建完成后：进入项目并检查实际结构

```bash
cd /Users/fly/code/mastra-demo/demo-flywiki

# 查看主要源码和配置；排除依赖及构建缓存
find . -maxdepth 4 \
  -not -path './node_modules*' \
  -not -path './.mastra*' \
  -not -path './src/mastra/public/*.db*' \
  | sort
```

本章使用 `create-mastra` 生成的 Agent Harness 项目结构如下：

```text
demo-flywiki/
├── .agents/
│   └── skills/
│       └── mastra/                  # 安装给 Codex 等助手的 Mastra skill
├── .claude/
│   └── skills/
│       └── mastra/                  # 安装给 Claude Code 的 Mastra skill
├── src/
│   └── mastra/
│       ├── agents/
│       │   └── agent.ts             # Harness Agent 定义
│       ├── tools/
│       │   ├── schedule-tools.ts    # 定时任务工具
│       │   └── web-fetch-tool.ts    # 网页抓取工具
│       ├── public/
│       │   ├── mastra.db            # 本地 libSQL 数据（运行后生成）
│       │   └── mastra.duckdb        # 本地观测数据（运行后生成）
│       └── index.ts                 # Mastra 实例、存储与观测配置
├── .env.example                     # 环境变量模板
├── AGENTS.md                        # 给编码 Agent 的项目说明
├── README.md                        # starter 使用与安全说明
├── package.json
├── package-lock.json
├── skills-lock.json                 # 已安装 skills 的锁文件
└── tsconfig.json
```

启动 `npm run dev` 后还会出现 `.mastra/`。它是 Mastra CLI 的本地构建输出与开发服务器状态目录，不是业务源码，通常无需手动修改。

这个结构与最小示例项目不同：Agent Harness starter 不会预先生成 weather workflow、scorer 等教学示例，而是直接提供一个能使用 workspace、Shell、memory、web fetch 和 schedule 的通用 Agent。

| 实际路径 | 作用 |
|---|---|
| `src/mastra/agents/agent.ts` | 修改模型、instructions、memory、workspace 和审批策略 |
| `src/mastra/tools/` | 定义 Harness Agent 可调用的业务工具 |
| `src/mastra/index.ts` | 注册 Agent，并配置 storage、logger 与 observability |
| `src/mastra/public/` | 开发运行时的本地数据库和静态资源 |
| `.agents/skills/mastra/`、`.claude/skills/mastra/` | 帮助编码助手使用当前版本的 Mastra API |
| `workspace/` | Agent 的文件工作区；首次使用相关工具时创建 |
| `.mastra/` | CLI 自动生成的构建产物和开发态文件 |

还可以核对 `package.json` 中的三个生命周期命令：

```json
{
  "scripts": {
    "dev": "mastra dev",
    "build": "mastra build",
    "start": "mastra start"
  }
}
```

Mastra **不强绑目录结构**；CLI 给出的是当前模板的可维护默认值。后续可以增加 `workflows/`、`scorers/`、`mcp/` 等目录，但无需为了匹配某个示意结构提前创建空目录。

### 创建完成后：配置模型 API Key

由于本章在 CLI 中选择了 `Skip for now`，启动 Agent 对话前需要创建 `.env`：

```bash
cd /Users/fly/code/mastra-demo/demo-flywiki
cp .env.example .env

# 用编辑器打开 .env，然后填写：
# OPENAI_API_KEY=sk-...
```

不要把 `.env` 或真实 API Key 提交到 Git。

<!-- 以下为其它提供商的环境变量对照；切换 provider 时使用。 -->

```bash
# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Google Gemini
GOOGLE_API_KEY=...

# xAI
XAI_API_KEY=...
```

### 使用 OpenAI-compatible API 地址和模型

如果使用的是 LM Studio、vLLM、LiteLLM，或云厂商提供的 **OpenAI-compatible API**，不必把请求固定发往 OpenAI 官方地址。Mastra 的 `model` 可以写成对象，在其中指定模型路由 ID、API Base URL 和密钥。

先在 `.env` 中保存配置：

```bash
# 必须是 API 的基础地址，通常以 /v1 结尾
OPENAI_COMPATIBLE_BASE_URL=https://api.example.com/v1
OPENAI_COMPATIBLE_API_KEY=你的密钥

# 服务端 /v1/models 返回的模型 id
OPENAI_COMPATIBLE_MODEL_ID=qwen-plus
```

然后修改 Agent，例如 `src/mastra/agents/agent.ts`：

```typescript
import { Agent } from '@mastra/core/agent'

export const agent = new Agent({
  id: 'openai-compatible-agent',
  name: 'OpenAI-compatible Agent',
  instructions: '你是一个有帮助的中文助手。',
  model: {
    // 前缀用于 Mastra 路由；真正发给直连服务的是后面的模型名
    id: `custom/${process.env.OPENAI_COMPATIBLE_MODEL_ID}`,
    url: process.env.OPENAI_COMPATIBLE_BASE_URL!,
    apiKey: process.env.OPENAI_COMPATIBLE_API_KEY!,
  },
})
```

> `url` 应填写 OpenAI-compatible **基础地址**（如 `https://api.example.com/v1`），不要填写具体的 `/chat/completions` 地址。修改 `.env` 后需重启 `npm run dev`。

#### 查询服务端支持的模型列表

OpenAI-compatible 服务通常通过 `GET /v1/models` 返回模型列表。若 Base URL 已包含 `/v1`，可以这样查询：

```bash
curl -sS \
  -H "Authorization: Bearer $OPENAI_COMPATIBLE_API_KEY" \
  "$OPENAI_COMPATIBLE_BASE_URL/models"
```

安装了 `jq` 时，只显示模型 ID：

```bash
curl -sS \
  -H "Authorization: Bearer $OPENAI_COMPATIBLE_API_KEY" \
  "$OPENAI_COMPATIBLE_BASE_URL/models" |
  jq -r '.data[].id'
```

把返回的某个 `id`（例如 `qwen-plus`）写入 `OPENAI_COMPATIBLE_MODEL_ID`。如果服务不需要鉴权，可去掉 `Authorization` 请求头；如果厂商没有实现 `/models`，则以其控制台或文档公布的模型 ID 为准。

#### `model.id` 应该怎样写

- **直连模型服务**：使用 `provider/model`，例如 `custom/qwen-plus`、`lmstudio/qwen/qwen3-30b-a3b-2507`。远端通常收到斜杠后面的裸模型名。
- **模型网关**：若远端要求保留上游提供商命名空间，则使用 `gateway/provider/model`，例如 `openrouter/google/gemini-2.5-flash`。

本地 LM Studio 的完整示例：

```typescript
model: {
  id: 'lmstudio/qwen/qwen3-30b-a3b-2507',
  url: 'http://localhost:1234/v1',
}
```

LM Studio 等无需鉴权的本地服务可以省略 `apiKey`。可用下面的命令查看其模型列表：

```bash
curl -sS http://localhost:1234/v1/models | jq -r '.data[].id'
```

Mastra 自带的模型目录则用于查询已收录的提供商及模型字符串：打开 [Model Providers](https://mastra.ai/models)，或在 Agent 的 `model` 字段中使用 IDE 自动补全。开发环境默认每小时刷新一次本地模型目录；如需关闭，可设置 `MASTRA_AUTO_REFRESH_PROVIDERS=false`。这里的目录与自建服务的 `/v1/models` 含义不同：前者是 Mastra 支持的提供商目录，后者是你的兼容端点实际开放的模型列表。

---

## 1.5 启动 Studio 并完成第一次对话

### 步骤 1：启动开发服务器

```bash
cd /Users/fly/code/mastra-demo/demo-flywiki
npm run dev

# 如果已安装 bgproc，也可以让开发服务在后台持续运行：
# npx bgproc start -n demo-flywiki -w -- npm run dev
```

本章实测 Mastra `1.20.1` 的成功输出为：

```text
mastra  1.20.1 ready in 3078 ms

│ Studio: http://localhost:4111
│ API:    http://localhost:4111/api

◯ watching for file changes...
```

看到 `ready`、Studio 地址和 API 地址，说明项目已经完成打包并进入文件监听状态：

- Studio：[http://localhost:4111](http://localhost:4111)
- API：[http://localhost:4111/api](http://localhost:4111/api)

`npm run dev` 会持续占用当前终端，这是正常现象；开发期间不要关闭它。需要停止服务时按 `Ctrl+C`。

### 步骤 2：打开 Studio

浏览器访问：

- Studio UI：http://localhost:4111  
- REST API 浏览：http://localhost:4111/swagger-ui  

Studio 是构建、测试和管理 Agents、Workflows 与 Tools 的本地界面。模型选择时也可以查看 [Mastra model router](https://mastra.ai/models)，它通过统一的 `provider/model` 字符串访问大量模型。

### 步骤 3：在 Studio 里测示例 Agent

按下面顺序操作（UI 文案可能随版本微调）：

1. 左侧进入 **Agents**
2. 选择本项目注册的 **Agent**
3. 输入一个能触发 Harness 能力的任务，例如：
   - `查询上海本周末的天气。`
   - `在 workspace 中创建一个日本樱花节落地页。`
   - `查询 SPCX 当前股价，然后每分钟检查一次。`
4. 发送后观察：
   - 流式回复文本
   - 是否出现 `web_search`、`web_fetch`、workspace 或 `start_schedule` 等 tool call
   - 涉及写文件、删除或执行命令时是否出现审批
   - Observability / traces 里是否有本次调用链路

若报错「缺少 API Key」，回到 `.env` 补全对应变量，**保存后重启** `npm run dev`（多数环境变量变更需要重启进程）。

### 步骤 4：熟悉 Studio 主要面板

| 面板 | 你可以做什么 |
|------|----------------|
| **Agents** | 聊天、切换模型、调 temperature / top-p；看推理步骤与 tool 输出 |
| **Workflows** | 看工作流图，带自定义输入逐步跑，观察当前步骤 |
| **Tools** | 脱离 Agent，单独测某个 Tool 的输入输出 |
| **MCP** | 查看已挂载的 MCP server 及其工具列表 |
| **Processors** | 检查 Agent 上的输入/输出处理器与护栏 |
| **Workspaces** | 浏览 Agent workspace 文件、Skills |
| **Scorers / Datasets / Experiments** | 跑评估、管理测试集、对比实验 |
| **Observability** | traces、耗时、错误定位 |
| **Settings** | 改 Mastra instance URL、API prefix、自定义 Header、主题 |

> **建议：** 前几章尽量在 Studio 里验证，再写业务前端。这是 Mastra 相对其它框架最省时间的一点。

### 步骤 5（可选）：改端口 / 开 HTTPS

默认端口 `4111`。可在 `src/mastra/index.ts` 的 `server` 配置里改 `host` / `port`（见 [Configuration](https://mastra.ai/reference/configuration)）。本地 HTTPS：

```bash
npx mastra dev --https
```

---

## 1.6 方式二：手动从零安装（对照官方 Manual Install）

不需要 CLI 脚手架时，按官方 [Manual Install](https://mastra.ai/docs/getting-started/manual-install) 做。适合想完全掌控依赖、或排查 CLI 问题。

### 步骤 1：建项目与 package.json

```bash
mkdir my-first-agent && cd my-first-agent
npm init -y
```

编辑 `package.json`，确保是 ESM，并加上脚本：

```json
{
  "name": "my-first-agent",
  "type": "module",
  "scripts": {
    "dev": "mastra dev",
    "build": "mastra build"
  }
}
```

### 步骤 2：安装依赖

```bash
npm install -D typescript @types/node mastra@latest
npm install @mastra/core@latest zod@^4
```

其它包管理器：

```bash
pnpm add -D typescript @types/node mastra@latest
pnpm add @mastra/core@latest zod@^4

yarn add --dev typescript @types/node mastra@latest
yarn add @mastra/core@latest zod@^4

bun add --dev typescript @types/node mastra@latest
bun add @mastra/core@latest zod@^4
```

> 不要为了「模型路由」去装一堆 AI SDK provider 包；用 `provider/model` 字符串即可。除非官方某页明确要求，否则不必装 `@ai-sdk/*`。

### 步骤 3：写 `tsconfig.json`

```bash
touch tsconfig.json
```

内容：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "noEmit": true,
    "allowImportingTsExtensions": true,
    "outDir": "dist"
  },
  "include": ["src/**/*"]
}
```

> Mastra 需要现代 `module` / `moduleResolution`。用 `CommonJS` 或 `"moduleResolution": "node"` 容易解析失败。

### 步骤 4：写 `.env`

```bash
touch .env
```

按提供商填写，例如 Google：

```bash
GOOGLE_API_KEY=你的密钥
```

或 OpenAI：`OPENAI_API_KEY=...`，Anthropic：`ANTHROPIC_API_KEY=...`。

### 步骤 5：创建 Tool（必须用 `createTool`）

```bash
mkdir -p src/mastra/tools
```

创建 `src/mastra/tools/weather-tool.ts`：

```typescript
import { createTool } from '@mastra/core/tools'
import { z } from 'zod'

export const weatherTool = createTool({
  id: 'get-weather',
  description: 'Get current weather for a location',
  inputSchema: z.object({
    location: z.string().describe('City name'),
  }),
  outputSchema: z.object({
    location: z.string(),
    temperatureCelsius: z.number(),
    conditions: z.string(),
  }),
  execute: async ({ location }) => {
    // 入门示例：先返回假数据；真实 API 可在第三章再接
    return {
      location,
      temperatureCelsius: 21,
      conditions: 'sunny',
    }
  },
})
```

> **踩坑：** 用普通对象当 tool 会「静默不执行」。工具必须经 `createTool()`，并提供 `id`、`description`、`inputSchema`（zod）、`execute`。

### 步骤 6：创建 Agent

```bash
mkdir -p src/mastra/agents
```

创建 `src/mastra/agents/weather-agent.ts`：

```typescript
import { Agent } from '@mastra/core/agent'
import { weatherTool } from '../tools/weather-tool.ts'

export const weatherAgent = new Agent({
  id: 'weather-agent',
  name: 'Weather Agent',
  instructions: `
You are a helpful weather assistant that provides accurate weather information.

When responding:
- Always ask for a location if none is provided
- If the location name isn't in English, translate it
- Keep responses concise but informative
- Use the weatherTool to fetch current weather data
`,
  // 与 .env 中的提供商一致；也可用 openai/gpt-5.5、anthropic/claude-sonnet-4-6
  model: 'google/gemini-2.5-flash',
  tools: { weatherTool },
})
```

### 步骤 7：注册到 Mastra 入口

创建 `src/mastra/index.ts`：

```typescript
import { Mastra } from '@mastra/core'
import { weatherAgent } from './agents/weather-agent.ts'

export const mastra = new Mastra({
  agents: { weatherAgent },
})
```

### 步骤 8：启动并验证

```bash
npm run dev
```

打开 http://localhost:4111 ，选中 Weather Agent，发送 `Weather in SF`。

也可用脚本直接调用（不启 Studio）。Node.js **22.18.0+** 可直接跑 TypeScript；本地 import 记得带扩展名：

```typescript
// run.mjs（或 .ts，视 Node 版本而定）
import { mastra } from './src/mastra/index.ts'

const agent = mastra.getAgentById('weather-agent')
const response = await agent.generate('Weather in SF')
console.log(response.text)
```

```bash
node run.mjs
```

类型检查（可选）：

```bash
npx tsc --noEmit
```

---

## 1.7 方式三：在已有项目中接入（`mastra init`）

已有 Next.js / Express 等仓库，不想另起目录时：

```bash
# 先安装 CLI 到当前项目（若尚未安装）
npm install -D mastra@latest
npm install @mastra/core@latest

# 在现有项目根目录初始化
npx mastra init
```

常用参数（见 [mastra CLI](https://mastra.ai/reference/cli/mastra)）：

```bash
# 一键默认：写到 src/，OpenAI，带示例
npx mastra init --default

# 指定目录与组件
npx mastra init --dir src --components agents,tools,workflows,scorers

# 指定提供商与 Key
npx mastra init --llm anthropic --llm-api-key sk-ant-...

# 不要示例代码
npx mastra init --default --no-example

# 给编辑器配 Mastra MCP：cursor | cursor-global | windsurf | vscode
npx mastra init --mcp cursor
```

框架级集成指南（创建或嵌入）：

- [Next.js](https://mastra.ai/guides/getting-started/next-js)
- [React (Vite)](https://mastra.ai/guides/getting-started/vite-react)
- [Astro](https://mastra.ai/guides/getting-started/astro)
- [SvelteKit](https://mastra.ai/guides/getting-started/sveltekit)
- [Nuxt](https://mastra.ai/guides/getting-started/nuxt)
- [Express](https://mastra.ai/guides/getting-started/express)
- [NestJS](https://mastra.ai/guides/getting-started/nestjs)
- [Hono](https://mastra.ai/guides/getting-started/hono)
- [Electron](https://mastra.ai/guides/getting-started/electron)

---

## 1.8 Hello World：自己加一个最简 Agent

在已有 CLI 项目或手动项目上，再加一个最小 Agent，巩固注册与调用路径。

### 步骤 1：新建 Agent 文件

创建 `src/mastra/agents/hello-agent.ts`：

```typescript
import { Agent } from '@mastra/core/agent'

export const helloAgent = new Agent({
  id: 'hello-agent',
  name: 'Hello Agent',
  instructions: '你是一个友好的中文助手，用简洁幽默的方式回答问题。',
  // 换成你已配置 Key 的提供商模型
  model: 'openai/gpt-5-mini',
})
```

### 步骤 2：注册到 `src/mastra/index.ts`

在现有 `Mastra` 配置里**追加**，不要删掉示例 Agent（除非你有意清理）：

```typescript
import { Mastra } from '@mastra/core'
import { helloAgent } from './agents/hello-agent'
// 若项目里已有 weatherAgent 等，保持原有 import
// import { weatherAgent } from './agents/weather-agent'

export const mastra = new Mastra({
  agents: {
    helloAgent,
    // weatherAgent,
  },
})
```

保存后，若 `npm run dev` 已在跑，Studio 一般会热更新；没有出现新 Agent 就重启一次 `dev`。

### 步骤 3：在 Studio 验证

1. 打开 http://localhost:4111  
2. Agents → 选择 **Hello Agent**  
3. 发送：`用一句话解释什么是 TypeScript`  
4. 确认有中文、风格符合 instructions

### 步骤 4：在代码里调用（推荐 `getAgentById`）

官方推荐通过 Mastra 实例按 **id** 取 Agent，这样能挂上实例级 storage、logging、observability 等共享能力；直接 `import` Agent 也能跑，但拿不到这些共享服务。

```typescript
import { mastra } from './src/mastra/index.ts'

// id 对应 Agent 构造函数里的 id 字段：'hello-agent'
const agent = mastra.getAgentById('hello-agent')

// 一次性拿完整回复
const response = await agent.generate('用一句话解释什么是 TypeScript')
console.log(response.text)
// response 还可能包含 toolCalls、toolResults、steps、usage 等

// 流式输出
const stream = await agent.stream('给我讲个程序员笑话')
for await (const chunk of stream.textStream) {
  process.stdout.write(chunk)
}
```

### 关键字段速查

| 字段 | 含义 |
|------|------|
| `id` | 稳定唯一标识；`getAgentById`、日志、API 路由都靠它 |
| `name` | Studio / UI 显示名 |
| `instructions` | 系统级行为与人格（系统提示词） |
| `model` | `provider/model-name`；Mastra 按提供商读对应环境变量 |
| `tools` | 可选；对象形式挂载 `createTool` 产物 |

Agent 适合「步骤事先不知道」的开放任务；流程固定时用 [Workflow](https://mastra.ai/docs/workflows/overview)（第四章）。

---

## 1.9 常见问题排查

| 现象 | 可能原因 | 处理 |
|------|----------|------|
| `EBADENGINE` / 安装警告 | Node &lt; 22.13.0 | 升级 Node 后重装依赖 |
| Studio 能开但 Agent 报鉴权错误 | `.env` 未配或变量名不对 | 对照提供商表；Google 用 `GOOGLE_API_KEY`；改完重启 `dev` |
| Tool「从来不跑」 | 没用 `createTool` | 按 1.6 节重写 Tool |
| `getAgent` / `getAgentById` 找不到 | 传了注册键名却用了 id，或反过来 | `getAgentById('hello-agent')` 用构造函数里的 `id` |
| TS 解析报错 | `moduleResolution` 过旧 | 按 1.6 的 `tsconfig` 改 |
| 模型字符串无效 | 用了 `:` 或错误 id | 用 `openai/gpt-5.5` 这种 `/` 形式；在 [Models](https://mastra.ai/models) 核对 |
| Studio 中消息偶尔没有回复 | Observational Memory 缺少 `threadId`、上下文过重，或旧 thread 留有中断状态 | 先关闭 `observationalMemory`，重启服务并新建 Chat；再查看 Logs / Traces |
| 依赖安装超时 | 网络慢 | `npx create-mastra@latest my-app -- --llm openai --timeout 120000` |

### Studio 消息无响应：关闭 Observational Memory

Agent Harness 模板可能启用了 Observational Memory：

```typescript
memory: new Memory({
  options: {
    generateTitle: true,
    observationalMemory: {
      model: compatibleModel,
    },
  },
}),
```

Observational Memory 是额外的输入处理器。它会基于 thread 维护观察记录，也会增加额外的模型调用和上下文。当 Studio 新建、切换或恢复会话时没有正确传入 `threadId`，服务端可能报：

```text
ObservationalMemory (scope: 'thread') requires a threadId
```

此时请求会返回 500；Studio 有时只留下用户消息，看起来像「发送后一直没有响应」。

排查时可以先删除 `observationalMemory` 配置，保留普通 Memory：

```typescript
import { Memory } from '@mastra/memory'

export const agent = new Agent({
  // ...id、name、instructions、model 等配置
  memory: new Memory({
    options: {
      generateTitle: true,
      // 不配置 observationalMemory，即为关闭
    },
  }),
})
```

也可以进一步简化为：

```typescript
memory: new Memory(),
```

如果暂时完全不需要跨轮对话记忆，则从 `new Agent({...})` 中删除整个 `memory` 字段，并移除未使用的 `Memory` import。

修改后执行：

```bash
# 先停止正在运行的开发服务：Ctrl+C
npm run dev
```

然后在 Studio 中点击 **New Chat**，不要继续复用已经失败或中断的旧 thread。依次测试：

1. `只回复 OK`：确认基础文本流稳定。
2. 连续发送两条有关联的问题：确认普通 Memory 正常。
3. 再测试工具调用，并在 **Logs / Traces** 中检查是否有 500、超时或未完成的 tool call。

如果关闭后恢复稳定，说明问题位于 Observational Memory 或 thread 上下文，而不是基础模型 API。稳定运行后仍可重新启用它，但必须保证所有代码调用都传入有效的 memory thread 和 resource：

```typescript
const response = await agent.generate('你好', {
  memory: {
    thread: 'thread-123',
    resource: 'user-123',
  },
})
```

> 对 OpenAI-compatible 网关做排查时，应分别测试基础 `/chat/completions` 流式请求和完整 Agent 请求。基础请求正常不代表 Memory processor、工具调用和 Studio thread 订阅一定正常。若简单消息也携带上万 token，可暂时减少 workspace tools、降低 `maxSteps`（例如从 `100` 调到 `10`），再逐项恢复能力。

本地用 CLI 快速打 API（需先 `npm run dev`）：

```bash
npx mastra api agent list
npx mastra api agent run hello-agent '{"messages":"你好"}'
```

---

## 1.10 本章小结

这一章你应该已经完成：

1. 确认 Node ≥ 22.13.0，并准备好模型 API Key  
2. 用 `npm create mastra@latest`（或手动安装 / `mastra init`）建好项目  
3. `npm run dev` 打开 Studio（`http://localhost:4111`）并完成首轮对话  
4. 理解 `src/mastra/index.ts` 注册模型，以及 Agent 三要素：`id` + `instructions` + `model`  
5. 用 `mastra.getAgentById()` + `generate` / `stream` 在代码里调用 Agent  

**官方延伸阅读：**

- [Quickstart](https://mastra.ai/guides/getting-started/quickstart)  
- [Build with AI](https://mastra.ai/docs/getting-started/build-with-ai)  
- [Mastra YouTube](https://www.youtube.com/@mastra-ai) / [Agent 课程视频](https://www.youtube.com/watch?v=lCmf_qrGfGA)  
- [GitHub mastra-ai/mastra](https://github.com/mastra-ai/mastra)

下一章深入 Agent：工具调用、结构化输出、多模态与更多运行时行为。
