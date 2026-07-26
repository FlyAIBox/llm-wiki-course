# LLM Wiki实战：让AI揪出 Agent Harness 各家说法的矛盾

> 文中的演示以"投喂几篇 Agent Harness（智能体框架）调研文章、提炼核心观点、交叉校验多来源之间是否矛盾"为例，你可以把来源换成任何主题的资料。
>
> **👉 想要 LLM Wiki实战 素材**
>
> **🚀 获取通道：**
>
> **1️⃣ 微信搜索关注公众号 👉 萤火AI百宝箱**
>
> **2️⃣ 后台私信回复暗号 👉 Agent**

---

## 先看效果：LLM Wiki 最终能帮你做什么

![image-20260723194456489](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723194456489.png)

把访谈、PDF、文章等资料放进 `raw` 文件夹，再让 AI 摄取后，原本散落在不同文档里的信息会被整理成一组**相互链接、可持续更新的知识页面**。打开 Obsidian 的关系图谱，你可以直观看到概念定义、框架组件、代表人物、工具产品、设计模式等主题如何彼此关联；点击任意节点，就能继续追溯具体观点和原始来源。

更实用的是，这不只是一张“看起来很酷”的图。你可以直接让 AI：

- 汇总某个概念或某个主题在多份资料中的核心观点
- 对比不同来源对同一问题的说法，并标记相互矛盾之处
- 从一个概念沿着双向链接快速发现相关人物、公司和工具
- 在加入新资料后自动更新已有页面，让知识库越用越完整
- 回答问题时给出具体 Wiki 页面和原始文件依据，而不是每次从零检索

本文最终会得到这样一个 LLM Wiki：一边是 AI 生成并维护的主题页面，一边是它们之间逐步形成的知识网络。下面就从为什么需要这种方式开始，再一步步搭建出来。

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

### 第 5 步：投喂第一批文档

准备好第一份要投喂的资料。本文的演示是先放入一篇入门文章——DataCamp 的《What Is an Agent Harness? A Beginner's Guide》（用 Web Clipper 剪藏成的 Markdown）。

![image-20260726135302405](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260726135302405.png)

1. 把文件**直接拖进 `raw` 文件夹**即可

> **重要提示：来源不必是 Markdown。** 有 PDF 就直接把 PDF 拖进 `raw` 文件夹——Claude Code 原生支持读取 PDF，txt、md 同理。什么格式的文档都直接丢进去，Claude 会处理。
>
> **另一个提示：** 技术调研的资料天然"说法不一"——不同作者对同一个概念（比如"什么是 harness"）给出不同定义、对同一个术语的出处各执一词，正好用得上 CLAUDE.md 的引用规则——每条论断标注来源文件、来源冲突时显式记录矛盾、无来源的论断标记"待核实"。LLM Wiki 天然适合梳理这类多方观点交织的调研材料。

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

![image-20260723175304975](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723175304975.png)

```
我刚在 raw 文件夹加了一个新来源，请读取它并更新 wiki
```

**④ 观察并确认**

![image-20260723181915511](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723181915511.png)

Claude 会：

- 读取文章全文
- 给出文章摘要
- 列出**计划创建的所有 Wiki 页面**（按主题/概念拆分的观点页，例如：agent-harness 概念页、"Agent = Model + Harness" 公式页、harness 与 framework/runtime 的区别、代表人物页等）

此时你可以调整范围（scope），如果计划看起来没问题，回复：

```
go ahead
```

本文中约 10 分钟后全部完成。

![image-20260723193907141](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723193907141.png)

### 第 7 步：回到 Obsidian 查看成果

1. 打开 `wiki` 文件夹——可以看到结构化的页面已经生成
2. 点开任意页面——页面之间已经带有相互链接
3. 打开**图谱视图（Graph View）**——可以看到连接网络正在成形

> 这只是**一份**文档的效果。想象 20 份来源之后是什么样子。

![image-20260723194001411](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723194001411.png)

### 第 8 步：加入更多来源，见证"增量整合 + 交叉校验"

这是整套系统最有意思的地方。继续往 `raw` 里放入新来源——本文接着放入了同一主题下不同视角的几篇文章：

- Viv Trivedy 的《The Anatomy of an Agent Harness》（X 长文，harness 组件的系统拆解）
- Addy Osmani 的《Agent Harness Engineering》（个人博客，工程实践视角）
- LangChain 的《How to Build a Custom Agent Harness》（框架厂商视角）
- Lilian Weng 的《Harness Engineering for Self-Improvement》（研究视角，harness 与递归自我改进）

1. 同样直接把文件拖进 `raw` 文件夹
2. 对 Claude 说**一模一样的话**：

```
我刚在 raw 文件夹加了一个新来源，请读取它并更新 wiki
```

**关键观察点：** Claude 不是为新来源另起炉灶，而是在**更新已有的主题观点页**——把新文章中的对应内容逐条并入已有页面（比如把各家对 harness 组件的清单并入同一个概念页），并按 CLAUDE.md 的引用规则给每条观点标注来源文件。

