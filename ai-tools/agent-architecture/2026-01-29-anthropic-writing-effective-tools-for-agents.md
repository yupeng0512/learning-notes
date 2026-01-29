---
title: "Anthropic：为 AI Agent 编写高效工具的实践指南"
source: "https://www.anthropic.com/engineering/writing-tools-for-agents"
author: "Ken Aizawa (Anthropic Applied AI Team)"
date: 2026-01-29
category: "Agent 架构"
tags:
  - Agent
  - MCP
  - Tool Design
  - Evaluation
  - Anthropic
  - Best Practices
---

# Anthropic：为 AI Agent 编写高效工具的实践指南

> **核心洞见**：Agents are only as effective as the tools we give them.

---

## 📋 文章概览

**作者**：Ken Aizawa (Anthropic Applied AI Team)  
**发布时间**：2025-09-11  
**文章类型**：技术博客（实践指南）  
**目标读者**：Agent 开发者、MCP 工具开发者、AI 工程师

**核心价值**：
- 提供为 AI Agent 编写高质量工具的系统方法论
- 展示如何用 Agent 优化 Agent 的工具（自举/元循环）
- 提炼工具设计的核心原则和最佳实践

---

## 🌳 根节点命题

> **工具设计 = 非确定性契约设计 + 评估驱动优化 + 元循环自举**

**为什么这是根节点**：

Anthropic 这篇文章揭示了为 Agent 编写工具的**范式转变**：

1. **非确定性契约**：传统软件是确定性系统之间的契约（`getWeather("NYC")` 总是一样），而工具是**确定性系统与非确定性 Agent 之间的契约**。Agent 可能调用工具、可能不调用、可能用错参数、可能理解错意图。这要求我们重新思考工具设计：不是为程序员写 API，而是为 Agent 写"符合人体工程学"的接口。

2. **评估驱动优化**：好的工具不是凭直觉写出来的，而是通过**系统化评估**迭代出来的。原型 → 评估 → 分析 → 优化 → 再评估，形成闭环。Anthropic 的 Slack、Asana 工具都是通过这个流程，从人工版提升到 Claude 优化版，准确率提升 10-20%。

3. **元循环自举**：最震撼的洞见——**用 Claude 优化 Claude 自己的工具**。Agent 不仅是工具的使用者，也是工具的优化者。Claude 可以分析自己的评估 transcripts，识别工具的问题，然后重构工具代码。这是软件工程的"自举"思想在 Agent 时代的体现。

这三个维度构成了完整的工具设计方法论，缺一不可。

---

## 📐 表示空间（核心维度）

Anthropic 的工具设计方法论可以用**三个正交维度**完整描述：

| 维度 | 含义 | Anthropic 实现 | 关键指标 |
|------|------|---------------|---------|
| **设计维度** | 如何设计符合 Agent 认知的工具 | 5 大原则（选择、命名空间、返回值、token 效率、描述）| 工具调用成功率、理解准确率 |
| **流程维度** | 如何系统化开发和优化工具 | 原型 → 评估 → 分析 → 优化（闭环）| 迭代速度、性能提升幅度 |
| **元认知维度** | 谁来优化工具 | Claude 分析 transcripts 并重构工具 | 自动化程度、优化质量 |

**维度关系**：

```
设计维度（原则）
    ↓ 指导
流程维度（方法）
    ↓ 执行
元认知维度（自举）
    ↓ 反馈
设计维度（改进）→ 闭环
```

**为什么是这三个维度？**

1. **设计维度**：回答"什么是好工具"——必须符合 Agent 的认知方式（非确定性）
2. **流程维度**：回答"如何做出好工具"——系统化方法（不靠直觉）
3. **元认知维度**：回答"谁来优化工具"——Agent 自己（自举）

三者缺一不可：没有设计原则，优化是盲目的；没有流程，原则无法落地；没有元认知，优化效率低下。

---

## 🌲 推论展开（核心结论树）

从根节点"工具设计 = 非确定性契约设计 + 评估驱动优化 + 元循环自举"，推导出 **5 个核心结论**：

---

### 推论 1：工具不是 API 的简单包装 → 需要重新设计

**推导**：
```
传统 API 设计：
  - 面向确定性系统（程序员）
  - 精确的参数定义（类型、约束）
  - 完整的返回值（所有字段）
  
Agent 工具设计：
  - 面向非确定性系统（LLM）
  - "符合人体工程学"的参数（自然语言友好）
  - 高信号返回值（过滤低价值字段）
  
→ 不能直接包装现有 API，必须重新设计
```

**应用场景**：
- ❌ **错误**：`list_contacts()` → 返回所有联系人（Agent 需要遍历）
- ✅ **正确**：`search_contacts(name)` → 只返回匹配的联系人

**实战案例**：
Anthropic 的地址簿工具：
- 传统 API：`GET /contacts` → 返回所有 1000 个联系人
- Agent 工具：`search_contacts(query="Jane")` → 只返回 3 个匹配的联系人

Agent 的"affordance"（可供性）不同于程序：Agent 的上下文是有限的，不能像程序那样高效遍历大列表。

---

### 推论 2：评估是工具优化的核心驱动力 → 必须系统化

**推导**：
```
直觉驱动：
  - 凭经验猜测"好工具"的样子
  - 修改后不知道是否真的变好
  - 容易过拟合到特定案例
  
评估驱动：
  - 生成 50-200 个真实场景任务
  - 量化指标（准确率、token 数、工具调用次数）
  - Held-out test set 防止过拟合
  
→ 评估让优化可测量、可重复
```

**应用场景**：
- **评估任务设计**：基于真实用户场景（不是简单的 sandbox）
- **多样化任务**：从简单（单工具调用）到复杂（20+ 工具调用）
- **验证策略**：精确匹配、LLM 判断、工具调用序列验证

**实战案例**：
Anthropic 内部 Slack/Asana 工具评估：
- 生成任务：分析真实项目、文档、消息
- 验证：Claude 判断响应质量
- 指标：准确率（人工版 vs Claude 优化版提升 10-20%）

**关键洞见**：评估不仅是"测试"，更是**发现工具问题的最佳途径**。

---

### 推论 3：Agent 是最好的工具优化者 → 元循环自举

**推导**：
```
人工优化：
  - 分析 transcripts（费时）
  - 识别模式（容易遗漏）
  - 重构工具（手动修改多个文件）
  
Agent 优化：
  - Claude 分析所有 transcripts
  - 识别工具描述矛盾、参数混淆、返回值冗余
  - 批量重构工具实现和描述
  
→ Agent 优化 Agent 的工具 = 自举
```

**应用场景**：
- 将评估 transcripts 粘贴到 Claude Code
- Claude 分析 Agent 的困惑点、错误调用模式
- Claude 自动重构工具代码和描述
- 重新运行评估，验证提升

**实战案例**：
Anthropic 的网页搜索工具：
- 问题：Claude 总是在 query 后面加 `2025`（如 `"AI agents 2025"`）
- 原因：工具描述不清晰，Agent 误以为需要指定年份
- 优化：Claude 分析 transcripts 后，改进工具描述
- 结果：性能修复

**关键洞见**：Claude 不仅能用工具，还能**优化自己的工具**——这是元循环的力量。

---

### 推论 4：工具设计有 5 大核心原则 → 系统化指南

**推导**：
```
从 Anthropic 的实践中提炼：
  
1. 选择正确的工具（而非包装所有 API）
2. 命名空间设计（避免工具混淆）
3. 返回有意义的上下文（高信号 > 完整性）
4. 优化 token 效率（分页、过滤、截断）
5. Prompt 工程工具描述（清晰、无歧义）

→ 5 大原则覆盖工具设计的全生命周期
```

**应用场景**：

| 原则 | 反例 | 正例 |
|------|------|------|
| **选择工具** | `list_users`, `list_events`, `create_event` | `schedule_event`（合并多步操作）|
| **命名空间** | `search`, `find` | `asana_search`, `jira_search` |
| **返回值** | 返回 `uuid`, `mime_type` | 返回 `name`, `file_type` |
| **Token 效率** | 返回完整 50K 日志 | 分页（`limit=100`）+ 过滤（`level=ERROR`）|
| **描述** | "搜索工具"（模糊）| "搜索 Slack 消息，支持 channel 和日期过滤"（清晰）|

**关键洞见**：这 5 大原则是**相互关联**的——好的工具选择 → 减少命名空间冲突 → 精简返回值 → 节省 tokens → 描述更清晰。

---

### 推论 5：对 Agent "ergonomic" 的工具，对人也直观 → 人机统一

