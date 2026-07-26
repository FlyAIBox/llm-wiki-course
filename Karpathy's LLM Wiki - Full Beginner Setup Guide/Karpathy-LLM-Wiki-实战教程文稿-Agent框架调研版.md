# LLM Wiki实战：让AI拆解 Agent Harness 的核心组成，对比各家框架

> 文中的演示以"投喂几篇 Agent Harness（智能体框架）文章，让 AI 拆解 harness 的核心组成、对比各家开源框架、并交叉校验各来源说法是否矛盾"为例，你可以把来源换成任何主题的资料。
>
> **👉 想要 LLM Wiki实战 素材**
>
> **🚀 获取通道：**
>
> **1️⃣ 微信搜索关注公众号 👉 萤火AI百宝箱**
>
> **2️⃣ 后台私信回复暗号 👉 Agent**

---

## 先看效果：和你现在用 AI 读资料的方式，差在哪

先说结论。把同一批资料丢给 ChatGPT / NotebookLM 这类工具（RAG 方式），和丢进 LLM Wiki，差别不在"这一次答得好不好"，而在**第二次、第十次提问之后**：

| | 传统 RAG（上传即问） | LLM Wiki（先建库，再问） |
|---|---|---|
| 资料怎么存 | 原文原样放着，每次提问现查 | 被拆解、互链成一组知识页面，建一次长期复用 |
| 提问时发生什么 | 从零检索、临时拼相关片段 | 直接命中已建好的主题页，跨来源的对照早已做好 |
| 多来源说法冲突 | 可能悄悄采信一种，你看不见 | 摄取时就显式标注矛盾，每条可追溯到原文 |
| 用得越久 | 每次都从头再来，什么都不积累 | 每加一份料，库更完整，新料还反过来校正旧记载 |

一句话：**RAG 把"读和综合"放在你每次提问时重做；LLM Wiki 把它挪到加资料时做一次，之后一直吃现成的。**

本文用一个真实案例走完全程——投喂 Agent Harness（智能体框架）调研资料，分两个阶段、用同一句摄取指令完成：

| 阶段 | 投喂的来源 | 得到的效果 |
|---|---|---|
| **阶段一：观点文章**（第 5-8 步） | 5 篇 2026 年的 Agent Harness 文章（博客/官博/X 长文） | **30 个互链页面**：1 张五方组件清单对比表、1 张 8 框架工具版图、12 个组件页，外加 AI 自动揪出的 **3 处来源矛盾** |
| **阶段二：工程资料**（第 9 步） | 85 个文件、31 MB：14 个开源框架的 README + 官方文档 + 7 篇论文 PDF | AI 自动盘点全部文件、给出约 20 个框架实体页 + 4 个研究页的**摄取计划**，供你确认范围后执行 |

**这套库能帮你做什么**，建好之后可以直接让 AI：

- **拆解一个概念的核心组成**：Agent Harness 由哪些组件构成、各家清单有何异同——AI 自动生成一张五方组件对比表，并指出"可观测性只有两家提、评估只有一家提"这类差异
- **对比各家开源框架**：Deep Agents、AgentScope、Mastra 等的定位与取舍，汇成一张工具版图页
- **交叉校验多来源**：不同作者说法冲突时显式标记矛盾（实操揪出 3 处，含一处"同一篇文章正文与图注打架"）

**而这些效果是怎么做出来的？** 不靠魔法，全靠 CLAUDE.md 里几条规则在背后起作用——**效果和机制一一对应**：

| 你看到的效果 | 背后是哪条机制 |
|---|---|
| 5 篇文章没各建一套页，而是编织进同一套概念页 | 摄取流程要求"更新已有页面、为新概念建页"——按主题归置，不按来源分档 |
| 各家分歧被一条条揪出、不丢 | 引用规则：每条论断标来源文件，两源冲突就显式记录矛盾 |
| 提问直接命中主题页、不再翻原文 | 问答规则先读 `index.md` 定位 + 页面早已互链——检索和综合在建库时就一次做完 |
| 全程你始终能把关 | 摄取第 2 步：AI 动笔前先讨论要点、你确认范围才落笔 |