更重要的是：当不同来源**说法冲突**时，Claude 会按规则在页面上**显式记录矛盾**并同时引用冲突双方。本案例中就有一个现成的例子：**"harness engineering" 这个术语到底是谁提出的**——Addy Osmani 的文章说这个词是 Viv Trivedy 创造的，而 DataCamp 的文章却说它是在 Mitchell Hashimoto（HashiCorp 联合创始人）2026 年 2 月的博客之后流行开来的。此外，各家列出的 harness 组件清单也不一致（有的强调 tools/memory/guardrails/observability，有的强调 prompts/hooks/sandboxes/subagents）——这正是校验"同一概念多方说法"是否自相矛盾的核心机制。

**这就是 Wiki 在履行它的职责**：知识不是堆积，而是被编织进已有的结构——来源之间的出入被系统性地暴露出来，而不是靠你逐篇肉眼比对。

再看图谱视图：节点更多、连接更多。**每加一份来源，Wiki 就变得更聪明。**

### 第 9 步：提一个"跨来源"的问题

现在问一个需要同时用到多份来源信息的问题。比如在多篇调研文章的场景下，可以这样问：

```
关于 "harness engineering" 这个术语是谁提出的，各个来源说法一致吗？
各家对 Agent Harness 组件的定义有什么差异，有没有只在其中一个来源出现的组件？
```

**观察 Claude 的回答方式：**

- 它**不是**在重新翻原始文档
- 它是从 **Wiki 页面**中提取相关主题页
- 它把散落在**不同来源**中的信息点连接、对照了起来
- 它**引用了具体的 Wiki 页面**作为依据，来源之间有分歧时会并列呈现而不是悄悄二选一

![image-20260723194856220](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723194856220.png)

这与基础 RAG 得到的结果有本质区别。

### 第 10 步：给 Wiki "做体检"——Lint（校验）

Karpathy 提出的一个巧妙概念：就像代码 Linter 检查代码问题一样，**定期让 AI 审计整个 Wiki**。它会检查：

- 页面之间的**矛盾**
- 可能**过时**的论断
- **孤儿页面**（没有任何链接指向它的页面）
- 被提到但**还没有独立页面**的概念

操作很简单，直接对 Claude 说：

```
请校验一下 wiki
```

![image-20260723195517896](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723195517896.png)

![image-20260723195559346](https://cdn.jsdelivr.net/gh/Fly0905/note-picture@main/mag/image-20260723195559346.png)

Claude 会遍历所有内容并给出一份报告：从孤儿页面、失效链接，到没有来源标注的论断、来源间的矛盾是否都已记录。在"多来源交叉校验"的场景里，Lint 尤其有价值——它相当于定期复查：**有没有哪条矛盾被摄取时漏标了？有没有观点只写了结论没标注出处？**

**随着 Wiki 增长，定期 Lint 是保持它健康的方式。**

---

## 四、日常使用循环（总结）

搭建完成后，日常就是三个动作的循环：

1. **喂料**：把新文档（PDF/文章/笔记）丢进 `raw` 文件夹 → 说 `我刚在 raw 文件夹加了一个新来源，请读取它并更新 wiki`
2. **提问**：直接问跨来源/跨作者的问题，AI 基于 Wiki 回答、引用页面，并指出来源间分歧
3. **体检**：定期说 "Please lint the wiki." 保持知识库健康

---

## 五、适用场景

| 身份 | 用法 |
|---|---|
| **学生 / 研究者** | 边读论文和文章边构建 Wiki，最后得到的是结构化知识库，而不是一堆划了高亮的 PDF |
| **教师** | 投喂课程文档、教研材料和文章，构建一个随时间成长的个人教学 Wiki |
| **企业 / 团队** | 投喂会议记录、客户通话转录、项目文档；新成员浏览有组织的 Wiki，而不是去翻 Slack 历史 |
| **爱阅读的人** | 追踪从书籍、播客、文章中学到的东西，等于建造自己的个人百科全书 |
| **技术调研 / 观点对比** | 投喂同一主题下不同作者、不同立场的文章（如本文的 Agent Harness 调研），提炼观点、逐条标注来源、系统性暴露各家说法的分歧 |

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
- [ ] 把第一篇文章放入 `raw`
- [ ] 终端 `cd` 到 Vault 目录，启动 `claude`
- [ ] 发送摄取指令 → 确认计划 → `go ahead`
- [ ] 在 Obsidian 图谱视图查看成果
- [ ] 继续放入更多来源（同一主题的其他文章），再次摄取，观察矛盾标记
- [ ] 持续喂料，定期 `Please lint the wiki.`

> 文中的演示以"投喂几篇 Agent Harness（智能体框架）调研文章、提炼核心观点、交叉校验多来源之间是否矛盾"为例，你可以把来源换成任何主题的资料。
>
> **👉 想要 LLM Wiki实战 素材**
>
> **🚀 获取通道：**
>
> **1️⃣ 微信搜索关注公众号 👉 萤火AI百宝箱**
>
> **2️⃣ 后台私信回复暗号 👉 Agent**
