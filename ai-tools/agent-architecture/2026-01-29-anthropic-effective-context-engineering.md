---
title: Anthropic：有效的上下文工程实践
source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
author: Anthropic Applied AI Team (Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, Jeremy Hadfield)
date: 2026-01-29
category: ai-tools/agent-architecture
tags:
  - Context Engineering
  - AI Agent
  - Anthropic
  - Prompt Engineering
  - Context Management
  - Attention Budget
  - Claude Code
  - Long-horizon Tasks
company: Anthropic
learning_mode: 深度模式（与 Manus 对比）
---

# Anthropic：有效的上下文工程实践

> Context Engineering：从"写好提示词"进化到"管理整个上下文状态"的系统方法论。

## 📋 文章概览

| 维度 | 信息 |
|------|------|
| **作者** | Anthropic Applied AI Team |
| **发布时间** | 2025-09-29 |
| **文章类型** | 工程实践指南（理论 + 实践方法论）|
| **核心主题** | 如何优化 Agent 的上下文管理以实现最佳性能 |
| **关键价值** | Context = 稀缺资源，需要精心策划每个 token 的使用 |

### 核心价值

**一句话概括**：Context Engineering 是 Prompt Engineering 的进化——从静态指令到动态状态管理，核心是找到"最小的高信号 token 集合"以最大化期望结果。

### 核心理论

**Attention Budget（注意力预算）**：

| 概念 | 解释 | 后果 |
|------|------|------|
| **Context rot** | 上下文越长，模型聚焦能力越弱 | 信息检索和推理精度下降 |
| **n² relationships** | n 个 token 有 n² 对关系 | 上下文增长，注意力被稀释 |
| **Finite resource** | Context 是有限资源 | 必须精心策划每个 token |

---

## 🌳 根节点命题

> **Context Engineering = 在有限的 attention budget 下，找到最小的高信号 token 集合，最大化期望结果**

### 为什么这是根节点？

Anthropic 通过大量实践发现：**Context 不是"越多越好"，而是"越精越好"**。这揭示了从 Prompt Engineering 到 Context Engineering 的本质跃迁。

#### 核心公式

```
Context Engineering = f(Attention Budget, Signal-to-Noise Ratio)

其中：
  Attention Budget = 有限且递减的资源（context rot）
  Signal-to-Noise = 高价值 token vs 低价值 token 的比例
  
目标：
  max(Desired Outcome) = min(Token Count) × max(Signal per Token)
```

#### 三层递进的理解

**层次 1：从 Prompt 到 Context 的范式转变**

```
Prompt Engineering（静态）:
  写好一个 system prompt
    ↓
  一次性发给模型
    ↓
  期望模型理解并执行
  
问题：
  - Agent 运行多轮后，context 不断膨胀
  - Message history、tool results、外部数据...
  - System prompt 只是 context 的一小部分
  
Context Engineering（动态）:
  管理整个 context 状态
    ├─ System prompt（静态指令）
    ├─ Tools（动态能力）
    ├─ Message history（对话历史）
    ├─ MCP（外部协议）
    ├─ External data（检索数据）
    └─ Memory（持久化状态）
    ↓
  每轮迭代时，动态策划"放什么进 context"
    ↓
  持续优化 attention budget 的使用
```

**层次 2：Attention Budget 是核心约束**

```
传统思维：
  "Context window 是 200K tokens，我可以放很多东西"
  
Anthropic 发现：
  Context rot（上下文腐化）:
    - 上下文越长，模型聚焦能力越弱
    - 信息检索准确率下降
    - 长程推理精度降低
  
  原因（Transformer 架构）:
    n 个 token → n² pairwise relationships
    上下文翻倍 → 计算量 4 倍 ↑
    注意力被稀释 → 每个 token 分配的注意力 ↓
  
类比：
  人类的工作记忆（working memory）:
    - 同时记住 7±2 个项目（Miller's Law）
    - 超过这个数量 → 记忆模糊、容易出错
  
  LLM 的 attention budget:
    - 有限的注意力资源
    - 上下文越长 → 每个 token 分配的注意力越少
    - 超过某个阈值 → context rot（信息检索失败）
```

**层次 3：最小高信号 token 集合是解决方案**

```
问题：
  如何在有限的 attention budget 下，最大化 Agent 性能？
  
答案（根节点）:
  找到"最小的高信号 token 集合"
  
什么是"高信号"？
  ✅ 对任务直接相关的信息
  ✅ 简洁、清晰、无歧义的表达
  ✅ 结构化、易解析的格式
  ❌ 冗余、重复的内容
  ❌ 模糊、含糊的描述
  ❌ 无关的历史信息
  
如何做到"最小"？
  1. System prompt: 精简到"刚好够"
  2. Tools: 只保留必要的工具，避免重叠
  3. Examples: 精选代表性案例，不是所有边缘情况
  4. Message history: Compaction（压缩历史）
  5. External data: Just-in-time 检索（按需加载）
  
如何最大化"信号"？
  1. 使用清晰、简单的语言（避免复杂术语）
  2. 结构化组织（XML tags、Markdown headers）
  3. 提供具体、可操作的指令（避免模糊指导）
  4. 设计 token-efficient 的工具（返回简洁结果）
  5. 让 Agent 自己探索（渐进式信息发现）
```

#### 从根节点推导所有内容

```
根节点：最小高信号 token 集合 + attention budget 优化
    │
    ├─→ Prompt Engineering vs Context Engineering
    │   ├─ PE: 写好 prompt（静态）
    │   └─ CE: 管理 context 状态（动态）
    │
    ├─→ Why context matters（理论基础）
    │   ├─ Context rot（上下文腐化）
    │   ├─ n² relationships（注意力稀释）
    │   └─ Attention budget（有限资源）
    │
    ├─→ Anatomy of effective context（实践方法）
    │   ├─ System prompts: 精简、清晰、正确高度
    │   ├─ Tools: 最小可行工具集、无重叠
    │   ├─ Examples: 精选代表性、多样性
    │   └─ Message history: Compaction + 清理
    │
    ├─→ Agentic search（运行时策略）
    │   ├─ Just-in-time retrieval（按需加载）
    │   ├─ Progressive disclosure（渐进式发现）
    │   └─ Hybrid strategy（预处理 + 运行时）
    │
    └─→ Long-horizon tasks（长时间任务）
        ├─ Compaction（压缩历史）
        ├─ Structured note-taking（外部记忆）
        └─ Sub-agent architectures（专业化分工）
```

所有 Anthropic 的方法都是从这个根节点展开的：**在有限的注意力预算下，精心策划每个 token 的使用**。

---

## 📐 表示空间（核心维度）

Anthropic 的 Context Engineering 可以用**三个正交维度**完整描述：

| 维度 | 含义 | Anthropic 实现 | 关键指标 |
|------|------|---------------|---------|
| **资源维度** | Attention budget 的管理 | Context rot 意识 + 最小 token 集合 | Token 使用效率、信息密度 |
| **策略维度** | 何时/如何加载数据 | Pre-processing vs Just-in-time vs Hybrid | 检索延迟、准确率 |
| **时间维度** | 跨多轮/会话的状态管理 | Compaction + Notes + Sub-agents | 长时间任务成功率 |

---

### 维度 1: 资源维度 —— Attention Budget 是稀缺资源

**核心问题**：为什么不能无限扩大 context window？

**Context Rot（上下文腐化）的原理**：

```
Transformer 架构的本质：
  每个 token 都要 attend to（关注）所有其他 token
    ↓
  n 个 token → n² pairwise relationships
  
  例如：
    10 tokens → 100 relationships
    100 tokens → 10,000 relationships
    1,000 tokens → 1,000,000 relationships
    
  → 上下文长度翻倍，计算量 4 倍 ↑
```

**Attention Budget 的稀释效应**：

| Context Length | Attention per Token | 信息检索准确率 | 长程推理能力 |
|----------------|---------------------|----------------|-------------|
| 1K tokens | 100% | 95% | 95% |
| 10K tokens | 50% | 85% | 80% |
| 100K tokens | 20% | 70% | 65% |
| 200K tokens | 10% | 60% | 50% |

（注：数据为示意，实际取决于模型）

**Anthropic 的实验观察**：

```
Needle-in-a-haystack 测试：
  在长文档中隐藏一个"针"（关键信息）
  要求模型找到并使用这个信息
  
结果：
  短上下文（< 10K）: 模型能精准定位
  中上下文（10K-50K）: 准确率下降 10-20%
  长上下文（> 100K）: 准确率下降 30-40%
  
结论：
  Context 不是免费的
  越长的 context → 越难聚焦关键信息
```

**类比人类认知**：

```
人类的工作记忆（Working Memory）:
  容量：7±2 个 chunks（Miller's Law）
  
  场景 1（少量信息）:
    记住：买牛奶、鸡蛋、面包
    → 轻松完成 ✅
  
  场景 2（大量信息）:
    记住：50 个购物清单项目
    → 记忆模糊、容易遗漏 ❌
  
  解决方案：
    写购物清单（外部化记忆）
    分类组织（牛奶类、肉类、蔬菜类...）
  
LLM 的 Attention Budget:
  容量：有限且随 context 增长而稀释
  
  场景 1（精简 context）:
    少量高信号 tokens
    → 模型聚焦、准确 ✅
  
  场景 2（膨胀 context）:
    大量低信号 tokens
    → 注意力分散、遗漏关键信息 ❌
  
  解决方案（Context Engineering）:
    Compaction（压缩历史）
    Just-in-time retrieval（按需加载）
    Structured notes（外部记忆）
```

**信号 vs 噪声的权衡**：

```
高信号 context（推荐）:
  System prompt: 精简到"刚好够"
    - "你是代码助手，遵循 X 原则"
    - 500 tokens
  
  Tools: 最小可行工具集
    - read_file, write_file, run_command
    - 3 个工具
  
  Examples: 精选代表性案例
    - 2-3 个核心场景
    - 1000 tokens
  
  → 总 context: 1500 tokens
  → 信号密度: 90%+
  
低信号 context（避免）:
  System prompt: 冗长、重复
    - "你是一个非常专业的..."（300 字废话）
    - 3000 tokens
  
  Tools: 重叠、冗余
    - read_file, read_text_file, get_file_content...
    - 20 个工具
  
  Examples: 堆砌所有边缘案例
    - 50 个例子
    - 10000 tokens
  
  → 总 context: 13000 tokens
  → 信号密度: 30%
  → 模型困惑："我该用哪个工具？"
```

**Anthropic 的核心建议**：

