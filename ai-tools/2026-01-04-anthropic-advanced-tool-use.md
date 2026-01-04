# Anthropic Advanced Tool Use 三件套

> 来源：Anthropic 官方博客
> 日期：2026-01-04
> 标签：#MCP #ToolUse #Agent #TokenOptimization

## 概述

Anthropic 针对 Agent 工具调用的三大痛点，推出了 Advanced Tool Use 三件套：

| 功能 | 解决的问题 | 效果 |
|------|-----------|------|
| Tool Search Tool | 工具太多撑爆 context | Token 节省 85% |
| Programmatic Tool Calling | 中间结果淹没上下文 | Token 节省 37% |
| Tool Use Examples | 参数用法猜不准 | 准确率 72% → 90% |

---

## 痛点一：工具定义撑爆 Context

### 传统做法的问题

```
所有 MCP 工具定义全部装弹
    ↓
七八万 token 起步
    ↓
系统 prompt + 对话历史和工具挤同一个 context
    ↓
真正干活之前，context 已经被消耗一大半
```

### 解决方案：Tool Search Tool

**核心思路**：按需加载工具，而非全量装载

```
传统模式：约 77K tokens
新模式：约 8.7K tokens
节省：85%
```

**工具准确率提升**：
- Opus 4: 49% → 74%
- Opus 4.5: 79.5% → 88.1%

### 实现方式

**单个工具延迟加载**：

```json
{
  "tools": [
    // 1. 注册工具搜索工具（约 500 token）
    {
      "type": "tool_search_tool_regex_20251119",
      "name": "tool_search_tool_regex"
    },

    // 2. 其他工具标记为按需加载
    {
      "name": "github.createPullRequest",
      "description": "Create a pull request",
      "input_schema": {...},
      "defer_loading": true  // 关键：延迟加载
    }
    // ... 数百个工具都标记 defer_loading: true
  ]
}
```

**整个 MCP Server 延迟加载**：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "google-drive",
  "default_config": {
    "defer_loading": true  // 整服延迟加载
  },
  "configs": {
    "search_files": {
      "defer_loading": false  // 只保留最常用的工具
    }
  }
}
```

---

## 痛点二：中间结果淹没上下文

### 传统做法的问题

以"找出 Q3 差旅超标的同事"为例：

```
1. 查询成员（20 人）
2. 对 20 人逐个查 Q3 报销记录 → 20 次工具调用
3. 查不同 level 对应的预算
4. 上百条记录原封不动进上下文
5. 模型自己算总和、对比预算、筛出超标
```

**问题**：
- Token 花在"中间垃圾数据"上
- 延迟高，每步都要模型推理
- 逻辑全靠模型"自由发挥"，出错概率高

### 解决方案：Programmatic Tool Calling (PTC)

**核心思路**：

> 让模型写代码来编排工具调用，工具结果先进代码执行环境，
> 在那儿完成循环、条件、聚合，最后只把"结论"送回上下文。

**模型生成的代码示例**：

```python
team = await get_team_members("engineering")

# 获取每个职级的预算
levels = list(set(m["level"] for m in team))
budget_results = await asyncio.gather(*[
    get_budget_by_level(level) for level in levels
])
budgets = {level: budget for level, budget in zip(levels, budget_results)}

# 并行获取所有人的费用
expenses = await asyncio.gather(*[
    get_expenses(m["id"], "Q3") for m in team
])

# 找出超标的人
exceeded = []
for member, exp in zip(team, expenses):
    budget = budgets[member["level"]]
    total = sum(e["amount"] for e in exp)
    if total > budget["travel_limit"]:
        exceeded.append({
            "name": member["name"],
            "spent": total,
            "limit": budget["travel_limit"]
        })

print(json.dumps(exceeded))  # 只有这几条结果回到上下文
```

**执行流程**：

```
Claude 提交 Python 代码
    ↓
代码运行时触发工具调用（如 get_expenses）
    ↓
工具结果返回给代码环境（不是 Claude 上下文）
    ↓
代码继续执行，最后 print 结果
    ↓
