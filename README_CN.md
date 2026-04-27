# Mini-Coding-Agent

一个极简的本地编码代理，仅依赖 Python 标准库，支持 **Ollama** 和 **DeepSeek V4** 两种后端。

- 代码：`mini_coding_agent.py`（单文件，约 40KB）
- CLI：`mini-coding-agent`

## 功能概览

- 自动收集工作区上下文（git 状态、文件布局、项目文档）
- 结构化提示词前缀 + 可复用的会话状态
- 7 种内置工具：文件读写、patch、搜索、Shell 执行、受限子代理委派
- 危险操作三级审批策略（ask / auto / never）
- 完整转写记录 + 精简工作记忆，支持断点续聊
- 上下文裁剪与去重，防止 prompt 膨胀
- 交互式 REPL，内置斜杠命令

<a href="https://magazine.sebastianraschka.com/p/components-of-a-coding-agent">
  <img src="https://substack-post-media.s3.amazonaws.com/public/images/49b97718-57f4-4977-99c8-8ad5c4d32af3_1548x862.png" width="500px">
</a>

<br>

**[详细教程：Components of a Coding Agent](https://magazine.sebastianraschka.com/p/components-of-a-coding-agent)**

## 核心组件

| # | 组件 | 说明 |
|---|------|------|
| 1 | **Live Repo Context** | 自动收集工作区事实：git 分支、状态、最近提交、项目文档 |
| 2 | **Prompt Shape & Cache Reuse** | 稳定的 prompt 前缀与变化的请求/转写/记忆分离，便于模型缓存 |
| 3 | **Structured Tools & Permissions** | 模型通过命名工具操作，含输入校验、路径验证和审批门控 |
| 4 | **Context Reduction** | 长输出裁剪、重复读取去重、旧转写压缩，控制 prompt 大小 |
| 5 | **Transcripts & Memory** | 持久化完整转写 + 精简工作记忆，支持会话恢复 |
| 6 | **Bounded Delegation** | 受限子代理可继承上下文执行只读子任务 |

## 安装

**前提条件：**

- Python 3.10+
- 选择以下任一后端：
  - **Ollama**：安装 Ollama 并拉取模型（如 `qwen3.5:4b`）
  - **DeepSeek API**：注册并获取 API Key

```bash
git clone https://github.com/rasbt/mini-coding-agent.git
cd mini-coding-agent
```

本项目零 Python 运行时依赖（仅标准库），可直接运行。可选使用 `uv` 管理环境。

## 快速开始

### 使用 Ollama 后端

```bash
# 确保 Ollama 服务已启动
ollama serve
# 已拉取默认模型 qwen3.5:4b

python3 mini_coding_agent.py
```

### 使用 DeepSeek V4 后端

```bash
# 方式一：通过环境变量设置 API Key
export DEEPSEEK_API_KEY=sk-xxx
python3 mini_coding_agent.py --backend deepseek

# 方式二：通过命令行参数
python3 mini_coding_agent.py --backend deepseek --deepseek-api-key sk-xxx
```

DeepSeek V4 提供两个模型：
- `deepseek-v4-pro`（默认，完整能力）
- `deepseek-v4-flash`（轻量快速）

切换模型：
```bash
python3 mini_coding_agent.py --backend deepseek --model deepseek-v4-flash
```

## 审批模式

对 Shell 命令和文件写入等危险操作进行审批管控：

- `--approval ask`（默认）：每次危险操作前等待确认
- `--approval auto`：自动批准所有危险操作
- `--approval never`：拒绝所有危险操作

## CLI 参数一览

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--backend` | `ollama` | 后端选择：`ollama` 或 `deepseek` |
| `--model` | 根据后端自动选择 | Ollama: `qwen3.5:4b`；DeepSeek: `deepseek-v4-pro` |
| `--cwd` | `.` | 工作目录 |
| `--host` | `http://127.0.0.1:11434` | Ollama 服务地址 |
| `--ollama-timeout` | `300` | Ollama 请求超时（秒） |
| `--deepseek-api-key` | 无 | DeepSeek API Key（或设 `DEEPSEEK_API_KEY` 环境变量） |
| `--deepseek-base-url` | `https://api.deepseek.com` | DeepSeek API 地址 |
| `--deepseek-timeout` | `300` | DeepSeek 请求超时（秒） |
| `--resume` | 无 | 恢复会话，支持 `latest` 或指定 ID |
| `--approval` | `ask` | 审批策略：`ask` / `auto` / `never` |
| `--max-steps` | `6` | 单轮最大工具调用次数 |
| `--max-new-tokens` | `512` | 单步最大输出 token 数 |
| `--temperature` | `0.2` | 采样温度 |
| `--top-p` | `0.9` | 核采样参数 |

## 内置工具

| 工具 | 风险 | 说明 |
|------|------|------|
| `list_files` | 安全 | 列出目录文件 |
| `read_file` | 安全 | 读取文件（按行范围） |
| `search` | 安全 | 全文搜索（优先 rg，无则内置搜索） |
| `delegate` | 安全 | 委派给受限只读子代理 |
| `run_shell` | 需审批 | 执行 Shell 命令 |
| `write_file` | 需审批 | 写入文件 |
| `patch_file` | 需审批 | 精确替换文件中的文本块 |

## 交互式命令

在 REPL 中可用以下斜杠命令：

- `/help` — 显示帮助
- `/memory` — 显示工作记忆（当前任务、跟踪文件、备注）
- `/session` — 显示当前会话文件路径
- `/reset` — 清空当前会话历史
- `/exit` / `/quit` — 退出

## 会话恢复

会话自动保存在工作区 `.mini-coding-agent/sessions/` 目录下：

```bash
python3 mini_coding_agent.py --resume latest
python3 mini_coding_agent.py --resume 20260401-144025-2dd0aa
```

## 输出格式

模型需输出以下两种格式之一：

- `<tool>{"name":"工具名","args":{...}}</tool>` 或 XML 风格
- `<final>最终回答</final>`

如模型不遵循格式，可尝试更强指令的模型。

## 项目定位

本项目专注于**可读性和教学性**，而非生产级鲁棒性。代码仅一个文件，适合用来：

- 理解 Coding Agent 的内部工作原理
- 学习六大核心组件的设计模式
- 作为个人本地编码代理使用