**推导**：
```
传统观念：
  - Agent 工具需要特殊设计
  - 人类 API 和 Agent 工具是两套系统
  
Anthropic 发现：
  - 对 Agent 最"符合人体工程学"的工具
  - 对人类开发者也最直观
  
原因：
  - Agent 的认知方式类似人类
  - 都需要"高信号、易理解、无歧义"
  
→ 好的 Agent 工具 = 好的人类接口
```

**应用场景**：
- **地址簿搜索**：人类也不会逐页翻联系人，而是直接搜索
- **日志查询**：人类也不会读完整日志，而是过滤关键词
- **客户信息**：人类也希望一次性获取"所有相关信息"（交易、备注、历史）

**实战案例**：
Anthropic 的客户上下文工具：
- ❌ 分散：`get_customer_by_id`, `list_transactions`, `list_notes`（需要 3 次调用）
- ✅ 合并：`get_customer_context(customer_id)`（一次性返回所有相关信息）

**关键洞见**：优化 Agent 工具的过程，也是**重新审视人类接口设计**的过程。

---

**推论关系图**：

```
推论 1（设计原则）
    ↓
推论 4（5 大原则）
    ↓
推论 5（人机统一）

推论 2（评估驱动）
    ↓
推论 3（元循环自举）
    ↓
回到推论 1（持续改进）
```

---

## 🔄 泛化模式（可迁移场景）

Anthropic 的工具设计方法论可以迁移到以下场景：

---

### 泛化 1：从 Agent 工具设计 → API 设计重新思考

**核心迁移**：非确定性契约思想 → 面向人类的 API 设计

**具体应用**：
```
传统 REST API 设计：
  GET /users → 返回所有用户（需要客户端分页）
  GET /logs → 返回完整日志（需要客户端过滤）
  
Agent-inspired API 设计：
  GET /users?search=jane → 服务端智能搜索
  GET /logs?level=ERROR&limit=100 → 服务端过滤 + 分页
  
→ 减少客户端复杂度，提升 API 易用性
```

**案例**：
- **GraphQL**：让客户端选择需要的字段（类似 Agent 的 `response_format`）
- **Stripe API**：高度语义化的参数名（`customer_email` 而非 `email`）

**关键洞见**：为 Agent 设计工具的思想，可以**反向优化人类 API 设计**。

---

### 泛化 2：从评估驱动优化 → 产品迭代方法论

**核心迁移**：原型 → 评估 → 分析 → 优化 → 闭环

**具体应用**：
```
产品开发传统流程：
  需求 → 开发 → 上线 → 用户反馈（慢、模糊）
  
Agent-inspired 产品流程：
  原型 → 模拟用户场景（50-200 个）→ 量化指标 → 优化 → 再测试
  
→ 快速迭代、数据驱动
```

**案例**：
- **A/B 测试**：类似 Anthropic 的 held-out test set
- **用户旅程测试**：类似 Agent 评估任务（真实场景 + 验证）
- **性能监控**：类似工具调用指标（耗时、错误率）

**关键洞见**：Agent 评估的思想，可以应用到**任何需要迭代优化的产品**。

---

### 泛化 3：从元循环自举 → 自动化优化系统

**核心迁移**：用 AI 优化 AI 的产出 → 用 AI 优化任意系统

**具体应用**：
```
传统优化：
  人工分析日志 → 识别问题 → 手动修复
  
AI-driven 优化：
  AI 分析日志 → AI 提出优化方案 → AI 实现优化 → 人类审核
  
→ 自动化优化闭环
```

**案例**：
- **代码审查**：Claude Code 分析 PR，提出改进建议
- **文档优化**：AI 分析用户困惑点，自动改进文档
- **性能优化**：AI 分析性能瓶颈，提出优化代码

**关键洞见**：元循环自举不局限于 Agent 工具，可以应用到**任何可量化优化的系统**。

---

### 泛化 4：从 5 大工具原则 → 接口设计通用原则

**核心迁移**：Agent 工具的 5 大原则 → 任意接口设计

**具体应用**：

| Anthropic 原则 | 通用化原则 | 应用场景 |
|---------------|-----------|---------|
| **选择正确的工具** | **功能合并 > 功能拆分** | CLI 命令设计、GUI 功能设计 |
| **命名空间** | **清晰边界 > 扁平结构** | 文件夹组织、模块划分 |
| **高信号返回值** | **相关性 > 完整性** | 搜索结果排序、通知设计 |
| **Token 效率** | **渐进式加载 > 一次性加载** | 网页分页、懒加载 |
| **清晰描述** | **无歧义 > 简洁** | 错误提示、帮助文档 |

**案例**：
- **Git 命令**：`git commit -am "msg"`（合并 add + commit）
- **macOS Spotlight**：智能搜索 > 文件夹浏览
- **Google 搜索**：相关结果 Top 10 > 所有匹配结果

**关键洞见**：Agent 工具设计原则，是**通用接口设计原则**在 AI 时代的体现。

---

## 小结：泛化的本质

Anthropic 的工具设计方法论，本质上是**在非确定性环境下的接口设计**。这个思想可以泛化到：

1. **API 设计**：面向人类的"非确定性"（不同技能水平）
2. **产品迭代**：在"非确定性"用户行为下优化
3. **自动化优化**：用 AI 处理"非确定性"优化任务
4. **接口设计**：在"非确定性"使用场景下保持易用性

→ **非确定性思维**是 Agent 时代的核心能力。

---

## 🔑 关键概念速查

| 概念 | 定义 | 为什么重要 | 实战应用 |
|------|------|-----------|---------|
| **Non-deterministic Contract** | 工具是确定性系统与非确定性 Agent 之间的契约 | 解释为何工具设计不同于传统 API | 设计工具时考虑 Agent 可能的"误解" |
| **Affordance（可供性）** | Agent 感知到的"可能行动"集合，不同于传统程序 | Agent 的上下文有限，不能像程序那样高效遍历 | 避免返回大列表，提供搜索/过滤工具 |
| **Ergonomic for Agents** | 对 Agent 来说"符合人体工程学"的设计 | 好的 Agent 工具对人类也直观 | 工具设计的通用原则 |
| **Evaluation-driven Development** | 基于系统化评估的开发流程 | 让优化可测量、可重复、可验证 | 原型 → 评估 → 优化 → 再评估 |
| **Bootstrapping（自举）** | 用 Agent 优化 Agent 的工具 | 自动化优化闭环，提升效率 | Claude 分析 transcripts 并重构工具 |
| **High-signal Context** | 返回高相关性、低噪音的信息 | Agent 的 attention budget 有限 | 过滤 `uuid`、`mime_type` 等低价值字段 |
| **Response Format Enum** | 工具提供多种返回格式（`concise` / `detailed`）| 让 Agent 控制返回粒度 | Slack 工具：`concise` 返回内容，`detailed` 返回 IDs |
| **Namespacing** | 工具分组（如 `asana_search`, `jira_search`）| 避免工具名冲突和 Agent 混淆 | 前缀/后缀命名规范 |
| **Tool Consolidation** | 合并多个 API 调用到单个工具 | 减少工具数量、降低决策复杂度 | `get_customer_context` 替代 3 个独立工具 |
| **Held-out Test Set** | 评估中保留的测试集（未用于训练/优化）| 防止过拟合，验证泛化能力 | Anthropic 的 Slack/Asana 工具评估 |
| **Transcript Analysis** | 分析 Agent 执行 transcripts 识别模式 | 发现工具问题的最佳途径 | 读取 CoT、工具调用序列、错误模式 |
| **Pagination & Truncation** | 分页和截断大返回值 | 控制 token 消耗 | Claude Code 默认 25K tokens 截断 |
| **Error Response Steering** | 用清晰的错误提示引导 Agent | 让 Agent 自我纠正 | 错误信息包含"如何修复"而非技术栈 trace |
| **Tool Annotations（MCP）** | 标注工具权限（open-world, destructive）| 安全性和透明度 | MCP 协议规范 |

---

### 概念关系图

```
Non-deterministic Contract（根源）
    ↓
Affordance（Agent 视角）
    ↓
Ergonomic for Agents（设计原则）
    ↓
High-signal Context + Namespacing + Consolidation（5 大原则）
    ↓
Evaluation-driven Development（方法论）
    ↓
Transcript Analysis + Held-out Test Set（技术手段）
    ↓
Bootstrapping（元循环）
```

---

## 🤔 推论深度展开（苏格拉底式讲解）

围绕 5 个核心推论，深度讲解关键问题。

---