只有最终结果回到 Claude 上下文
```

**效果**：
- Token 消耗：43,588 → 27,297（节省 37%）
- GIA 基准：46.5% → 51.2%

### 实现方式

```json
{
  "tools": [
    // 1. 注册代码执行工具
    {
      "type": "code_execution_20250825",
      "name": "code_execution"
    },

    // 2. 业务工具声明允许被代码调用
    {
      "name": "get_team_members",
      "description": "Get all members of a department...",
      "input_schema": {...},
      "allowed_callers": ["code_execution_20250825"]  // 关键
    },
    {
      "name": "get_expenses",
      ...
    }
  ]
}
```

**适用场景**：
- 需要并行跑很多工具（如 50 个 endpoint 健康检查）
- 中间数据量巨大，但只要聚合结果
- 不希望中间过程影响模型"思考方向"

---

## 痛点三：参数用法猜不准

### 传统做法的问题

JSON Schema 只能约束"格式对不对"，无法表达"怎么用"：

```json
{
  "name": "create_ticket",
  "input_schema": {
    "properties": {
      "due_date": {"type": "string"},      // 什么格式？
      "reporter": {
        "properties": {
          "id": {"type": "string"}         // USR-12345 还是裸数字？
        }
      },
      "priority": {"enum": ["low", "medium", "high", "critical"]},
      "escalation": {...}                  // 什么时候要填？
    }
  }
}
```

**模型常犯的错误**：
- `due_date` 格式不确定
- `reporter.id` 格式不确定
- 字段组合逻辑不清楚

### 解决方案：Tool Use Examples

**核心思路**：用真实调用示例教模型"正确姿势"

```json
{
  "name": "create_ticket",
  "input_schema": {...},
  "input_examples": [
    // 示例 1：紧急 Bug
    {
      "title": "Login page returns 500 error",
      "priority": "critical",
      "labels": ["bug", "authentication", "production"],
      "reporter": {
        "id": "USR-12345",           // 学到：ID 格式是 USR-XXXXX
        "name": "Jane Smith",
        "contact": {
          "email": "jane@acme.com",
          "phone": "+1-555-0123"
        }
      },
      "due_date": "2024-11-06",      // 学到：日期格式是 YYYY-MM-DD
      "escalation": {
        "level": 2,
        "notify_manager": true,
        "sla_hours": 4               // 学到：critical 要配 escalation
      }
    },

    // 示例 2：功能请求（部分字段）
    {
      "title": "Add dark mode support",
      "labels": ["feature-request", "ui"],
      "reporter": {
        "id": "USR-67890",
        "name": "Alex Chen"          // 学到：feature 可以不填 contact
      }
    },

    // 示例 3：最简形式
    {
      "title": "Update API documentation"  // 学到：只有 title 是必填
    }
  ]
}
```

**模型从示例中学到**：
- 日期统一用 `YYYY-MM-DD`
- 用户 ID 形如 `USR-XXXXX`
- 标签用 `kebab-case`
- 严重故障带 `escalation` 和紧凑 SLA
- feature 请求可以只填部分字段

**效果**：复杂参数处理准确率 72% → 90%

---

## 使用决策指南

### 什么时候开启？

| 功能 | 适用场景 |
|------|----------|
| **Tool Search Tool** | 工具总数 > 10；工具定义 > 10K tokens；多 MCP server |
| **Programmatic Tool Calling** | 多步骤依赖复杂；数据量大但只需聚合结果；适合并行和重试 |
| **Tool Use Examples** | Schema 复杂嵌套多；可选参数多；有明显业务约定 |

### 整体配置示例

```python
client.beta.messages.create(
    betas=["advanced-tool-use-2025-11-20"],
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    tools=[
        # 工具搜索
        {
            "type": "tool_search_tool_regex_20251119",
            "name": "tool_search_tool_regex"
        },
        # 代码执行
        {
            "type": "code_execution_20250825",
            "name": "code_execution"
        },
        # 业务工具（带 defer_loading、allowed_callers、input_examples）
        ...
    ]
)
```

---

## 核心洞见

### 设计哲学

> 真正强大的 Agent，不是"参数多、schema 花哨"，
> 而是能在工具巨多、数据巨大、约定巨多的情况下，依然做到：
> - 找到对的工具
> - 用对的参数
> - 在对的地方看结果

### 三件套定位

| 功能 | 解决的本质问题 |
|------|---------------|
| Tool Search Tool | **规模问题**：工具太多怎么找 |
| Programmatic Tool Calling | **编排与效率问题**：复杂流程怎么跑 |
| Tool Use Examples | **语义与约定问题**：参数怎么填 |

### 工程思维

> 让 Agent 更像一个靠谱的工程同事：
> - 该查文档时查文档
> - 该写脚本时写脚本
> - 该少说废话时，老老实实给结果

---

## 官方资源

- [Tool Search Tool 文档](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/tool-search-tool)
- [Tool Search + 向量检索示例](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/tool_search_tool.ipynb)
- [Programmatic Tool Calling 文档](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/programmatic-tool-calling)
- [PTC 示例 notebook](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/programmatic_tool_calling.ipynb)
- [Tool Use Examples 使用说明](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/tool-use-examples)

---

## 与其他技术的关联

| 技术 | 关系 |
|------|------|
| MCP | Tool Search Tool 直接支持 MCP Server 级别的延迟加载 |
| Agentic Patterns | PTC 本质是 Tool Use Pattern 的代码化实现 |
| LangChain/CrewAI | 这些框架未来可能会集成类似能力 |
