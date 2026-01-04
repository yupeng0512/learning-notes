# Agentic Patterns - AI Agent 设计模式原理

> 📅 学习日期：2026-01-04
> 🔗 项目地址：https://github.com/neural-maze/agentic_patterns
> 📦 安装：`pip install agentic-patterns`

## 项目概述

| 项目 | 内容 |
|------|------|
| **名称** | agentic_patterns |
| **类型** | AI Agent 设计模式教程（从零实现） |
| **核心价值** | 不依赖框架，用纯 API 实现 4 种核心代理模式 |
| **目标用户** | 想深入理解 Agent 原理的开发者 |
| **学习价值** | ⭐⭐⭐⭐⭐ 原理级教程，理解 > 使用 |

> 💡 **一句话总结**：不用 LangChain，从零实现 Andrew Ng 定义的 4 种代理模式，彻底理解 Agent 本质。

## 为什么要学这个？

使用 LangChain、CrewAI 等框架开发智能体时，框架封装了底层实现细节。这个项目帮助你：

1. **理解本质**：知其然更知其所以然
2. **调试能力**：遇到问题能定位到原理层面
3. **定制能力**：能根据需求修改或扩展模式
4. **框架选型**：理解原理后能更好地评估不同框架

## 知识地图

```
┌─────────────────────────────────────────────────────────────┐
│                  4 种代理模式架构                            │
├─────────────────────────────────────────────────────────────┤
│  🤔 反思模式 (Reflection)                                    │
│  └── LLM 自我审视 → 迭代改进                                │
├─────────────────────────────────────────────────────────────┤
│  🛠️ 工具模式 (Tool Use)                                     │
│  └── LLM + 外部工具 → 获取真实信息                          │
├─────────────────────────────────────────────────────────────┤
│  🧠 规划模式 (Planning / ReAct)                              │
│  └── 大任务 → 子目标分解 → 逐步执行                         │
├─────────────────────────────────────────────────────────────┤
│  🤝 多代理模式 (Multi-Agent)                                 │
│  └── 多角色协作 → 任务流编排                                │
└─────────────────────────────────────────────────────────────┘
```

## 四种模式详解

### 1. 反思模式 (Reflection)

**核心思想**：让 LLM 审视自己的输出，提出改进建议，迭代优化。

**流程**：
```
用户请求 → 生成初稿 → 自我反思 → 改进 → 再反思 → ... → 最终输出
```

**代码示例**：
```python
from agentic_patterns import ReflectionAgent

agent = ReflectionAgent()
final_response = agent.run(
    user_msg="Generate a Python implementation of Merge Sort",
    generation_system_prompt="You are a Python programmer",
    reflection_system_prompt="You are Andrej Karpathy",
    n_steps=10,  # 迭代次数
)
```

**应用场景**：
- 代码生成与优化
- 文章写作与润色
- 方案设计与改进

**框架对应**：LangChain 的 `LLMChain` + 自定义反思逻辑

---

### 2. 工具模式 (Tool Use)

**核心思想**：LLM 本身无法获取实时信息，通过工具函数连接外部世界。

**流程**：
```
用户请求 → LLM 判断需要工具 → 调用工具 → 获取结果 → 生成回答
```

**代码示例**：
```python
from agentic_patterns.tool_pattern.tool import tool
from agentic_patterns.tool_pattern.tool_agent import ToolAgent

@tool
def fetch_top_hacker_news_stories(top_n: int):
    """获取 Hacker News 热门故事"""
    import requests
    response = requests.get("https://hacker-news.firebaseio.com/v0/topstories.json")
    story_ids = response.json()[:top_n]
    # ... 获取故事详情
    return stories

tool_agent = ToolAgent(tools=[fetch_top_hacker_news_stories])
output = tool_agent.run(user_msg="Tell me top 5 HN stories")
```

**应用场景**：
- 搜索引擎集成
- 数据库查询
- API 调用
- 数学计算

**框架对应**：LangChain 的 `Tool` 和 `Agent`

---

### 3. 规划模式 (Planning / ReAct)

**核心思想**：将复杂任务分解为子目标，按步骤执行，不丢失最终目标。

**ReAct 循环**：
```
Thought（思考）→ Action（行动）→ Observation（观察）→ 循环
```

**代码示例**：
```python
from agentic_patterns.planning_pattern.react_agent import ReactAgent
from agentic_patterns.tool_pattern.tool import tool

@tool
def sum_two_elements(a: int, b: int) -> int:
    """Add two integers"""
    return a + b

@tool
def multiply_two_elements(a: int, b: int) -> int:
    """Multiply two integers"""
    return a * b

agent = ReactAgent(tools=[sum_two_elements, multiply_two_elements])
agent.run(user_msg="Calculate (1234 + 5678) * 5")
```

**执行过程**：
```
Thought: 我需要先计算 1234 + 5678
Action: sum_two_elements(1234, 5678)
Observation: 6912

Thought: 现在我需要将结果乘以 5
Action: multiply_two_elements(6912, 5)
Observation: 34560

Thought: 计算完成
Final Answer: 34560
```

