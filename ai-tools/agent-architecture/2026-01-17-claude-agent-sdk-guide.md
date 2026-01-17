# Claude Agent SDK：把 Claude Code 能力嵌入你的应用

> 精读日期：2026-01-17
> 原文来源：Nader Dabit 推文（320万浏览，1万点赞）
> 官方仓库：https://github.com/anthropics/claude-agent-sdk-python
> Claude Code：https://github.com/anthropics/claude-code（57.4k stars）

## 一、核心定位

**一句话概括**：

> **Claude Agent SDK = 把 Claude Code 的能力开放成可编程的库**

### 1.1 Claude Code 是什么？

Claude Code 是 Anthropic 推出的**终端 AI 编程助手**（57.4k GitHub Stars），核心能力：

| 能力 | 说明 |
|------|------|
| 读文件 | 理解整个代码库上下文 |
| 跑命令 | 执行 Shell/Git 操作 |
| 编辑代码 | 精确修改文件 |
| 任务规划 | 自主分解和执行多步骤任务 |

**以前**：只能在终端里用
**现在**：通过 SDK 在 Python/TypeScript 项目中调用

### 1.2 与直接调 API 的本质区别

```
┌─────────────────────────────────────────────────────────────┐
│                    调用方式对比                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  直接调 Anthropic API                                       │
│  ─────────────────────                                      │
│  ┌──────┐    prompt    ┌───────┐    response   ┌──────┐    │
│  │ You  │ ──────────▶  │ Claude │ ──────────▶  │ You  │    │
│  └──────┘              └───────┘               └──────┘    │
│       │                                            │        │
│       └──── 你自己写循环处理 tool_use ─────────────┘        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  用 Claude Agent SDK                                        │
│  ──────────────────                                         │
│  ┌──────┐    task     ┌───────────────────────────────┐    │
│  │ You  │ ─────────▶  │       Claude Agent            │    │
│  └──────┘             │  ┌─────────────────────────┐  │    │
│       ▲               │  │  自动管理工具调用循环     │  │    │
│       │               │  │  ┌──────┐  ┌──────┐     │  │    │
│       │               │  │  │ Read │──│ Edit │──…  │  │    │
│       │               │  │  └──────┘  └──────┘     │  │    │
│       │               │  └─────────────────────────┘  │    │
│       │               └───────────────────────────────┘    │
│       │                              │                      │
│       └──────────── 最终结果 ────────┘                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**代码对比**：

```python
# ❌ 直接调 API：你得自己管理工具循环
response = await client.messages.create(...)
while response.stop_reason == "tool_use":
    result = your_tool_executor(response.tool_use)
    response = await client.messages.create(tool_result=result, ...)

# ✅ 用 SDK：Claude 自己处理
async for message in query(prompt="Fix the bug in auth.py"):
    print(message)