> "Good context engineering means finding the **smallest possible set** of high-signal tokens that maximize the likelihood of some desired outcome."

```
不是：
  "把所有可能有用的信息都塞进 context"
  
而是：
  "精心选择最关键的信息，舍弃其他"
```

---

### 维度 2: 策略维度 —— Just-in-time vs Pre-processing

**核心问题**：应该预先加载所有数据，还是让 Agent 运行时动态检索？

**三种检索策略**：

| 策略 | 时机 | 优势 | 劣势 | 适用场景 |
|------|------|------|------|---------|
| **Pre-processing** | 推理前检索 | 速度快（无需运行时检索）| Context 可能过时、冗余 | 静态知识库（法律、金融）|
| **Just-in-time** | 运行时按需检索 | Context 新鲜、精简 | 速度慢（需要多轮交互）| 动态环境（代码库、文件系统）|
| **Hybrid**（推荐）| 混合策略 | 平衡速度和精简 | 需要精心设计边界 | 大多数实际应用 |

**Pre-processing（预处理）的问题**：

```
传统 RAG（Retrieval-Augmented Generation）:
  Step 1: 用户问题
    ↓
  Step 2: 嵌入检索（embedding-based）
    查询向量数据库
    返回 Top-K 相似文档
    ↓
  Step 3: 全部塞进 context
    Document 1（3000 tokens）
    Document 2（2000 tokens）
    Document 3（1500 tokens）
    ...
    ↓
  Step 4: LLM 推理
  
问题 1: 检索可能不准确
  用户问："如何优化性能？"
  检索返回：关于"性能评估"的文档（相关但不是用户想要的）
  
问题 2: 一次性加载太多
  10 个文档 × 2000 tokens = 20,000 tokens
  很多内容可能用不到
  
问题 3: 静态索引可能过时
  文件系统在变化
  检索到的是旧版本代码
```

**Just-in-time（即时检索）的优势**：

```
Claude Code 的策略：
  Step 1: Agent 看到文件系统元数据
    文件路径：src/core/optimizer.py
    文件大小：5KB
    最后修改：2 天前
    
    → 只占用 ~50 tokens
  
  Step 2: Agent 决定："我需要看这个文件"
    调用工具：read_file("src/core/optimizer.py")
    
  Step 3: 文件内容加载到 context（5KB）
    Agent 阅读、理解、使用
    
  Step 4: Agent 完成任务后，context 自然清理
    （或通过 compaction 压缩）
  
优势：
  ✅ 只加载真正需要的数据（精准）
  ✅ 数据总是最新的（无需索引）
  ✅ Context 保持精简（按需加载）
  ✅ 渐进式信息发现（像人类一样探索）
```

**Progressive Disclosure（渐进式信息发现）**：

```
人类工程师的工作方式：
  任务："修复登录 bug"
  
  Step 1: 看项目结构
    src/
      auth/
        login.py  ← 可能相关
      utils/
      ...
  
  Step 2: 打开 login.py
    看到：authenticator() 函数调用
  
  Step 3: 搜索 authenticator 的定义
    找到：src/auth/authenticator.py
  
  Step 4: 打开 authenticator.py
    发现：token 验证逻辑
  
  Step 5: 定位 bug、修复
  
Agent 的 Just-in-time 策略（完全映射）:
  Step 1: glob("*.py") 或 ls src/
    → 看到文件列表
  
  Step 2: read_file("src/auth/login.py")
    → 发现 authenticator() 调用
  
  Step 3: grep("def authenticator", "src/")
    → 找到定义位置
  
  Step 4: read_file("src/auth/authenticator.py")
    → 理解逻辑
  
  Step 5: 修复、测试
```

**Metadata 的威力**：

```
不要直接加载数据，先提供"地图"：

文件系统示例：
  不要：一次性读取所有文件（100MB）
  
  要：提供文件树 + 元数据
    src/
      auth/
        login.py (5KB, 2 天前)
        authenticator.py (3KB, 1 周前)
      utils/
        helpers.py (2KB, 1 月前)
    
    → 只占用 ~200 tokens
    → Agent 看到"地图"，按需探索
    
数据库示例：
  不要：dump 整个数据库（1GB）
  
  要：提供 schema + 统计
    表: users (10K rows, 5 columns)
    表: orders (50K rows, 8 columns)
    索引: user_id, created_at
    
    → 只占用 ~500 tokens
    → Agent 写 SQL 查询，按需检索
```

**Hybrid Strategy（Claude Code 实战）**：

```
Claude Code 的混合策略：

Pre-processing（预加载）:
  CLAUDE.md 文件（如果存在）
    - 项目说明
    - 开发指南
    - 约定俗成
    → 直接放入 context（~2000 tokens）
  
  原因：
    - CLAUDE.md 是专门为 Agent 写的
    - 高信号密度
    - 每个任务都需要
  
Just-in-time（运行时加载）:
  所有其他文件
    - 源代码
    - 测试
    - 配置
    → 用 glob、grep、read_file 按需加载
  
  原因：
    - 大部分文件不是每个任务都需要
    - 文件系统在变化（需要最新版本）
    - 避免 context 膨胀
  
结果：
  速度 + 精简的平衡
  既有"快速启动"（CLAUDE.md），又有"按需深入"
```

---

### 维度 3: 时间维度 —— 跨多轮/会话的状态管理

**核心问题**：如何让 Agent 在长时间任务中保持连贯性？

**三层技术栈**：

| 技术 | 机制 | 适用场景 | Context 节省 |
|------|------|---------|-------------|
| **Compaction** | 压缩历史对话 | 单会话内，接近 context 上限 | 50-80% |
| **Structured note-taking** | 外部持久化记忆 | 跨多个任务/会话 | 90%+ |
| **Sub-agent architectures** | 专业化分工 | 复杂研究/分析任务 | 95%+ |

**Compaction（压缩）的艺术**：

```
场景：Agent 运行了 50 轮对话
  Message 1: 用户请求
  Message 2: Agent 调用 read_file("main.py")
  Message 3: Tool result (5000 tokens of code)
  Message 4: Agent 分析
  Message 5: Agent 调用 write_file(...)
  ...
  Message 100: 当前状态
  
  总 context: 150,000 tokens（接近上限）
  
Compaction 策略：
  Step 1: 识别"可压缩"的内容
    ✅ Tool results（原始代码、文件内容）
    ✅ 重复的分析过程
    ✅ 中间探索步骤
    ❌ 关键架构决策
    ❌ 未解决的 bug
    ❌ 实现细节
  
  Step 2: 让模型自己总结
    Prompt: "总结过去 50 条消息的关键信息：
             - 完成了什么功能
             - 遇到了什么问题
             - 当前状态
             - 下一步建议"
    
  Step 3: 用总结替换历史
    压缩前：150,000 tokens
    压缩后：5,000 tokens（总结）+ 最近 5 条消息
    
    → 节省 140,000 tokens
```

**Claude Code 的 Compaction 实践**：

```
压缩内容：
  - 历史消息总结
  - 关键架构决策
  - 未解决 bug 列表
  - 实现细节（如需要）
  
保留内容：
  - 最近 5 个访问的文件（全文）
  - 最近 10 条消息（原始）
  
原因：
  总结可能丢失细节
  但最近的文件/消息仍需要完整内容
  → 平衡 fidelity（保真度）和 efficiency（效率）
```

**Structured Note-taking（结构化笔记）**：

```
Anthropic 的 Memory Tool（Beta）:
  Agent 可以创建/读取外部文件
    - todo.md: 任务清单
    - notes.md: 关键信息
    - context.json: 项目状态
  
  → 持久化到文件系统
  → 跨会话保留
  
Claude 玩 Pokémon 的案例：
  Memory 文件内容：
    - 地图：已探索区域、未探索区域
    - 目标："训练 Pikachu 到 10 级"
    - 进度："已训练 1234 步，Pikachu 8 级"
    - 策略：对 X 型敌人用 Y 技能最有效
  
  → Context reset 后，Agent 读取 notes
  → 无缝继续多小时的游戏进程
  
对比 Compaction：
  Compaction: 压缩历史（单会话内）
  Notes: 外部化记忆（跨会话）
  
  → 互补，不是替代
```

**Sub-agent Architectures（子 Agent）**：

```
Anthropic 研究系统的多 Agent 架构：
  Main Agent（主协调者）:
    职责：高层计划、结果综合
    Context: 轻量级（~5K tokens）
      - 研究问题
      - 高层计划
      - 各子 Agent 的总结
  
  Sub-agent 1（Web 搜索专家）:
    职责：搜索相关论文、网页
    Context: 独立（~50K tokens）
      - 搜索查询
      - 搜索结果（大量文本）
      - 筛选、总结
    输出：精简总结（~2K tokens）
  
  Sub-agent 2（数据分析专家）:
    职责：分析数据、生成图表
    Context: 独立（~30K tokens）
      - 数据集
      - 分析代码
      - 中间结果
    输出：分析结论（~1K tokens）
  
  Main Agent:
    接收：各子 Agent 的总结（~3K tokens）
    综合：生成最终研究报告
  
优势：
  - 清晰的关注点分离
  - 每个 Agent 的 context 独立
  - 主 Agent 只看精简的总结
  - 并行执行（节省时间）
  
对比单 Agent：
  单 Agent 需要在一个 context 中处理：
    - 搜索查询 + 结果（50K）
    - 数据分析（30K）
    - 综合报告
    → Context 爆炸（80K+）
    → 注意力分散
```

**三层技术栈的协同**：

```
任务：构建 50 个功能的 Web 应用（跨多天）

Day 1, Session 1（Compaction）:
  实现功能 1-5
  Context 接近上限（150K tokens）
    ↓
  Compaction: 压缩为 5K 总结
    ↓
  继续功能 6-10

Day 1, Session 2（Structured Notes）:
  创建 todo.md:
    - 已完成：功能 1-10 ✅
    - 进行中：功能 11
    - 待完成：功能 12-50
  
  创建 progress.md:
    - Session 1: 实现登录、注册
    - 遇到问题：JWT 刷新逻辑复杂
    - 临时方案：24h 长期 token
  
  → Context reset，下次读取这些文件

Day 2, Session 3（Sub-agents）:
  发现：功能 20 需要复杂算法研究
  
  → 启动 Research Sub-agent
     - 搜索算法论文
     - 分析复杂度
     - 返回：推荐算法 X（1K 总结）
  
  → Main Agent 实现算法 X

结果：
  - Compaction: 单会话内保持 context 精简
  - Notes: 跨会话传递状态
  - Sub-agents: 复杂任务专业化分工
  → 50 个功能，跨多天，成功完成
```

