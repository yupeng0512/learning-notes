# Planning with Files Skill 深度解析

> 精读日期：2026-01-20
> 来源：https://github.com/OthmanAdi/planning-with-files
> 口号：Work like Manus — the AI agent company Meta acquired for $2 billion.

---

## 一、根节点公式

```
Context Window = RAM (volatile, limited)
Filesystem = Disk (persistent, unlimited)

→ Anything important gets written to disk.
```

**核心洞察**：AI Agent 的瓶颈不是模型能力，而是上下文管理。

---

## 二、问题定义

Claude Code（以及大多数 AI Agent）存在的问题：

| 问题 | 描述 |
|------|------|
| **Volatile memory** | TodoWrite 工具在 context reset 后消失 |
| **Goal drift** | 50+ 次工具调用后，原始目标被遗忘 |
| **Hidden errors** | 错误没有被记录，导致重复犯错 |
| **Context stuffing** | 所有内容塞进 context 而不是存储 |

---

## 三、解决方案：3-File Pattern

```
task_plan.md      → 阶段和进度追踪（主控文件）
findings.md       → 研究和发现存储
progress.md       → 会话日志和测试结果
```

### 关键规则

1. **Create Plan First** — 永远不要在没有 task_plan.md 的情况下开始
2. **The 2-Action Rule** — 每 2 次浏览操作后，立即保存发现到文件
3. **Read Before Decide** — 做重大决策前，重新阅读计划文件（Attention Manipulation）
4. **Log ALL Errors** — 所有错误都要记录
5. **Never Repeat Failures** — 追踪尝试过的方法，变换方式

---

## 四、核心技巧深挖

### 4.1 Attention Manipulation（注意力操控）

> 决策前重读 task_plan.md = 强制刷新 Attention 窗口

**原理**：
- Transformer 的 Attention 机制对「最近出现的 token」更敏感
- 重读计划 = 把目标重新拉到 Attention 窗口的前排
- 等于给 AI 做「工作记忆刷新」

**类比**：
```
人类：开会前重读会议议程
AI Agent：执行前重读 task_plan.md
```

### 4.2 The 2-Action Rule

```
if action_count >= 2 && action_type in [view, browser, search]:
    IMMEDIATELY save to findings.md
```

**为什么是 2 而不是 3 或 5？**
- 多模态信息（图片、PDF）在 context 中衰减最快
- 2 是「安全边际」，超过 2 就有遗忘风险

### 4.3 3-Strike Error Protocol

```
ATTEMPT 1: Diagnose & Fix（诊断修复）
ATTEMPT 2: Alternative Approach（换方法）
ATTEMPT 3: Broader Rethink（质疑假设）
AFTER 3 FAILURES: Escalate to User（升级给用户）
```

**设计哲学**：
- 给 AI 足够的自主空间（3 次机会）
- 但设置明确的止损点
- 避免无限循环

---

## 五、关联历史笔记

> 以下是与本文**主题相关**的历史学习笔记（按相关性排序，非时间顺序）

| 历史笔记 | 关系类型 | 关联说明 |
|----------|----------|----------|
| [Agent Skills for Context Engineering](../agent-skill/2026-01-09-agent-skills-context-engineering.md) | **深化** | 同为 Context Engineering 实践，Planning with Files 是 3-File Pattern，该项目是完整的 5 大技能模块体系 |
| [AgeMem 统一长短期记忆框架](../../ai-research/2026-01-16-agentic-memory-unified-ltm-stm.md) | **对比** | 都解决 AI 记忆问题，Planning with Files 用外部文件（工程方案），AgeMem 用 RL 训练（学术方案）|
| [Claude-Mem 持久化记忆系统](../agent-architecture/2026-01-17-claude-mem-analysis.md) | **互补** | Claude-Mem 自动捕获 + 语义压缩，Planning with Files 手动规划 + 显式存储 —— 两者可结合使用 |
| [Agentic Patterns](../agent-skill/2026-01-04-agentic-patterns.md) | **应用** | Planning with Files 是「规划模式（Planning/ReAct）」的工程化实现 |