```

---

## 二、内置 9 大工具

SDK 封装了 Claude Code 最核心的 9 个工具，**开箱即用**：

| 工具 | 功能 | 典型用途 |
|------|------|----------|
| **Read** | 读取工作目录中的任意文件 | 理解代码上下文 |
| **Write** | 创建新文件 | 生成配置、代码文件 |
| **Edit** | 精确编辑现有文件 | 修改代码（非全量覆盖） |
| **Bash** | 运行终端命令 | 执行脚本、Git 操作 |
| **Glob** | 按模式查找文件 | `**/*.ts` 找所有 TS 文件 |
| **Grep** | 正则搜索文件内容 | 查找函数定义、引用 |
| **WebSearch** | 搜索网络 | 获取最新信息 |
| **WebFetch** | 获取网页内容 | 爬取文档、API |
| **AskUserQuestion** | 向用户提问 | 澄清模糊需求 |

**核心价值**：
- 不用自己实现文件系统操作
- 不用处理命令执行的安全问题
- 不用自己写爬虫
- Claude 知道**何时用什么工具**

---

## 三、快速上手

### 3.1 安装

```bash
# Python SDK
pip install claude-agent-sdk

# 设置 API 密钥
export ANTHROPIC_API_KEY=your-api-key
```

> 注：CLI 已自动随包捆绑，无需单独安装 Claude Code

### 3.2 最简示例

```python
import anyio
from claude_agent_sdk import query

async def main():
    # 让 Agent 列出当前目录文件
    async for message in query(
        prompt="What files are in this directory?",
        options={"allowed_tools": ["Bash", "Glob"]}
    ):
        print(message)

anyio.run(main)
```

### 3.3 实战示例：代码审查 Agent

```python
from claude_agent_sdk import query, ClaudeAgentOptions

async def code_review():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep"],
        permission_mode="bypassPermissions"  # 只读操作，跳过确认
    )
    
    async for message in query(
        prompt="""Review this codebase for:
        - Bugs
        - Security issues  
        - Code quality problems
        Return a structured report.""",
        options=options
    ):
        if hasattr(message, 'result'):
            print(message.result)
```

**这个 Agent 会自动**：
1. 用 Glob 找到所有代码文件
2. 用 Read 读取文件内容
3. 用 Grep 搜索潜在问题模式
4. 生成结构化审查报告

---

## 四、高级特性

### 4.1 自定义工具（进程内 MCP 服务器）

可以定义自己的工具，无需外部进程：

```python
from claude_agent_sdk import tool, create_sdk_mcp_server, ClaudeAgentOptions, ClaudeSDKClient

# 使用装饰器定义工具
@tool("greet", "Greet a user", {"name": str})
async def greet_user(args):
    return {
        "content": [{"type": "text", "text": f"Hello, {args['name']}!"}]
    }

# 创建 MCP 服务器
server = create_sdk_mcp_server(
    name="my-tools",
    version="1.0.0",
    tools=[greet_user]
)

# 配置使用
options = ClaudeAgentOptions(
    mcp_servers={"tools": server},
    allowed_tools=["mcp__tools__greet"]
)

async with ClaudeSDKClient(options=options) as client:
    await client.query("Greet Alice")
    async for msg in client.receive_response():
        print(msg)
```

**进程内 MCP 优势**：
- 无需子进程管理
- 性能更好（无 IPC 开销）
- 部署更简单（单进程）
- 调试更容易
- 类型安全

### 4.2 Hooks 系统

在工具执行前后插入自定义逻辑：

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

# 定义 Hook：检查 Bash 命令安全性
async def check_bash_command(input_data, tool_use_id, context):
    if input_data["tool_name"] != "Bash":
        return {}
    
    command = input_data["tool_input"].get("command", "")
    blocked = ["rm -rf", "sudo", "chmod 777"]
    
    for pattern in blocked:
        if pattern in command:
            return {
                "hookSpecificOutput": {
                    "hookEventName": "PreToolUse",
                    "permissionDecision": "deny",
                    "permissionDecisionReason": f"Blocked: {pattern}",
                }
            }
    return {}

# 应用 Hook
options = ClaudeAgentOptions(
    allowed_tools=["Bash"],
    hooks={
        "PreToolUse": [
            HookMatcher(matcher="Bash", hooks=[check_bash_command]),
        ],
    }
)
```

**Hook 类型**：
- `PreToolUse` - 工具执行前（可拦截/修改）
- `PostToolUse` - 工具执行后（可审计/记录）

### 4.3 子 Agent（Subagents）

定义专门的 Agent 处理特定任务：

```python
# 概念示例（TypeScript 风格）
agents = {
    "security-reviewer": {
        "description": "Security expert for vulnerability analysis",
        "prompt": "Analyze code for security vulnerabilities.",
        "tools": ["Read", "Glob", "Grep"]
    },
    "performance-analyzer": {
        "description": "Performance optimization specialist",
        "prompt": "Find performance bottlenecks.",
        "tools": ["Read", "Grep", "Bash"]
    }
}
```

主 Agent 可以根据任务类型**派遣子 Agent**。

### 4.4 会话管理

支持跨多次交互保持上下文：

```python
async with ClaudeSDKClient(options=options) as client:
    # 第一次交互
    await client.query("Read the auth module")
    async for msg in client.receive_response():
        pass
    
    # 第二次交互（Agent 记得 "它" 指什么）
    await client.query("Find all places that call it")
    async for msg in client.receive_response():
        print(msg)
```

### 4.5 MCP 协议集成

接入外部 MCP 服务器扩展能力：

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "playwright": {
            "command": "npx",
            "args": ["@playwright/mcp@latest"]
        },
        "database": {
            "command": "python",
            "args": ["db_mcp_server.py"]
        }
    }
)
```

---

## 五、架构理解

### 5.1 工作流程

```
┌─────────────────────────────────────────────────────────────┐
│                 Claude Agent SDK 工作流程                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                           │
│  │  Your Code   │                                           │
│  │  query(...)  │                                           │
│  └──────┬───────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   Claude Agent SDK                    │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │                 Agent Loop                      │  │  │
│  │  │                                                 │  │  │
│  │  │   ┌─────────┐    ┌─────────┐    ┌─────────┐   │  │  │
│  │  │   │  Plan   │───▶│  Tool   │───▶│ Reflect │   │  │  │
│  │  │   └─────────┘    └────┬────┘    └────┬────┘   │  │  │
│  │  │        ▲              │              │        │  │  │
│  │  │        └──────────────┴──────────────┘        │  │  │
│  │  │              (循环直到任务完成)                 │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │                          │                            │  │
│  │                          ▼                            │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │              Built-in Tools                    │  │  │
│  │  │  Read │ Write │ Edit │ Bash │ Glob │ Grep...  │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │                          │                            │  │
│  │                          ▼                            │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │           MCP Servers (可选)                   │  │  │
│  │  │    进程内工具 │ Playwright │ Database │ ...    │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────┐                                           │
│  │  File System │  实际的文件读写、命令执行                   │
│  └──────────────┘                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 与 LangChain/LangGraph 对比

| 维度 | Claude Agent SDK | LangChain Agent |
|------|------------------|-----------------|
| **工具** | 内置 9 个开发工具 | 需自己定义 |
| **循环管理** | 自动 | 需配置 |
| **模型绑定** | 仅 Claude | 多模型 |
| **定位** | 开发任务 Agent | 通用 Agent 框架 |
| **学习曲线** | 低 | 中高 |
| **灵活性** | 中 | 高 |

**选择建议**：
- 构建**开发相关** Agent → Claude Agent SDK
- 构建**通用业务** Agent → LangChain/LangGraph
- 两者可以**结合使用**

---

## 六、适用场景分析

### 6.1 适合的场景

| 场景 | 说明 |
|------|------|
| **CI/CD 自动化代码审查** | 每次 PR 自动扫描问题 |
| **自定义开发辅助工具** | 团队专属的代码助手 |
| **文件系统操作的 AI 应用** | 自动化文档处理、代码生成 |
| **企业内部自动化脚本** | 替代复杂的 Bash 脚本 |
| **代码迁移/重构工具** | 批量修改代码模式 |

### 6.2 不适合的场景

| 场景 | 原因 | 替代方案 |
|------|------|----------|
| 简单问答 | 杀鸡用牛刀 | 直接调 API |
| 极低延迟要求 | 多轮工具调用增加延时 | 单次 API 调用 |
| 非开发任务 | SDK 工具聚焦开发 | 通用 Agent 框架 |
| 多模型支持 | 仅支持 Claude | LangChain |

---

## 七、最佳实践

### 7.1 权限控制

```python
# 生产环境：最小权限原则
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep"],  # 只给只读工具
    permission_mode="default"  # 需要用户确认危险操作
)

# 开发/测试环境
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Write", "Edit", "Bash"],
    permission_mode="acceptEdits"  # 自动接受文件编辑
)
```

### 7.2 错误处理

```python
from claude_agent_sdk import (
    ClaudeSDKError,
    CLINotFoundError,
    CLIConnectionError,
    ProcessError,
)

try:
    async for message in query(prompt="..."):
        pass
except CLINotFoundError:
    print("Claude Code CLI 未安装")
except ProcessError as e:
    print(f"进程失败，退出码: {e.exit_code}")
except ClaudeSDKError as e:
    print(f"SDK 错误: {e}")
```

### 7.3 工作目录隔离

```python
from pathlib import Path

# 限制 Agent 只能操作特定目录
options = ClaudeAgentOptions(
    cwd=Path("/safe/sandbox/directory"),
    allowed_tools=["Read", "Write", "Edit"]
)
```

---

## 八、与你探索方向的关联

作为 AI Agent 方向的探索者，Claude Agent SDK 是一个**重要的参考实现**：

### 8.1 核心洞察

1. **工具封装的艺术**
   - 9 个工具覆盖 80% 开发场景
   - 启发：好的 Agent 不需要无限工具，需要**精选的高质量工具**

2. **循环管理的价值**
   - SDK 最大价值是自动管理工具调用循环
   - 启发：Agent 框架的核心是**决策-执行-反思循环**

3. **MCP 协议的威力**
   - 进程内 MCP + 外部 MCP 双模式
   - 启发：可扩展性通过**标准协议**实现

### 8.2 与已学项目的关系

| 项目 | 与 Claude Agent SDK 的关系 |
|------|---------------------------|
| **Memvid** | 可作为 Agent 的记忆后端 |
| **Claude-Mem** | 互补：SDK 负责执行，Mem 负责记忆 |
| **LangGraph** | 竞品/互补：LangGraph 更通用，SDK 更开箱即用 |
| **MCP 协议** | SDK 是 MCP 协议的消费者 |

### 8.3 可实践的下一步

1. **快速体验**：用 SDK 构建一个代码审查 Agent
2. **深入探索**：尝试自定义工具和 Hooks
3. **整合实践**：将 SDK 与 Memvid 结合，构建有记忆的开发助手

---

## 九、关键代码模式

### 9.1 流式处理消息

```python
from claude_agent_sdk import query, AssistantMessage, TextBlock, ToolUseBlock

async for message in query(prompt="..."):
    if isinstance(message, AssistantMessage):
        for block in message.content:
            if isinstance(block, TextBlock):
                print(f"Claude: {block.text}")
            elif isinstance(block, ToolUseBlock):
                print(f"使用工具: {block.name}")
```

### 9.2 完整的审计日志

```python
async def audit_logger(input_data, tool_use_id, context):
    log_entry = {
        "timestamp": datetime.now().isoformat(),
        "tool": input_data["tool_name"],
        "input": input_data["tool_input"],
        "session": context.get("session_id")
    }
    with open("agent_audit.log", "a") as f:
        f.write(json.dumps(log_entry) + "\n")
    return {}

options = ClaudeAgentOptions(
    hooks={
        "PostToolUse": [
            HookMatcher(matcher=".*", hooks=[audit_logger]),
        ],
    }
)
```

---

## 十、总结

### 10.1 一句话评价

> **Claude Agent SDK 是把 Claude Code 的「靠谱工程师」能力 API 化的产物**

### 10.2 核心价值

| 价值 | 说明 |
|------|------|
| **降低门槛** | 不用自己实现工具循环和文件操作 |
| **开箱即用** | 9 个内置工具覆盖开发常见场景 |
| **可扩展** | MCP 协议支持无限扩展 |
| **可控制** | Hooks + 权限模式保证安全 |

### 10.3 Nader 的观点是否成立？

> "用 Claude Agent SDK 来构建自己的 Agent，会是未来 AI 开发中最有用的技能之一"

**评估**：
- ✅ 方向正确：Agent 开发确实是重要趋势
- ⚠️ 有点夸张：SDK 是工具之一，不是唯一
- ✅ 值得学习：掌握 SDK 确实能快速构建有用的 Agent

---

## 参考资源

- GitHub（Python SDK）：https://github.com/anthropics/claude-agent-sdk-python
- Claude Code：https://github.com/anthropics/claude-code
- Nader 原推文：https://x.com/dabit3/status/2009131298250428923
- 官方文档：https://code.claude.com/docs/en/overview
