# OpenKB 安装与使用教程

> 项目地址：[VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB)  
> 本文基于 OpenKB `main` 分支及 PyPI `openkb 0.4.5` 整理（2026-07-26）。

## 1. OpenKB 是什么

OpenKB 是一个开源的 LLM 知识库工具。它会把 PDF、Word、Markdown、PPT、Excel、CSV、网页等原始资料编译成互相链接的 Wiki，并支持：

- 基于知识库进行单轮问答和多轮对话；
- 使用 PageIndex 处理长 PDF，无需向量数据库；
- 自动生成摘要、概念页、实体页和交叉引用；
- 用 Obsidian 打开生成的 Markdown Wiki；
- 生成知识图谱、Agent Skill 和 HTML 幻灯片；
- 通过本地 Web UI 浏览、上传和查询资料。

OpenKB 在导入文档时会调用 LLM，因此除了安装软件，还需要准备受支持模型的 API Key，或者使用支持 OAuth 的模型提供商。

## 2. 环境要求

官方包声明支持 Python 3.10 及以上，并列出了 Python 3.10～3.13。实际安装建议使用：

- **Python 3.11 或 3.12（推荐）**
- macOS、Linux 或 Windows
- 可访问 PyPI 和所用模型 API 的网络

不要直接安装到 Conda 的 `base` 环境。独立环境可以避免 OpenKB 的严格依赖版本影响其他项目。

安装前先确认解释器和 `pip` 属于同一个环境：

```bash
python --version
python -m pip --version
```

## 3. 推荐安装方式

### 3.1 使用 Conda

```bash
conda create -n openkb python=3.11 -y
conda activate openkb

python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb

openkb --help
```

截至本文记录时间，PyPI 稳定版 0.4.5 **尚未提供** `web` extra 和
`openkb-web` 命令。需要 Web UI 时安装已包含这些功能的 0.5.0
候选版本：

```bash
python -m pip install --upgrade --pre "openkb[web]==0.5.0rc1"
```

### 3.2 使用 Python venv

macOS / Linux：

```bash
python3.11 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb
openkb --help
```

