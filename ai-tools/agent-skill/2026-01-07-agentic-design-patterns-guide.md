---
title: Agentic Design Patterns: A Hands-On Guide to Building Intelligent Systems
source: https://github.com/sarwarbeing-ai/Agentic_Design_Patterns
author: Antonio Gulli (Google 资深工程师)
date: 2026-01-07
category: ai-tools
tags: [智能体, 设计模式, LangChain, CrewAI, Google ADK, MCP, 多智能体协作]
---

# Agentic Design Patterns: 智能体设计模式完整指南

> 📖 原文：[Agentic Design Patterns](https://github.com/sarwarbeing-ai/Agentic_Design_Patterns)  
> 🇨🇳 中文版：[Jimmy Song 翻译版](https://jimmysong.io/zh/book/agentic-design-patterns/) | [GitHub 中文仓库](https://github.com/ginobefun/agentic-design-patterns-cn)  
> 📅 学习日期：2026-01-07  
> 🏷️ 分类：AI 工具与效率

---

## 一句话总结

这是一本将 AI 智能体设计模式系统化的实战指南，从基础的提示链到复杂的多智能体协作，涵盖 21 种核心模式，配套完整的 Python 代码实现。中文翻译版提供了双语对照和本土化的代码示例，大大降低了学习门槛。

## 📚 资源概览

| 资源类型 | 链接 | 特点 | 推荐指数 |
|---------|------|------|----------|
| **原版英文** | [GitHub 仓库](https://github.com/sarwarbeing-ai/Agentic_Design_Patterns) | 最新内容，Jupyter 代码 | ⭐⭐⭐⭐⭐ |
| **中文在线版** | [Jimmy Song 网站](https://jimmysong.io/zh/book/agentic-design-patterns/) | 专业排版，目录导航 | ⭐⭐⭐⭐⭐ |
| **中文 GitHub** | [ginobefun 仓库](https://github.com/ginobefun/agentic-design-patterns-cn) | 双语对照，可运行代码 | ⭐⭐⭐⭐⭐ |
| **PDF 下载** | [在线版 PDF](https://jimmysong.io/zh/book/agentic-design-patterns/) | 离线阅读，完整内容 | ⭐⭐⭐⭐ |

---

## 核心要点

1. **模式可组合**：21 种模式不是孤立的，可以像乐高积木一样组合使用
2. **反思是关键**：Reflection 模式是 Agent 从"工具"升级为"智能体"的核心能力
3. **安全不可缺**：Guardrails 必须在设计阶段就考虑，而非事后补丁
4. **标准化趋势**：MCP 和 A2A 正在成为 Agent 生态的"USB 标准"
5. **渐进式采用**：从简单模式（Prompt Chaining）开始，逐步引入复杂模式
6. **中文生态**：高质量的中文翻译版本为国内开发者提供了完整的学习路径

## 🔥 中文翻译版亮点

### 翻译质量
- **专业术语精准**：Prompt Chaining → 提示链，Context Engineering → 上下文工程
- **双语对照结构**：英文段落 + 中文段落逐段对照，便于理解细微差别
- **本土化处理**：角色名称、案例场景适配中文语境

### 独特增值内容
- **代码运行结果**：译者补充了具体的 JSON 输出示例，便于验证学习效果
- **环境配置指南**：详细的 Python 环境搭建和依赖安装说明
- **在线运行支持**：提供 Google Colab 链接，无需本地配置即可实践

### 翻译进度（截至 2026-01-07）
| 部分 | 章节范围 | 完成状态 | 说明 |
|------|----------|----------|------|
| **核心模式** | 第 1-7 章 | ✅ 已完成 | 提示链、路由、并行化、反思等 |
| **高级模式** | 第 8-11 章 | ⏳ 部分完成 | 记忆管理、MCP 等 |
| **集成模式** | 第 12-14 章 | ⏳ 交叉评审中 | 异常处理、HITL、RAG |
| **生产模式** | 第 15-21 章 | ⏳ 进行中 | A2A 通信、安全护栏等 |

---

## 21 种设计模式全景图

### 基础层 (Foundation)
- **Ch1. Prompt Chaining**：线性链式调用，将复杂任务分解为多个提示序列
- **Ch2. Routing**：动态任务分发，根据输入特征智能路由到专业处理器
- **Ch3. Parallelization**：并行执行，同时处理多个独立任务
- **Ch4. Reflection**：自我反思改进，生成→评估→改进的迭代循环
- **Ch5. Tool Use**：工具调用扩展，通过外部 API 扩展能力边界

### 协作层 (Collaboration)
- **Ch6. Planning**：任务规划分解，将目标拆解为可执行的步骤
- **Ch7. Multi-Agent**：多智能体协作，不同角色的 Agent 分工合作
- **Ch8. Memory Management**：状态与记忆管理，短期和长期记忆的协同
- **Ch9. Adaptation**：动态适应调整，根据反馈调整行为策略

### 通信层 (Communication)
- **Ch10. MCP**：模型上下文协议，标准化的工具接口
- **Ch11. Goal Monitoring**：目标设定与监控，追踪任务进度和完成度
- **Ch12. Exception Handling**：异常处理恢复，容错和降级机制
- **Ch13. Human-in-the-Loop**：人机协作介入，关键决策点的人工确认
- **Ch14. Knowledge Retrieval**：RAG 知识增强，外部知识库的检索整合
- **Ch15. A2A Communication**：智能体间通信，Agent 之间的标准化协议

### 运维层 (Operations)
- **Ch16. Resource Optimization**：资源成本优化，性能与成本的平衡
- **Ch17. Reasoning Techniques**：推理技术增强，CoT、自纠正等方法
- **Ch18. Guardrails/Safety**：安全护栏边界，输入输出的安全验证
- **Ch19. Evaluation/Monitoring**：评估与监控，质量评估和可观测性
- **Ch20. Prioritization**：任务优先级调度，资源分配和任务排序
- **Ch21. Exploration**：自主探索发现，创新和自主学习能力

---

## 关键概念

| 概念 | 定义 | 应用场景 | 中文版补充 |
|------|------|----------|------------|
| **Prompt Chaining** | 将复杂任务分解为多个提示的线性序列 | 信息提取→格式转换→结果输出 | 强调结构化输出（JSON/XML）的重要性 |
| **Routing** | 根据输入特征将请求动态分发到专业处理器 | 客服系统、多领域问答 | 使用 LLM 作为语义路由器的最佳实践 |
| **Reflection** | Agent 对自身输出进行批判性评估并迭代改进 | 代码生成、文章写作 | 生产者-评论者双角色模型详解 |
| **Multi-Agent** | 多个专业 Agent 协作完成复杂任务 | 内容创作团队、研发流程 | CrewAI 框架的角色定义和任务协作 |
| **Guardrails** | 输入验证、输出校验、行为约束的安全机制 | 生产环境部署 | Pydantic 结构验证的具体实现 |
| **MCP** | Anthropic 提出的工具标准化协议 | 文件系统、数据库访问 | 与传统 API 集成的对比分析 |
| **Memory Management** | 短期（对话）和长期（持久化）记忆的管理 | 有状态对话、知识积累 | 记忆摘要和淘汰策略的实现 |

## 🎯 核心模式深度解析

### 1. 提示链 (Prompt Chaining)
**中文版独特见解**：
- **上下文工程**：从简单的"问答"向高级"智能体"演进的关键概念
- **结构化传递**：强烈推荐在步骤间使用 JSON/XML 格式，减少自然语言解析的歧义
- **模块化角色**：在链的不同阶段为 LLM 分配不同角色（分析师→撰稿人）

**7 大应用场景**：
1. 信息处理工作流（提取→总结→查库→生成报告）
2. 复杂问答（拆解问题→分步检索→综合回答）
3. 数据提取与转换（OCR→规范化→外部计算）
4. 内容生成工作流（构思→大纲→起草→润色）
5. 有状态对话智能体（维护对话历史）
6. 代码生成与优化（伪代码→草稿→分析→修正）
7. 多模态推理（图像文本→关联标签→表格解读）

### 2. 反思模式 (Reflection)
**核心架构**：生产者与评论者双智能体模型
- **生产者**：专注生成内容（写代码、写文章、做计划）
- **评论者**：作为"高级工程师"、"事实核查员"挑刺并提出改进建议

**技术实现要点**：
```python
# LangChain 实现核心
message_history = []  # 维护对话历史
while not is_perfect(output):
    feedback = reflector_prompt.invoke(output)
    message_history.append(feedback)
    output = generator.invoke(message_history)
```

**适用原则**：当质量比速度和成本更重要时使用
- ✅ 长文写作、代码开发、复杂规划
- ❌ 简单问答、实时对话、成本敏感场景

---

## 实用技巧

| 模式 | 技巧 | 操作方法 | 效果 |
|------|------|----------|------|
| **提示链** | 使用 LCEL 构建管道 | `prompt1 \| llm \| parser \| prompt2` | 可调试、可复用 |
| **路由** | LLM 作为 Router | 定义子 Agent 描述，协调器自动分发 | 智能分发、易扩展 |
| **反思** | 设置最大迭代次数 | 生成→评估→改进，max_iterations=3 | 质量提升、避免死循环 |
| **多智能体** | 定义角色+目标+背景 | CrewAI 的 Agent 和 Task 定义 | 专业分工、交叉验证 |
| **安全护栏** | Pydantic 结构验证 | 输入过滤+输出验证+业务规则检查 | 安全可靠、格式规范 |
| **MCP** | 标准化工具接口 | MCPToolset 连接各种 MCP Server | 一次开发、多处复用 |

---

## 技术栈推荐

| 场景 | 推荐框架 | 说明 |
|------|----------|------|
| 快速原型 | LangChain | 丰富的组件和社区支持 |
| 复杂工作流 | LangGraph | 状态机和条件分支 |
| 多智能体 | CrewAI | 角色定义和任务协作 |
| Google 生态 | Google ADK | 与 Gemini 深度集成 |
| 工具标准化 | MCP | 未来的工具协议标准 |

---

## 模式选择决策树

```
需要构建 Agent 系统？
│
├─ 任务可分解？ ─────────────→ Prompt Chaining
│
├─ 需要动态分发？ ───────────→ Routing
│
├─ 需要自我改进？ ───────────→ Reflection
│
├─ 需要多角色协作？ ─────────→ Multi-Agent
│
├─ 需要调用外部工具？ ───────→ Tool Use / MCP
│
├─ 需要记住历史？ ───────────→ Memory Management
│
├─ 需要人工确认？ ───────────→ Human-in-the-Loop
│
└─ 需要安全控制？ ───────────→ Guardrails
```

---

## 行动清单

### 🚀 立即可做（今天）
- [ ] **克隆原版仓库**：`git clone https://github.com/sarwarbeing-ai/Agentic_Design_Patterns.git`
- [ ] **克隆中文仓库**：`git clone https://github.com/ginobefun/agentic-design-patterns-cn.git`
- [ ] **环境配置**：
  ```bash
  cd agentic-design-patterns-cn
  python3 -m venv venv
  source venv/bin/activate  # Windows: venv\Scripts\activate
  pip install langchain langchain-community langchain-openai langgraph
  ```
- [ ] **配置 API 密钥**：创建 `.env` 文件并添加 `OPENAI_API_KEY`
- [ ] **运行第一个示例**：`python codes/Chapter-01-Prompt-Chaining-Example.py`

### 📖 短期学习（本周）
- [ ] **精读核心模式**：完成第 1-7 章的中文版学习
- [ ] **对比学习**：英文原版 + 中文翻译双语对照
- [ ] **实践验证**：
  - [ ] 实现一个提示链 Agent（信息提取→格式转换→输出）
  - [ ] 构建反思模式 Agent（代码生成→评审→改进）
  - [ ] 尝试路由 Agent（多领域问答分发）

### 🏗️ 中期实践（本月）
- [ ] **多智能体协作**：使用 CrewAI 搭建团队协作系统
- [ ] **安全护栏**：集成 Pydantic 验证和异常处理
- [ ] **记忆管理**：实现对话历史和知识积累
- [ ] **工具集成**：连接外部 API 和数据库

### 🎯 长期提升（持续）
- [ ] **完整项目**：构建个人 Agent 系统（组合多种模式）
- [ ] **生产部署**：添加监控、评估和资源优化
- [ ] **社区贡献**：参与中文翻译项目的改进
- [ ] **前沿跟踪**：关注 MCP/A2A 生态发展

## 💡 学习路径建议

### 初学者路径（0-1 个月）
1. **基础概念**：阅读介绍和第 1-4 章（提示链、路由、并行、反思）
2. **动手实践**：运行所有基础模式的代码示例
3. **项目练习**：构建一个简单的多步骤 Agent

### 进阶路径（1-3 个月）
1. **高级模式**：学习第 5-11 章（工具使用、规划、多智能体、记忆）
2. **框架对比**：LangChain vs CrewAI vs Google ADK
3. **生产考虑**：安全、监控、资源优化

### 专家路径（3+ 个月）
1. **完整系统**：设计和实现复杂的多智能体系统
2. **标准协议**：深入 MCP 和 A2A 协议的实现
3. **创新探索**：结合最新研究，探索新的设计模式

---

## 常见误区与最佳实践

| 误区 | 正确做法 |
|------|----------|
| ❌ 一个 Agent 做所有事情 | ✅ 按职责拆分多个专业 Agent |
| ❌ 忽略错误处理 | ✅ 实现异常处理和 Fallback 机制 |
| ❌ 无限制调用 LLM | ✅ 设置反思循环的最大迭代次数 |
| ❌ 输出直接使用 | ✅ 通过 Pydantic 验证输出结构 |
| ❌ 硬编码路由规则 | ✅ 使用 LLM 进行语义路由 |
| ❌ 记忆无限增长 | ✅ 实现记忆摘要和淘汰策略 |

---

## 个人思考

这个项目最大的价值在于：
1. **系统化**：将分散的 Agent 技术整理成完整的设计模式体系
2. **实战性**：每个模式都有完整的代码实现，可直接运行学习
3. **前瞻性**：涵盖了 MCP、A2A 等最新的标准化协议
4. **可组合性**：模式之间可以自由组合，构建复杂系统

未来 Agent 开发的趋势：
- 标准化协议（MCP/A2A）将成为主流
- 多智能体协作将是复杂任务的标准解决方案
- 安全和可观测性将成为生产部署的必备要素

---

## 延伸阅读

### 官方资源
- [LangChain 官方文档](https://python.langchain.com/) - 链式调用和 Agent 构建
- [CrewAI 文档](https://docs.crewai.com/) - 多智能体框架
- [Google ADK](https://google.github.io/adk-docs/) - Google 的 Agent 开发套件
- [MCP 规范](https://modelcontextprotocol.io/) - Anthropic 的工具协议标准
- [Building Effective Agents (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents) - Anthropic 官方 Agent 构建指南

### 中文学习资源
- [Jimmy Song 在线版](https://jimmysong.io/zh/book/agentic-design-patterns/) - 专业排版的中文翻译
- [GitHub 中文仓库](https://github.com/ginobefun/agentic-design-patterns-cn) - 可运行的双语代码
- [中文技术社区](https://github.com/ginobefun/agentic-design-patterns-cn/discussions) - 学习讨论和问题解答

### 相关项目
- [LangGraph 教程](https://langchain-ai.github.io/langgraph/) - 复杂工作流构建
- [AutoGen 框架](https://microsoft.github.io/autogen/) - 微软的多智能体框架
- [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/) - 微软的 AI 编排框架

## 🤝 社区参与

### 贡献方式
- **翻译改进**：Fork [中文仓库](https://github.com/ginobefun/agentic-design-patterns-cn)，提交 PR
- **错误反馈**：在 GitHub Issues 中报告翻译错误或技术问题
- **经验分享**：在 Discussions 中分享实践经验和项目案例
- **代码贡献**：补充更多的实践示例和应用场景

### 版权说明
- **原书版权**：Springer 出版社，版税捐赠给救助儿童会
- **翻译协议**：CC BY-NC 4.0（署名-非商业性使用）
- **使用限制**：允许学习交流，禁止商业使用