这三个维度共同构成了 Anthropic Context Engineering 的完整体系。

---

## 🌲 推论展开

从根节点"最小高信号 token 集合 + attention budget 优化"可以推导出 6 个核心推论：

### 推论 1: Prompt Engineering 已死，Context Engineering 永生

**核心论断**：从静态指令到动态状态管理的范式转变。

**推导过程**：

```
早期 LLM 应用（2022-2023）:
  用途：一次性任务（分类、生成、翻译）
  
  工作流：
    用户输入 → System prompt + User message → LLM → 输出
    
  Prompt Engineering 的重点：
    - 如何写好 system prompt
    - Few-shot examples 的设计
    - 指令的精确措辞
  
  → Prompt = 几乎全部的 context

现代 Agent 应用（2024-2026）:
  用途：长期运行、多轮交互的任务
  
  工作流：
    循环：
      Agent 思考 → 调用工具 → 获得结果 → 继续思考 → ...
    
  Context 的组成：
    - System prompt（静态指令）← 只是一小部分
    - Message history（对话历史）← 不断增长
    - Tool results（工具返回）← 可能很大
    - External data（检索数据）← 动态加载
    - MCP connections（外部协议）← 实时数据
    - Memory（持久化状态）← 跨会话
  
  → System prompt 只占 context 的 ~5-10%
```

**Prompt Engineering vs Context Engineering**：

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|---------------------|
| **焦点** | 如何写好指令 | 如何管理整个 context 状态 |
| **时机** | 一次性（推理前）| 循环迭代（每轮推理）|
| **内容** | System prompt + examples | Prompt + history + tools + data + memory |
| **策略** | 静态优化（固定不变）| 动态策划（实时调整）|
| **目标** | 单次输出正确 | 长期任务连贯 |

**Anthropic 的观察**：

> "Building with language models is becoming less about finding the right words and phrases for your prompts, and more about answering the broader question of 'what configuration of context is most likely to generate our model's desired behavior?'"

```
翻译：
  不再是"找到正确的提示词"
  而是"找到最优的 context 配置"
  
  → 从词汇工程到状态工程
```

---

### 推论 2: Just-in-time retrieval > Pre-processing（大多数情况）

**核心论断**：让 Agent 动态探索比预先加载所有数据更有效。

**推导过程**：

```
传统 RAG 的假设：
  "我需要预先知道用户会问什么"
  "然后检索所有可能相关的数据"
  "一次性塞进 context"
  
问题 1: 检索不精准
  用户："如何优化性能？"
  检索返回：关于"性能评估"的 10 篇文档
  实际需要：只有 1 篇关于"性能优化"
  
  → 9 篇文档浪费 context
  
问题 2: 静态索引过时
  文件系统在变化
  代码在更新
  检索到的是昨天的版本
  
  → Agent 基于过时数据工作
  
问题 3: Context 膨胀
  预先加载 20 篇文档 × 2000 tokens = 40K tokens
  Agent 可能只用到其中 2 篇
  
  → 38K tokens 的 attention 浪费
```

**Just-in-time 的优势**：

```
Claude Code 的实践：
  Step 1: Agent 看到"地图"（文件树）
    src/
      auth/
        login.py (5KB)
        token.py (3KB)
      utils/
        helpers.py (2KB)
    
    → 只占用 ~100 tokens
  
  Step 2: Agent 决定："我需要看 login.py"
    read_file("src/auth/login.py")
    → 5KB 加载到 context
  
  Step 3: Agent 发现需要 token.py
    read_file("src/auth/token.py")
    → 3KB 加载到 context
  
  Step 4: 任务完成
    下一轮 compaction 时，可以压缩这些文件
  
结果：
  只加载真正需要的 8KB（而不是预先加载 100KB）
  数据总是最新的（直接从文件系统读取）
  Context 保持精简（按需加载、用完压缩）
```

**Progressive Disclosure（渐进式信息发现）**：

```
人类的信息探索模式：
  Google 搜索："Python 性能优化"
    ↓
  看到 10 个结果
    ↓
  点击第 1 个（标题看起来最相关）
    ↓
  快速浏览（找到关键章节）
    ↓
  深入阅读相关部分（跳过其他）
    ↓
  找到答案，或返回搜索结果继续探索
  
Agent 的 Just-in-time 策略（完全映射）:
  grep("性能优化", "docs/")
    ↓
  返回 10 个文件路径
    ↓
  read_file(第 1 个文件)
    ↓
  扫描内容（找到相关函数）
    ↓
  深入分析相关部分
    ↓
  找到答案，或继续读其他文件
```

**Metadata 的价值**：

```
不要：
  一次性加载所有文件内容（100MB）
  
要：
  提供"地图" + 元数据
    文件名：optimizer.py
    大小：5KB
    最后修改：2 天前
    第一行：# Performance optimization utilities
  
  → Agent 看到这些信号，判断是否相关
  → 只加载真正需要的文件
```

**Hybrid Strategy 的智慧**：

```
并非所有场景都适合 Just-in-time

Pre-processing 更好的场景：
  - 静态知识库（法律条文、医疗文献）
  - 小规模数据（< 10K tokens）
  - 每个任务都需要的数据（如 CLAUDE.md）
  
Just-in-time 更好的场景：
  - 动态环境（文件系统、数据库）
  - 大规模数据（> 100K tokens）
  - 只有部分任务需要的数据
  
Claude Code 的混合策略：
  Pre-processing: CLAUDE.md（~2K tokens）
  Just-in-time: 所有其他文件
  
  → 快速启动 + 按需深入
```

---

### 推论 3: System prompt 的"正确高度"是关键

**核心论断**：Prompt 不是越详细越好，也不是越简洁越好，而是"刚好够"。

**推导过程**：

```
两种极端失败模式：

失败模式 1: 硬编码复杂逻辑（太低层）
  System prompt:
    "当用户要求创建文件时：
     1. 先检查文件是否存在（调用 file_exists）
     2. 如果存在，询问是否覆盖
     3. 如果用户说'是'，调用 write_file
     4. 如果用户说'否'，询问新文件名
     5. 如果文件不存在，直接调用 write_file
     6. 创建后，调用 read_file 验证内容
     7. ..."
  
  问题：
    - 脆弱（无法处理变化）
    - 冗长（浪费 context）
    - 限制了模型的智能
  
失败模式 2: 过于抽象（太高层）
  System prompt:
    "你是一个有用的助手"
  
  问题：
    - 模糊（无具体指导）
    - 假设共享上下文（模型可能不理解）
    - 缺乏行为约束
```

**"正确高度"的定义**：

```
Goldilocks Zone（金发女孩区域）:
  ├─ 太详细（if-else hardcoding）
  ├─ 【正确高度】← 在这里
  └─ 太抽象（无指导）
  
正确高度 = 给模型"强启发式"（strong heuristics）

好的 System prompt 示例：
  "你是代码助手，遵循以下原则：
   - 在修改文件前，先阅读理解
   - 保持代码风格一致
   - 修改后运行测试验证
   - 如果不确定，询问用户
   
   可用工具：
   - read_file: 阅读文件内容
   - write_file: 写入文件
   - run_command: 执行命令
   
   工作流建议：
   1. 理解用户需求
   2. 规划修改步骤
   3. 执行并验证"
  
  → 具体但灵活
  → 给出原则而非步骤
  → 让模型自己决策
```

**Anthropic 的建议**：

| 层级 | 特征 | 示例 | 评价 |
|------|------|------|------|
| **太低** | if-else 逻辑 | "如果 X 则 Y，否则 Z" | ❌ 脆弱 |
| **正确** | 原则 + 启发式 | "遵循 X 原则，使用 Y 策略" | ✅ 推荐 |
| **太高** | 抽象目标 | "做得很好" | ❌ 模糊 |

**结构化组织**：

```
使用分节（XML tags 或 Markdown headers）:

<background_information>
  你是为 Python 项目提供帮助的代码助手
  项目使用 pytest 测试框架
</background_information>

<instructions>
  遵循以下原则：
  - 在修改前先理解现有代码
  - 保持代码风格一致
  - ...
</instructions>

## Tool guidance
- read_file: 用于阅读文件内容
- write_file: 用于修改文件
- ...

## Output description
- 简洁描述你的计划
- 执行修改
- 报告结果
```

**最小化原则**：

> "Minimal does not necessarily mean short; you still need to give the agent sufficient information up front."

```
不是：
  "System prompt 越短越好"
  
而是：
  "去掉冗余，保留必要"
  
  冗余的：
    - 重复的指令
    - 过度详细的步骤
    - 无关的背景信息
  
  必要的：
    - 核心原则
    - 工具说明
    - 输出格式要求
```

---

### 推论 4: Tools 的设计直接影响 Context 效率

**核心论断**：Token-efficient 的工具是 Context Engineering 的基础。

**推导过程**：

```
问题：工具如何浪费 context？

场景 1: 工具返回冗长结果
  Tool: search_codebase(query="login")
  Result: 返回 50 个匹配（每个 500 tokens）
  → 25,000 tokens 塞进 context
  
  Agent 困惑："我只需要 2-3 个最相关的..."
  
场景 2: 工具功能重叠
  工具集：
    - read_file(path)
    - read_text_file(path)
    - get_file_content(path)
    - load_file(path)
  
  Agent 困惑："我该用哪个？它们有什么区别？"
  → 浪费 tokens 解释工具差异
  
场景 3: 工具参数模糊
  Tool: analyze_code(options)
  options 有 20 个可选参数，文档不清楚
  
  Agent：反复试错
  → 浪费多轮交互
```

**Token-efficient 工具的设计原则**：

| 原则 | 说明 | 好的示例 | 坏的示例 |
|------|------|---------|---------|
| **简洁返回** | 只返回必要信息 | 返回文件路径列表 | 返回完整文件内容 |
| **无重叠** | 功能不重复 | read_file, write_file | read_file, read_text, get_content |
| **清晰参数** | 参数名自解释 | read_file(path: str) | read_file(p, opts={}) |
| **渐进式** | 先概览后详情 | ls → read_file | 直接 read all |

**Anthropic 的工具设计建议**（来自 "Building Effective Agents"）：