### 🤔 问题 1：为什么"对 Agent ergonomic 的工具"不能通过简单包装现有 API 实现？LLM 不是号称能理解任何代码吗？

<details>
<summary>💡 点击查看深度解答</summary>

这个问题触及了 **Agent affordance**（可供性）的本质。

**表面原因**：

LLM 确实能"理解"现有 API 的语义，比如看到 `GET /contacts` 会知道这是"获取联系人列表"。但"理解"不等于"高效使用"。

**深层原因——Transformer 的架构限制**：

```
传统程序处理 list_contacts():
  for contact in all_contacts:  # O(n) 时间，O(1) 内存
      if match(contact, query):
          return contact
  → 高效

Agent (Transformer) 处理 list_contacts():
  1. 读取所有 1000 个联系人 → 50K tokens 进入 context
  2. Self-attention 计算 n² relationships → 2.5B pairwise relationships
  3. 从 50K tokens 中搜索匹配
  → Context rot、注意力稀释、成本高昂
```

**关键洞见——affordance 的物理限制**：

- **程序的 affordance**：无限内存（相对）、高效循环、精确匹配
- **Agent 的 affordance**：有限 attention budget、n² 复杂度、概率匹配

**类比人类认知**：

你不会通过"读完整本通讯录"来找联系人，而是：
1. 打开搜索框
2. 输入 "Jane"
3. 看到 3 个匹配结果

这就是 `search_contacts("Jane")` 的设计——符合 Agent（和人类）的认知方式。

**实战启示**：

```
❌ 错误思维："LLM 很智能，给它所有数据，它自己会处理"
✅ 正确思维："LLM 的 affordance 类似人类，需要符合人体工程学的工具"
```

Anthropic 的地址簿案例就是典型：不是给 Agent 1000 个联系人让它遍历，而是提供搜索工具。

**延伸思考**：

这也解释了为何 **RAG（Retrieval-Augmented Generation）** 有效——不是把整个知识库塞进 context，而是先检索相关片段。本质上，RAG 就是为 LLM 设计的"搜索工具"。

</details>

---

### 🤔 问题 2：Anthropic 说"用 Claude 优化 Claude 的工具"——但 Agent 怎么知道自己的工具有问题？它不会"感到困惑"啊？

<details>
<summary>💡 点击查看深度解答</summary>

这是理解**元循环自举**的关键问题。

**误解**：Agent 需要"主动发现"工具问题

**实际**：Agent 通过 **transcript 分析** 揭示问题

**工作原理**：

```
Step 1: 评估阶段
  Agent 执行 50-200 个任务
  → 生成 transcript（包含 CoT、工具调用、结果）
  
Step 2: Transcript 内容
  {
    "task": "安排与 Jane 的会议",
    "reasoning": "我需要先找到 Jane 的 user_id...",
    "tool_calls": [
      {"tool": "list_users", "params": {}},  ← 调用了错误的工具
      {"tool": "search_user", "params": {"name": "Jane"}},  ← 纠正
      ...
    ],
    "errors": ["Tool 'list_users' returned 5000 users, context overflow"],
    "outcome": "success (but inefficient)"
  }
  
Step 3: Claude 分析 transcripts
  输入：所有 transcripts（可能 100K+ tokens）
  Claude 识别模式：
    - "list_users 被频繁误用"
    - "Agent 先调用 list_users，再调用 search_user"
    - "描述不清晰导致混淆"
  
Step 4: Claude 提出优化
  - 移除 list_users 工具（容易误用）
  - 增强 search_user 描述："直接搜索用户，无需先列出所有用户"
  - 修改参数名：user_name → search_query（更直观）
```

**关键洞见——Agent 不需要"感到困惑"**：

- Agent 的"困惑"表现在 **transcript 中**：
  - 重复调用工具
  - 调用错误的工具
  - 冗长的 reasoning
  - 工具错误（如 context overflow）

- **另一个 Claude** 分析这些 transcripts，识别模式

**为什么这是"元循环"？**

```
Claude (Agent) → 使用工具 → 生成 transcripts
    ↓
Transcripts → 揭示问题
    ↓
Claude (Optimizer) → 分析 transcripts → 重构工具
    ↓
Claude (Agent) → 使用新工具 → 生成新 transcripts
    ↓
循环...
```

**实战案例——Anthropic 网页搜索工具**：

```
问题发现（从 transcripts）:
  Agent 总是这样调用：
    web_search(query="AI agents 2025")
    web_search(query="prompt engineering 2025")
  → 模式：总是加 "2025"

Claude 分析：
  "工具描述可能让 Agent 误以为需要指定年份"

优化：
  修改描述："搜索最新信息，无需指定年份（系统自动匹配时效性）"

结果：
  Agent 不再加 "2025"，性能修复
```

**延伸思考**：

这种"通过行为分析优化"的方法，也可以应用到人类产品优化：
- 分析用户行为日志（类似 transcripts）
- 识别困惑模式（如频繁点击错误按钮）
- 优化 UI/UX（类似优化工具）

元循环自举的本质：**行为是最诚实的反馈**。

</details>

---

### 🤔 问题 3：为什么 Anthropic 强调"更少的工具 > 更多的工具"？不是应该给 Agent 尽可能多的能力吗？

<details>
<summary>💡 点击查看深度解答</summary>

这触及了 **Agent 决策复杂度** 和 **Context Engineering** 的核心。

**直觉认知**：工具越多 → Agent 能力越强

**实际情况**：工具越多 → Agent 越困惑

**原因分析**：

```
20 个工具的情况：
  工具描述：20 × 200 tokens = 4K tokens（挤占 context）
  决策空间：Agent 需要在 20 个工具中选择
  困惑度：工具功能重叠 → Agent 不确定用哪个
  
3 个精心设计的工具：
  工具描述：3 × 200 tokens = 600 tokens
  决策空间：清晰、边界明确
  效率：Agent 快速决策、准确调用
```

**Anthropic 的实战案例**：

❌ **过度设计**（拆分）：
```python
list_users()  # 列出所有用户
get_user_by_id(id)  # 通过 ID 获取用户
search_user(name)  # 搜索用户
filter_users(department)  # 按部门过滤
```
→ Agent 困惑："我应该先 list 还是直接 search？"

✅ **合并设计**（consolidation）：
```python
search_users(query, filters={})  
  # 统一搜索接口
  # 示例：search_users("Jane", {"department": "Engineering"})
```
→ Agent 清晰："用 search_users 就对了"

**关键洞见——工具合并的三大收益**：

1. **Context 节省**：
   ```
   4 个工具 × 200 tokens = 800 tokens
   1 个合并工具 × 250 tokens = 250 tokens
   → 节省 550 tokens
   ```

2. **决策简化**：
   ```
   Agent 思考：
     "我需要找用户... 有 4 个工具... 先 list 还是 search？"
   vs
     "我需要找用户... 用 search_users"
   ```

3. **错误减少**：
   ```
   工具重叠 → Agent 可能选错
   工具唯一 → 没有歧义
   ```

**Anthropic 的客户上下文案例**：

❌ **拆分**：
```python
get_customer_by_id(id)  # 获取客户基本信息
list_transactions(customer_id)  # 获取交易记录
list_notes(customer_id)  # 获取备注
list_support_tickets(customer_id)  # 获取工单
```
→ Agent 需要 4 次工具调用，4 次 context 污染

✅ **合并**：
```python
get_customer_context(customer_id)
  # 返回：基本信息 + 最近交易 + 关键备注 + 未解决工单
```
→ 1 次工具调用，精简高信号的返回值

**设计原则**：

```
工具设计问题清单：
  ❓ 这两个工具是否经常连续调用？ → 考虑合并
  ❓ Agent 需要在多个工具中选择？ → 考虑统一接口
  ❓ 工具返回的数据是否关联？ → 考虑一次性返回
```

**延伸思考——类比 Unix 哲学的反思**：

Unix 哲学："做一件事，做好"（Single Responsibility）

这对**确定性系统**有效（程序员可以精确组合工具）。

但对**非确定性系统**（Agent），需要权衡：
- 工具太细粒度 → Agent 困惑
- 工具合并 → Agent 高效

**结论**：为 Agent 设计工具，**减少决策点 > 增加灵活性**。

</details>

---

### 🤔 问题 4：工具返回 `uuid` 和 `mime_type` 有什么问题？这不是标准字段吗？

<details>
<summary>💡 点击查看深度解答</summary>

这个问题揭示了 **High-signal Context** 原则的深层逻辑。

**误解**：返回"完整"信息 = 好工具

**实际**：返回"高信号"信息 = 好工具