Windows PowerShell：

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb
openkb --help
```

### 3.3 安装 GitHub 最新版

PyPI 版本落后于仓库时，可以在独立的 Python 3.11/3.12 环境中安装最新版：

```bash
python -m pip install "git+https://github.com/VectifyAI/OpenKB.git"
```

需要修改源码时：

```bash
git clone https://github.com/VectifyAI/OpenKB.git
cd OpenKB
python -m pip install -e .
```

## 4. 本次安装报错分析

原始环境中的关键信息是：

```text
/Users/fly/miniconda3/lib/python3.14/site-packages
Python reports SOABI: cpython-314-darwin
Collecting litellm
Using cached litellm-1.93.0.tar.gz
Rust not found
ImportError: Using SOCKS proxy, but the 'socksio' package is not installed
```

这不是 OpenKB 本身运行时报错，而是依赖解析和构建阶段失败：

1. 当前使用的是 Conda `base` 中的 **Python 3.14**。
2. `pip` 检查多个 OpenKB 版本后发生回退，最终尝试安装依赖约束较宽的旧版本。
3. 该版本解析到 `litellm` 源码包，而不是可直接安装的 wheel。
4. `litellm` 的构建依赖尝试通过 `maturin`/`puccinialin` 准备 Rust 工具链。
5. 当前网络配置使用 SOCKS 代理，但临时构建环境中没有 `socksio`，所以 Rust 下载过程失败。

### 推荐修复

直接离开 `base`，使用 Python 3.11 新环境重新安装：

```bash
conda create -n openkb python=3.11 -y
conda activate openkb
python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb
openkb --help
```

这是最稳妥的解法。通常**不需要**为这次错误单独安装 Rust，也不建议先在 Python 3.14 的 `base` 环境中不断补构建依赖。

### 清理缓存后重试

如果切换到 Python 3.11 后仍错误地使用旧缓存，可执行：

```bash
python -m pip cache purge
python -m pip install --no-cache-dir openkb
```

### SOCKS 代理的补充处理

先检查代理变量：

```bash
env | grep -i proxy
```

如果代理是必须的，可在当前环境补装 SOCKS 支持：

```bash
python -m pip install "httpx[socks]"
```

但请注意，`pip` 的隔离构建环境不一定继承当前环境中的 Python 包，所以这不能替代切换到受支持 Python 版本。若代理并非必需，可只对当前终端临时取消：

```bash
unset ALL_PROXY all_proxy HTTP_PROXY HTTPS_PROXY http_proxy https_proxy
```

不要在不清楚公司或校园网络要求时永久删除代理配置。

## 5. 创建第一个知识库

### 5.1 初始化

```bash
mkdir my-kb
cd my-kb
openkb init
```

初始化时按提示选择模型和输出语言。生成的主要配置位于：

```text
.openkb/config.yaml
```

一个基础配置示例：

```yaml
model: gpt-5.4
language: zh
pageindex_threshold: 20
```

模型名称采用 LiteLLM 的 `provider/model` 格式，例如：

```yaml
model: anthropic/claude-sonnet-4-6
```

OpenAI 模型可以省略提供商前缀。

### 5.2 配置 API Key

在知识库根目录创建 `.env`：

```dotenv
LLM_API_KEY=your_llm_api_key
```

使用 OpenAI 兼容网关时，还可以设置：

```dotenv
LLM_API_KEY=your_llm_api_key
OPENAI_API_BASE=https://your-api-endpoint.example/v1
```

不要把真实 API Key 提交到 Git 仓库。建议确认 `.gitignore` 中包含 `.env`。

## 6. 导入资料

导入单个文件：

```bash
openkb add paper.pdf
```

导入整个目录：

```bash
openkb add ~/Documents/papers/
```

导入网页或在线 PDF：

```bash
openkb add "https://arxiv.org/pdf/2509.11420"
```

支持的常见格式包括 PDF、DOCX、PPTX、XLSX、XLS、Markdown、HTML、CSV 和纯文本。

导入过程中 OpenKB 会调用 LLM 并产生费用。建议先用一篇较短的文档验证模型、网络和额度，再批量导入。长 PDF 默认在达到 `pageindex_threshold` 后使用 PageIndex 建立层次索引；扫描版 PDF 如需 OCR，可选配 PageIndex Cloud。

## 7. 查询和管理

单轮问答：

```bash
openkb query "这篇论文的主要结论是什么？"
```

保存查询结果：

```bash
openkb query "总结关键方法和局限性" --save
```

进入多轮对话：

```bash
openkb chat
```

查看知识库状态：

```bash
openkb status
openkb list
openkb lint
```

监听 `raw/` 目录并自动编译新文件：

```bash
openkb watch
```

重新编译已有文档：

```bash
openkb recompile --all
```

重新编译会覆盖由 OpenKB 生成页面中的手工修改，执行前应备份或提交 Git。

安全预览删除影响：

```bash
openkb remove "文档名" --dry-run
```

确认无误后再执行实际删除：

```bash
openkb remove "文档名"
```

## 8. 可视化、Skill 和幻灯片

生成交互式知识图谱：

```bash
openkb visualize
```

输出文件默认位于：

```text
output/visualize/graph.html
```

从知识库生成 Agent Skill：

```bash
openkb skill new my-expert "根据知识库，以领域专家方式分析问题"
openkb skill validate my-expert
openkb skill eval my-expert
```

生成单文件 HTML 幻灯片：

```bash
openkb deck new intro "用中文介绍知识库中的核心主题"
```

## 9. 启动 Web UI

### 9.1 版本说明

OpenKB 的仓库文档描述的是较新的代码，而 PyPI 稳定版 0.4.5 的 wheel
中只有 `openkb` CLI，不包含：

- `web` extra；
- `openkb.api`；
- `openkb-web` / `openkb-api` 命令；
- 编译后的 Web 前端。

因此，在 0.4.5 环境执行下面的命令：

```bash
python -m pip install "openkb[web]"
```

会出现：

```text
WARNING: openkb 0.4.5 does not provide the extra 'web'
```

这条命令虽然可能以成功状态结束，但 **Web UI 并未安装**。

### 9.2 安装包含 Web UI 的候选版本

目前可以显式安装 PyPI 的 0.5.0rc1：

```bash
python -m pip install --upgrade --pre "openkb[web]==0.5.0rc1"
```

`rc` 表示发布候选版，并非最终稳定版。它适合体验 Web UI；重要知识库
应先备份，生产环境则建议等待正式的 0.5.0。

验证安装内容：

```bash
python -m pip show openkb
command -v openkb-web
openkb-web --help
```

`pip show` 应显示版本 `0.5.0rc1`，并且能够找到 `openkb-web`。

### 9.3 启动服务

启动服务：

```bash
openkb-web
```

浏览器打开：

```text
http://127.0.0.1:7566/
```

本地模式默认不启用认证。如果服务要监听非本机地址或通过网络暴露，务必先设置令牌：

```bash
export OPENKB_API_TOKEN="replace-with-a-strong-random-token"
openkb-web
```

交互式 REST API 文档位于：

```text
http://127.0.0.1:7566/docs
```

## 10. 生成文件与 Obsidian

OpenKB 生成的 Wiki 是普通 Markdown 文件，并使用 `[[wikilinks]]` 建立链接。可以直接用 Obsidian 打开知识库中的 `wiki/` 目录：

1. 在 Obsidian 中选择“打开文件夹作为仓库”；
2. 选择 `my-kb/wiki/`；
3. 浏览摘要、概念、实体和探索页面；
4. 使用 Obsidian 图谱查看知识之间的连接。

`wiki/AGENTS.md` 定义 Wiki 的结构和维护规范。修改后会影响后续编译行为，建议先用 Git 管理知识库目录。

## 11. 常见问题

### `openkb: command not found`

通常是环境未激活，或者 `pip` 与 `python` 不属于同一环境：

```bash
conda activate openkb
which python
which openkb
python -m pip show openkb
```

Windows 可用：

```powershell
where.exe python
where.exe openkb
```

### `CondaToSNonInteractiveError`

如果创建环境时看到：

```text
CondaToSNonInteractiveError: Terms of Service have not been accepted
```

说明 Conda 在下载 Python 之前就停止了，环境实际上没有创建。因此紧接着执行 `conda activate openkb` 还会出现：

```text
EnvironmentNameNotFound: Could not find conda environment: openkb
```

可根据自己的情况选择以下一种方式。

**方式一：接受 Anaconda 默认仓库条款**

阅读并同意相关条款后执行报错中给出的命令：

```bash
conda tos accept --override-channels \
  --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels \
  --channel https://repo.anaconda.com/pkgs/r