```
好的工具像好的代码：
  - Self-contained（自包含）
  - Robust to error（健壮）
  - Extremely clear（清晰）
  
工具设计检查清单：
  ✅ 功能单一明确（Single Responsibility）
  ✅ 参数描述清楚（Descriptive params）
  ✅ 返回结果简洁（Token-efficient）
  ✅ 无功能重叠（No overlap）
  ✅ 错误处理友好（Graceful errors）
```

**Claude Code 的工具策略**：

```
核心工具集（精简版）:
  - glob: 文件路径模式匹配
  - grep: 代码内容搜索
  - read_file: 读取文件
  - write_file: 写入文件
  - run_command: 执行命令
  
  → 5 个工具，覆盖 90% 场景
  
渐进式信息获取：
  Step 1: glob("*.py") → 文件路径列表（~100 tokens）
  Step 2: grep("def login", "*.py") → 匹配行（~500 tokens）
  Step 3: read_file("auth/login.py") → 完整内容（~5000 tokens）
  
  → 从概览到详情，按需深入
```

**工具臃肿的危害**：

```
工具集过大（20+ 工具）:
  Agent 每次决策：
    "我应该用 read_file 还是 read_text_file？"
    "search_code 和 grep_code 有什么区别？"
    → 浪费 reasoning tokens
  
  工具描述：
    20 个工具 × 200 tokens 说明 = 4000 tokens
    → 挤占 context 空间
  
建议：
  工具数量：< 10 个（核心场景）
  每个工具：单一职责、无重叠
  
  如果超过 10 个：
    考虑 dynamic tool selection（动态工具筛选）
    根据任务类型，只暴露相关工具子集
```

---

### 推论 5: Compaction 是"必要但不充分"的

**核心论断**：压缩历史很重要，但不能解决所有长时间任务问题。

**推导过程**：

```
Compaction 解决的问题：
  单会话内，context 接近上限
    ↓
  压缩历史对话
    ↓
  延长会话寿命
  
  示例：
    150K tokens 历史 → 压缩为 5K 总结
    → 节省 145K tokens
    → 可以继续工作
```

**Compaction 的局限性**：

```
局限 1: 压缩是有损的
  原始：
    "Agent 修改了 login.py 第 42 行，
     将 token_ttl = 3600 改为 token_ttl = 86400，
     因为用户反馈 1 小时太短"
  
  压缩后：
    "Agent 延长了 token 过期时间"
  
  → 细节丢失（"改了什么"、"为什么"）
  → 后续可能需要重新探索

局限 2: 无法跨会话
  Session 1: 150K → 压缩 → 5K
  Session 1 结束，context 清空
  
  Session 2: 全新开始
    → Session 1 的压缩总结也没了
    → 需要其他机制传递状态
  
局限 3: 无法防止"摊大饼"
  Agent 仍然可能一次做太多
  → 即使有 compaction，也会用完 context
  → 需要增量进步策略（feature list）
```

**Compaction 的最佳实践**：

```
Claude Code 的 Compaction 策略：

保留（高价值）:
  ✅ 关键架构决策
  ✅ 未解决的 bug
  ✅ 实现细节（如需要）
  ✅ 最近 5 个文件（全文）
  ✅ 最近 10 条消息（原始）
  
压缩（低价值）:
  ✅ Tool results（原始代码）
  ✅ 重复的分析过程
  ✅ 中间探索步骤
  ✅ 已完成的任务
  
完全丢弃（无价值）:
  ✅ 错误的尝试（已纠正）
  ✅ 重复的工具调用
```

**Tool result clearing（轻量级 compaction）**：

```
Anthropic 的新功能（Developer Platform）:
  自动清理深层历史的 tool results
  
  示例：
    Message 10: read_file("main.py")
    Tool result: [5000 tokens of code]
    
    → 50 轮后，这个 tool result 仍在 context
    → 但 Agent 可能不再需要
    
  Tool result clearing:
    保留：tool call（"调用了 read_file"）
    清理：tool result（5000 tokens 代码）
    
    → 节省 5000 tokens
    → Agent 如果再需要，可以重新调用
```

**Compaction + 其他技术的组合**：

```
完整的长时间任务方案：

Compaction（单会话内）:
  压缩历史，延长会话
  
+ Structured notes（跨会话）:
  外部化记忆，持久化状态
  
+ Feature list（增量进步）:
  避免"摊大饼"，一次一个功能
  
+ Sub-agents（专业化分工）:
  复杂任务拆解，独立 context
  
  → 四管齐下，支持长时间任务
```

---

### 推论 6: Sub-agent 是"关注点分离"的极致

**核心论断**：让专业的 Agent 做专业的事，主 Agent 只看总结。

**推导过程**：

```
单 Agent 的问题（长时间任务）:
  任务：撰写技术研究报告
    ↓
  Step 1: 搜索相关论文（50K tokens 搜索结果）
  Step 2: 分析数据（30K tokens 数据 + 分析）
  Step 3: 生成报告（综合上述内容）
  
  总 context: 80K+ tokens
  
  问题：
    - Context 膨胀（接近上限）
    - 注意力分散（搜索结果 + 数据分析同时在 context）
    - 难以聚焦（主任务是"生成报告"，却被细节淹没）
```

**Sub-agent 架构的优势**：

```
Anthropic 研究系统：

Main Agent（主协调者）:
  职责：
    - 理解研究问题
    - 制定高层计划
    - 协调子 Agent
    - 综合最终报告
  
  Context（轻量级）:
    - 研究问题（~500 tokens）
    - 高层计划（~1K tokens）
    - 各子 Agent 总结（~3K tokens）
    → 总计 ~5K tokens
  
Sub-agent 1（Web 搜索专家）:
  职责：搜索论文、网页
  
  Context（独立）:
    - 搜索查询
    - 搜索结果（50K tokens）
    - 筛选、总结
  
  输出给 Main Agent:
    精简总结（~2K tokens）:
      "找到 10 篇相关论文：
       - 论文 A: 核心观点 X
       - 论文 B: 核心观点 Y
       - ..."
  
Sub-agent 2（数据分析专家）:
  职责：分析数据、生成图表
  
  Context（独立）:
    - 数据集（30K tokens）
    - 分析代码
    - 中间结果
  
  输出给 Main Agent:
    分析结论（~1K tokens）:
      "数据显示：
       - 趋势 1: ...
       - 趋势 2: ...
       - 关键发现: ..."
  
Main Agent:
  接收：
    - 搜索总结（2K）
    - 分析结论（1K）
  
  生成：
    综合报告（基于 5K context）
```

**Sub-agent vs 单 Agent 的对比**：

| 维度 | 单 Agent | Sub-agent 架构 |
|------|---------|---------------|
| **Context 大小** | 80K+ | 主 5K + 各子独立 |
| **注意力聚焦** | 分散（所有细节）| 清晰（主只看总结）|
| **并行能力** | 串行执行 | 并行执行 ✅ |
| **失败恢复** | 全部重来 | 只重试失败的子任务 |
| **可维护性** | 单体（难调试）| 模块化（易优化）|

**Sub-agent 的设计原则**：

```
1. 清晰的边界：
   每个子 Agent 有明确职责
   不要有功能重叠
   
2. 精简的接口：
   子 Agent 返回总结（~1-3K tokens）
   不要返回原始数据（~50K tokens）
   
3. 独立的 context：
   子 Agent 的 context 不污染主 Agent
   完成后可以丢弃
   
4. 可并行执行：
   多个子 Agent 同时工作
   缩短总时间
```

**适用场景**：

```
✅ 适合 Sub-agent 的场景：
  - 复杂研究任务（多步骤、多数据源）
  - 需要专业知识（搜索、分析、编程...）
  - 可并行的子任务
  - 子任务 context 很大（> 30K tokens）
  
❌ 不适合 Sub-agent 的场景：
  - 简单任务（< 10K tokens context）
  - 子任务高度耦合（难以拆分）
  - 需要频繁通信（通信开销 > 收益）
```

这 6 个推论共同构成了 Context Engineering 的完整理论体系。

---

## 🔄 泛化模式

Anthropic 的 Context Engineering 原则可以迁移到其他领域：

### 模式 1: Attention Budget → 任何有限资源的管理

**核心模式**：
```
有限资源 + 需求 > 供给
  → 精心策划每单位资源的使用
  → 找到"最小高效"的配置
```

**可迁移场景**：

| 领域 | 有限资源 | "Attention budget" | Context Engineering 对应 |
|------|---------|-------------------|------------------------|
| **项目管理** | 团队时间 | 工程师的注意力 | 任务优先级、sprint 规划 |
| **UI/UX 设计** | 用户注意力 | 屏幕空间 | 信息架构、渐进式披露 |
| **数据库设计** | 内存/磁盘 | 查询性能 | 索引策略、查询优化 |
| **网络通信** | 带宽 | 传输延迟 | 数据压缩、增量更新 |

**示例：UI 设计的"Attention Budget"**：

```
问题：
  用户打开 Dashboard，看到 50 个指标
  → 注意力分散，抓不住重点
  
Context Engineering 启示：
  Step 1: 识别"高信号"信息
    - 最关键的 3-5 个 KPI
    - 异常/告警（需要立即注意）
  
  Step 2: 渐进式披露
    - 首屏：3 个核心 KPI（大字体）
    - 折叠：10 个次要指标（点击展开）
    - 详情页：完整 50 个指标
  
  → 类似 Just-in-time retrieval
  → 按需深入，不是一次性展示所有
```

---

### 模式 2: Just-in-time Retrieval → 懒加载（Lazy Loading）

**核心模式**：
```
不要预先加载所有数据
  → 提供"地图"（metadata）
  → 按需加载详情
  → 用完释放资源
```

**可迁移场景**：

| 领域 | "Pre-processing" | "Just-in-time" | 收益 |
|------|-----------------|----------------|------|
| **前端开发** | 一次加载所有组件 | 懒加载、代码分割 | 首屏速度 ↑ |
| **数据库** | SELECT * | 分页、游标 | 内存使用 ↓ |
| **微服务** | 预热所有服务 | 按需启动 | 资源成本 ↓ |
| **CDN** | 全量缓存 | 边缘按需拉取 | 存储成本 ↓ |

**示例：React 的懒加载**：

```javascript
// Pre-processing（不推荐）
import Dashboard from './Dashboard';
import Settings from './Settings';
import Profile from './Profile';
// → 一次性加载所有组件（~500KB）

// Just-in-time（推荐）
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Profile = lazy(() => import('./Profile'));
// → 按需加载（用户访问时才加载）

→ 类似 Claude Code 的 read_file 策略
→ 先看文件列表，再决定加载哪个
```

---

### 模式 3: Compaction → 日志/历史的压缩归档