**问题分析——`uuid` 和 `mime_type` 的低信号特征**：

```
返回值示例：
{
  "id": "a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3f",  ← uuid（对 Agent 无意义）
  "name": "年度报告.pdf",  ← 高信号（Agent 理解）
  "mime_type": "application/pdf",  ← 低信号（Agent 无法直接理解）
  "file_type": "PDF",  ← 高信号（Agent 理解）
  "url": "https://storage.example.com/files/a3f7c8d2...",  ← 中等信号
  "download_url": "点击下载",  ← 高信号（自然语言）
}
```

**为什么 uuid 是低信号？**

1. **对 Agent 无意义**：
   ```
   Agent reasoning:
     "我看到 file id 是 a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3f..."
     → Agent 无法从 uuid 推断任何语义
   ```

2. **消耗 attention budget**：
   ```
   uuid: 36 characters = ~10 tokens
   × 100 个文件 = 1000 tokens（浪费）
   ```

3. **容易幻觉**：
   ```
   Agent 可能生成错误的 uuid：
     "a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3f"
   vs （幻觉）
     "a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3e"  ← 最后一位错误
   ```

**Anthropic 的解决方案——`response_format` enum**：

```python
def list_files(response_format: ResponseFormat):
    if response_format == "concise":
        # 只返回自然语言字段
        return {
            "files": [
                {"name": "年度报告.pdf", "type": "PDF", "size": "2.3 MB"},
                {"name": "预算表.xlsx", "type": "Excel", "size": "1.1 MB"}
            ]
        }
    elif response_format == "detailed":
        # 返回完整字段（包含 uuid）
        return {
            "files": [
                {
                    "id": "a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3f",
                    "name": "年度报告.pdf",
                    "mime_type": "application/pdf",
                    "file_type": "PDF",
                    "size_bytes": 2419200,
                    "size_human": "2.3 MB",
                    "url": "https://..."
                }
            ]
        }
```

**使用场景**：

```
Scenario 1: 用户问"我有哪些 PDF 文件？"
  → list_files(response_format="concise")
  → Agent 回答："你有 年度报告.pdf 和 项目提案.pdf"

Scenario 2: 用户问"下载年度报告"
  → list_files(response_format="detailed")  # 获取 file_id
  → download_file(file_id="a3f7c8d2...")  # 使用 id 下载
```

**关键洞见——"自然语言 ID" 的威力**：

Anthropic 发现：用自然语言 ID 替代 uuid，可以大幅减少幻觉。

```
❌ 低信号 ID：
  user_id: "a3f7c8d2-4e1b-4a9c-8f6d-2b5e9c7a1d3f"
  → Agent 容易生成错误 uuid

✅ 高信号 ID：
  user_id: 12345  # 或者 "user_12345"
  → Agent 很少生成错误数字

✅✅ 最高信号 ID：
  user_name: "jane@example.com"
  → Agent 几乎不会错
```

**实战案例——Slack 工具**：

```
❌ 原版（低信号）：
{
  "thread_ts": "1641024000.123456",  ← 技术标识符
  "channel_id": "C01ABC123",  ← 混淆
  "user_id": "U01XYZ789",  ← 混淆
  "text": "会议改到明天"
}
→ 206 tokens

✅ 优化版（高信号）：
{
  "channel": "#engineering",  ← 自然语言
  "user": "Jane Smith",  ← 自然语言
  "text": "会议改到明天",
  "replies": 3
}
→ 72 tokens（节省 65%）
```

**设计原则**：

```
返回值设计清单：
  ✅ Agent 能否从字段名理解含义？
  ✅ Agent 能否将字段值用于回答用户？
  ❌ 字段是否只用于"下游 API 调用"？
  ❌ 字段是否是技术标识符？

如果后两项为"是" → 使用 response_format 控制是否返回
```

**延伸思考**：

这也解释了为何 **GraphQL** 成功——让客户端选择需要的字段，避免 over-fetching。

Anthropic 的 `response_format` 本质上是"Agent-friendly GraphQL"。

</details>

---

### 🤔 问题 5：评估任务应该有多复杂？是不是越接近真实场景越好？

<details>
<summary>💡 点击查看深度解答</summary>

这个问题触及了 **Evaluation Design** 的平衡艺术。

**直觉认知**：评估任务 = 完全模拟真实场景

**实际情况**：需要在**真实性**和**可测试性**之间平衡

**Anthropic 的评估设计原则**：

```
✅ 好的评估任务：
  - 基于真实用户场景
  - 需要多步工具调用（3-20+ 次）
  - 有明确的验证标准
  - 数据复杂度接近生产环境

❌ 差的评估任务：
  - 过于简化的 sandbox 环境
  - 单步工具调用（无法测试组合能力）
  - 模糊的验证标准
  - 人工构造的测试数据
```

**任务复杂度光谱**：

```
简单任务（单工具调用）：
  "查找 jane@example.com 的用户 ID"
  → 测试：Agent 能否正确调用 search_user
  
中等任务（3-5 工具调用）：
  "安排与 Jane 下周的会议，讨论 Acme 项目"
  → 测试：search_user + list_availability + create_event
  
复杂任务（10-20+ 工具调用）：
  "客户 9182 报告重复扣费，调查是否影响其他客户"
  → 测试：search_customer + list_transactions + search_logs + 
          filter_affected_customers + generate_report
  
真实任务（20-50+ 工具调用）：
  "准备 Q4 财务报告，包含所有部门预算对比和异常分析"
  → 测试：完整的多步推理 + 数据聚合 + 异常检测
```

**Anthropic 的实战案例**：

**强评估任务示例**：
```
任务：客户 Sarah Chen 提交取消请求。准备挽留方案：
  1. 为什么取消？
  2. 什么挽留优惠最有吸引力？
  3. 有哪些风险因素需要注意？

需要的工具调用：
  - get_customer_context(customer_id)  # 获取客户历史
  - list_support_tickets(customer_id)  # 查看投诉
  - analyze_usage_patterns(customer_id)  # 分析使用情况
  - get_churn_risk_factors(customer_id)  # 流失风险
  - suggest_retention_offers(customer_id, context)  # 推荐优惠

验证标准：
  - 是否识别取消原因？（LLM 判断）
  - 是否提出合理优惠？（规则验证）
  - 是否标注风险因素？（关键词匹配）
```

**弱评估任务示例**：
```
任务：安排与 jane@acme.corp 下周的会议

问题：
  - 过于简单（2-3 工具调用）
  - 无法测试复杂推理
  - 不反映真实场景复杂度
```

**数据真实性的重要性**：

```
❌ 人工构造数据：
  {
    "customer_id": 1,
    "name": "Test User",
    "transactions": [{"amount": 100}, {"amount": 200}]
  }
  → Agent 不会遇到边缘情况

✅ 真实生产数据：
  {
    "customer_id": 9182,
    "name": "王小明（测试账号）",  ← 混合中英文
    "transactions": [
      {"amount": 99.99, "status": "pending"},  ← 未完成交易
      {"amount": 99.99, "status": "failed"},  ← 失败交易
      {"amount": 99.99, "status": "success"},  ← 成功交易
      {"amount": 99.99, "status": "refunded"}  ← 退款
    ]
  }
  → Agent 需要处理复杂情况
```

**验证策略的多样性**：

```
1. 精确匹配（简单任务）：
   预期：response == "user_id: 12345"
   实际：response == "user_id: 12345"
   → 通过

2. LLM 判断（复杂任务）：
   预期：response 包含 "客户因价格不满"
   Claude 评估：response 是否准确识别取消原因？
   → Claude 判断：是/否

3. 工具调用序列验证（流程任务）：
   预期工具调用：[search_user, list_availability, create_event]
   实际工具调用：[search_user, list_availability, create_event]
   → 通过
```

**关键洞见——避免"过度简化"和"过度复杂"**：

```
过度简化：
  - 任务太简单 → 无法发现工具问题
  - sandbox 数据 → 无法测试边缘情况
  
过度复杂：
  - 任务太难 → 验证困难
  - 多个正确路径 → 难以判断对错
  
平衡点：
  - 任务复杂度：5-15 工具调用
  - 数据真实性：生产环境的副本
  - 验证策略：规则 + LLM 判断
```

**实战建议**：

```
评估任务设计流程：
  1. 收集真实用户请求（10-20 个）
  2. 选择代表性场景（覆盖常见 + 边缘）
  3. 设计验证标准（可量化 + 人工审核）
  4. 运行基线评估（人工工具 vs Agent）
  5. 迭代任务难度（确保 60-80% 通过率）
```

**Anthropic 的 Held-out Test Set**：

