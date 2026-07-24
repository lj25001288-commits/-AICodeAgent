# AI Code Auditor

> **商业级多模型 Agent 审计工具** — 不止审代码，能 grep 代码库、跑测试、验证 patch、红蓝对抗辩论、自动生成可应用 patch。

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Code Size](https://img.shields.io/badge/code%20size-7300%2B-brightgreen.svg)](#)
[![Quality Score](https://img.shields.io/badge/quality%20score-90.9%2F100-brightgreen.svg)](#)
[![AI Smell](https://img.shields.io/badge/AI%20smell-3.5%2F100-green.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

支持用户自定义配置多个不同家族的模型（如 GPT-4o / Claude / DeepSeek / Qwen / GLM 等），配合 **tree-sitter 多语言 AST + ReAct Agent Loop + 红蓝对抗辩论 + Sandbox 沙箱验证**，把 AI 生成代码的审计做到商业级。

---

## 🎯 核心能力

| 能力                      | 描述                                                                    |
| ----------------------- | --------------------------------------------------------------------- |
| 🔍 **多语言 AST 静态分析**     | tree-sitter 统一支持 Python / JS / TS / Go / Rust / Java / Ruby / C / C++ |
| 🧬 **AI 代码指纹识别**        | 10 维特征向量判断代码来自 Cursor / Copilot / ChatGPT / 人类                        |
| 🤖 **ReAct Agent Loop** | Agent 能主动 grep 代码、读文件、跑测试、查 git log，最多 8 步                            |
| ⚔️ **红蓝对抗辩论**           | 红队 3 模型找 bug（安全/性能/考古 3 视角）vs 蓝队 3 模型反驳（律师/测试/文档 3 视角），法官仲裁           |
| 🧪 **Sandbox 沙箱验证**     | patch 应用到临时副本跑测试，失败自动反馈 LLM 重生成（最多 3 轮）                               |
| 📊 **SWE-bench 风格评估集**  | 内置 21 个金标 bug 样本，跑 benchmark 输出 P/R/F1                                |
| 🔄 **多模型 fallback**     | 主模型挂了自动切备选，单点失败不阻塞                                                    |
| 💾 **函数级缓存**            | SHA256 hash + TTL + 模型签名，二次审计 0 token                                 |
| 🛠️ **git diff 模式**     | 只审 PR 改动 hunks，CI/CD 真实场景                                             |
| 📝 **自动 patch 生成**      | 输出 unified diff，可直接 `git apply`                                       |

---

## 🏗️ 架构图

```
┌──────────────────────────────────────────────────────────────────┐
│  Step 0: tree-sitter 多语言 AST 分析                              │
│  支持 Python/JS/TS/Go/Rust/Java/Ruby/C/C++                       │
│  量化 8 类 AI smell → AI 风格评分 0-100                           │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  Step 0.5: AI 指纹识别                                            │
│  10 维特征向量 → Cursor/Copilot/ChatGPT/Human 概率分布            │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  Step 1: 函数级缓存检查                                           │
│  命中 → 直接返回，0 token 消耗                                    │
└──────────────────────────────────────────────────────────────────┘
                              ↓ 未命中
┌──────────────────────────────────────────────────────────────────┐
│  Round 1: 多模型独立审计                                          │
│  [GPT-4o 拆解] → 检查清单                                         │
│  [Claude 推理] + [DeepSeek-Coder 代码专项] 并发审计                │
└──────────────────────────────────────────────────────────────────┘
                              ↓ (full 模式)
┌──────────────────────────────────────────────────────────────────┐
│  Agent Loop（ReAct 模式）                                         │
│  Agent 自主调用工具：grep_code / read_file / run_tests /          │
│  read_git_log / search_docs / search_similar_cases                │
│  最多 8 步，验证关键 findings                                      │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  红蓝对抗辩论                                                     │
│  红队 3 模型（安全/性能/考古）攻击找 bug                           │
│  蓝队 3 模型（律师/测试/文档）反驳                                 │
│  法官模型仲裁 → confirmed / rejected / needs_review               │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  仲裁成文（Qwen-Max）                                             │
│  交叉验证 → 共识/多重确认/争议/单方 标签                           │
│  生成 Markdown 报告 + unified diff patch                          │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  Sandbox 沙箱验证                                                 │
│  patch 应用到临时副本 → 跑测试 → 失败反馈 LLM 重生成               │
│  通过 → 标记 verified ✅                                          │
└──────────────────────────────────────────────────────────────────┘
```

---

## ✨ 项目特色

- 🔍 **tree-sitter 多语言 AST（10 语言）** — 统一支持 Python / JavaScript / TypeScript / TSX / Go / Rust / Java / Ruby / C / C++
- 🧬 **AI 代码指纹识别** — 10 维特征向量，区分 Cursor / Copilot / ChatGPT / 人类
- 🤖 **ReAct Agent Loop** — Agent 自主 grep 代码、读文件、跑测试、查 git log，最多 8 步
- ⚔️ **红蓝对抗辩论** — 红队 3 视角攻击 vs 蓝队 3 视角反驳，法官模型仲裁
- 🧪 **Sandbox 沙箱验证** — patch 应用到临时副本跑测试，失败自动重生成
- 📊 **双评分体系** — AI 风格评分（越低越好）+ 代码质量评分（越高越好），维度互补
- 📦 **21 个金标评估样本** — 内置 SWE-bench 风格 benchmark，跑 P / R / F1

---

## 📦 安装

### 环境要求

- Python 3.10+
- 任意 OpenAI 兼容的 LLM API（自建中转站 / OpenAI 官方 / Anthropic / DeepSeek / 阿里云）

### 方式 1：源码安装（推荐）

```bash
# 1. 克隆项目
git clone https://github.com/lj25001288-commits/-AICodeAgent.git
cd ai-code-auditor

# 2. 创建虚拟环境（推荐）
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows

# 3. 安装依赖
pip install -r requirements.txt
```

依赖只有 5 个：

- `httpx` — HTTP 客户端
- `PyYAML` — 配置文件解析
- `pydantic` — 结构化输出 schema
- `tree_sitter` — 多语言 AST 解析
- `tree-sitter-language-pack` — 10 种语言的 grammar

**零 LLM 框架依赖**（不用 LangChain / LlamaIndex / AutoGen）。

### 方式 2：Docker 部署

```bash
# 构建镜像
docker build -t ai-code-auditor .

# 运行审计
docker run --rm -v $(pwd):/code ai-code-auditor /code/examples/bad_code.py
```

镜像基于 `python:3.12-slim`，内置全部依赖；通过 volume 挂载源码即可审计，无需在宿主机装任何依赖。CI/CD 流水线直接拉镜像即可使用。

### 方式 3：pip install（未来支持）

```bash
pip install ai-code-auditor
```

> 🚧 计划发布到 PyPI，发布后即可一行命令安装。当前请使用方式 1 或方式 2。

---

## 🔧 配置

复制 `config.example.yaml` 为 `config.yaml`，填入你的中转站信息：

```yaml
# 你的 AI 中转站 base_url（OpenAI 兼容格式）
base_url: "https://your-relay-site.com/v1"
api_key: "sk-xxxxxxxxxxxxxxxxxxxxxxxx"

# 请求超时（秒）
timeout: 180.0

# 并发上限（中转站 QPS 限制较严时调小）
max_concurrency: 4

# 模型角色分配：每个角色可配多候选，主模型挂了自动 fallback
models:
  # Round 1: 拆解
  decomposer:
    - "gpt-4o"

  # Round 1: 逻辑审计（主 Claude，备 GPT-4o）
  logic_auditor:
    - "claude-3-5-sonnet-20241022"
    - "gpt-4o"

  # Round 1: 代码专项（主 DeepSeek-Coder，备 GPT-4o-mini）
  code_specialist:
    - "deepseek-coder"
    - "gpt-4o-mini"

  # Agent Loop 用的模型
  agent:
    - "gpt-4o"

  # 红队 3 个视角
  red_security:      ["gpt-4o"]
  red_performance:   ["claude-3-5-sonnet-20241022"]
  red_archaeologist: ["deepseek-coder"]

  # 蓝队 3 个视角
  blue_devils:       ["claude-3-5-sonnet-20241022"]
  blue_test:         ["deepseek-coder"]
  blue_doc:          ["gpt-4o"]

  # 法官
  judge: ["qwen-max", "gpt-4o"]

  # 仲裁成文
  arbiter: ["qwen-max", "gpt-4o"]

  # Sandbox 重生成 patch
  sandbox: ["gpt-4o"]

# 功能开关（默认全开）
enable_debate: true
enable_agent: true
enable_sandbox: true
enable_fingerprint: true

# 函数级缓存
cache_path: "./.auditor-cache.json"
```

#### 环境变量方式

如果不方便维护 `config.yaml`（例如 CI/CD、容器、临时审计），也可以用环境变量代替：

```bash
# 也可以用环境变量代替 config.yaml
export RELAY_BASE_URL="https://your-relay.com/v1"
export RELAY_API_KEY="sk-xxxxx"
python -m auditor examples/bad_code.py
```

环境变量优先级高于 `config.yaml`，便于在密钥管理系统中注入凭据。

> 支持 OpenAI 兼容协议的任何 API 服务。可配置多个不同家族的模型实现多模型协作。

---

## 🚀 使用

### 三种审计模式

```bash
# Simple 模式：单模型直接审，最快最省 token
python -m auditor examples/bad_code.py --mode simple

# Standard 模式（默认）：多模型 + 辩论
python -m auditor examples/bad_code.py --mode standard

# Full 模式：Agent Loop + 红蓝对抗 + Sandbox 验证，最准最慢
python -m auditor examples/bad_code.py --mode full --verbose
```

### 三种使用场景

```bash
# 1. 审单文件
python -m auditor examples/bad_code.py --out report.md

# 2. 审 git diff 改动（CI/CD 友好）
python -m auditor --diff HEAD~1
python -m auditor --diff main
python -m auditor --diff --cached

# 3. 跑评估集（benchmark）
python -m auditor --evaluate --samples 5 --out eval.md
```

### Patch 操作

```bash
# 审完直接应用 patch 到工作区
python -m auditor app/api.py --apply-patch

# 保存 patch 到目录（不直接应用）
python -m auditor app/api.py --save-patches ./patches/

# 跳过沙箱验证（不推荐，patch 可能破坏代码）
python -m auditor app/api.py --no-sandbox
```

### 省钱组合

```bash
# 跳过红蓝对抗（省 40% token）
python -m auditor main.py --no-debate

# 跳过 Agent Loop（省 30% token）
python -m auditor main.py --no-agent

# 只跑静态分析 + 单模型（最省，仅 1 次模型调用）
python -m auditor main.py --mode simple --no-fingerprint
```

---

## 🔁 CI/CD 集成

AI Code Auditor 可直接接入 GitHub Actions，在 PR 上自动跑审计并把报告作为 artifact / 评论输出。

```yaml
# .github/workflows/audit.yml
name: AI Code Audit
on: [pull_request]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: python -m auditor --diff main --out audit-report.md
        env:
          RELAY_BASE_URL: ${{ secrets.RELAY_BASE_URL }}
          RELAY_API_KEY: ${{ secrets.RELAY_API_KEY }}
```

**步骤说明**：

1. `--diff main` 只审 PR 相对 `main` 分支的改动 hunks，省 token。
2. `--out audit-report.md` 把报告写到文件，可通过 `actions/upload-artifact` 上传或 bot 评论到 PR。
3. 中转站凭据从 GitHub Secrets 注入，避免硬编码。

> 💡 也可以直接用 Docker 镜像：`docker run --rm -v $GITHUB_WORKSPACE:/code -e RELAY_BASE_URL -e RELAY_API_KEY ai-code-auditor --diff main --out /code/audit-report.md`。

---

## 📂 项目结构

```
ai-code-auditor/
├── auditor/                          # 核心代码
│   ├── __init__.py                   # 73 行   — 包入口
│   ├── __main__.py                   # 289 行  — CLI 入口（3 模式 + 评估）
│   ├── client.py                     # 468 行  — 多模型客户端（fallback + reasoning recovery）
│   ├── schemas.py                    # 363 行  — Pydantic 结构化输出 schema
│   ├── tree_sitter_analyzer.py       # 702 行  — 多语言 AST 分析（10 语言）
│   ├── static_analyzer.py            # 548 行  — Python ast 静态分析
│   ├── ai_fingerprint.py             # 422 行  — AI 来源指纹识别
│   ├── tools.py                      # 473 行  — 7 个 Agent 工具（参数容错）
│   ├── agent_loop.py                 # 367 行  — ReAct Agent 主循环
│   ├── adversarial_debate.py         # 686 行  — 红蓝对抗辩论引擎
│   ├── sandbox.py                    # 414 行  — Patch 沙箱验证
│   ├── cache.py                      # 118 行  — 函数级 hash 缓存
│   ├── git_integration.py            # 199 行  — git diff 解析
│   ├── patcher.py                    # 115 行  — unified diff 提取/应用
│   ├── evaluation.py                 # 770 行  — 评估集（21 金标样本）
│   ├── prompts.py                    # 257 行  — v2 prompt（兼容）
│   ├── prompts_v3.py                 # 195 行  — v3 prompt
│   └── pipeline.py                   # 705 行  — 主 orchestrator
├── examples/
│   └── bad_code.py                   # 262 行  — 演示用例
├── config.example.yaml
├── requirements.txt
└── README.md

总核心代码 7316 行，零 LLM 框架依赖。
```

### 🔍 自审结果

本项目用自身审计工具跑过自审，结果如下：

| 指标                        | 数值                     | 方向                    |
| ------------------------- | ---------------------- | --------------------- |
| HIGH 级 smell              | **0** ✅                | 越少越好                  |
| 总 smell 数                 | 77（37 MEDIUM + 40 LOW） | 越少越好                  |
| **AI 风格评分** (tree-sitter) | **3.5 / 100** ✅        | ↓ **越低越好**（越不像 AI 写的） |
| **代码质量评分** (tree-sitter)  | **90.9 / 100** ✅       | ↑ **越高越好**（工程质量越好）    |
| AI 风格评分 (Python ast)      | 8.4 / 100              | ↓ 越低越好                |
| 代码质量评分 (Python ast)       | 89.3 / 100             | ↑ 越高越好                |
| 非 demo unused imports     | **0** ✅                | 越少越好                  |

> ⚠️ **方向说明**：本项目使用**双评分体系**，避免单一评分方向歧义：
> 
> - **AI 风格评分**（0-100，越低越好）：量化 AI smell（异常吞咽、unused imports、过度防御等），分数低 = 代码不像 AI 写的。
> - **代码质量评分**（0-100，越高越好）：量化工程质量（复杂度、嵌套、命名一致性等），分数高 = 工程质量优秀。
> - 两个评分方向相反但维度互补，合在一起才能完整描述代码质量。

剩余的 37 个 MEDIUM 主要是 `nesting_depth 4-5`（LLM 调用 try-except、AST 遍历、JSON 容错解析的合理嵌套，再拆会过度工程化）。40 个 LOW 主要是 `examples/bad_code.py` demo 文件故意塞的 unused imports（不动）。

**经过 3 轮自审 + 修复迭代，AI 风格评分 3.5（几乎无 AI 味），代码质量评分 90.9（优秀），项目已达到可商用标准**。

---

## 📊 输出示例

### 标准输出 Banner

```
========================================================================
  审计完成: bad_code.py  (262 行, 模式: full)
  总耗时: 28.50s
  总调用次数: 12 (失败 0)
  总 token: 输入 27,906 + 输出 12,300 = 40,206
  各模型消耗:
    gpt-4o                                     21,439 tokens
    qwen-max                                   12,145 tokens
    claude-3-5-sonnet                           8,392 tokens
    deepseek-coder                              5,541 tokens
  静态分析 (tree-sitter/python): 262 行, 16 函数, 18 smell 信号, 评分 48.0/100
  AI 指纹识别:
    Cursor     ██████░░░░░░░░░░░░░░ 31.3%
    ChatGPT    ████░░░░░░░░░░░░░░░░ 24.5%
    Copilot    ████░░░░░░░░░░░░░░░░ 22.6%
    Human      ████░░░░░░░░░░░░░░░░ 21.7%
  Agent Loop: 5 步, 8 次工具调用, 8.50s
  红蓝对抗: 1 轮, 红队提出 8 个 finding, 法官 confirmed 5
  Sandbox 验证: 2/3 通过
  提取到 3 个 patch
  ✅ 其中 2 个通过沙箱验证
========================================================================
```

### 评估集输出

```markdown
# 评估结果汇总

- 总样本数: 21
- 平均 Precision: 87.5%
- 平均 Recall:    78.3%
- 平均 F1:        82.6%
- 平均耗时:       4520ms

## 按语言分组

| 语言 | Precision | Recall | F1 | 样本数 |
|------|-----------|--------|----|----|
| python | 88.2% | 80.1% | 84.0% | 17 |
| javascript | 85.0% | 75.0% | 79.7% | 3 |
| go | 87.5% | 75.0% | 80.8% | 1 |

## 按严重度分组

| 严重度 | Recall | 总数 |
|--------|--------|------|
| critical | 92.3% | 13 |
| high | 81.5% | 14 |
| medium | 73.2% | 11 |
| low | 65.0% | 6 |
```

---

## 🧪 评估集

内置 21 个金标 bug 样本，覆盖：

- **Python (17 个)**：SQL 注入、硬编码 key、异常静默吞咽、N+1 查询、路径穿越、命令注入、过度防御、unused imports、命名混用、_pickle 反序列化、docstring 撒谎、同步阻塞、md5/sha1 弱哈希、线程不安全单例、递归无终止、全局变量副作用、资源泄漏、None 解引用
- **JavaScript (3 个)**：eval 注入、innerHTML XSS、callback 嵌套地狱
- **Go (1 个)**：SQL 拼接 + 错误忽略

跑评估：

```bash
python -m auditor --evaluate --samples 0 --concurrency 2 --out eval.md
```

---

## ❓ 常见问题（FAQ）

**Q: reasoning 模型（GLM-5.2 / o1）content 为空怎么办？**
A: 项目已内置 `chat_with_reasoning_recovery`，自动扩容 `max_tokens` 重试，把思考过程腾出空间留给最终回答，无需手动调参。

**Q: 中转站不支持某些模型怎么办？**
A: `config.yaml` 每个角色可配多个候选模型，主模型挂了自动 fallback。例如 `logic_auditor` 配 `[claude-3-5-sonnet, gpt-4o]`，Claude 不支持时自动切 GPT-4o。

**Q: Full 模式太慢怎么办？**
A: 用 Simple 模式（~90s）或 Standard 模式（~2min），Full 模式（含 Agent Loop + 红蓝对抗 + Sandbox）适合 nightly audit / 周末跑批。CI 场景推荐 Standard + `--diff main`。

**Q: 支持哪些语言？**
A: tree-sitter 统一支持 10 种语言：Python / JavaScript / TypeScript / TSX / Go / Rust / Java / Ruby / C / C++。指纹识别和静态分析对 Python 额外做了深度 `ast` 模块支持。

**Q: 单次审计要烧多少 token？**
A: Simple ~5K，Standard ~30K，Full ~80-150K。跑评估集（21 样本）约 100-500 万 token。开启函数级缓存后，二次审计相同函数 0 token。

**Q: 必须用中转站吗？**
A: 不必须。任何 OpenAI 兼容协议（官方 OpenAI / DeepSeek 官方 / Together / OpenRouter / 阿里云 DashScope）都可以，但只有中转站能让你用一个 key 调多家模型——这才是本项目多模型协作的前提。

---

## 🛣️ Roadmap

- [ ] GitHub Action 集成（PR 自动评论 + 自动 patch）
- [ ] VSCode 插件
- [ ] MCP Server 接口（让 Cursor / Cline 直接调用）
- [ ] 跨文件 taint analysis（用户输入 → SQL sink 数据流追踪）
- [ ] 向量化审计经验库（ChromaDB 集成）
- [ ] OpenTelemetry 接入
- [ ] Streaming TUI（rich/textual 实时输出）
- [ ] 自进化规则库（DISAGREE 后的反馈学习）

---

## 📝 License

MIT — 随便用，欢迎 PR。

---

## 🙏 Acknowledgements

- [tree-sitter](https://github.com/tree-sitter/tree-sitter) — 多语言 AST 解析
- [tree-sitter-language-pack](https://github.com/Goldziher/tree-sitter-language-pack) — 10 种语言 grammar 打包
- [pydantic](https://github.com/pydantic/pydantic) — 结构化输出 schema
- [httpx](https://github.com/encode/httpx) — 异步 HTTP 客户端

---

## 💼 关于多模型 API 接入

本项目通过 OpenAI 兼容协议调用 4-10 个不同家族的模型，支持多模型协作审计。

如果你是开发者，手头有多种大模型 API 需求但又不想分别申请 OpenAI / Anthropic / DeepSeek / 阿里云的账号（也不想在代码里维护多套不同的 SDK），找一个靠谱的中转站是最省事的方案。

单次完整审计 token 消耗：

- Simple 模式：~5K
- Standard 模式：~30K
- Full 模式：~80-150K

跑评估集（21 样本）大约消耗 100-500 万 token，是验证"多模型协作是否有用"最直接的方式。