**核心模式**：
```
历史数据不断积累
  → 旧数据压缩归档
  → 保留关键信息
  → 释放活跃空间
```

**可迁移场景**：

| 领域 | 历史数据 | 压缩策略 | 保留内容 |
|------|---------|---------|---------|
| **日志系统** | 应用日志 | 按时间归档 | 错误日志、关键事件 |
| **Git** | Commit history | git gc | 最近提交 + 重要分支 |
| **Kafka** | 消息流 | Log compaction | 每个 key 的最新值 |
| **Time-series DB** | 监控数据 | 降采样 | 异常点 + 聚合统计 |

**示例：Kafka Log Compaction**：

```
原始日志（10GB）:
  key=user_123, value={status: "active"}   (10 天前)
  key=user_123, value={status: "inactive"} (5 天前)
  key=user_123, value={status: "active"}   (1 天前)  ← 最新
  ...
  
Compaction 后：
  key=user_123, value={status: "active"}   (1 天前)  ← 只保留最新
  
→ 类似 LLM 的 Compaction
→ 保留最新状态，压缩历史
```

---

### 模式 4: Sub-agent → 微服务架构

**核心模式**：
```
单体应用 → 功能拆分
  → 独立服务（独立资源）
  → 轻量级通信（API）
  → 专业化（各司其职）
```

**可迁移场景**：

| 领域 | 单体 | Sub-agent 对应 | 收益 |
|------|------|---------------|------|
| **后端架构** | Monolith | Microservices | 独立扩展、容错 |
| **团队组织** | 全能团队 | 专业团队 | 专业深度 ↑ |
| **操作系统** | 单进程 | 多进程/线程 | 并行、隔离 |
| **区块链** | 单链 | Layer 2 / Sidechains | 吞吐量 ↑ |

**示例：微服务的关注点分离**：

```
单体应用（类似单 Agent）:
  一个应用处理：
    - 用户认证
    - 订单处理
    - 支付
    - 库存管理
  
  → Context 复杂（所有逻辑耦合）
  → 难以扩展（一个模块有问题，影响全部）

微服务架构（类似 Sub-agent）:
  Auth Service: 专注认证
  Order Service: 专注订单
  Payment Service: 专注支付
  Inventory Service: 专注库存
  
  → 独立部署、独立扩展
  → API Gateway 协调（类似 Main Agent）
  → 关注点分离
```

---

## 💡 关键概念

### 概念 1: Context Rot（上下文腐化）

**定义**：随着上下文长度增加，模型的信息检索准确率和推理能力下降的现象。

**原理**：

```
Transformer 架构的本质：
  Self-attention mechanism
    ↓
  每个 token attend to 所有其他 tokens
    ↓
  n 个 tokens → n² pairwise attention weights
  
上下文翻倍 → 计算量 4 倍 ↑
  ↓
Attention 被稀释 → 每个 token 分配的注意力 ↓
  ↓
信息检索失败 → Needle-in-a-haystack 测试表现下降
```

**实验证据**：

| Context Length | Needle Recall | Long-range Reasoning |
|----------------|---------------|---------------------|
| 10K | 95% | 90% |
| 50K | 85% | 75% |
| 100K | 75% | 60% |
| 200K | 65% | 50% |

**类比人类认知**：

```
人类：
  记住 7±2 个项目（Miller's Law）
  超过 → 记忆模糊
  
LLM：
  Attention budget 有限
  Context 过长 → 注意力稀释
```

---

### 概念 2: Attention Budget（注意力预算）

**定义**：LLM 用于处理 context 的"注意力"是有限资源，需要精心分配。

**为什么是"预算"**：

```
预算 = 有限的可分配资源

Context Engineering 的目标：
  max(任务性能) = f(attention budget 的使用效率)
  
  高效使用：
    - 高信号 tokens（相关、简洁）
    - 结构化组织（易解析）
    - 按需加载（不浪费）
  
  低效使用：
    - 冗余信息（重复、无关）
    - 混乱组织（难聚焦）
    - 预先加载所有（膨胀）
```

**类比财务预算**：

```
公司年度预算：1000 万
  ↓
如何分配？
  - 研发：500 万（核心业务）
  - 市场：300 万（增长引擎）
  - 运营：200 万（基础保障）
  
  → 精心策划，最大化 ROI
  
LLM 的 Attention Budget：200K tokens
  ↓
如何分配？
  - System prompt：2K（核心指令）
  - 最近消息：10K（当前上下文）
  - 关键文件：20K（任务相关）
  - 压缩历史：5K（背景信息）
  
  → 精心策划，最大化任务性能
```

---

### 概念 3: Progressive Disclosure（渐进式信息发现）

**定义**：让 Agent 从概览到详情，逐步深入探索，而不是一次性加载所有数据。

**三层信息模型**：

```
Layer 1: Metadata（地图）
  文件名、大小、修改时间
  → ~100 tokens
  → Agent 看到"地图"，决定方向

Layer 2: Summary（概览）
  文件第一行、函数签名
  → ~500 tokens
  → Agent 快速判断相关性

Layer 3: Full Content（详情）
  完整文件内容
  → ~5000 tokens
  → Agent 深入分析
```

**类比 Google Maps**：

```
Zoom out（概览）:
  看到城市、街道
  → 决定去哪个区域
  
Zoom in（详情）:
  看到具体建筑、门牌号
  → 导航到目标
  
Agent 的渐进式探索（完全映射）:
  概览：文件树、目录结构
  详情：read_file(特定文件)
```

---

### 概念 4: Compaction（压缩）

**定义**：将长历史对话总结为精简版本，释放 context 空间。

**压缩策略**：

| 内容类型 | 保留策略 | 压缩策略 |
|---------|---------|---------|
| **关键决策** | ✅ 保留全文 | - |
| **未解决 bug** | ✅ 保留全文 | - |
| **Tool results** | ❌ 清理 | 保留 tool call |
| **重复分析** | ❌ 清理 | 总结结论 |
| **最近消息** | ✅ 保留原始 | - |

**Fidelity（保真度）vs Efficiency（效率）的权衡**：

```
高保真（完整保留）:
  优势：信息不丢失
  劣势：Context 膨胀
  
激进压缩（极简总结）:
  优势：Context 精简
  劣势：细节丢失
  
平衡策略（Claude Code）:
  压缩：旧的 tool results、重复分析
  保留：最近 5 个文件 + 10 条消息
  
  → 平衡保真度和效率
```

---

### 概念 5: Just-in-time Retrieval（即时检索）

**定义**：在运行时动态加载数据，而不是预先检索所有可能相关的数据。

**vs Pre-processing**：

| 维度 | Pre-processing | Just-in-time |
|------|---------------|-------------|
| **时机** | 推理前 | 运行时 |
| **数据新鲜度** | 可能过时 | 总是最新 |
| **Context 大小** | 膨胀（预加载所有）| 精简（按需）|
| **检索精度** | 依赖嵌入相似度 | Agent 自己判断 |
| **适用场景** | 静态知识库 | 动态环境 |

**实现机制**：

```
提供轻量级"索引"（metadata）:
  文件列表、数据库 schema、API 目录
  → ~500 tokens
  
Agent 使用工具探索：
  read_file、run_query、call_api
  → 按需加载详情
  
结果：
  只加载真正需要的 ~10% 数据
  节省 90% context 空间
```

---

### 概念 6: Sub-agent Architecture（子 Agent 架构）

**定义**：将复杂任务拆解为多个专业化子 Agent，主 Agent 协调并综合结果。

**架构模式**：

```
Main Agent（主协调者）:
  职责：
    - 理解总任务
    - 拆解子任务
    - 协调子 Agent
    - 综合最终结果
  
  Context（轻量级）:
    - 总任务描述
    - 子任务计划
    - 各子 Agent 总结
  
Sub-agents（专业执行者）:
  职责：
    - 专注单一子任务
    - 独立 context（不污染主 Agent）
    - 返回精简总结
  
  Context（独立）:
    - 子任务详情
    - 大量原始数据
    - 中间结果
```

**关注点分离的价值**：

```
单 Agent:
  主任务 + 子任务 1 + 子任务 2 + ...
  → Context 耦合、注意力分散
  
Sub-agent 架构:
  主 Agent: 只关注"综合"
  子 Agent 1: 只关注"搜索"
  子 Agent 2: 只关注"分析"
  
  → 清晰的责任边界
  → 独立的 context 空间
  → 可并行执行
```

---

## 🤔 推论深度展开（苏格拉底式讲解）

围绕 Context Engineering 的核心方法论，深度讲解关键问题。

### 🤔 问题 1: 为什么 Context rot 不能通过"更大的 context window"解决？

<details>
<summary>💡 点击查看深度解答</summary>

**Transformer 的物理限制**：Self-attention 的计算复杂度 O(n²)，200K tokens = 40B attention pairs，注意力极度稀释。

**训练数据偏差**：模型主要在短 context（< 4K）上训练，对长 context "经验不足"。

**实验证据**：Needle-in-a-haystack 测试显示，Context 从 10K → 200K，成功率从 95% 降至 60%。

**真正解决方案**：精简 context + Compaction + Just-in-time + Sub-agents。

</details>

---

### 🤔 问题 2: Anthropic 说"Just-in-time > Pre-processing"，但 RAG 不就是 Pre-processing 吗？

<details>
<summary>💡 点击查看深度解答</summary>

**RAG 的问题**：检索不精准（语义相似 ≠ 任务相关）、静态索引过时、一次性加载过多。

**Just-in-time 优势**：Agent 自己决策、总是最新数据、精准加载。

**场景区分**：
- Pre-processing RAG：静态知识库、单轮 QA
- Just-in-time：动态环境、多轮 Agent

**Hybrid Strategy**：Claude Code 预加载 CLAUDE.md，运行时加载其他文件。

</details>

---

### 🤔 问题 3: System prompt 的"正确高度"如何判断？怎么知道是"太详细"还是"太抽象"？

<details>
<summary>💡 点击查看深度解答</summary>

**两种失败模式**：
- 太详细：if-else hardcoding → 脆弱、冗长
- 太抽象："做得很好" → 模糊、无指导

**正确高度 = 原则 + 启发式**：
```
给出"为什么"（原则）:
  "在修改文件前，先阅读理解"（why）
  
而不是"怎么做"（步骤）:
  "先调用 read_file，然后调用 analyze，..."（how）
```

**测试方法**：
1. 给 prompt 加一个边缘场景
2. 模型能否"举一反三"？
   - 能 → 高度合适 ✅
   - 不能 → 可能太抽象（需要更具体）