**应用场景**：
- 复杂推理任务
- 多步骤问题解决
- 数学计算
- 研究和分析

**框架对应**：LangChain 的 `ReAct Agent`

---

### 4. 多代理模式 (Multi-Agent)

**核心思想**：不同角色的代理协作完成任务，形成工作流。

**流程**：
```
任务 → Agent1（角色1）→ Agent2（角色2）→ ... → 最终输出
```

**代码示例**：
```python
from agentic_patterns.multiagent_pattern.agent import Agent
from agentic_patterns.multiagent_pattern.crew import Crew

with Crew() as crew:
    agent_1 = Agent(
        name="Poet Agent",
        backstory="You are a well-known poet, skilled in writing emotional poems",
        task_description="Write a poem about the meaning of life",
    )
    agent_2 = Agent(
        name="Translator Agent",
        backstory="You are an expert translator, fluent in Spanish",
        task_description="Translate the poem to Spanish",
    )
    
    # 定义依赖关系（DAG）
    agent_1 >> agent_2

# 可视化工作流
crew.plot()

# 执行
crew.run()
```

**应用场景**：
- 软件开发团队模拟
- 内容创作流水线
- 审批流程自动化
- 辩论和决策系统

**框架对应**：CrewAI、AutoGen

## 核心洞见

### 1. 框架是封装，原理是本质

```
LangChain/CrewAI
      ↓
  封装了这些模式
      ↓
理解原理 → 更好地使用框架
         → 能够定制和扩展
         → 遇到问题能调试
```

### 2. 四种模式的组合使用

```
实际应用 = 反思 + 工具 + 规划 + 多代理
```

单独使用任何一种都有局限，组合使用才能解决复杂问题：

| 场景 | 推荐组合 |
|------|----------|
| 代码生成 | 工具 + 反思 |
| 研究分析 | 规划 + 工具 |
| 内容创作 | 多代理 + 反思 |
| 复杂任务 | 全部组合 |

### 3. ReAct 是规划模式的核心

- **Re**asoning + **Act**ing 的结合
- 让 LLM 边思考边行动，而非一次性输出
- 每一步都有明确的思考过程，便于调试

### 4. 多代理的本质是分工

- 每个代理专注一个角色/任务
- 通过 DAG（有向无环图）定义执行顺序
- 上游代理的输出是下游代理的输入

## 项目结构

```
agentic_patterns/
├── notebooks/                    # Jupyter 教程（推荐入口）
│   ├── reflection_pattern.ipynb  # 反思模式教程
│   ├── tool_pattern.ipynb        # 工具模式教程
│   ├── planning_pattern.ipynb    # 规划模式教程
│   └── multiagent_pattern.ipynb  # 多代理模式教程
├── src/agentic_patterns/         # 源代码实现
│   ├── reflection_pattern/       # 反思模式实现
│   ├── tool_pattern/             # 工具模式实现
│   ├── planning_pattern/         # 规划模式实现
│   └── multiagent_pattern/       # 多代理模式实现
└── .env                          # API 密钥配置
```

## 学习路径

```
Step 1: 观看 YouTube 视频（理解概念）
    ↓
Step 2: 运行 Notebook（动手实践）
    ↓
Step 3: 阅读源代码（深入原理）
    ↓
Step 4: 修改代码实现自己的变体
    ↓
Step 5: 在实际项目中应用
```

## 与框架的对应关系

| 模式 | 原理实现 | LangChain | CrewAI |
|------|----------|-----------|--------|
| 反思 | ReflectionAgent | 自定义 Chain | - |
| 工具 | ToolAgent | Tool + Agent | Tool |
| 规划 | ReactAgent | ReAct Agent | - |
| 多代理 | Crew + Agent | - | Crew + Agent |

## 行动清单

- [ ] 安装项目：`pip install agentic-patterns`
- [ ] 获取 Groq API Key（免费）：https://console.groq.com/
- [ ] 运行 4 个 Notebook，理解每种模式
- [ ] 阅读每种模式的源代码实现
- [ ] 尝试组合多种模式解决一个实际问题
- [ ] 对比 LangChain 的实现，加深理解

## 相关资源

- **GitHub**：https://github.com/neural-maze/agentic_patterns
- **Groq API**：https://console.groq.com/
- **Andrew Ng 的 Agent 课程**：https://www.deeplearning.ai/
- **LangChain 文档**：https://python.langchain.com/
- **CrewAI 文档**：https://docs.crewai.com/

## 思考题

1. 在你的工作中，哪种代理模式最有用？
2. 你会如何组合这些模式来解决一个实际问题？
3. 理解原理后，你对 LangChain 的使用有什么新认识？

---

> 💡 **学习心得**：使用框架可以快速开发，但理解原理才能走得更远。这个项目是连接"会用"和"精通"的桥梁。