**知识网络**：

```
Planning with Files（Context Engineering 实践）
│
├─ 深化：Agent Skills for Context Engineering → 单一模式 vs 完整体系
├─ 对比：AgeMem → 工程方案（文件）vs 学术方案（RL 训练）
├─ 互补：Claude-Mem → 自动捕获 vs 手动规划
└─ 应用：Agentic Patterns → 规划模式的工程化实现
```

### AI 记忆方案对比

| 方案 | 存储位置 | 触发方式 | 用户控制 | 适用场景 |
|------|----------|----------|----------|----------|
| **Planning with Files** | 本地文件 | 手动 | 完全控制 | 复杂任务执行 |
| **Claude-Mem** | 本地 + 向量库 | 自动 Hook | 部分控制 | 日常开发记忆 |
| **AgeMem** | 外部记忆系统 | RL 学习 | 无直接控制 | 学术研究 |
| **Claude Cowork 知识库** | 云端 | 用户管理 | UI 可控 | 长期偏好存储 |

**洞察**：不同方案针对不同的记忆层级（L1 任务层 → L3 身份层），可组合使用

---

## 六、Manus 20 亿美金的秘密

### 6.1 技术路线

```
Context Engineering > Prompt Engineering
```

Manus 创始人的原话：
> "Markdown is my 'working memory' on disk."

### 6.2 商业价值

- 8 个月做到 1 亿美金营收
- 被 Meta 以 20 亿美金收购
- 证明了「AI Agent 的瓶颈是上下文管理」

### 6.3 KV-Cache 优化

Manus 的 context engineering 让相同前缀的 KV-Cache 可以复用：
- 成本下降 10x
- 这是 Manus 能盈利的核心原因

---

## 七、5-Question Reboot Test

如果你能回答这 5 个问题，说明 context 管理到位：

| 问题 | 答案来源 |
|------|----------|
| Where am I? | task_plan.md 当前阶段 |
| Where am I going? | 剩余阶段 |
| What's the goal? | 目标声明 |
| What have I learned? | findings.md |
| What have I done? | progress.md |

---

## 八、Anti-Patterns 反模式

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| 用 TodoWrite 做持久化 | 创建 task_plan.md |
| 目标说一次就忘 | 决策前重读计划 |
| 隐藏错误静默重试 | 记录错误到计划文件 |
| 所有内容塞进 context | 大内容存文件 |
| 立即开始执行 | 先创建计划文件 |
| 重复失败的操作 | 追踪尝试，变换方法 |

---

## 九、实践行动清单

### 立即可做

- [ ] 下一个复杂任务，先创建 task_plan.md
- [ ] 建立个人的 3-File Pattern 模板
- [ ] 在 Claude Code 中测试 Attention Manipulation 效果

### 深入探索

- [ ] 研究 Manus 的 KV-Cache 优化细节
- [ ] 对比 Cursor/Windsurf 的 context 管理策略
- [ ] 设计自己的 Hook 系统

### 长期目标

- [ ] 把 Planning with Files 集成到自己的 Agent 工作流
- [ ] 结合 Claude Cowork 知识库，构建分层记忆系统

---

## 十、深层思考

### Q1: 为什么文件比内存好？

**表面答案**：文件持久，内存易失
**深层答案**：文件迫使你「显式化」思考过程，而显式化本身就是一种对齐机制

### Q2: Context Engineering 会成为新的护城河吗？

Manus 证明了：
- 模型能力趋同后，差异化在于「怎么用」
- 上下文管理 = 产品体验的核心

### Q3: 这个 Skill 的局限性？

- 依赖用户遵守规则（没有强制机制）
- 文件数量增多后，管理成本上升
- 需要用户理解 Context Window 原理

---

## 参考资料

- [GitHub: OthmanAdi/planning-with-files](https://github.com/OthmanAdi/planning-with-files)
- [Manus AI 被 Meta 收购报道](https://www.theinformation.com/articles/meta-acquires-manus-ai-for-2-billion)
- [Context Engineering 概念解析](https://www.anthropic.com/research/context-engineering)