```
Training Set（优化用）:
  - 100 个评估任务
  - 用于迭代优化工具
  
Held-out Test Set（验证用）:
  - 50 个评估任务（未用于优化）
  - 用于验证泛化能力
  - 防止过拟合
```

**延伸思考**：

这种"真实场景 + 系统化评估"的方法，也可以应用到传统软件测试：
- 不是写单元测试（过于简化）
- 而是写"用户旅程测试"（接近真实场景）

</details>

---

### 🤔 问题 6：命名空间（namespacing）到底有多重要？是前缀好还是后缀好？

<details>
<summary>💡 点击查看深度解答</summary>

这个问题触及了 **Agent 工具选择** 的认知机制。

**命名空间的核心目的**：**减少歧义、清晰边界**

**为什么需要命名空间？**

```
场景：你有 2 个 MCP 服务器
  - Asana MCP: 提供 search, create_task, list_projects
  - Jira MCP: 提供 search, create_issue, list_projects

Agent 看到的工具列表：
  [search, search, create_task, create_issue, list_projects, list_projects]
  
Agent 困惑：
  "我需要搜索任务... 用哪个 search？"
  "我需要列出项目... 用哪个 list_projects？"
```

**命名空间解决方案**：

```
前缀命名：
  asana_search, asana_create_task, asana_list_projects
  jira_search, jira_create_issue, jira_list_projects
  
后缀命名：
  search_asana, create_task_asana, list_projects_asana
  search_jira, create_issue_jira, list_projects_jira
```

**前缀 vs 后缀——Anthropic 的发现**：

```
Anthropic 的实验结果：
  前缀命名：准确率 85%
  后缀命名：准确率 78%
  
原因推测：
  - LLM 训练数据中，前缀命名更常见
  - Agent 从左到右阅读，前缀更早提供上下文
```

**但这不是绝对的**：

```
场景决定命名策略：
  
场景 1：按服务分组（多个 MCP 服务器）
  → 前缀命名
  例：asana_search, jira_search, slack_search
  
场景 2：按资源分组（单个 MCP 服务器）
  → 后缀命名
  例：users_search, projects_search, tasks_search
  
场景 3：混合分组（服务 + 资源）
  → 双层命名
  例：asana_users_search, asana_projects_search
```

**命名空间的层次设计**：

```
层次 1：服务命名空间
  asana_*, jira_*, slack_*
  
层次 2：资源命名空间
  asana_users_*, asana_projects_*, asana_tasks_*
  
层次 3：操作命名空间
  asana_users_search, asana_users_create, asana_users_update
```

**Anthropic 的实战案例**：

**Slack MCP 工具**：
```
❌ 扁平命名（无命名空间）：
  search, get_message, send_message, list_channels, join_channel
  → Agent 容易与其他 MCP 的 search 混淆

✅ 前缀命名：
  slack_search, slack_get_message, slack_send_message, 
  slack_list_channels, slack_join_channel
  → 清晰的边界

✅✅ 资源分组：
  slack_messages_search, slack_messages_get, slack_messages_send,
  slack_channels_list, slack_channels_join
  → 最清晰的结构
```

**命名空间的副作用——Token 消耗**：

```
无命名空间：
  search: 1 token
  
前缀命名：
  asana_search: 3 tokens
  
双层命名：
  asana_users_search: 5 tokens

Trade-off：
  - 命名空间增加 token 消耗
  - 但减少 Agent 困惑（提升准确率）
  
→ 准确率提升 > token 成本
```

**关键洞见——命名即文档**：

```
好的工具名：
  ✅ asana_users_search_by_email
  → Agent 立即理解：在 Asana 中搜索用户，按邮箱
  
差的工具名：
  ❌ search_user
  → Agent 困惑：搜索哪里的用户？用什么字段搜索？
```

**实战建议**：

```
命名空间设计清单：
  1. 识别工具集的"自然分组"（服务/资源/操作）
  2. 选择命名策略（前缀/后缀/混合）
  3. 运行评估（测试 Agent 工具选择准确率）
  4. 迭代优化（根据 transcripts 调整）
```

**Claude Code 的命名空间实践**：

```
文件系统工具：
  read_file, write_file, list_directory, search_files
  → 无前缀（因为只有一个"文件系统服务"）
  
如果有多个文件系统（如本地 + 云端）：
  local_read_file, cloud_read_file
  或
  read_file_local, read_file_cloud
```

**延伸思考**：

命名空间不仅是 Agent 工具的问题，也是传统软件设计的问题：
- **Python**：`from sklearn.linear_model import LinearRegression`（模块命名空间）
- **Kubernetes**：`namespace: production`（资源隔离）
- **DNS**：`www.example.com`（域名层次）

→ **命名空间 = 认知边界的显式表达**

</details>

---

### 🤔 问题 7：Anthropic 说"人工优化版 vs Claude 优化版"有 10-20% 性能提升——这提升真的归功于"工具优化"吗？会不会是模型本身变好了？

<details>
<summary>💡 点击查看深度解答</summary>

这是理解 **Evaluation 科学性** 的关键问题。

**合理怀疑**：性能提升可能来自：
1. 工具优化（Anthropic 的主张）
2. 模型升级（Claude Sonnet 4.5 vs 3.5）
3. 评估任务偏见（过拟合）
4. 随机波动

**Anthropic 如何确保实验有效性？**

```
实验设计：

控制变量：
  ✅ 同一个模型（Claude Sonnet 4.5）
  ✅ 同样的评估任务
  ✅ 同样的系统 prompt
  
唯一变量：
  ❌ 人工编写的工具（版本 A）
  ✅ Claude 优化的工具（版本 B）
  
测试集：
  Training Set（优化用）: 100 个任务
  Held-out Test Set（验证用）: 50 个任务（未用于优化）
  
结果：
  版本 A（人工）: 70% 准确率（held-out test）
  版本 B（Claude 优化）: 85% 准确率（held-out test）
  → 15% 提升
```

**关键设计——Held-out Test Set 防止过拟合**：

```
为什么需要 Held-out Test Set？

场景：优化 Slack 工具
  
Training Set（用于优化）:
  - 任务 1："搜索 #engineering 频道的消息"
  - 任务 2："发送消息到 #general"
  - ...（100 个任务）
  
风险：过拟合
  - Claude 可能针对这 100 个任务优化工具
  - 在新任务上表现不佳
  
Held-out Test Set（用于验证）:
  - 任务 101："搜索 @Jane 发送的所有消息"
  - 任务 102："在 #marketing 创建提醒"
  - ...（50 个任务，从未用于优化）
  
验证：
  如果在 Held-out Test 上仍有 10-20% 提升
  → 证明工具优化有效（而非过拟合）
```

**实验结果的可信度分析**：

| 可能原因 | Anthropic 的控制措施 | 可信度 |
|---------|---------------------|--------|
| **工具优化** | ✅ 唯一变量 | ⭐⭐⭐⭐⭐ |
| **模型升级** | ✅ 同一模型 | 排除 |
| **评估任务偏见** | ✅ Held-out Test Set | 排除 |
| **随机波动** | ✅ 多次运行（统计显著性）| 排除 |

**Anthropic 的透明度——图表数据**：

```
文章中的图表（Slack MCP）:
  
人工版（蓝色柱）:
  Training Set: 72%
  Held-out Test: 70%
  
Claude 优化版（橙色柱）:
  Training Set: 88%
  Held-out Test: 85%
  
关键观察：
  - Training 和 Held-out 差距小（2-3%）
  → 没有过拟合
  - Claude 版在两个集合上都更好
  → 工具优化真实有效
```

**更深层的验证——Transcript 分析**：

```
Anthropic 不仅看"准确率"，还看"为什么准确"：

人工版的 transcript:
  - Agent 频繁调用错误工具
  - Agent reasoning 冗长（困惑）
  - 工具错误率高（如 context overflow）
  
Claude 优化版的 transcript:
  - Agent 直接调用正确工具
  - Agent reasoning 简洁（清晰）
  - 工具错误率低
  
→ 定性分析支持定量结果
```

**关键洞见——多维度验证**：

Anthropic 的验证不是单一指标，而是多维度：

1. **准确率**（Held-out Test Set）
2. **工具调用次数**（效率）
3. **Token 消耗**（成本）
4. **错误率**（工具调用失败）
5. **Reasoning 长度**（Agent 困惑度）

**实战案例——Asana 工具优化**：