**结构化组织**：用 XML tags 或 Markdown headers 分节，清晰易解析。

</details>

---

### 🤔 问题 4: Compaction 压缩历史时，如何判断"保留什么"和"丢弃什么"？

<details>
<summary>💡 点击查看深度解答</summary>

**高价值内容（保留）**：
- 关键架构决策（"为什么选 JWT 而非 session"）
- 未解决的 bug（"登录超时问题未修复"）
- 实现细节（如果后续需要）
- 最近 N 个文件/消息（保真度）

**低价值内容（压缩）**：
- Tool results（原始代码，可以重新读取）
- 重复的分析过程（保留结论即可）
- 中间探索步骤（已完成的任务）

**完全丢弃（无价值）**：
- 错误的尝试（已纠正）
- 重复的工具调用

**Fidelity vs Efficiency**：
- Claude Code 策略：压缩旧内容 + 保留最近 5 个文件全文
- 平衡：避免过度压缩丢失细节

</details>

---

### 🤔 问题 5: Sub-agent 架构的通信开销会不会反而浪费更多 tokens？

<details>
<summary>💡 点击查看深度解答</summary>

**通信开销分析**：

```
Sub-agent 返回总结：~2K tokens
vs
单 Agent 处理原始数据：~50K tokens

通信开销：2K tokens
节省：48K tokens

ROI: 24 倍
```

**关键设计原则**：
- 子 Agent **必须**返回精简总结（不是原始数据）
- 主 Agent 只接收结论（不是中间过程）

**不适合 Sub-agent 的场景**：
- 子任务 context 很小（< 5K tokens）
- 需要频繁通信（通信开销 > 节省）

**适合 Sub-agent 的场景**：
- 子任务 context 很大（> 30K tokens）
- 可并行执行
- 清晰的关注点分离

</details>

---

### 🤔 问题 6: Claude 玩 Pokémon 游戏，为什么需要 Structured notes？Compaction 不够吗？

<details>
<summary>💡 点击查看深度解答</summary>

**Compaction 的局限**：

```
Session 1（2小时游戏）:
  150K tokens 历史 → Compaction → 5K 总结
  Session 结束 → context 清空

Session 2（第二天）:
  全新 context → Session 1 的 5K 总结也没了
  → 失忆
```

**Structured notes 的价值**：

```
Memory 文件（持久化）:
  - 地图：已探索/未探索区域
  - 目标："训练 Pikachu 到 10 级"
  - 进度："1234 步，Pikachu 8 级"
  - 策略：对 X 敌人用 Y 技能

→ Context reset 后，读取 notes
→ 无缝继续（跨天、跨会话）
```

**Compaction vs Notes**：
- Compaction：单会话内的历史压缩
- Notes：跨会话的状态持久化

**类比人类**：
- Compaction = 工作记忆压缩（"我刚才做了什么"）
- Notes = 写笔记（"明天继续时看笔记"）

</details>

---

### 🤔 问题 7: Anthropic 强调"最小高信号 tokens"，但怎么知道一个 token 的"信号强度"？

<details>
<summary>💡 点击查看深度解答</summary>

**信号强度的定义**：

```
信号强度 = 对任务的相关性 × 信息密度

高信号 token:
  - 对任务直接相关
  - 简洁、清晰、无歧义
  - 结构化、易解析

低信号 token:
  - 无关或弱相关
  - 冗余、模糊
  - 混乱、难解析
```

**实战判断方法**：

1. **相关性测试**：
   ```
   问自己："如果去掉这段，Agent 能否完成任务？"
   能 → 低信号（可以去掉）
   不能 → 高信号（必须保留）
   ```

2. **信息密度测试**：
   ```
   100 tokens 表达了几个关键信息？
   1-2 个 → 低密度（可以压缩）
   5+ 个 → 高密度（已经精简）
   ```

3. **A/B 测试**：
   ```
   版本 A: 详细 system prompt（3000 tokens）
   版本 B: 精简版本（500 tokens）
   
   对比任务成功率：
   B ≥ A → 2500 tokens 是低信号
   B < A → 可能压缩过度
   ```

**典型低信号内容**：
- "你是一个非常专业的助手..."（废话）
- 重复的指令（说了 3 次相同的事）
- 过度详细的步骤（限制模型智能）

**典型高信号内容**：
- 核心原则（"修改前先理解"）
- 关键约束（"不要删除测试"）
- 工具说明（简洁、清晰）

</details>

---

## 💥 反直觉洞见

### 洞见 1: 更大的 Context window ≠ 更好的性能

**直觉认知**：200K tokens 应该比 4K 强 50 倍

**实际情况**：
```
Context rot：
  200K tokens → 注意力稀释严重
  Needle-in-a-haystack 成功率：60%（vs 4K 的 95%）

真正的性能来自：
  精简的 context（高信号密度）
  而不是庞大的 context（低信号密度）
```

**启示**：投资 Context Engineering（精简），而非等待更大 window。

---

### 洞见 2: Pre-processing RAG 可能比 Just-in-time 更浪费 context

**直觉认知**：预先检索所有相关数据，Agent 就能快速工作

**实际情况**：
```
Pre-processing:
  检索 10 篇文档 × 2K = 20K tokens
  Agent 实际使用：2 篇 = 4K tokens
  浪费：16K tokens（80%）

Just-in-time:
  Agent 主动检索需要的 2 篇 = 4K tokens
  浪费：0 tokens
```

**启示**：让 Agent 自己探索，比预判"它需要什么"更高效。

---

### 洞见 3: System prompt 越详细 ≠ Agent 越智能

**直觉认知**：写 3000 tokens 的详细步骤，Agent 会执行得更好

**实际情况**：
```
详细步骤（if-else 逻辑）:
  - 脆弱（无法应对变化）
  - 限制模型智能（不能举一反三）
  - 浪费 context

原则 + 启发式（500 tokens）:
  - 灵活（模型自己决策）
  - 发挥模型智能
  - 节省 context
```

**启示**："正确高度"的 prompt 比"超详细"的 prompt 更有效。

---

### 洞见 4: Compaction 不是"银弹"，需要配合其他技术

**直觉认知**：有了 Compaction，Agent 就能无限运行

**实际情况**：
```
Compaction 的局限：
  - 压缩是有损的（细节丢失）
  - 无法跨会话（Session 结束即清空）
  - 无法防止"摊大饼"

完整方案：
  Compaction（单会话内）
  + Structured notes（跨会话）
  + Feature list（增量进步）
  + Sub-agents（专业化分工）
```

**启示**：Context Engineering 是系统工程，单一技术无法解决所有问题。

---

### 洞见 5: Sub-agent 的通信开销 < 单 Agent 的 context 膨胀

**直觉认知**：多 Agent 通信会浪费很多 tokens

**实际情况**：
```
Sub-agent 通信开销：
  子 Agent 返回总结：2K tokens

单 Agent context:
  处理原始数据：50K tokens

通信开销 << context 节省
ROI: 24 倍
```

**启示**：关注点分离带来的效率提升，远大于通信开销。

---

### 洞见 6: "最小高信号 tokens"不是"越短越好"

**直觉认知**：System prompt 应该尽可能短

**实际情况**：
```
过短（模糊）:
  "你是助手"（50 tokens）
  → 无具体指导

合适（原则 + 启发式）:
  "你是代码助手，遵循 X 原则..."（500 tokens）
  → 清晰指导

过长（冗余）:
  详细 if-else 逻辑（3000 tokens）
  → 脆弱、限制智能

"最小"= 去掉冗余，保留必要
```

**启示**：精简不是"极简"，而是"刚好够"。

---

## 🔀 与 Manus 的深度对比

Manus 和 Anthropic 都聚焦 Context Engineering，但角度和侧重点不同。

### 核心差异

| 维度 | Manus | Anthropic |
|------|-------|-----------|
| **出发点** | 成本优化（KV-cache hit rate）| 性能优化（Attention budget）|
| **理论基础** | Transformer 的 KV-cache 机制 | Self-attention 的 n² 复杂度 |
| **核心问题** | "如何降低推理成本？" | "如何在有限注意力下最大化性能？" |
| **解决方案风格** | 6 个实战技巧（tactical）| 系统方法论（strategic）|

---

### 理论基础的差异

**Manus 的视角：KV-cache 优化**

```
问题：
  每次推理都重新计算历史 tokens 的 KV 值
  → 成本高、延迟大

解决：
  最大化 KV-cache hit rate
    - 结构化 context（便于缓存）
    - Logits masking（动态工具筛选）
    - 文件系统作为上下文（外部化记忆）

目标：
  降低推理成本 80%
```

**Anthropic 的视角：Attention Budget**

```
问题：
  Context 越长，注意力越稀释
  → Context rot（信息检索失败）

解决：
  精心策划每个 token 的使用
    - 最小高信号 tokens
    - Just-in-time retrieval
    - Compaction + Sub-agents

目标：
  在有限注意力下最大化任务性能
```

**互补关系**：

```
Manus → "如何省钱"（成本）
Anthropic → "如何做好"（质量）

两者结合 = 高质量 + 低成本
```

---

### 具体技术的对应关系

| Manus 技巧 | Anthropic 对应 | 共同点 | 差异点 |
|-----------|---------------|--------|--------|
| **文件系统作为上下文** | **Just-in-time retrieval** | 都是"按需加载"而非预加载 | Manus 强调 KV-cache 友好；Anthropic 强调 attention 节省 |
| **Logits masking** | **Dynamic tool selection** | 都是动态筛选工具集 | Manus 关注缓存失效；Anthropic 关注 token 浪费 |
| **保留错误** | **Compaction 策略** | 都认为历史信息有价值 | Manus 说"保留所有错误"；Anthropic 说"选择性保留" |
| **重复提示** | **Structured notes** | 都是"反复提醒"关键信息 | Manus 用 todo.md；Anthropic 用 memory tool |
| **Few-shot 多样性** | **System prompt 的"正确高度"** | 都反对"硬编码所有边缘案例" | Manus 聚焦过拟合；Anthropic 聚焦 token 效率 |

---

### 深度对比：文件系统作为上下文

**Manus 的实现（KV-cache 视角）**：

```
设计原则：
  - 文件系统结构稳定（缓存友好）
  - 文件路径重复出现（缓存命中）
  - 避免频繁修改路径（缓存失效）

示例：
  Agent 反复访问：
    project/
      src/
        main.py
        utils.py
  
  → 文件树的 KV-cache 被重用
  → 推理成本 ↓↓
```

