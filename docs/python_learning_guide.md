# Python 学习对照文档

本文档结合 `mini-coding-agent` 项目中的实际代码，帮助学习 Python 核心知识点。每个知识点都对应项目中的真实代码示例，配合行号方便对照阅读。

---

## 目录

1. [模块导入 (Module Imports)](#1-模块导入-module-imports)
2. [数据类型 (Data Types)](#2-数据类型-data-types)
3. [字符串操作 (String Operations)](#3-字符串操作-string-operations)
4. [控制流 (Control Flow)](#4-控制流-control-flow)
5. [函数定义 (Functions)](#5-函数定义-functions)
6. [面向对象编程 (OOP)](#6-面向对象编程-oop)
7. [异常处理 (Exception Handling)](#7-异常处理-exception-handling)
8. [文件与路径操作 (File I/O with pathlib)](#8-文件与路径操作-file-io-with-pathlib)
9. [子进程管理 (Subprocess)](#9-子进程管理-subprocess)
10. [网络请求 (HTTP with urllib)](#10-网络请求-http-with-urllib)
11. [JSON 处理 (JSON)](#11-json-处理-json)
12. [正则表达式 (Regex)](#12-正则表达式-regex)
13. [类方法与静态方法 (@classmethod / @staticmethod)](#13-类方法与静态方法-classmethod--staticmethod)
14. [解包与列表推导式 (Unpacking & Comprehensions)](#14-解包与列表推导式-unpacking--comprehensions)
15. [作用域与嵌套函数 (Scope & Nested Functions)](#15-作用域与嵌套函数-scope--nested-functions)
16. [命令行参数解析 (argparse)](#16-命令行参数解析-argparse)
17. [环境变量与上下文管理器 (Environment & Context Manager)](#17-环境变量与上下文管理器)

---

## 1. 模块导入 (Module Imports)

Python 使用 `import` 引入标准库或第三方模块。项目演示了两种导入方式。

### 1.1 直接导入整个模块

```python
# mini_coding_agent.py:1-10
import argparse
import json
import os
import re
import shutil
import subprocess
import sys
import urllib.error
import urllib.request
import uuid
```

使用时需要通过 `模块名.` 访问，如 `json.dumps()`、`re.search()`。

### 1.2 从模块中导入特定名称

```python
# mini_coding_agent.py:11-12
from datetime import datetime, timezone
from pathlib import Path
```

使用时直接写名称，不需要前缀，如 `datetime.now()`、`Path("some_dir")`。

### 关键区别

| 方式 | 使用示例 |
|------|----------|
| `import json` | `json.dumps(data)` |
| `from json import dumps` | `dumps(data)` |

---

## 2. 数据类型 (Data Types)

### 2.1 字符串 (str)

```python
# mini_coding_agent.py:15-16
DOC_NAMES = ("AGENTS.md", "README.md", "pyproject.toml", "package.json")
HELP_TEXT = "/help, /memory, /session, /reset, /exit"
```

### 2.2 整数 (int) 与浮点数 (float)

```python
# mini_coding_agent.py:35-36
MAX_TOOL_OUTPUT = 4000
MAX_HISTORY = 12000
```

```python
# mini_coding_agent.py:1059-1060
parser.add_argument("--temperature", type=float, default=0.2, ...)
parser.add_argument("--top-p", type=float, default=0.9, ...)
```

### 2.3 元组 (tuple) — 不可变序列

```python
# mini_coding_agent.py:17-24
WELCOME_ART = (
    "/\\     /\\\\",
    "{  `---'  }",
    "{  O   O  }",
    "~~>  V  <~~",
    "\\\\  \\|/  /",
    "`-----'__",
)
```

元组用 `()` 定义，一旦创建不能修改元素。适合存放固定不变的数据。

### 2.4 集合 (set) — 无序不重复元素

```python
# mini_coding_agent.py:37
IGNORED_PATH_NAMES = {".git", ".mini-coding-agent", "__pycache__", ".pytest_cache", ...}
```

集合用 `{}` 定义，查找速度快，自动去重。常用于成员检测 `x in set`。

### 2.5 字典 (dict) — 键值对映射

```python
# mini_coding_agent.py:336-342
tools = {
    "list_files": {
        "schema": {"path": "str='.'"},
        "risky": False,
        "description": "List files in the workspace.",
        "run": self.tool_list_files,
    },
    ...
}
```

字典是项目中使用最频繁的数据结构，用于存储工具定义、会话数据、模型参数等。

---

## 3. 字符串操作 (String Operations)

### 3.1 f-string 格式化（推荐方式）

```python
# mini_coding_agent.py:59
return text[:limit] + f"\n...[truncated {len(text) - limit} chars]"
```

f-string 是 Python 3.6+ 的语法，在字符串前加 `f`，用 `{}` 嵌入表达式。

```python
# mini_coding_agent.py:131-133
f"- cwd: {self.cwd}",
f"- repo_root: {self.repo_root}",
```

### 3.2 join — 拼接列表/元组为字符串

```python
# mini_coding_agent.py:25-33
HELP_DETAILS = "\n".join([
    "Commands:",
    "/help    Show this help message.",
    "/memory  Show the agent's distilled working memory.",
    ...
])
```

`"\n".join([...])` 用换行符将列表中的字符串连接起来。

### 3.3 split — 拆分字符串

```python
# mini_coding_agent.py:122
recent_commits = [line for line in git(["log", "--oneline", "-5"]).splitlines() if line]
```

`.splitlines()` 按行拆分字符串为列表。

### 3.4 strip / lstrip / rstrip — 去除空白

```python
# mini_coding_agent.py:100
return result.stdout.strip() or fallback
```

```python
# mini_coding_agent.py:183
self.host = host.rstrip("/")
```

`.strip()` 去除两端空白字符，`.rstrip("/")` 去除右端指定字符。

### 3.5 find — 查找子串位置

```python
# mini_coding_agent.py:672
if "<tool>" in raw and ("<final>" not in raw or raw.find("<tool>") < raw.find("<final>")):
```

`.find()` 返回子串首次出现的索引，找不到时返回 `-1`。

### 3.6 replace — 替换子串

```python
# mini_coding_agent.py:63
text = str(text).replace("\n", " ")
```

### 3.7 字符串对齐方法

```python
# mini_coding_agent.py:819
body = "\n".join(f"{number:>4}: {line}" for number, line in enumerate(lines[start - 1:end], start=start))
```

`{number:>4}` 表示右对齐占 4 个字符宽度，用于格式化输出行号。

```python
# mini_coding_agent.py:942
body = middle(f"{label:<9} {value}", size)
```

`{label:<9}` 表示左对齐占 9 个字符宽度。

### 3.8 removeprefix — 去除前缀（Python 3.9+）

```python
# mini_coding_agent.py:120
default_branch = (...).removeprefix("origin/")
```

---

## 4. 控制流 (Control Flow)

### 4.1 if / elif / else 条件判断

```python
# mini_coding_agent.py:55-59
def clip(text, limit=MAX_TOOL_OUTPUT):
    text = str(text)
    if len(text) <= limit:
        return text
    return text[:limit] + f"\n...[truncated {len(text) - limit} chars]"
```

### 4.2 for 循环

```python
# mini_coding_agent.py:106-114
for base in (repo_root, cwd):
    for name in DOC_NAMES:
        path = base / name
        if not path.exists():
            continue
        ...
```

`continue` 跳过本次循环剩余代码，直接进入下一次迭代。

### 4.3 while 循环

```python
# mini_coding_agent.py:509-538
while tool_steps < self.max_steps and attempts < max_attempts:
    attempts += 1
    raw = self.model_client.complete(self.prompt(user_message), self.max_new_tokens)
    kind, payload = self.parse(raw)

    if kind == "tool":
        tool_steps += 1
        ...
        continue

    if kind == "retry":
        ...
        continue

    final = (payload or raw).strip()
    ...
    return final
```

这是 agent 的核心循环：不断让模型输出，解析结果，如果是工具调用则执行，直到得到最终答案或超过步数限制。

---

## 5. 函数定义 (Functions)

### 5.1 基本函数定义

```python
# mini_coding_agent.py:50-51
def now():
    return datetime.now(timezone.utc).isoformat()
```

### 5.2 带默认参数的函数

```python
# mini_coding_agent.py:55
def clip(text, limit=MAX_TOOL_OUTPUT):
```

调用时可以省略 `limit`，将使用默认值 `4000`。

### 5.3 带 `**kwargs` 的函数

```python
# mini_coding_agent.py:314
def from_session(cls, model_client, workspace, session_store, session_id, **kwargs):
    return cls(
        model_client=model_client,
        workspace=workspace,
        session_store=session_store,
        session=session_store.load(session_id),
        **kwargs,   # 将额外的关键字参数传递给构造函数
    )
```

`**kwargs` 收集所有未明确命名的关键字参数为一个字典，常用于参数透传。

### 5.4 嵌套函数（闭包）

```python
# mini_coding_agent.py:90-102
def git(args, fallback=""):        # 嵌套在 build() 内部
    try:
        result = subprocess.run(
            ["git", *args],
            cwd=cwd,                # 使用了外层函数的变量 cwd
            capture_output=True,
            text=True,
            check=True,
            timeout=5,
        )
        return result.stdout.strip() or fallback
    except Exception:
        return fallback
```

嵌套函数可以访问外层函数作用域中的变量，这叫**闭包**。

### 5.5 Lambda 表达式（匿名函数）

```python
# mini_coding_agent.py:164
files = sorted(self.root.glob("*.json"), key=lambda path: path.stat().st_mtime)
```

`lambda path: path.stat().st_mtime` 等价于：

```python
def get_mtime(path):
    return path.stat().st_mtime
```

lambda 适合一次性使用的简单函数。

---

## 6. 面向对象编程 (OOP)

### 6.1 类定义与 `__init__` 构造方法

```python
# mini_coding_agent.py:76-84
class WorkspaceContext:
    def __init__(self, cwd, repo_root, branch, default_branch, status, recent_commits, project_docs):
        self.cwd = cwd
        self.repo_root = repo_root
        self.branch = branch
        self.default_branch = default_branch
        self.status = status
        self.recent_commits = recent_commits
        self.project_docs = project_docs
```

`self` 指代实例本身，`self.xxx = xxx` 设置实例属性。

### 6.2 实例方法

```python
# mini_coding_agent.py:126-141
def text(self):
    commits = "\n".join(f"- {line}" for line in self.recent_commits) or "- none"
    ...
    return "\n".join([...])
```

实例方法的第一个参数必须是 `self`。

### 6.3 类方法 (@classmethod)

```python
# mini_coding_agent.py:86-124
@classmethod
def build(cls, cwd):
    cwd = Path(cwd).resolve()
    ...
    return cls(           # 用 cls 创建并返回新实例
        cwd=str(cwd),
        repo_root=str(repo_root),
        ...
    )
```

`@classmethod` 第一个参数是 `cls`（类本身），常用于工厂方法模式。调用时用 `WorkspaceContext.build(cwd)` 而非 `WorkspaceContext().__init__(...)`。

### 6.4 多态与接口约定

项目中有两个模型客户端类遵循相同的 `complete` 接口：

```python
# OllamaModelClient:188
def complete(self, prompt, max_new_tokens):
    ...

# DeepSeekModelClient:240
def complete(self, prompt, max_new_tokens):
    ...
```

`MiniAgent` 不关心具体用的是哪个客户端，只要它有 `complete(prompt, max_new_tokens)` 方法即可。这叫**鸭子类型**（Duck Typing）。

### 6.5 函数作为值存入字典

```python
# mini_coding_agent.py:341
"run": self.tool_list_files,
```

注意这里没有括号 `()` —— 存的是函数引用，不是调用结果。之后可以通过 `tool["run"](args)` 来调用。

---

## 7. 异常处理 (Exception Handling)

### 7.1 try / except

```python
# mini_coding_agent.py:206-219
try:
    with urllib.request.urlopen(request, timeout=self.timeout) as response:
        data = json.loads(response.read().decode("utf-8"))
except urllib.error.HTTPError as exc:
    body = exc.read().decode("utf-8", errors="replace")
    raise RuntimeError(f"Ollama request failed with HTTP {exc.code}: {body}") from exc
except urllib.error.URLError as exc:
    raise RuntimeError(...) from exc
```

### 7.2 raise — 主动抛出异常

```python
# mini_coding_agent.py:650
raise ValueError("delegate depth exceeded")
```

### 7.3 捕获后返回错误信息（不抛出）

```python
# mini_coding_agent.py:554-561
try:
    self.validate_tool(name, args)
except Exception as exc:
    example = self.tool_example(name)
    message = f"error: invalid arguments for {name}: {exc}"
    if example:
        message += f"\nexample: {example}"
    return message   # 返回错误信息而不是抛异常
```

这里选择将异常转为可读的错误消息返回给模型，适合 agent 场景。

### 7.4 异常的 `from` 链

```python
# mini_coding_agent.py:212
raise RuntimeError(...) from exc
```

`from exc` 保留原始异常的堆栈信息，方便调试。

### 7.5 测试中的异常断言

```python
# tests/test_mini_coding_agent.py:195-196
with pytest.raises(ValueError, match="path escapes workspace"):
    agent.path("../outside.txt")
```

---

## 8. 文件与路径操作 (File I/O with pathlib)

项目使用 `pathlib.Path` 而非 `os.path`，这是现代 Python 推荐的路径操作方式。

### 8.1 创建路径与检查存在性

```python
# mini_coding_agent.py:108-110
path = base / name           # 用 / 拼接路径
if not path.exists():        # 检查是否存在
    continue
```

### 8.2 读取文件内容

```python
# mini_coding_agent.py:114
docs[key] = clip(path.read_text(encoding="utf-8", errors="replace"), 1200)
```

`Path.read_text()` 一行代码完成打开-读取-关闭。

### 8.3 写入文件内容

```python
# mini_coding_agent.py:157
path.write_text(json.dumps(session, indent=2), encoding="utf-8")
```

### 8.4 创建目录

```python
# mini_coding_agent.py:150
self.root.mkdir(parents=True, exist_ok=True)
```

`parents=True` 会递归创建所有不存在的父目录，`exist_ok=True` 目录已存在时不报错。

### 8.5 路径操作常用方法

```python
# mini_coding_agent.py:88
cwd = Path(cwd).resolve()          # 获取绝对路径
```

```python
# mini_coding_agent.py:111
key = str(path.relative_to(repo_root))  # 获取相对路径
```

```python
# mini_coding_agent.py:798-799
if not path.is_dir():
    raise ValueError("path is not a directory")
```

```python
# mini_coding_agent.py:818
lines = path.read_text(encoding="utf-8", errors="replace").splitlines()
```

```python
# mini_coding_agent.py:839
for item in path.rglob("*"):       # 递归遍历所有文件
```

```python
# mini_coding_agent.py:164
self.root.glob("*.json")           # 匹配文件模式
```

```python
# mini_coding_agent.py:165
return files[-1].stem if files else None   # .stem 获取文件名（不含扩展名）
```

---

## 9. 子进程管理 (Subprocess)

项目用 `subprocess.run` 执行 git 命令和 shell 命令。

### 9.1 基本用法

```python
# mini_coding_agent.py:92-99
result = subprocess.run(
    ["git", *args],              # 命令列表（安全，防止 shell 注入）
    cwd=cwd,                     # 工作目录
    capture_output=True,         # 捕获 stdout 和 stderr
    text=True,                   # 返回字符串而非字节
    check=True,                  # 非零退出码时抛异常
    timeout=5,                   # 超时秒数
)
return result.stdout.strip() or fallback
```

### 9.2 执行 shell 命令

```python
# mini_coding_agent.py:857-864
result = subprocess.run(
    command,                     # 字符串命令（需要 shell=True）
    cwd=self.root,
    shell=True,                  # 通过 shell 执行（允许管道等）
    capture_output=True,
    text=True,
    timeout=timeout,
)
```

**注意**：`shell=True` 有安全风险，只在可信输入下使用。

### 9.3 检查命令是否可用

```python
# mini_coding_agent.py:828
if shutil.which("rg"):
    # rg 可用，使用 ripgrep 搜索
```

`shutil.which("命令")` 返回命令的完整路径，找不到返回 `None`。

---

## 10. 网络请求 (HTTP with urllib)

### 10.1 发起 POST 请求

```python
# mini_coding_agent.py:201-206
request = urllib.request.Request(
    self.host + "/api/generate",
    data=json.dumps(payload).encode("utf-8"),   # JSON 编码为字节
    headers={"Content-Type": "application/json"},
    method="POST",
)
```

### 10.2 发送请求并处理响应

```python
# mini_coding_agent.py:207-209
with urllib.request.urlopen(request, timeout=self.timeout) as response:
    data = json.loads(response.read().decode("utf-8"))
```

`urllib` 是标准库，无需安装第三方包。对于更复杂的 HTTP 需求，通常使用 `requests` 库。

### 10.3 错误处理

```python
# mini_coding_agent.py:210-218
except urllib.error.HTTPError as exc:
    body = exc.read().decode("utf-8", errors="replace")
    raise RuntimeError(f"Ollama request failed with HTTP {exc.code}: {body}") from exc
except urllib.error.URLError as exc:
    raise RuntimeError(
        "Could not reach Ollama.\n"
        "Make sure `ollama serve` is running and the model is available.\n"
        f"Host: {self.host}\n"
        f"Model: {self.model}"
    ) from exc
```

`HTTPError` 表示服务器返回了错误状态码，`URLError` 表示网络不可达。

---

## 11. JSON 处理 (JSON)

### 11.1 字典转 JSON 字符串

```python
# mini_coding_agent.py:157
path.write_text(json.dumps(session, indent=2), encoding="utf-8")
```

`indent=2` 使输出格式化，便于阅读。

```python
# mini_coding_agent.py:203
data=json.dumps(payload).encode("utf-8")
```

### 11.2 JSON 字符串转字典

```python
# mini_coding_agent.py:161
return json.loads(self.path(session_id).read_text(encoding="utf-8"))
```

```python
# mini_coding_agent.py:209
data = json.loads(response.read().decode("utf-8"))
```

### 11.3 安全访问字典键

```python
# mini_coding_agent.py:221-223
if data.get("error"):
    raise RuntimeError(f"Ollama error: {data['error']}")
return data.get("response", "")
```

`.get(key, default)` 安全取值，键不存在时返回默认值而不会抛 `KeyError`。

---

## 12. 正则表达式 (Regex)

### 12.1 re.search — 查找匹配

```python
# mini_coding_agent.py:717
match = re.search(r"<tool(?P<attrs>[^>]*)>(?P<body>.*?)</tool>", raw, re.S)
```

- `r"..."` — 原始字符串，`\` 不需要转义
- `(?P<attrs>...)` — 命名捕获组，通过 `match.group("attrs")` 获取
- `[^>]*` — 匹配任意非 `>` 字符
- `.*?` — 非贪婪匹配任意字符
- `re.S` — 让 `.` 也能匹配换行符

### 12.2 re.finditer — 查找所有匹配

```python
# mini_coding_agent.py:741
for match in re.finditer(r"""([A-Za-z_][A-Za-z0-9_]*)\s*=\s*(?:"([^"]*)"|'([^']*)')""", text):
    attrs[match.group(1)] = match.group(2) if match.group(2) is not None else match.group(3)
```

解析 XML 属性如 `name="write_file"` 或 `path='file.py'`。

---

## 13. 类方法与静态方法 (@classmethod / @staticmethod)

### 13.1 @classmethod

```python
# mini_coding_agent.py:86
@classmethod
def build(cls, cwd):
    ...
    return cls(...)
```

接收 `cls` 参数，可以创建并返回类实例。常用于替代构造函数。

### 13.2 @staticmethod

```python
# mini_coding_agent.py:323-330
@staticmethod
def remember(bucket, item, limit):
    if not item:
        return
    if item in bucket:
        bucket.remove(item)
    bucket.append(item)
    del bucket[:-limit]
```

不接收 `self` 或 `cls`，就是一个普通函数放在类里面。适合工具函数。

```python
# mini_coding_agent.py:669-701
@staticmethod
def parse(raw):
    ...
```

`parse` 方法不需要访问实例属性，所以定义为静态方法。

---

## 14. 解包与列表推导式 (Unpacking & Comprehensions)

### 14.1 列表推导式

```python
# mini_coding_agent.py:122
recent_commits = [line for line in git(["log", "--oneline", "-5"]).splitlines() if line]
```

等价于：

```python
recent_commits = []
for line in git(["log", "--oneline", "-5"]).splitlines():
    if line:
        recent_commits.append(line)
```

### 14.2 带条件的推导式

```python
# mini_coding_agent.py:800-802
entries = [
    item for item in sorted(path.iterdir(), key=lambda item: (item.is_file(), item.name.lower()))
    if item.name not in IGNORED_PATH_NAMES
]
```

### 14.3 生成器表达式

```python
# mini_coding_agent.py:389
fields = ", ".join(f"{key}: {value}" for key, value in tool["schema"].items())
```

与列表推导式类似，但用 `()` 而不是 `[]`。生成器惰性计算，更省内存。

### 14.4 星号解包 (* unpacking)

```python
# mini_coding_agent.py:92
["git", *args]
```

`*args` 将列表展开为独立元素。如 `args = ["status", "--short"]` 则变为 `["git", "status", "--short"]`。

```python
# mini_coding_agent.py:780
for candidate in (probe, *probe.parents):
```

### 14.5 多变量赋值

```python
# mini_coding_agent.py:512
kind, payload = self.parse(raw)
```

函数返回元组 `(kind, payload)`，直接解包到两个变量。

---

## 15. 作用域与嵌套函数 (Scope & Nested Functions)

### 15.1 嵌套函数访问外层变量

```python
# mini_coding_agent.py:87-124
@classmethod
def build(cls, cwd):
    cwd = Path(cwd).resolve()

    def git(args, fallback=""):   # 内层函数
        ...
        result = subprocess.run(
            ["git", *args],
            cwd=cwd,              # 使用外层的 cwd 变量
            ...
        )
        ...

    repo_root = Path(git([...], str(cwd))).resolve()   # 调用内层函数
```

### 15.2 嵌套函数作为局部辅助

```python
# mini_coding_agent.py:930-948
def build_welcome(agent, model, backend, host=None):
    ...
    def row(text):
        body = middle(text, width - 4)
        return f"| {body.ljust(width - 4)} |"

    def divider(char="-"):
        return "+" + char * (width - 2) + "+"

    def center(text):
        body = middle(text, inner)
        return f"| {body.center(inner)} |"
    ...
    line = divider("=")
    rows = [center(text) for text in WELCOME_ART]
```

在 `build_welcome` 内部定义了多个小函数（`row`, `divider`, `center`, `cell`, `pair`），这些函数共享外部函数的局部变量（`width`, `inner`, `gap` 等），使代码更模块化。

---

## 16. 命令行参数解析 (argparse)

```python
# mini_coding_agent.py:1028-1061
def build_arg_parser():
    parser = argparse.ArgumentParser(
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
        description="Minimal coding agent for Ollama and DeepSeek models.",
    )
    parser.add_argument("prompt", nargs="*", help="Optional one-shot prompt.")
    parser.add_argument("--cwd", default=".", help="Workspace directory.")
    parser.add_argument(
        "--backend",
        choices=("ollama", "deepseek"),
        default="deepseek",
        help="LLM backend to use.",
    )
    parser.add_argument("--max-steps", type=int, default=6, help="...")
    parser.add_argument("--temperature", type=float, default=0.2, help="...")
    ...
    return parser
```

### 常用参数类型

| 写法 | 说明 |
|------|------|
| `nargs="*"` | 零个或多个位置参数，收集为列表 |
| `type=int` | 自动转换为整数 |
| `type=float` | 自动转换为浮点数 |
| `choices=("a", "b")` | 限制只能选这些值 |
| `default="..."` | 未提供时的默认值 |

### 使用方式

```python
# mini_coding_agent.py:1065
args = build_arg_parser().parse_args(argv)
```

`args` 是一个命名空间对象，通过 `args.cwd`、`args.backend` 等属性访问。

---

## 17. 环境变量与上下文管理器

### 17.1 读取环境变量

```python
# mini_coding_agent.py:978
api_key = os.environ.get("DEEPSEEK_API_KEY", "")
if not api_key:
    raise RuntimeError(
        "DeepSeek API key is required.\n"
        "Set the DEEPSEEK_API_KEY environment variable."
    )
```

### 17.2 上下文管理器 (with 语句)

```python
# mini_coding_agent.py:207-209
with urllib.request.urlopen(request, timeout=self.timeout) as response:
    data = json.loads(response.read().decode("utf-8"))
```

`with` 语句确保资源在使用后自动关闭（如网络连接、文件句柄），即使发生异常也会正确释放。

```python
# mini_coding_agent.py:208
with urllib.request.urlopen(request, timeout=self.timeout) as response:
```

---

## 附录：核心类关系图

```
MiniAgent                          核心 agent 类
├── model_client                   模型客户端（多态）
│   ├── OllamaModelClient          Ollama 后端
│   └── DeepSeekModelClient        DeepSeek 后端
├── workspace (WorkspaceContext)   工作区上下文信息
├── session_store (SessionStore)   会话持久化
├── tools (dict)                   工具注册表
│   ├── list_files
│   ├── read_file
│   ├── search
│   ├── run_shell
│   ├── write_file
│   ├── patch_file
│   └── delegate (条件注册)
└── session (dict)                 会话数据
    ├── history[]                  完整对话记录
    └── memory{}                   工作记忆

WorkspaceContext                   工作区快照
├── cwd / repo_root / branch       基本信息
├── status                         git 状态
├── recent_commits                 最近提交
└── project_docs                   项目文档内容

SessionStore                       会话存储
├── save()                         保存会话到 JSON
├── load()                         加载会话
└── latest()                       获取最新会话
```

---

## 附录：学习方法建议

1. **按照模块文件顺序阅读**：从顶部的导入和常量开始，往下阅读 `WorkspaceContext` → `SessionStore` → `MiniAgent` → 辅助函数 → `main`
2. **结合测试学习**：`tests/test_mini_coding_agent.py` 展示了如何使用 `FakeModelClient` 来测试 agent 行为
3. **动手修改**：尝试添加一个新工具（如 `grep_file`），或修改现有工具的逻辑
4. **运行 lint**：用 `ruff check .` 检查代码风格，理解 Python 代码规范