```
优化前（人工版）：
  准确率: 65%
  平均工具调用: 12 次/任务
  平均 token 消耗: 15K tokens/任务
  
优化后（Claude 版）：
  准确率: 82% （+17%）
  平均工具调用: 8 次/任务 （-33%）
  平均 token 消耗: 10K tokens/任务 （-33%）
  
→ 多维度全面提升
```

**可重复性——开源 Evaluation**：

Anthropic 提供 cookbook：
- https://github.com/anthropics/anthropic-cookbook
- 包含工具评估模板
- 任何人可以复现实验

**延伸思考——科学方法论在 AI 工程中的应用**：

```
传统软件工程：
  - 依赖单元测试（确定性验证）
  - 性能测试（benchmark）
  
AI 工程：
  - 需要统计方法（非确定性验证）
  - Held-out Test Set（防止过拟合）
  - 多维度指标（全面评估）
  
→ AI 工程需要"实验科学"思维
```

**结论**：

Anthropic 的 10-20% 提升是**科学严谨**的结果：
- ✅ 控制变量
- ✅ Held-out Test Set
- ✅ 多维度验证
- ✅ Transcript 定性分析
- ✅ 开源可复现

这不是营销数字，而是系统工程的成果。

</details>

---

## 💡 反直觉洞见

### 洞见 1："更多工具" ≠ "更强能力"，反而可能导致性能下降

**直觉认知**：工具越多 → Agent 能力越强 → 用户体验越好

**实际情况**：

```
20 个工具：
  - Agent 需要在 20 个工具中选择
  - 工具描述占用 4K tokens
  - 功能重叠导致困惑
  → 准确率: 60%

5 个精心设计的工具：
  - Agent 决策空间清晰
  - 工具描述仅占 1K tokens
  - 功能边界明确
  → 准确率: 85%
```

**为什么反直觉？**

传统软件设计：功能越多 → 系统越强大  
Agent 设计：**决策点越少 → 系统越可靠**

**启示**：工具设计是"做减法"而非"做加法"。

---

### 洞见 2：用 Agent 优化 Agent 的工具，比人工优化更有效

**直觉认知**：人类专家（精通工具设计）> Agent（自动化优化）

**实际情况**：

```
Anthropic 的实验结果：
  人工专家优化：70% 准确率
  Claude 自动优化：85% 准确率
  
原因：
  - Agent 分析 transcripts 无遗漏（人类会忽略）
  - Agent 识别模式更系统（人类凭直觉）
  - Agent 批量重构工具更快（人类手动修改）
```

**为什么反直觉？**

我们通常认为"理解问题的人（人类）最会优化"。  
但 Agent 有独特优势：**它既是使用者，也是分析者**。

**启示**：元循环自举不是"偷懒"，而是"更高效"。

---

### 洞见 3：返回"完整信息"不如返回"高信号信息"

**直觉认知**：API 应该返回所有字段（完整性 > 相关性）

**实际情况**：

```
完整返回值（206 tokens）：
  {
    "thread_ts": "1641024000.123456",
    "channel_id": "C01ABC123",
    "user_id": "U01XYZ789",
    "text": "会议改到明天",
    "reactions": [...],
    "replies": 3
  }

高信号返回值（72 tokens）：
  {
    "channel": "#engineering",
    "user": "Jane Smith",
    "text": "会议改到明天",
    "replies": 3
  }
  
→ 节省 65% tokens，准确率不变
```

**为什么反直觉？**

传统 API 设计："返回所有数据，让客户端选择"  
Agent 工具设计：**"返回相关数据，减少 Agent 困惑"**

**启示**：Agent 的 attention budget 有限，"less is more"。

---

### 洞见 4：对 Agent "ergonomic" 的工具，对人类也最直观

**直觉认知**：Agent 工具和人类接口是两套系统

**实际情况**：

```
Agent 工具优化：
  ❌ list_contacts() → ✅ search_contacts(name)
  
这个优化对人类也有效：
  人类也不会逐页翻通讯录，而是直接搜索
```

**为什么反直觉？**

我们认为 Agent 是"外星人"，需要特殊设计。  
实际上，**Agent 的认知方式接近人类**。

**启示**：优化 Agent 工具的过程，也是重新审视人类接口设计的过程。

---

### 洞见 5：评估不是"测试"，而是"发现工具问题的最佳途径"

**直觉认知**：评估 = 验证工具是否正确（测试阶段）

**实际情况**：

```
传统思维：
  开发 → 测试 → 上线
  
Agent 工具思维：
  原型 → 评估（发现问题）→ 优化 → 再评估
  
评估的价值：
  - Transcripts 揭示 Agent 的困惑点
  - 工具调用模式暴露设计缺陷
  - 错误率指向需要改进的地方
```

**为什么反直觉？**

传统测试："验证正确性"  
Agent 评估：**"发现优化方向"**

**启示**：评估驱动开发（Evaluation-driven Development）是 Agent 工程的核心方法论。

---

### 洞见 6：工具描述的"小改动"能带来"大提升"

**直觉认知**：性能提升需要大的架构改动

**实际情况**：

```
Anthropic 网页搜索工具案例：
  
问题：Claude 总是在 query 后加 "2025"
  
原描述：
  "搜索网页信息"
  
优化描述：
  "搜索最新网页信息，无需指定年份（系统自动匹配时效性）"
  
结果：性能修复，准确率提升 5%
```

**为什么反直觉？**

我们认为优化需要"重构代码"或"增加功能"。  
实际上，**Prompt 工程工具描述**往往是最高 ROI 的优化。

**启示**：不要忽视"工具描述"——它是 Agent 理解工具的唯一途径。

---

## 小结：反直觉的核心

这些反直觉洞见有一个共同主题：

**为 Agent 设计工具 ≠ 为程序员设计 API**

原因：
- Agent 是**非确定性**系统（需要减少歧义）
- Agent 有**有限 attention budget**（需要高信号）
- Agent 能**自我优化**（元循环自举）

→ 工具设计需要重新思考"确定性"假设。

---

## ✅ 行动清单

### Top 3 立即可行动作

#### 1. **为你的 Agent 工具建立评估体系** 📊

- [ ] 收集 20-50 个真实用户场景任务
- [ ] 为每个任务定义验证标准（精确匹配 / LLM 判断 / 工具调用序列）
- [ ] 实现自动化评估脚本（while 循环 + LLM API + 工具调用）
- [ ] 记录基线指标（准确率、工具调用次数、token 消耗、错误率）
- [ ] 建立 Held-out Test Set（30% 任务用于验证泛化能力）

**工具模板**：

```python
def run_evaluation(tasks, tools, held_out=False):
    results = []
    for task in tasks:
        agent = Agent(tools=tools)
        transcript = agent.run(task["prompt"])
        
        # 验证结果
        correct = verify(transcript, task["expected"])
        results.append({
            "task_id": task["id"],
            "correct": correct,
            "tool_calls": len(transcript["tool_calls"]),
            "tokens": count_tokens(transcript),
            "errors": transcript["errors"]
        })
    
    # 计算指标
    accuracy = sum(r["correct"] for r in results) / len(results)
    avg_tool_calls = mean([r["tool_calls"] for r in results])
    avg_tokens = mean([r["tokens"] for r in results])
    
    return {
        "accuracy": accuracy,
        "avg_tool_calls": avg_tool_calls,
        "avg_tokens": avg_tokens,
        "held_out": held_out
    }
```

**预计时间**：2-3 天  
**收益**：建立优化的科学基础，让后续改进可测量

---

#### 2. **审查现有工具，识别"简单包装 API"的问题** 🔍

- [ ] 列出所有 Agent 工具
- [ ] 检查是否直接包装 API（如 `list_all_*`, `get_*_by_id`）
- [ ] 识别 Agent 频繁误用的工具（从 logs/transcripts）
- [ ] 设计合并方案（如 3 个工具 → 1 个统一搜索工具）
- [ ] A/B 测试：原版 vs 优化版

**审查清单**：

```
工具问题检查：
  ❌ 返回大列表（> 100 项）？
  ❌ Agent 需要遍历数据？
  ❌ 功能与其他工具重叠？
  ❌ Agent 频繁调用错误的工具？
  ❌ 工具调用失败率高（> 10%）？

如果 3+ 项为"是" → 需要重新设计
```

**预计时间**：1-2 天  
**收益**：快速识别低效工具，制定优化计划

---

#### 3. **实现 `response_format` 控制返回值粒度** 🎛️

- [ ] 识别返回大量数据的工具（> 1K tokens）
- [ ] 设计 `response_format` enum（`concise` / `detailed`）
- [ ] `concise`：只返回自然语言字段（名称、类型、状态）
- [ ] `detailed`：返回完整字段（包括 IDs、技术标识符）
- [ ] 更新工具描述，引导 Agent 使用 `concise` 模式
- [ ] 测试 token 节省效果