conda create -n openkb python=3.11 -y
conda activate openkb
```

**方式二：只使用 conda-forge，绕开 Anaconda 默认仓库**

```bash
conda create -n openkb \
  --override-channels \
  --channel conda-forge \
  python=3.11 -y

conda activate openkb
```

`--override-channels` 很重要，它表示本次创建环境不再同时使用配置中的 `defaults`。

**方式三：不用 Conda**

如果本机已经安装 Python 3.11，可以直接使用标准 `venv`：

```bash
python3.11 -m venv ~/openkb-venv
source ~/openkb-venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb
```

环境创建后可以这样确认：

```bash
conda info --envs
python --version
```

其中 `python --version` 应显示 Python 3.11.x。

### 安装时长时间显示 `pip is looking at multiple versions`

这是依赖回溯。先确认不是 Python 3.14，再升级 `pip`：

```bash
python --version
python -m pip install --upgrade pip setuptools wheel
```

仍无法安装时，新建干净的 Python 3.11 环境，避免在原环境继续叠加依赖。

### 调用模型时报 401

检查：

- `.env` 是否在当前知识库根目录；
- `LLM_API_KEY` 是否正确；
- 模型与 API Key 的提供商是否匹配；
- 自定义网关的 `OPENAI_API_BASE` 是否包含正确路径。

如果错误明确显示：

```text
key not allowed to access model
This key can only access models=[...]
Tried to access gpt-5.4
```

说明 API Key 本身已被网关识别，但 `.openkb/config.yaml` 中配置的模型不在该 Key 的允许列表里。这不是重新安装 OpenKB 可以解决的问题。

进入执行过 `openkb init` 的知识库根目录，检查配置：

```bash
pwd
grep -n "^model:" .openkb/config.yaml
```

然后编辑 `.openkb/config.yaml`，把模型改成错误信息允许的模型，例如：

```yaml
model: gpt-5.5
language: zh
pageindex_threshold: 20
```

如果使用官方 Anthropic API，应使用 LiteLLM 的提供商前缀，例如：

```yaml
model: anthropic/claude-opus-4-6
```

如果使用的是提供 OpenAI 兼容接口的第三方网关，应以该网关文档给出的模型名称和调用协议为准；不要因为模型名字包含 `claude` 就擅自添加 `anthropic/` 前缀。

保存后重新运行被中止的命令：

```bash
openkb add /path/to/document.pdf
```

如果上次中止已留下文档记录，可先查看状态，再决定重新添加或重新编译：

```bash
openkb status
openkb list
openkb recompile --all
```

### 调用模型时报 429

通常是余额、速率或并发限制。可在 `.openkb/config.yaml` 中降低并发：

```yaml
concurrency: 2
```

也可以减少一次导入的文件数量，或更换额度充足的模型。

### 本地模型响应超时或参数不兼容

可以增加超时，并允许 LiteLLM 丢弃提供商不支持的参数：

```yaml
litellm:
  timeout: 1200
  drop_params: true
  num_retries: 3
```

### 扫描版 PDF 无法提取文字

本地 PageIndex 不等同于 OCR 服务。扫描版 PDF 可先自行 OCR，或配置 PageIndex Cloud：

```dotenv
PAGEINDEX_API_KEY=your_pageindex_api_key
```

## 12. 最小可运行流程

```bash
# 创建独立环境
conda create -n openkb python=3.11 -y
conda activate openkb

# 安装
python -m pip install --upgrade pip setuptools wheel
python -m pip install openkb

# 初始化知识库
mkdir my-kb
cd my-kb
openkb init

# 在当前目录创建 .env，并填写真实 Key
# LLM_API_KEY=...

# 导入、查询
openkb add /path/to/document.pdf
openkb status
openkb query "请概括文档的核心内容"
openkb chat
```

## 参考资料

- [OpenKB 官方仓库](https://github.com/VectifyAI/OpenKB)
- [OpenKB PyPI 页面](https://pypi.org/project/openkb/)
- [LiteLLM 模型提供商文档](https://docs.litellm.ai/docs/providers)
- [PageIndex](https://github.com/VectifyAI/PageIndex)