看懂这张表，就看懂了整套系统：**你在实操里读到的每一个"AI 居然会这样做"，都能在 CLAUDE.md 里找到对应的那行规则。** 后面第 3 步讲规则、第 5-11 步逐一验证效果，就是在把这张表展开。

而你要做的事情非常少。整个流程中**人只负责三件事**：把资料丢进 `raw`、对 AI 说一句固定话术、在它给出建页计划时点头或调整范围。写作、组织、链接、矛盾标记全部由 AI 完成。下面就从为什么需要这种方式开始，再一步步搭建出来。

## 一、为什么要做 LLM Wiki：现有方式的问题

### 1. RAG 的瓶颈

目前大多数人用 AI 处理文档的方式（如 ChatGPT、NotebookLM 上传文件问答）叫 **RAG（Retrieval Augmented Generation，检索增强生成）**，流程是：

1. 上传文件
2. 提问
3. AI 在文件中检索，抓取"看起来相关"的片段
4. 基于片段生成答案

问题在于：**每次提问都是从零开始**。今天问一个问题，明天问一个类似的问题，AI 会把全部检索、拼接工作重做一遍。没有任何东西被保存、被积累，知识无法复利（nothing compounds）。

打个比方：你是一个研究者，几周来读了大量论文。用 RAG 的话，每次向 AI 提问，它都像从来没读过这些论文一样，每次都"第一次见"。

### 2. Karpathy 的解法：LLM Wiki

Andrej Karpathy（OpenAI 联合创始人、前 Tesla AI 总监）提出的思路把这个过程反过来了：

**不是每次提问时去搜原始文档，而是让 AI 把文档读一遍，构建出一个结构化的 Wiki**——一个由互相链接的 Markdown 文件组成的、持久存在的知识库。

每当你加入一个新来源（PDF、文章等），AI 不是简单存起来，而是：

- 真正读完它
- 提取核心观点
- **整合**进 Wiki：更新已有页面、为新概念创建新页面
- 把相关概念互相链接
- 如果新来源与 Wiki 里已有内容**矛盾**，会标记出来

随着时间推移，Wiki 越长越丰富。连接已经建好，综合已经完成。你提问时，AI 不是从零开始，而是在一个**预先构建好、已经组织好的知识库**上工作。

Karpathy 的类比：

> 把 Obsidian 当作 IDE，LLM 当作程序员，Wiki 当作代码库。你几乎不亲手写 Wiki——AI 负责写作和组织，你只负责决定"喂什么进去"和"问什么问题"。

### 3. 系统的三层结构

| 层 | 内容 | 说明 |
|---|---|---|
| **第一层：原始来源（Raw Sources）** | PDF、文章、会议记录等原始文档 | **只读**。AI 只读不改，这是你的"事实来源"（source of truth） |
| **第二层：Wiki 本体** | AI 创建并维护的一个 Markdown 文件夹 | 包含索引页、概念页、实体页、汇总对比页等，全部互相链接，全部由 AI 维护 |
| **第三层：Schema（规则文档）** | 一份"规则说明书" | 告诉 AI 如何组织 Wiki 结构、如何处理新来源、如何格式化。用 Claude Code 的话，这就是你的 `CLAUDE.md` 文件 |

---

## 二、准备工作：需要安装什么

### 需要两样东西

**1. Obsidian（免费笔记软件）**

- 下载地址：obsidian.md
- 作用：作为 Wiki 的**查看器**。它基于纯 Markdown 文件工作，且自带**图谱视图（Graph View）**，能把页面间的连接可视化
- 说明：Obsidian 不是必须的。整个 Wiki 本质上就是一个装 Markdown 文件的文件夹，用 VS Code 或任何文本编辑器都行，选你最顺手的即可

**2. 一个 AI 编程代理（AI Coding Agent）**

- 本文使用 **Claude Code**
- 也可以用 OpenAI Codex、Cursor 等任何**能读写你电脑上文件**的工具
- 这是整个系统的引擎——Obsidian 本身不会做任何 AI 工作

---

## 三、实操步骤（逐步还原本文操作）

### 第 1 步：创建 Vault（仓库）

![image-20260723174210372](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723174210372.png)

1. 打开安装好的 Obsidian
2. 点击 **Create new vault**（新建仓库）——"vault" 只是"文件夹"的花哨叫法
3. 命名为 `LLM Wiki`（可自定义）
4. 保存位置选一个简单的路径，本文中放在 **Documents（文档）** 目录下
5. 点击 **Create**