**实现示例**：

```python
from enum import Enum

class ResponseFormat(Enum):
    CONCISE = "concise"
    DETAILED = "detailed"

def list_files(directory: str, response_format: ResponseFormat = ResponseFormat.CONCISE):
    files = get_files_from_fs(directory)
    
    if response_format == ResponseFormat.CONCISE:
        return {
            "files": [
                {
                    "name": f["name"],
                    "type": get_human_readable_type(f["mime_type"]),
                    "size": format_size(f["size_bytes"]),
                    "modified": format_date(f["modified_timestamp"])
                }
                for f in files
            ]
        }
    else:  # DETAILED
        return {"files": files}  # 返回完整数据
```

**预计时间**：1 天  
**收益**：Token 消耗减少 30-60%，Agent 决策更清晰

---

### 完整行动清单（10 项）

#### 4. **用 Claude 分析你的 Agent transcripts 并优化工具** 🤖

- [ ] 收集最近 50-100 个 Agent 执行 transcripts
- [ ] 将 transcripts 粘贴到 Claude Code
- [ ] 提示 Claude："分析这些 transcripts，识别工具问题，提出优化建议"
- [ ] Claude 重构工具代码和描述
- [ ] 重新运行评估，验证提升

**Prompt 模板**：

```
我有 100 个 Agent 执行 transcripts，请分析并优化我的工具：

1. 识别模式：
   - Agent 频繁误用的工具
   - 冗长的 reasoning（困惑信号）
   - 工具调用失败模式

2. 提出优化：
   - 哪些工具应该合并？
   - 哪些描述需要澄清？
   - 哪些参数名容易混淆？

3. 重构代码：
   - 修改工具实现
   - 更新工具描述
   - 确保工具之间一致性

Transcripts:
[... 粘贴 transcripts ...]
```

**预计时间**：1-2 天  
**收益**：自动化优化，发现人类忽略的模式

---

#### 5. **为工具实现分页、过滤、截断机制** 📄

- [ ] 识别可能返回大数据的工具（日志、消息、文件列表）
- [ ] 添加 `limit` 参数（默认 100）
- [ ] 添加过滤参数（如 `status`, `date_range`, `category`）
- [ ] 实现智能截断（如 Claude Code 的 25K tokens 限制）
- [ ] 在错误信息中引导 Agent 使用分页

**实现示例**：

```python
def search_logs(
    query: str,
    level: str = None,  # "ERROR", "WARN", "INFO"
    limit: int = 100,
    max_tokens: int = 25000
):
    """
    搜索日志。默认返回 100 条，最多 25K tokens。
    
    提示：如果结果被截断，使用更具体的 query 或 level 过滤。
    """
    logs = get_logs_from_db(query, level=level)
    
    # 分页
    logs = logs[:limit]
    
    # Token 截断
    result = format_logs(logs)
    if count_tokens(result) > max_tokens:
        result = truncate_with_hint(result, max_tokens)
    
    return result

def truncate_with_hint(content, max_tokens):
    truncated = content[:estimate_char_for_tokens(max_tokens)]
    return f"{truncated}\n\n⚠️ 结果被截断。建议：使用更具体的过滤（如 level='ERROR'）"
```

**预计时间**：1-2 天  
**收益**：防止 context overflow，引导 Agent 精准查询

---

#### 6. **设计命名空间策略** 🏷️

- [ ] 识别工具的自然分组（服务 / 资源 / 操作）
- [ ] 选择命名策略（前缀 / 后缀 / 混合）
- [ ] 统一命名规范（如 `{service}_{resource}_{action}`）
- [ ] 重命名现有工具
- [ ] 运行评估，验证工具选择准确率提升

**命名空间设计模板**：

```
场景 1：多个 MCP 服务器
  → 前缀命名
  例：asana_search, jira_search, slack_search

场景 2：单个服务，多种资源
  → 资源分组
  例：users_search, projects_search, tasks_search

场景 3：复杂系统（服务 + 资源）
  → 双层命名
  例：asana_users_search, asana_projects_create

命名检查清单：
  ✅ 工具名能否立即理解功能？
  ✅ 是否避免与其他工具混淆？
  ✅ 是否遵循一致的命名模式？
```

**预计时间**：1 天  
**收益**：工具选择准确率提升 5-15%

---

#### 7. **优化工具描述的"正确高度"** 📝

- [ ] 审查所有工具描述
- [ ] 识别"过度详细"（if-else 逻辑）和"过于模糊"（"搜索工具"）
- [ ] 重写为"原则 + 启发式"风格
- [ ] 添加具体示例（1-2 个典型用法）
- [ ] A/B 测试：原版 vs 优化版描述

**工具描述模板**：

```python
❌ 过度详细（脆弱）：
"""
搜索用户工具。
如果提供 email，按邮箱搜索。
如果提供 name，按姓名搜索。
如果提供 department，按部门过滤。
如果同时提供 name 和 department，先按姓名搜索，再按部门过滤。
...
"""

❌ 过于模糊（无指导）：
"""
搜索用户。
"""

✅ 正确高度（原则 + 示例）：
"""
在组织中搜索用户。支持按姓名、邮箱、部门等字段搜索。

原则：
- 使用最具体的查询（避免返回过多结果）
- 优先使用 email（最精确）

示例：
- search_users("jane@example.com")  # 按邮箱（最精确）
- search_users("Jane Smith", department="Engineering")  # 组合查询
"""
```

**预计时间**：1-2 天  
**收益**：Agent 理解准确率提升，减少工具误用

---

#### 8. **将 UUID 替换为自然语言 ID 或 0-indexed ID** 🆔

- [ ] 识别工具返回的 UUID 字段
- [ ] 设计替代方案（自然语言 ID / 简单数字 ID）
- [ ] 修改工具实现（内部仍可用 UUID，对外映射）
- [ ] 测试幻觉率降低效果

**实现策略**：

```python
# 内部映射：UUID → 简单 ID
_user_id_map = {}

def search_users(query):
    users = db.query("SELECT uuid, name, email FROM users WHERE ...")
    
    # 映射 UUID → 简单 ID
    results = []
    for i, user in enumerate(users):
        simple_id = f"user_{i+1}"
        _user_id_map[simple_id] = user["uuid"]
        
        results.append({
            "id": simple_id,  # 对 Agent 友好
            "name": user["name"],
            "email": user["email"]
        })
    
    return results

def send_message(user_id, message):
    # 内部映射回 UUID
    uuid = _user_id_map.get(user_id)
    if not uuid:
        raise ValueError(f"Invalid user_id: {user_id}")
    
    db.send_message(uuid, message)
```

**预计时间**：1 天  
**收益**：减少 Agent 幻觉，提升准确率 5-10%

---

#### 9. **建立工具优化的迭代流程** 🔄

- [ ] 建立每周/每月工具优化会议
- [ ] 制定工具质量指标仪表板（准确率、token 效率、错误率）
- [ ] 定期运行评估，追踪趋势
- [ ] 优先优化"高频 + 低准确率"的工具
- [ ] 文档化优化决策和结果

**优化优先级矩阵**：

```
工具优先级 = 使用频率 × (1 - 准确率) × token 消耗

高优先级工具（立即优化）：
  - 使用频率 > 50%
  - 准确率 < 70%
  - Token 消耗 > 5K

中优先级工具（下次迭代）：
  - 使用频率 20-50%
  - 准确率 70-85%

低优先级工具（监控）：
  - 使用频率 < 20%
  - 准确率 > 85%
```

**预计时间**：持续投入  
**收益**：系统化优化流程，持续提升

---

#### 10. **参与社区，分享你的工具设计实践** 🌐

- [ ] 撰写博客：你的工具设计案例（Before/After）
- [ ] 开源你的评估框架（GitHub）
- [ ] 贡献 MCP 工具到社区（modelcontextprotocol.io）
- [ ] 参与讨论：Anthropic Discord、Reddit r/ClaudeAI
- [ ] 学习他人案例，持续迭代

**分享模板**：

```markdown
# 我的 Agent 工具优化案例：Slack MCP

## 背景
- Agent 类型：内部协作助手
- 原工具：直接包装 Slack API（15 个工具）
- 主要问题：Agent 困惑、token 浪费

## 优化措施
1. 合并工具：15 → 5 个工具
2. response_format：节省 60% tokens
3. 命名空间：slack_* 前缀
4. 描述优化：澄清歧义

## 效果
- 准确率：68% → 87% （+19%）
- 工具调用次数：12 → 7 次/任务 （-42%）
- Token 消耗：18K → 8K tokens/任务 （-56%）

## 代码
[链接到 GitHub repo]
```