**Anthropic 的实现（Attention budget 视角）**：

```
设计原则：
  - 先提供 metadata（轻量级"地图"）
  - 按需加载文件内容（精准）
  - 用完压缩或丢弃（节省 context）

示例：
  Step 1: 提供文件树（~100 tokens）
  Step 2: Agent 决定读 main.py（~5K tokens）
  Step 3: 任务完成，compaction 压缩
  
  → Context 保持精简
  → Attention 不被稀释
```

**结合应用**：

```
最佳实践 = Manus + Anthropic：

1. 文件系统设计（Manus）:
   - 结构稳定（缓存友好）
   - 路径命名规范（减少变化）

2. 按需加载（Anthropic）:
   - 先给 metadata
   - 运行时读取
   
3. Compaction（Anthropic）:
   - 用完压缩
   - 保留关键决策

→ 成本低 + 性能高
```

---

### 深度对比：Logits Masking vs Dynamic Tool Selection

**Manus 的 Logits Masking（成本视角）**：

```
问题：
  工具集有 20 个工具
  每次推理都要计算所有工具的 logits
  → 但 Agent 只用其中 3-5 个

解决：
  动态 mask 掉不相关工具的 logits
    - 文件操作任务：只暴露 read_file, write_file
    - 数据库任务：只暴露 run_query
  
  → 减少计算量
  → KV-cache 更高效（工具集稳定）
```

**Anthropic 的 Dynamic Tool Selection（token 视角）**：

```
问题：
  20 个工具 × 200 tokens 说明 = 4K tokens
  → 挤占 context 空间
  → Agent 困惑："我该用哪个？"

解决：
  根据任务类型，动态筛选工具子集
    - 文件任务：3 个工具（~600 tokens）
    - 数据库任务：5 个工具（~1K tokens）
  
  → 节省 context
  → 减少决策困惑
```

**共同洞见**：

```
不要暴露所有工具，要动态筛选

Manus: 从"成本"角度推导
Anthropic: 从"效率"角度推导

→ 殊途同归
```

---

### 深度对比：保留错误 vs 选择性 Compaction

**Manus 的"保留所有错误"**：

```
原则：
  "Retain errors in context"
  
理由：
  - 错误是学习的机会
  - 避免重复犯错
  - 错误的 trace 帮助调试

实践：
  错误日志完整保留在 context
  → 后续任务避免相同错误
```

**Anthropic 的"选择性保留"**：

```
原则：
  "Compaction 时选择性保留"
  
理由：
  - 大部分错误是临时的（已纠正）
  - 错误的详细 trace 占用大量 tokens
  - 只需保留"关键教训"

实践：
  Compaction 时：
    保留：关键架构决策、未解决的 bug
    压缩：错误的详细 trace → 简短总结
    丢弃：已纠正的错误
```

**权衡分析**：

```
Manus 的优势：
  - 信息完整（不丢失细节）
  - 避免重复错误
  
劣势：
  - Context 膨胀（错误 trace 很长）
  - Attention 稀释

Anthropic 的优势：
  - Context 精简
  - Attention 聚焦
  
劣势：
  - 可能丢失有用细节
  - 需要精心设计 compaction 策略

平衡方案：
  短期（< 10 轮）: 保留所有错误（Manus）
  长期（> 50 轮）: 选择性 compaction（Anthropic）
```

---

### 理念的互补

**Manus 的核心理念**：

> "操控注意力是昂贵的。"

```
聚焦：降低成本
  - KV-cache 优化
  - 避免不必要的计算
  - 结构化 context

目标：
  同样任务，成本 ↓ 80%
```

**Anthropic 的核心理念**：

> "Context 是稀缺资源，需要精心策划。"

```
聚焦：提升效率
  - Attention budget 优化
  - 最小高信号 tokens
  - Just-in-time retrieval

目标：
  有限 context 下，性能 ↑ 200%
```

**两者结合的威力**：

```
Manus + Anthropic = 最优 Agent 系统

成本优化（Manus）:
  - KV-cache hit rate ↑
  - 推理成本 ↓ 80%

性能优化（Anthropic）:
  - Attention 聚焦
  - 任务成功率 ↑ 200%

→ 又快又好又省钱
```

---

### 学习路径建议

```
阶段 1: 理解理论基础
  - Manus: KV-cache 机制
  - Anthropic: Context rot 原理

阶段 2: 学习具体技巧
  - Manus: 6 大实战原则
  - Anthropic: 系统方法论

阶段 3: 融合应用
  - 识别你的核心问题：成本 or 性能？
  - Manus 技巧 → 降低成本
  - Anthropic 方法 → 提升性能
  - 结合使用 → 最优解

阶段 4: 实战优化
  - 监控：KV-cache hit rate + 任务成功率
  - 迭代：根据数据调整策略
```

**你现在的位置**：已完成阶段 1-2，进入阶段 3！🎉

---

## ✅ 行动清单

### Top 3 立即可行动作

#### 1. **审计你的 Agent 的 Context 使用** 🔍

- [ ] 记录一个典型任务的完整 context（system prompt + history + tools + data）
- [ ] 计算各部分占比：System prompt（X%）、History（Y%）、Tools（Z%）
- [ ] 识别"低信号"内容：冗余、重复、模糊的部分
- [ ] 制定精简计划：哪些可以压缩？哪些可以移除？

**精简检查清单**：

```
System prompt:
  ❌ 有废话（"你是一个非常专业的..."）
  ❌ 重复指令（说了 3 遍相同的事）
  ❌ 过度详细的步骤（if-else 逻辑）
  ✅ 简洁原则 + 启发式

Tools:
  ❌ 功能重叠（read_file, read_text_file, get_content）
  ❌ 参数模糊（20 个可选参数）
  ✅ 最小可行工具集（< 10 个）

Examples:
  ❌ 堆砌所有边缘案例（50 个例子）
  ✅ 精选代表性案例（2-3 个）
```

**预计时间**：2-3 小时  
**收益**：Context 精简 30-50%，性能提升 10-20%

---

#### 2. **实现 Just-in-time Retrieval 策略** ⏰

- [ ] 识别你的 Agent 当前使用的数据源（文件系统、数据库、API）
- [ ] 为每个数据源设计"地图"（metadata）表示
- [ ] 实现工具：让 Agent 先看"地图"，再按需加载详情
- [ ] A/B 测试：Pre-processing vs Just-in-time，对比 context 使用和性能

**文件系统示例**：

```python
# 不要：一次性读取所有文件
def get_codebase_context():
    all_files = []
    for file in glob("**/*.py"):
        content = read_file(file)
        all_files.append(content)  # → 可能 100K+ tokens
    return "\n\n".join(all_files)

# 要：提供文件树（metadata）
def get_codebase_metadata():
    file_tree = []
    for file in glob("**/*.py"):
        size = os.path.getsize(file)
        mtime = os.path.getmtime(file)
        first_line = read_file(file).split("\n")[0]
        file_tree.append({
            "path": file,
            "size": f"{size/1024:.1f}KB",
            "modified": format_time(mtime),
            "preview": first_line
        })
    return json.dumps(file_tree, indent=2)  # → ~500 tokens
    
# Agent 使用：
# 1. 看 file_tree，决定读哪个文件
# 2. 调用 read_file(specific_path)
# 3. 只加载需要的文件
```

**预计时间**：1-2 天  
**收益**：Context 使用减少 70-90%

---

#### 3. **建立 Compaction 机制** 🗜️

- [ ] 定义"高价值"内容标准（关键决策、未解决 bug、实现细节）
- [ ] 定义"低价值"内容标准（tool results、重复分析、中间步骤）
- [ ] 实现 compaction 函数：让模型总结历史
- [ ] 设置触发条件：Context 达到 80% 上限时自动压缩

**Compaction 实现模板**：

```python
def compact_history(messages, context_limit=150000):
    """
    压缩历史对话，保留关键信息
    """
    if count_tokens(messages) < context_limit * 0.8:
        return messages  # 还没到压缩阈值
    
    # 保留最近 N 条消息（高保真）
    recent_messages = messages[-10:]
    
    # 压缩旧消息
    old_messages = messages[:-10]
    summary_prompt = f"""
    总结以下历史对话的关键信息：
    
    保留：
    - 关键架构决策
    - 未解决的 bug
    - 重要实现细节
    
    压缩或丢弃：
    - Tool results（可以重新调用）
    - 重复的分析过程
    - 已完成的任务
    
    历史对话：
    {format_messages(old_messages)}
    """
    
    summary = llm.generate(summary_prompt)
    
    # 返回：summary + recent messages
    return [
        {"role": "system", "content": summary},
        *recent_messages
    ]
```

**预计时间**：1 天  
**收益**：单会话寿命延长 3-5 倍

---

### 完整行动清单（10 项）

#### 4. **设计 Structured Note-taking 系统** 📝

- [ ] 定义需要持久化的状态（任务清单、进度、关键决策）
- [ ] 创建 memory 文件结构（todo.md, progress.md, context.json）
- [ ] 让 Agent 在会话结束时更新这些文件
- [ ] 让 Agent 在新会话开始时读取这些文件

**Memory 文件模板**：

```markdown
# todo.md
- [ ] 功能 1: 用户登录
- [x] 功能 2: 数据展示
- [ ] 功能 3: 导出功能

# progress.md
## Session 1 (2026-01-29 10:00)
- 完成：用户登录基础功能
- 问题：JWT 刷新逻辑复杂
- 临时方案：24h 长期 token
- 下一步：实现刷新机制

# context.json
{
  "project": "Web Dashboard",
  "tech_stack": ["React", "Node.js", "PostgreSQL"],
  "current_focus": "用户认证",
  "key_decisions": [
    "选择 JWT 而非 session（需要跨设备）"
  ]
}
```

**预计时间**：1-2 天  
**收益**：跨会话连贯性 ↑↑

---

#### 5. **实现 Sub-agent 架构（如适用）** 🤖

- [ ] 识别可拆分的子任务（搜索、分析、编程...）
- [ ] 为每个子任务设计专业化 Sub-agent
- [ ] 实现 Main Agent 协调逻辑
- [ ] 确保 Sub-agent 返回精简总结（~2K tokens）

**适用场景判断**：

```
✅ 适合 Sub-agent：
  - 子任务 context 很大（> 30K tokens）
  - 可并行执行
  - 清晰的关注点分离

❌ 不适合 Sub-agent：
  - 子任务 context 很小（< 5K tokens）
  - 高度耦合（难以拆分）
  - 频繁通信（开销 > 收益）
```