### 第 2 步：建立三个文件夹

![图片来自https://github.com/nashsu/llm_wiki](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/llm_wiki_arch.jpg)

在 Vault 内点击左上角的 **New folder（新建文件夹）** 按钮，依次创建：

| 文件夹 | 作用 |
|---|---|
| `raw` | 存放原始来源文档。**AI 只从这里读取，永远不修改这里的任何内容** |
| `wiki` | AI 构建和维护所有 Wiki 页面的地方 |
| `templates` | **可选**。如果你想在 Obsidian 里手动创建格式统一的笔记，可以放模板。本教程中所有 Wiki 页面都由 Claude 生成，所以用不到，仅作介绍 |

完成后的结构：

```
LLM Wiki/              ← Vault 根目录
├── raw/               ← 原始文档（只读）：要投喂的 PDF/文章放这里
├── wiki/              ← AI 维护的 Wiki 页面
├── templates/         ← 可选模板
└── CLAUDE.md          ← 下一步创建的规则文件
```

### 第 3 步：创建 Schema 文件（CLAUDE.md）—— 最关键的一步

这是告诉 AI "如何运营这个 Wiki" 的规则文档。

**操作：** 在 Vault 的**根目录**创建一个名为 `CLAUDE.md` 的文件（Claude Code 打开项目时会自动读取这个文件）。本文是直接把准备好的模板文件拖进根目录的。

**① Purpose（目的）——唯一需要按情况调整的部分**

一句话说明这个知识库是关于什么的。注意：**这里写的是整个 Wiki 的定位，不是某一批文档的说明**——Wiki 会长期容纳很多不同主题的资料，所以 Purpose 不要写得过于具体。

- 如果 Wiki 专门用于某个主题（研究可再生能源、追踪读书学习），就写那个主题
- 如果像本文一样是一个**综合知识库**（今天放一批技术调研文章，明天放论文、会议记录），就写成通用定位，让 AI 按主题自行组织
- **模板中其他内容全部开箱即用，只有这一节需要你按情况调整**

**② Folder structure（文件夹结构）**

说明原始资料在哪、Wiki 输出到哪、`index.md` 和 `log.md` 分别管什么。

**③ Ingest workflow（摄取工作流）**

定义"当我往 raw 里加入新文档时，AI 应该做什么"。关键的一步是**动笔前先和你讨论要点**（第 2 步），把关机会留给你；之后才是建摘要页、建/更新概念页、加链接、更新索引、写日志。

**④ Page format（页面格式）**

给每个 Wiki 页面规定统一骨架：顶部摘要、来源列表、最后更新时间，正文用短段落，结尾列相关页面。这样 AI 产出的页面整齐一致，方便日后维护和 Lint。

**⑤ Citation rules（引用规则）——多来源校验的核心**

每条事实论断必须标注来源文件（`(source: 文件名.md)`）；**两个来源说法冲突时显式记录矛盾**；没有来源的论断标记为"待核实"。校验"同一概念多篇文章的说法是否自相矛盾"，靠的就是这一节。

**⑥ Question answering（问答行为）**

提问时 AI 应先读 `index.md` 找到相关页、再综合作答并引用具体页面；答不出就明说；有价值的答案主动提议回填成新页面（让知识复利）。

**⑦ Lint（校验）**

规定 `Please lint the wiki.` 时检查哪些项：页面间矛盾、孤儿页、缺独立页面的概念、可能过时的论断、不合格式的页面。

**⑧ Rules（总则）**

兜底铁律：永不修改 `raw/`、每次改动都更新 `index.md` 和 `log.md`、页面命名规范、拿不准如何归类就问用户。

> 本文提醒：**不要在这一步想太多**。模板给的是一个扎实的起点，可以边用边完善——Schema 随着 Wiki 的成长而演化，这本身就是流程的一部分。

**本文实际使用的 CLAUDE.md（通用版，完整文件随本文稿一同提供，以下为全文）：**

这一版不绑定任何具体主题或文档，可以直接长期使用。有三处设计对实际效果影响最大：**Page format** 统一了每个页面的骨架（摘要、来源列表、最后更新时间），让 AI 生成的页面整齐一致；**引用规则**要求每条论断标注来源文件、来源之间冲突时显式记录矛盾——这是"多来源交叉校验"的核心机制；**摄取流程第 2 步**要求 AI 动笔前先和你讨论要点，给了你把关的机会。

```markdown
# LLM Wiki

A personal knowledge base maintained by Claude Code.
Based on Andrej Karpathy's LLM Wiki pattern.

## Purpose

This wiki is a structured, interlinked knowledge base. It may hold many
unrelated topics over time — each source you add is organized into the
existing structure and linked to related material.
Claude maintains the wiki. The human curates sources, asks questions,
and guides the analysis.

## Folder structure

raw/          -- source documents (immutable -- never modify these)
wiki/         -- markdown pages maintained by Claude
wiki/index.md -- table of contents for the entire wiki
wiki/log.md   -- append-only record of all operations

## Ingest workflow

When the user adds a new source to `raw/` and asks you to ingest it:

1. Read the full source document
2. Discuss key takeaways with the user before writing anything
3. Create a summary page in `wiki/` named after the source
4. Create or update concept pages for each major idea or entity
5. Add wiki-links ([[page-name]]) to connect related pages
6. Update `wiki/index.md` with new pages and one-line descriptions
7. Append an entry to `wiki/log.md` with the date, source name,
   and what changed

A single source may touch 10-15 wiki pages. That is normal.

## Page format

Every wiki page should follow this structure:

    # Page Title

    **Summary**: One to two sentences describing this page.

    **Sources**: List of raw source files this page draws from.

    **Last updated**: Date of most recent update.

    ---

    Main content goes here. Use clear headings and short paragraphs.

    Link to related concepts using [[wiki-links]] throughout the text.

    ## Related pages

    - [[related-concept-1]]
    - [[related-concept-2]]

## Citation rules

- Every factual claim should reference its source file
- Use the format (source: filename.pdf) after the claim
- If two sources disagree, note the contradiction explicitly
- If a claim has no source, mark it as needing verification

## Question answering

When the user asks a question:

1. Read `wiki/index.md` first to find relevant pages
2. Read those pages and synthesize an answer
3. Cite specific wiki pages in your response
4. If the answer is not in the wiki, say so clearly
5. If the answer is valuable, offer to save it as a new wiki page

Good answers should be filed back into the wiki so they compound
over time.

## Lint

When the user asks you to lint or audit the wiki:

- Check for contradictions between pages
- Find orphan pages (no inbound links from other pages)
- Identify concepts mentioned in pages that lack their own page
- Flag claims that may be outdated based on newer sources
- Check that all pages follow the page format above
- Report findings as a numbered list with suggested fixes

## Rules

- Never modify anything in the `raw/` folder
- Always update `wiki/index.md` and `wiki/log.md` after changes
- Keep page names lowercase with hyphens (e.g. `machine-learning.md`)
- Write in clear, plain language
- When uncertain about how to categorize something, ask the user
```

### 第 4 步（可选，但本案例强烈推荐）：安装 Obsidian Web Clipper 浏览器扩展

- 在 Chrome 应用商店添加 **Obsidian Web Clipper**（免费）
- 作用：把任何网页文章一键转换成 Markdown 文件
- 本文的 Agent Harness 调研来源全部是网页文章（个人博客、公司博客、X 长文），正是用 Web Clipper 一键剪藏成 Markdown 后放进 `raw` 的；如果你的来源本来就是现成的 PDF，用不到它

### 第 5 步：准备并投喂调研文章

本文围绕"Agent Harness 的核心组成与各家框架"这一调研主题，选了 5 篇 2026 年发布、视角互补的文章（全部用 Web Clipper 剪藏成 Markdown）：

| 文章 | 作者/出处 | 视角 |
|---|---|---|
| The Anatomy of an Agent Harness | Viv Trivedy（X 长文） | 组件拆解的"骨架"：从想要的行为反推每个 harness 组件为何存在 |
| What Is an Agent Harness? A Beginner's Guide | Khalid Abdelaty（DataCamp） | 入门科普 + harness/framework/runtime 三层辨析 + 2026 工具版图 |
| Agent Harness Engineering | Addy Osmani（个人博客） | 工程实践：每个错误变成一条规则的"棘轮"方法 |
| How to Build a Custom Agent Harness | Sydney Runkle（LangChain 官博） | 框架厂商视角：middleware 定制与 task-harness fit |
| Harness Engineering for Self-Improvement | Lilian Weng（Lil'Log） | 研究视角：harness 与递归自我改进（RSI） |

操作上把文件**直接拖进 `raw` 文件夹**即可。本文的实操节奏是：**先放入主干文章**《The Anatomy of an Agent Harness》走一遍完整摄取流程（让 AI 先讨论要点、你确认建页计划），**再把其余 4 篇一起放入**、一次性摄取——两种节奏都支持，AI 会自动把多篇内容编织进同一套概念页。

> **重要提示：来源不必是 Markdown。** 有 PDF 就直接把 PDF 拖进 `raw` 文件夹——Claude Code 原生支持读取 PDF，txt、md 同理。什么格式的文档都直接丢进去，Claude 会处理。
>
> **另一个提示：** 技术调研的资料天然"说法不一"——不同作者对同一个概念给出不同定义、组件清单各有侧重、术语出处各执一词，正好用得上 CLAUDE.md 的引用规则——每条论断标注来源文件、来源冲突时显式记录矛盾、无来源的论断标记"待核实"。LLM Wiki 天然适合梳理这类多方观点交织的调研材料。

### 第 6 步：打开 Claude Code 并执行首次摄取

**① 切换到 Vault 目录**

打开终端，先 `cd` 到 Vault 所在位置（必须让 Claude Code 在正确的目录下工作，它才能读到 CLAUDE.md 和你的文件夹）：

```bash
cd ~/Documents/"LLM Wiki"
```

**② 启动 Claude Code**

```bash
claude
```

**③ 输入摄取指令**

本文中使用的原话（这也是之后每次加新资料时用的固定话术）：

```
我刚在 raw 文件夹加了一些来源，请读取它并更新 wiki（如果没有先创建）
```

**④ 观察并确认**

![image-20260726151658303](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726151658303.png)

Claude 会：

- 读取文章全文
- 给出核心要点摘要（本文实操中它总结了 4 条：Agent = Model + Harness 公式、从行为反推组件的方法、模型-harness 协同训练的副作用、harness 的未来判断）
- 列出**计划创建的所有 Wiki 页面**。本文实操中它列了约 12 页的计划：索引页、日志页、来源摘要页，加上 agent-harness（核心概念）、harness-engineering（方法论）、文件系统、bash 与代码执行、沙箱、记忆与持续学习、context rot、长时程执行、模型-harness 协同训练、人物页等概念页

此时你可以调整范围（scope），如果计划看起来没问题，回复：

```
go ahead
```

本文中 5 篇全部摄取完成约 10 分钟。

### 第 7 步：回到 Obsidian 查看成果

本文实操 5 篇文章摄取完成后，`wiki` 文件夹里共生成 **30 个文件**：**5 个来源摘要页**（每篇文章一页）、**23 个概念页**（6 个核心概念页 + 12 个组件页 + 4 个生态研究页 + 1 个人物页），外加全站索引 `index.md` 和操作日志 `log.md`。

两个页面最能体现"AI 帮你拆解、对比"的价值，值得点开看：

1. **`agent-harness.md`**：一张**五方组件清单对比表**，Viv Trivedy、DataCamp、Osmani、LangChain、Weng 各强调哪些组件一目了然，表下附差异分析（如"可观测性只有两家提、评估只有 Weng 提"）。
2. **`harness-tooling-landscape.md`**：一张 **8 个框架/产品对比表**（Deep Agents、`create_agent`、Anthropic Agent SDK、OpenAI Agents SDK、Google ADK、MAF、CrewAI、Temporal/Inngest），每个定位与特点各占一行。

打开**图谱视图（Graph View）**——`agent-harness` 位于网络中心，组件页环绕四周，连接网络已经成形。

![image-20260726151911546](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726151911546.png)

### 第 8 步：多来源增量整合 + 交叉校验的实际效果

这是整套系统最有意思的地方。**关键观察点：** Claude 不是给每篇文章各建一套页面，而是把 5 篇文章的内容**编织进同一套概念页**——比如 `context-rot.md` 一页里就汇集了 4 篇文章对同一问题的表述（Viv 的三种对策、Osmani 转述 Anthropic 的"完整上下文重置"、Weng 的研究化版本），每条都按 CLAUDE.md 的引用规则标注了来源文件。

更重要的是矛盾标记。本文实操中，AI 在摄取过程中**自动揪出了 3 处来源矛盾/出入**，全部显式记录在对应页面并汇总在 `index.md` 底部：

1. **"harness engineering" 术语归属**：Osmani 说这个词是 Viv Trivedy 创造（coined）的；DataCamp 却说它是随 Mitchell Hashimoto（HashiCorp 联合创始人）2026 年 2 月的博客流行开来的。AI 还进一步指出时间线疑点——Viv 的《Anatomy》发布于 3 月 11 日，晚于 Hashimoto 的博客，于是把"coined"的说法标为**待核实**。
2. **"If you're not the model, you're the harness" 引语归属**：DataCamp 把这句话归为"来自 LangChain 的定义"，但它的原始出处是 Viv 的帖子。两种归属被并列记录，而不是悄悄二选一。
3. **同一来源内部的数据出入**：Lilian Weng 文章里引用 Lin et al. 2026 的实验，正文写模型范围是 "Qwen3.5-9B to Opus 4.6"，图注却写 "Qwen2-32B to Opus 4.6"——连同一篇文章正文和图注打架，AI 也标出来了。

**这就是 Wiki 在履行它的职责**：知识不是堆积，而是被编织进已有的结构——来源之间的出入被系统性地暴露出来，而不是靠你逐篇肉眼比对。

> **回到开头那张"效果↔机制"表**：这里的两个效果，正对应两条 CLAUDE.md 规则——"编织进同一套概念页"来自摄取流程"更新已有页面而非按来源分档"，"3 处矛盾一条不丢"来自引用规则"两源冲突就显式记录"。你不必信任 AI 的自觉，是规则逼它这么做的。

再看图谱视图：节点更多、连接更多。**每加一份来源，Wiki 就变得更聪明。**

### 第 9 步：规模升级——投喂 85 个文件的工程资料，看 AI 如何自己拿方案

前 8 步处理的是 5 篇观点文章。这一步把强度直接拉满：往 `raw` 里放入一个 **`Agent Harness 开源项目`** 文件夹，里面是 **85 个文件、约 31 MB** 的一手工程资料，分三个子目录：

| 子目录 | 内容 |
|---|---|
| `官方仓库README` | 14 个开源框架的 GitHub README：AgentScope、AutoGen、Microsoft Agent Framework、Google ADK、smolagents、DeepAgents、LangGraph、Mastra、CrewAI、PydanticAI、OpenAI Agents SDK 等 |
| `官方文档` | 59 个文档快照：多数框架的 `llms.txt` 目录 + 关键页面正文（HTML/Markdown），部分框架有完整文档聚合文本（单文件最大 14 MB） |
| `论文与研究` | 7 篇论文 PDF：Agentic AI Frameworks 综述、AgentScope 系列 3 篇、框架失效模式研究、Auto-SLURP 基准等 |

![image-20260726152152885](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726152152885.png)

然后说的还是**那句一模一样的固定话术**：

```
我刚在 raw 文件夹加了一些来源，请读取它并更新 wiki（如果没有先创建）
```

**这一步值得观察的是 AI 面对大规模资料时的反应**（这正是 CLAUDE.md 摄取流程第 2 步"动笔前先讨论"的价值）：它没有闷头开写，而是先扫完 85 个文件按类型汇报（甚至转述了资料自带的版本提醒，如"AutoGen 已进维护模式"），判断这批是一手工程资料、该建成**框架实体页**（每个框架一页：定位、核心抽象、harness 组件、部署、成熟度），然后给出**带选项的计划**交你定夺——约 20 个新页 + 6 个已有页更新，并说明会派多个子 agent 并行读（31 MB 一次读不完）。

![image-20260726153343463](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726153343463.png)

确认范围、回复 `go ahead` 后，AI 按计划跑完整批摄取。**本案例真实结果：**

- **新建 17 页**：14 个框架/harness 实体页 + 3 个研究页；8 个子 agent 分组并行阅读、各自起草再汇总。7 篇论文按之前说的"可收缩选项"做了归并——Auto-SLURP 与渗透测试合一页，AgentScope 系列 3 篇并进实体页，其余两篇各自成页。
- **`harness-tooling-landscape.md` 从 8 行扩成完整版图**：框架名都变成可点进去的实体页，并分出"编码/编排型框架"与"个人/自托管 harness"两个象限。
- **交叉校验继续发力**：拿官方文档核对第一批 DataCamp 科普，又揪出 **6 处新出入**（矛盾清单 3 → 9 条），如 Deep Agents 的"PII middleware"官方查无此项、Google ADK 托管服务已改名、CrewAI 三层记忆已被统一 Memory 取代。

**这就是"知识复利"的具象化：第一批文章搭好概念骨架，第二批工程资料往骨架上挂肉——新资料还会反过来校正旧记载。** 这正是开头对比表最后一行"用得越久"的兑现：RAG 每次从头再来，LLM Wiki 每加一份料都站在上一批的肩膀上。



> 顺带验证了一个前面的提示：`raw` 里可以混放 md、html、txt、PDF，甚至整个带子目录的文件夹——AI 都能自己盘明白。

### 第 10 步：提一个"跨来源"的问题

现在问一个需要同时用到多份来源信息的问题。围绕本案例的调研主题，可以这样问：

```
Agent Harness 的核心组成，各家清单有什么差异？哪些组件只有某一家强调？
```

```
LangChain Deep Agents、AgentScope、Mastra 这几个框架
各自的定位和取舍是什么？如果我要做一个个人知识库agent，该从哪个入手？
```

**实操中 Claude 的两次回答：**

![image-20260726174354591](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726174354591.png)

第一问，它直接命中 `agent-harness.md` 里的五方组件对比表，先点出共识地基（`Agent = Model + Harness`，工具和编排是公约数），再指出只有某一家强调的组件：**可观测性只有 DataCamp、Osmani 两家列**，**评估作为一等组件只有 Lilian Weng 提**，**middleware 作为统一定制原语是 LangChain 独有主张**——并解释这源于各自立场（构建者/科普/产品/研究视角）不同。

![image-20260726174556632](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726174556632.png)

![image-20260726174924234](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726174924234.png)

第二问，它读取了 `deepagents.md`、`agentscope.md`、`mastra.md` 三个实体页加 `harness-tooling-landscape.md`，用一张表对齐三家的出身/语言/定位/取舍，然后先泼冷水——"你这套 LLM Wiki 本身就是 Claude Code+markdown 方案，只做知识库其实不必引框架"，再给分场景建议（Python 最快上手选 Deep Agents、要多租户+权限选 AgentScope、嵌 TS 前端选 Mastra）。

**这两次回答体现了四点，正是它区别于基础 RAG 的地方：**

- **不重新翻原始文档**，而是从已建好的 Wiki 页面取答案
- **精准命中相关主题页**，而非全库检索拼片段
- **把散落在不同来源的信息点连接、对照**起来（第一问跨五位作者、第二问跨三个框架）
- **引用具体 Wiki 页面作依据**，来源有分歧时并列呈现而不是悄悄二选一

问题问得越"跨来源"，这种预先组织好的知识库相比每次从零检索的 RAG，优势就越明显。

> **再对回"效果↔机制"表**：这里的"直接命中主题页、不翻原文"就是问答规则（先读 `index.md` 定位 → 读相关页综合）加上摄取时早已建好的互链在起作用。综合的重活在建库时做完了，提问只是取现成的——这正是它和 RAG 分道扬镳的地方。

### 第 11 步：给 Wiki "做体检"——Lint（校验）

Karpathy 提出的一个巧妙概念：就像代码 Linter 检查代码问题一样，**定期让 AI 审计整个 Wiki**。它会检查：

- 页面之间的**矛盾**
- 可能**过时**的论断
- **孤儿页面**（没有任何链接指向它的页面）
- 被提到但**还没有独立页面**的概念

操作很简单，直接对 Claude 说：

```
请校验一下 wiki
```

![image-20260726175333876](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726175333876.png)

Claude 会遍历所有内容并给出一份报告：从孤儿页面、失效链接，到没有来源标注的论断、来源间的矛盾是否都已记录。在"多来源交叉校验"的场景里，Lint 尤其有价值——它相当于定期复查：**有没有哪条矛盾被摄取时漏标了？有没有观点只写了结论没标注出处？**

**随着 Wiki 增长，定期 Lint 是保持它健康的方式。**

---

## 四、日常使用循环（总结）

搭建完成后，日常就是三个动作的循环：

1. **喂料**：把新文档（PDF/文章/笔记）丢进 `raw` 文件夹 → 说 `我刚在 raw 文件夹加了一个新来源，请读取它并更新 wiki`
2. **提问**：直接问跨来源/跨作者的问题（组件差异、框架选型、说法冲突），AI 基于 Wiki 回答、引用页面，并指出来源间分歧
3. **体检**：定期说 "Please lint the wiki." 保持知识库健康

---

## 五、适用场景

| 身份 | 用法 |
|---|---|
| **学生 / 研究者** | 边读论文和文章边构建 Wiki，最后得到的是结构化知识库，而不是一堆划了高亮的 PDF |
| **教师** | 投喂课程文档、教研材料和文章，构建一个随时间成长的个人教学 Wiki |
| **企业 / 团队** | 投喂会议记录、客户通话转录、项目文档；新成员浏览有组织的 Wiki，而不是去翻 Slack 历史 |
| **爱阅读的人** | 追踪从书籍、播客、文章中学到的东西，等于建造自己的个人百科全书 |
| **技术调研 / 框架选型** | 投喂同一主题下不同作者、不同立场的文章（如本文的 Agent Harness 调研），拆解概念组成、生成框架对比表、系统性暴露各家说法的分歧 |

**通用规律：只要你在长期积累知识、并希望它被组织起来而不是散落各处，这个模式就适用。**

---

## 六、局限性（本文的坦诚提醒）

1. **最适合个人规模。** Karpathy 谈的是约 100 篇文章量级的 Wiki。如果要做几万页的系统，需要比 Markdown 文件更重的基础设施。
2. **垃圾进，垃圾出。** Wiki 的质量取决于你投喂的来源质量，你仍然需要**筛选**进入的内容。
3. **必须有编程代理。** Obsidian 自己什么都不会做，AI 才是引擎。需要 Claude Code、Codex 或类似工具。
4. **AI 会犯错。** 可能分类错误或连接错误——这正是 Lint 功能存在的意义。尤其在早期，建议人工复核它构建的内容。

尽管如此，作者认为这是他见过最实用的 AI 工作流之一：解决真实问题、免费搭建、数据以纯文本形式保存在你自己的电脑上，**完全归你所有**。

---

## 附：快速检查清单

- [ ] 安装 Obsidian（obsidian.md）
- [ ] 准备好 Claude Code（或 Codex/Cursor 等）
- [ ] 创建 Vault：`LLM Wiki`
- [ ] 创建三个文件夹：`raw` / `wiki` / `templates`
- [ ] 在根目录创建 `CLAUDE.md`（通用模板可直接用，按需微调 Purpose）
- [ ] （可选）安装 Obsidian Web Clipper 浏览器扩展（网页文章来源必备）
- [ ] 把调研文章放入 `raw`（可先放一篇主干文章，也可多篇一起）
- [ ] 终端 `cd` 到 Vault 目录，启动 `claude`
- [ ] 发送摄取指令 → 确认建页计划 → `go ahead`
- [ ] 在 Obsidian 查看成果：中心概念页的组件对比表、框架工具版图页、图谱视图
- [ ] 检查 `index.md` 底部的矛盾汇总，追溯 AI 标记的来源分歧
- [ ] 规模升级：投喂框架 README/官方文档/论文等大批量工程资料，观察 AI 的盘点与建页方案，确认范围后执行
- [ ] 持续喂料，定期 `Please lint the wiki.`

> 文中的演示以"投喂几篇 Agent Harness（智能体框架）文章，让 AI 拆解 harness 的核心组成、对比各家开源框架、并交叉校验各来源说法是否矛盾"为例，你可以把来源换成任何主题的资料。
>
> **👉 想要 LLM Wiki实战 素材**
>
> **🚀 获取通道：**
>
> **1️⃣ 微信搜索关注公众号 👉 萤火AI百宝箱**
>
> **2️⃣ 后台私信回复暗号 👉 Agent**