**预计时间**：持续投入  
**收益**：社区反馈、持续学习、建立影响力

---

## 🔗 知识网络关联

| 笔记 | 关系类型 | 关联点 |
|------|----------|--------|
| **Context Engineering** | 🔗 **互补** | Context Engineering 讲"如何管理 context"，本文讲"如何设计工具减少 context 浪费"；两者结合 = 完整的 Agent 优化 |
| **Building Effective Agents** | 🎯 **基础** | Building Agents 讲"架构选择"，本文讲"工具实现"；前者是战略，后者是战术 |
| **Long-running Agents** | 🔗 **互补** | Long-running 需要高效工具（减少 token 消耗、跨会话一致性）；本文提供工具设计方法 |
| **Agent Evals** | 🎯 **应用** | 本文的评估方法是 Agent Evals 的具体实践；transcript 分析 = eval 的核心技术 |
| **Manus Context Engineering** | 🔗 **互补** | Manus 讲 KV-cache 优化（成本），本文讲工具设计（性能）；工具设计影响 KV-cache hit rate |

---

## 📌 金句摘录

1. **"Agents are only as effective as the tools we give them."**  
   *价值：定义了工具设计的核心重要性*

2. **"Tools are a new kind of software which reflects a contract between deterministic systems and non-deterministic agents."**  
   *价值：揭示了工具设计的范式转变*

3. **"When we traditionally write software, we're establishing a contract between deterministic systems. For instance, a function call like `getWeather("NYC")` will always fetch the weather in New York City in the exact same manner every time it is called."**  
   *价值：对比传统软件开发，突出 Agent 工具的特殊性*

4. **"Fundamentally rethinking our approach when writing software for agents: instead of writing tools and MCP servers the way we'd write functions and APIs for other developers or systems, we need to design them for agents."**  
   *价值：强调需要"重新思考"工具设计方法*

5. **"Our goal is to increase the surface area over which agents can be effective in solving a wide range of tasks by using tools to pursue a variety of successful strategies."**  
   *价值：定义了工具设计的目标——增加 Agent 的"可供性"*

6. **"Fortunately, in our experience, the tools that are most 'ergonomic' for agents also end up being surprisingly intuitive to grasp as humans."**  
   *价值：揭示了人机统一的洞见*

7. **"LLM agents have limited 'context' (that is, there are limits to how much information they can process at once), whereas computer memory is cheap and abundant."**  
   *价值：解释了为何不能简单包装 API*

8. **"Tools can consolidate functionality, handling potentially multiple discrete operations (or API calls) under the hood."**  
   *价值：提出工具合并的原则*

9. **"Namespacing (grouping related tools under common prefixes) can help delineate boundaries between lots of tools."**  
   *价值：强调命名空间的重要性*

10. **"Tool implementations should take care to return only high signal information back to agents. They should prioritize contextual relevance over flexibility, and eschew low-level technical identifiers."**  
    *价值：定义了高信号返回值原则*

11. **"Fields like `name`, `image_url`, and `file_type` are much more likely to directly inform agents' downstream actions and responses [than `uuid`, `256px_image_url`, `mime_type`]."**  
    *价值：具体说明什么是"高信号"*

12. **"We've found that merely resolving arbitrary alphanumeric UUIDs to more semantically meaningful and interpretable language (or even a 0-indexed ID scheme) significantly improves Claude's precision in retrieval tasks by reducing hallucinations."**  
    *价值：揭示自然语言 ID 的威力*

13. **"Even your tool response structure—for example XML, JSON, or Markdown—can have an impact on evaluation performance: there is no one-size-fits-all solution."**  
    *价值：强调需要根据具体场景选择格式*

14. **"We suggest implementing some combination of pagination, range selection, filtering, and/or truncation with sensible default parameter values for any tool responses that could use up lots of context."**  
    *价值：提出 token 效率优化策略*

15. **"If a tool call raises an error (for example, during input validation), you can prompt-engineer your error responses to clearly communicate specific and actionable improvements, rather than opaque error codes or tracebacks."**  
    *价值：强调错误信息也是引导 Agent 的机会*

16. **"Think of how you would describe your tool to a new hire on your team. Consider the context that you might implicitly bring—specialized query formats, definitions of niche terminology, relationships between underlying resources—and make it explicit."**  
    *价值：提供工具描述的写作指南*

17. **"Even small refinements to tool descriptions can yield dramatic improvements. Claude Sonnet 3.5 achieved state-of-the-art performance on the SWE-bench Verified evaluation after we made precise refinements to tool descriptions."**  
    *价值：强调 Prompt 工程工具描述的高 ROI*

18. **"To build effective tools for agents, we need to re-orient our software development practices from predictable, deterministic patterns to non-deterministic ones."**  
    *价值：总结全文核心思想*

19. **"Through the iterative, evaluation-driven process we've described in this post, we've identified consistent patterns in what makes tools successful."**  
    *价值：强调评估驱动的重要性*

20. **"With a systematic, evaluation-driven approach to improving tools for agents, we can ensure that as agents become more capable, the tools they use will evolve alongside them."**  
    *价值：展望未来——工具与 Agent 共同进化*

---

## 📚 延伸阅读

### Anthropic 官方资源

1. **Building effective AI agents**  
   https://www.anthropic.com/engineering/building-effective-agents  
   *前置文章：Workflows vs Agents 架构选择*

2. **Effective context engineering for AI agents**  
   https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents  
   *姊妹篇：Context 管理方法论*

3. **Effective harnesses for long-running agents**  
   https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents  
   *姊妹篇：长时间运行 Agent 的基础设施*

4. **How we built our multi-agent research system**  
   https://www.anthropic.com/engineering/multi-agent-research-system  
   *Sub-agent 架构案例*

5. **Anthropic Cookbook - Tool Evaluation**  
   https://github.com/anthropics/anthropic-cookbook  
   *官方 Cookbook：工具评估模板*

6. **Claude Developer Guide - Tool Use**  
   https://docs.anthropic.com/en/docs/build-with-claude/tool-use  
   *官方文档：工具使用最佳实践*

---

### Model Context Protocol (MCP)

7. **MCP 官方文档**  
   https://modelcontextprotocol.io/docs/getting-started/intro  
   *MCP 协议规范和入门指南*

8. **MCP Tool Annotations**  
   https://modelcontextprotocol.io/docs/concepts/tools#tool-annotations  
   *工具权限标注规范*

9. **MCP SDK (Python/TypeScript)**  
   https://github.com/modelcontextprotocol  
   *官方 SDK 实现*

---

### 工具设计理论

10. **Don Norman - The Design of Everyday Things**  
    *经典书籍：Affordance（可供性）理论*

11. **Poka-yoke (Mistake-proofing)**  
    https://en.wikipedia.org/wiki/Poka-yoke  
    *防错设计原则：让工具"难以误用"*

12. **GraphQL - Flexible Query API**  
    https://graphql.org/  
    *类比：客户端选择需要的字段（类似 `response_format`）*

---

### 评估方法论

13. **SWE-bench: Can Language Models Resolve Real-World GitHub Issues?**  
    https://www.swebench.com/  
    *Agent 评估的学术标准*

14. **Needle-in-a-Haystack Test**  
    https://research.trychroma.com/context-rot  
    *Context rot 的测试方法*

---

### 元循环与自举

15. **Bootstrapping (Compilers)**  
    https://en.wikipedia.org/wiki/Bootstrapping_(compilers)  
    *自举思想在编译器中的应用*

16. **Meta-circular Evaluator (SICP)**  
    https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-26.html  
    *经典：Lisp 的元循环解释器*

---

### 相关工具与实践

17. **Claude Code**  
    https://www.anthropic.com/claude-code  
    *实战案例：Anthropic 的 Agent 编程助手*

18. **Stripe API Design Guidelines**  
    https://stripe.com/docs/api  
    *高质量 API 设计案例：语义化参数名*

19. **llms.txt**  
    https://llmstxt.org/  
    *LLM 友好文档格式标准*

---

### 社区讨论

20. **Anthropic Discord**  
    https://discord.com/invite/anthropic  
    *官方社区：工具设计讨论*

21. **Reddit r/ClaudeAI**  
    https://www.reddit.com/r/ClaudeAI/  
    *用户社区：实战经验分享*

22. **MCP Community Servers**  
    https://github.com/modelcontextprotocol/servers  
    *开源 MCP 工具集合*