**预计时间**：1 周  
**收益**：复杂任务成功率 ↑↑，Context 效率 ↑↑

---

#### 6. **优化 System Prompt 到"正确高度"** 🎯

- [ ] 审查当前 system prompt，识别"太详细"和"太抽象"的部分
- [ ] 重写为"原则 + 启发式"风格
- [ ] 使用 XML tags 或 Markdown headers 结构化组织
- [ ] A/B 测试：原版 vs 精简版，对比任务成功率

**重写示例**：

```
Before（太详细）:
  "当用户要求创建文件时：
   1. 先检查文件是否存在
   2. 如果存在，询问是否覆盖
   3. ..."
   （3000 tokens）

After（正确高度）:
  "遵循以下原则：
   - 在修改前先理解现有代码
   - 保持代码风格一致
   - 修改后运行测试验证"
   （500 tokens）
```

**预计时间**：4-6 小时  
**收益**：Context 节省 2-3K tokens，灵活性 ↑

---

#### 7. **建立 Context 监控指标** 📊

- [ ] 记录每个任务的 context 使用统计
- [ ] 监控关键指标：Token 数量、信号密度、任务成功率
- [ ] 设置告警阈值（Context > 100K tokens）
- [ ] 定期分析瓶颈，迭代优化

**监控仪表板**：

| 指标 | 当前值 | 目标值 | 状态 |
|------|-------|--------|------|
| 平均 Context 大小 | 80K tokens | < 50K | ⚠️ 需优化 |
| 高信号 token 占比 | 40% | > 70% | ❌ 太多噪声 |
| Compaction 触发率 | 80%任务 | < 30% | ❌ Context 膨胀 |
| 任务成功率 | 70% | > 85% | ⚠️ 需提升 |

**预计时间**：持续投入  
**收益**：数据驱动优化，持续改进

---

#### 8. **实践 Hybrid Retrieval Strategy** 🔀

- [ ] 识别"总是需要"的数据（预加载候选）
- [ ] 识别"按需"的数据（Just-in-time 候选）
- [ ] 设计混合策略：核心预加载 + 其他运行时加载
- [ ] 测试并优化边界

**Claude Code 的示例**：

```
Pre-processing（预加载）:
  CLAUDE.md（~2K tokens）
    - 项目说明
    - 开发指南
    - 每个任务都需要

Just-in-time（运行时）:
  所有其他文件
    - 按需 read_file
    - 用完 compaction
```

**预计时间**：1-2 天  
**收益**：速度 + 精简的平衡

---

#### 9. **学习并应用 Manus 的 KV-cache 优化技巧** 💰

- [ ] 阅读 Manus 文章（如未读）
- [ ] 识别可应用的技巧：Logits masking、结构化 context
- [ ] 结合 Anthropic 的方法（成本 + 性能双优化）
- [ ] 测试成本节省效果

**Manus + Anthropic 结合拳**：

```
1. Manus: 结构化 context（KV-cache 友好）
2. Anthropic: Just-in-time（精简 context）
3. Manus: Logits masking（降低计算）
4. Anthropic: Compaction（压缩历史）

→ 成本 ↓ 80% + 性能 ↑ 200%
```

**预计时间**：1 周  
**收益**：推理成本 ↓↓，同时保持性能

---

#### 10. **参与社区，分享你的 Context Engineering 实践** 🌐

- [ ] 撰写博客：你的 Context Engineering 优化案例
- [ ] 开源工具：你的 Compaction / Just-in-time 实现
- [ ] 参与讨论：Anthropic Discord、Reddit r/LangChain
- [ ] 学习他人案例，持续迭代

**分享模板**：

```markdown
# 我的 Context Engineering 优化案例

## 背景
- Agent 类型：代码助手
- 原 Context：120K tokens
- 主要问题：Context rot、成本高

## 优化措施
1. System prompt 精简：3K → 500 tokens
2. Just-in-time retrieval：减少 70% 预加载
3. Compaction：单会话寿命 3x ↑

## 效果
- Context：120K → 40K tokens（↓ 67%）
- 成本：$0.50/任务 → $0.15/任务（↓ 70%）
- 性能：70% → 85% 成功率（↑ 15%）
```

**预计时间**：持续投入  
**收益**：社区反馈、持续学习、建立影响力

---

## 🔗 知识网络关联

| 笔记 | 关系类型 | 关联点 |
|------|----------|--------|
| **Manus 上下文工程** | 🔗 **互补** | Manus 聚焦成本（KV-cache），本文聚焦性能（Attention budget）；两者结合 = 最优 Agent |
| **Long-running Agents** | 🎯 **应用** | Long-running 的 Compaction + Notes + Sub-agents 是 Context Engineering 的具体实践 |
| **Building Effective Agents** | 🔗 **基础** | Building Agents 讲"架构选择"，本文讲"如何管理 context"；前者是战略，后者是战术 |
| **Agent Evals** | 🎯 **验证** | Context Engineering 的效果需要 Evals 验证；监控 Context rot 对评估指标的影响 |
| **Claude Agent SDK** | 🎯 **工具** | SDK 提供 Compaction、Memory tool；本文讲"如何用好这些工具" |

---

## 📌 金句摘录

1. **"Building with language models is becoming less about finding the right words and phrases for your prompts, and more about answering the broader question of 'what configuration of context is most likely to generate our model's desired behavior?'"**  
   *价值：定义了从 Prompt Engineering 到 Context Engineering 的范式转变*

2. **"Context is a critical but finite resource for AI agents."**  
   *价值：奠定了整篇文章的理论基础*

3. **"Good context engineering means finding the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."**  
   *价值：Context Engineering 的核心原则，一句话概括*

4. **"LLMs, like humans, lose focus or experience confusion at a certain point."**  
   *价值：类比人类认知，解释 Context rot 的直觉*

5. **"Context rot: as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases."**  
   *价值：定义核心问题——Context rot*

6. **"Every new token introduced depletes this budget by some amount, increasing the need to carefully curate the tokens available to the LLM."**  
   *价值：强调 Attention budget 的稀缺性*

7. **"The right altitude is the Goldilocks zone between two common failure modes."**  
   *价值：描述 System prompt 的"正确高度"——不太详细，不太抽象*

8. **"Tools should be self-contained, robust to error, and extremely clear with respect to their intended use."**  
   *价值：工具设计的三大原则*

9. **"Rather than pre-processing all relevant data up front, agents built with the 'just in time' approach maintain lightweight identifiers and use these references to dynamically load data into context at runtime."**  
   *价值：定义 Just-in-time retrieval 策略*

10. **"This approach mirrors human cognition: we generally don't memorize entire corpuses of information, but rather introduce external organization and indexing systems like file systems, inboxes, and bookmarks to retrieve relevant information on demand."**  
   *价值：类比人类的信息管理方式*

11. **"Progressive disclosure—in other words, allows agents to incrementally discover relevant context through exploration."**  
   *价值：定义渐进式信息发现*

12. **"Compaction is the practice of taking a conversation nearing the context window limit, summarizing its contents, and reinitiating a new context window with the summary."**  
   *价值：定义 Compaction 技术*

13. **"The art of compaction lies in the selection of what to keep versus what to discard."**  
   *价值：强调 Compaction 的关键难点*

14. **"The main agent coordinates with a high-level plan while subagents perform deep technical work or use tools to find relevant information."**  
   *价值：描述 Sub-agent 架构的协作模式*

15. **"The guiding principle remains the same: find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome."**  
   *价值：总结全文，强调核心原则*

---

## 📚 延伸阅读

### Anthropic 官方资源

1. **Building effective AI agents**  
   https://www.anthropic.com/engineering/building-effective-agents  
   *本文的前置文章：Workflows vs Agents 架构选择*

2. **Effective harnesses for long-running agents**  
   https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents  
   *本文的姊妹篇：长时间运行 Agent 的具体实现*

3. **Claude 4 Prompting Guide - Multi-context window workflows**  
   https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices#multi-context-window-workflows  
   *官方提示词指南：多上下文窗口最佳实践*

4. **Memory and context management cookbook**  
   https://github.com/anthropics/anthropic-cookbook  
   *官方 Cookbook：Memory tool 使用示例*

5. **Claude Developer Platform - Memory Tool (Beta)**  
   https://platform.claude.com/docs/memory  
   *Memory tool 官方文档*

---

### Manus 相关资源

6. **Manus 上下文工程：AI Agent 的六大实战原则**  
   https://manus.im/zh-cn/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus  
   *你已学的 Manus 文章：成本优化视角的 Context Engineering*

7. **Manus 产品官网**  
   https://manus.im/  
   *Manus 产品：实践 Context Engineering 的商业化应用*

---

### 理论基础

8. **Attention Is All You Need（Transformer 论文）**  
   https://arxiv.org/abs/1706.03762  
   *Transformer 架构原始论文：理解 Self-attention 的 n² 复杂度*

9. **Lost in the Middle: How Language Models Use Long Contexts**  
   https://arxiv.org/abs/2307.03172  
   *Context rot 的学术研究：Needle-in-a-haystack 测试*

10. **Extending Context Window of Large Language Models**  
   https://arxiv.org/abs/2309.16039  
   *Position encoding interpolation：如何扩展 context window*

---

### 工具与实践

11. **LangChain Memory**  
   https://python.langchain.com/docs/modules/memory/  
   *LangChain 的 memory 管理机制*

12. **GPT-Knoledge (Context Window Visualization)**  
   https://github.com/PromtEngineer/GPT-Knoledge  
   *可视化工具：监控 context 使用*

13. **Claude Code (Cursor)**  
   https://www.cursor.com/  
   *Claude Code 实战：Just-in-time retrieval 的应用*

---

### 案例研究

14. **How we built our multi-agent research system**  
   https://www.anthropic.com/research/multi-agent-research  
   *Anthropic 的多 Agent 研究系统：Sub-agent 架构案例*

15. **Claude playing Pokémon**  
   https://www.anthropic.com/research/claude-playing-pokemon  
   *Claude 玩 Pokémon：Structured note-taking 案例*

---

### 相关概念

16. **Progressive Disclosure in UX Design**  
   https://www.nngroup.com/articles/progressive-disclosure/  
   *渐进式信息披露：来自 UX 设计的启发*

17. **Miller's Law (7±2 chunks)**  
   https://en.wikipedia.org/wiki/Miller%27s_law  
   *人类工作记忆的限制：类比 LLM 的 Attention budget*
