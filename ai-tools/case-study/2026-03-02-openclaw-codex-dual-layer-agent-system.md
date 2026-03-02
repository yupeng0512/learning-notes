---
title: OpenClaw + Codex/CC 双层 AI Agent 系统：一人 94 次提交的实战架构
source: https://x.com/elvissun/status/2025920521871716562
author: Elvis Sun (@elvissun)
date: 2026-03-02
category: ai-tools
subcategory: case-study
tags: [OpenClaw, Codex, Claude Code, Agent 编排, 双层架构, 独立开发者, B2B SaaS, Ralph Loop]
---

# OpenClaw + Codex/CC 双层 AI Agent 系统：一人 94 次提交的实战架构

> 📖 原文：[Elvis Sun Twitter Thread](https://x.com/elvissun/status/2025920521871716562)
> 📅 学习日期：2026-03-02
> 🏷️ 分类：AI 工具 / 开发案例

---

## 根节点命题

> **上下文的专业化分层（Context Specialization Layering），而非更强的模型，是让 AI Agent 系统从"能用"到"自动化生产力"的关键跃迁。**

**为什么这是根节点**：文章中所有令人惊叹的数据（日均 50 次提交、30 分钟 7 个 PR、$190/月成本）都不是来自更强的模型或更多的 token，而是来自一个核心架构决策——把"知道为什么做"（业务上下文）和"知道怎么做"（代码执行）分成两层。上下文窗口是固定的，你只能在业务上下文和代码上下文之间二选一。分层架构消除了这个困境，让每一层都在自己的领域达到最优。

**公式表达**：`系统生产力 = 编排层(业务上下文 × 决策质量) × 执行层(代码能力 × 并发数) × 反馈闭环(学习速率)`

---

## 表示空间

> 理解这套系统需要 4 个核心维度：

| 维度 | 含义 | 本文位置 |
|------|------|----------|
| **上下文分层** | 业务上下文与代码上下文是否分离 | 强分离：编排层持有全量业务上下文（会议记录、客户数据、历史决策），执行层仅接收任务级最小上下文 |
| **反馈闭环** | 系统是否能从失败中学习并改进 | 改进版 Ralph Loop：失败时动态重写 prompt（非静态重试），成功模式被记录复用 |
| **自主程度** | 人工介入的频率和深度 | 高度自主：cron 监控 + 三重 AI code review + CI 全绿后才通知人类，review 仅需 5-10 分钟 |
| **Agent 路由** | 是否根据任务特征选择合适的 Agent | 是：Codex=后端逻辑（90%）、Claude Code=前端/git、Gemini=设计审美，Zoe 自动路由 |

---

## 推论展开

> 从根节点推导出的核心结论（树状结构）

```
根节点：上下文专业化分层 = 自动化生产力的关键跃迁
│
├─ 推论1：编排层必须持有"全量业务上下文"，而不仅仅是代码上下文
│   ├─ 证据：Zoe 自动同步 Obsidian 会议记录、访问生产 DB（只读）、获取客户配置
│   ├─ 反例：直接用 Codex/CC，上下文窗口塞满代码 → 无空间放业务上下文
│   └─ 应用：零解释成本——作者和 Zoe 聊需求时，Zoe 已经知道客户背景
│
├─ 推论2：执行层只需要"完成任务的最小上下文"，刻意隔离其他信息
│   ├─ 安全边界：Agent 永远不接触生产数据库，看不到客户敏感信息
│   ├─ 效率收益：更少的上下文 → 更精准的代码输出 → 更低的 token 消耗
│   └─ 应用：类比"主厨 vs 厨师"——主厨管口味/库存/菜单，厨师只管做好手头的菜
│
├─ 推论3：反馈闭环必须是"动态学习"而非"静态重试"
│   ├─ 改进版 Ralph Loop：失败时 Zoe 分析原因 + 重写 prompt（而非同样 prompt 重试）
│   ├─ 奖励信号：CI 通过 + 三重 code review 通过 + 人工合并
│   ├─ 记忆沉淀："这种 prompt 结构对账单功能很有效"
│   └─ 应用：系统越用越聪明，prompt 质量随时间提升
│
├─ 推论4：Agent 路由策略让"对的 Agent 做对的事"
│   ├─ Codex (gpt-5.3)：90% 任务，后端逻辑、多文件重构、慢但彻底
│   ├─ Claude Code (opus-4.5)：前端、git 操作、速度型选手
│   ├─ Gemini：设计审美，先生成 HTML/CSS 规范再交给 CC 实现
│   └─ 应用：专业化分工 > 通用 Agent 独挑大梁
│
└─ 推论5："完成"的定义必须是多维度自动验证，而非 PR 创建
    ├─ 7 项检查：PR 创建 + 分支同步 + CI 通过 + 三重 review + UI 截图
    ├─ 确定性监控：cron 检查客观事实（tmux 存活、PR 状态、CI 结果），不消耗 token
    └─ 应用：人类只在"一切就绪"时才被打扰，review 从"审查"降级为"批准"
```

---

## 泛化模式

> 这个洞见可以迁移到哪些其他场景？

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| B2B SaaS 开发（编排层管理客户上下文） | **客户支持 Agent 集群** | 编排层持有客户画像/工单历史/SLA 要求，执行层 Agent 专注处理具体问题类型（技术/账单/功能） |
| OpenClaw 编排 Codex/CC 写代码 | **内容生产流水线** | 编排层持有品牌调性/受众画像/内容日历，执行层 Agent 分别负责文案/设计/SEO/分发 |
| 改进版 Ralph Loop 动态调整 prompt | **Trading System 自进化** | Reflect Agent 分析失败交易，动态调整策略 prompt 而非固定参数重试（你的 trading-system 已在做这件事！） |
| Agent 路由策略 | **DevOps 自动化** | 编排层分析 incident 类型，路由到不同 Agent：性能问题→分析 Agent，配置错误→修复 Agent，安全事件→响应 Agent |

---

## 关键概念

> 只保留理解根节点必需的概念

| 概念 | 定义 | 与根节点的关系 |
|------|------|----------------|
| **双层架构** | 编排层（OpenClaw）+ 执行层（Codex/CC/Gemini）的分层系统 | 根节点的物理实现形式，上下文分层的载体 |
| **Ralph Loop** | 记忆拉取 → 生成输出 → 评估结果 → 保存学习的循环 | 反馈闭环的基础框架，本文在此基础上做了"动态 prompt 重写"改进 |
| **Git Worktree** | Git 的轻量级并行工作目录，每个任务独立分支 | 并发执行的隔离机制，让多个 Agent 同时工作不冲突 |
| **tmux 会话** | 终端复用器，让 Agent 在后台持续运行 | 执行层的运行基础设施，支持中途人工干预 |
| **确定性监控** | cron 检查客观事实（tmux 存活/PR/CI 状态），不用 AI 判断 | 省 token 的监控策略，只在需要时触发人工介入 |
| **三重 Code Review** | Codex + Gemini + Claude Code 三个 Agent 交叉审查 | 质量保障机制，不同 Agent 有不同的审查优势 |
| **"完成"的 7 项标准** | PR + 同步 + CI + 三重 review + UI 截图 | 自动化的质量门禁，人类只处理"完全就绪"的 PR |

---

## 完整工作流：从客户需求到 PR 合并的 8 步

```
客户需求 ──→ ① Zoe 理解拆解（零解释成本，已读会议记录）
                │
                ├─ 给客户充值（管理员 API）
                ├─ 拉取客户配置（生产 DB 只读）
                └─ 生成 prompt 并启动 Agent
                        │
                        ▼
              ② 启动 Agent（git worktree + tmux）
                        │
                        ▼
              ③ 自动监控（cron 每 10 分钟，确定性检查）
                ├─ tmux 会话存活？
                ├─ PR 已创建？
                ├─ CI 状态？
                └─ 失败 → 最多重试 3 次
                        │
                        ▼
              ④ Agent 创建 PR（gh pr create --fill）
                        │
                        ▼
              ⑤ 三重 Code Review
                ├─ Codex：逻辑/边界/竞态（最靠谱）
                ├─ Gemini：安全/扩展性（免费）
                └─ Claude Code：过度谨慎（基本跳过）
                        │
                        ▼
              ⑥ 自动化 CI 测试
                ├─ Lint + TypeScript
                ├─ 单元测试 + E2E
                ├─ Playwright（预览环境）
                └─ UI 截图规则（无截图 → CI 失败）
                        │
                        ▼
              ⑦ 人工 Review（5-10 分钟，Telegram 通知）
                └─ 很多 PR 只看截图就合并
                        │
                        ▼
              ⑧ 合并 + 清理（cron 清理孤立 worktree）
```

**关键代码片段**：

```bash
# 创建隔离环境
git worktree add ../feat-custom-templates -b feat/custom-templates origin/main
cd ../feat-custom-templates && pnpm install

# 启动 Agent（tmux 后台运行）
tmux new-session -d -s "codex-templates" \
  -c "/path/to/worktree/feat-custom-templates" \
  "$HOME/.codex-agent/run-agent.sh templates gpt-5.3-codex high"

# 中途干预（不杀进程，直接发指令）
tmux send-keys -t codex-templates "停一下。先做 API 层,别管 UI。" Enter
```

**任务状态记录（JSON）**：
```json
{
  "id": "feat-custom-templates",
  "tmuxSession": "codex-templates",
  "agent": "codex",
  "description": "企业客户的自定义邮件模板功能",
  "repo": "medialyst",
  "worktree": "feat-custom-templates",
  "branch": "feat/custom-templates",
  "startedAt": 1740268800000,
  "status": "running",
  "notifyOnComplete": true
}
```

---

## 推论深度展开

### 推论 1：为什么"上下文窗口固定"是根本性的约束？

**问题**：Codex 和 Claude Code 的上下文窗口越来越大（200K+），为什么作者说"只能二选一"？

**回答**：窗口大小是必要条件但不是充分条件。即使窗口足够大，同时塞入代码库 + 业务上下文会产生三个问题：(1) 注意力稀释——模型在海量上下文中找到相关信息的精度下降；(2) 干扰——客户情感信息可能污染代码逻辑推理；(3) 安全风险——执行层不应该看到客户敏感数据。分层不是因为窗口不够大，而是因为**关注点分离（Separation of Concerns）本身就是更优的架构模式**。

### 推论 2：编排层和执行层之间的"信息翻译"如何保证质量？

**问题**：Zoe 把业务上下文翻译成 prompt 喂给 Codex，如果翻译不准确怎么办？

**回答**：这正是改进版 Ralph Loop 解决的问题。第一次翻译可能不准确，但系统有反馈机制：如果 Agent 产出不符合预期（CI 失败、review 不通过），Zoe 会分析失败原因，带着**客户原话**重写 prompt。关键洞察：Zoe 能做这种修正，因为她同时持有"客户说了什么"和"Agent 做错了什么"两个上下文。这是执行层自己永远做不到的。

### 推论 3：为什么用三个不同的 Agent 做 Code Review？

**问题**：一个强大的 reviewer 不就够了吗？为什么要三个？

**回答**：因为每个 Agent 有不同的"审查盲区"：Codex 擅长逻辑/边界情况但可能忽略安全问题；Gemini 擅长安全/扩展性但可能过度设计；Claude Code 过度谨慎但偶尔能抓到前两者遗漏的问题。三重 review 的本质是**用多样性对冲单点盲区**，类似于集成学习（Ensemble Learning）的思路。

### 推论 4：确定性监控为什么比 AI 监控更好？

**问题**：为什么不让 AI 自己判断"任务完成了没"？

**回答**：用 AI 判断进度需要消耗大量 token，而且 AI 可能"自以为完成"。作者用 cron 脚本检查客观事实（tmux 会话存活否、PR 是否已创建、CI 状态），这是 100% 确定性的，零 token 消耗。只有当需要分析失败原因时，才调用 AI。这体现了一个设计原则：**能用确定性逻辑解决的，就不要用概率性 AI**。

### 推论 5：为什么 RAM 是瓶颈而非 token 成本？

**问题**：AI 系统的瓶颈通常是 token/API 速率，为什么这里是内存？

**回答**：因为这套系统是本地运行的。每个 Agent 需要独立的 worktree + node_modules + TypeScript 编译器 + 测试运行器，5 个 Agent 并行 = 5 倍的内存消耗。16GB Mac Mini 最多跑 4-5 个 Agent。这揭示了一个反直觉的事实：**在本地多 Agent 系统中，物理资源（RAM/CPU）而非 AI 资源（token/API）才是真正的扩展瓶颈**。

### 推论 6：Zoe 的"主动找活干"机制是如何实现的？

**问题**：Zoe 不等分配就主动扫描 Sentry/会议记录/git log，这是怎么做到的？

**回答**：通过定时任务（cron）+ 数据源扫描 + 规则匹配。早上扫描 Sentry 新错误 → 自动启动调查 Agent；会议后扫描 Obsidian 笔记 → 识别功能需求 → 启动 Codex；晚上扫描 git log → 更新文档。本质是**把"人类需要手动检查的东西"变成自动化数据管道**，编排层根据规则自动触发任务。

### 推论 7："完成"的 7 项标准为什么比 4 项更好？

**问题**：大多数团队只要求"CI 通过 + 人工 review"就合并，为什么要 7 项？

**回答**：因为这套系统的目标是**让人类 review 降级为"批准"而非"审查"**。当 PR 到达作者面前时，CI 全绿 + 三个 AI 都说 OK + 截图展示了 UI 变化，作者只需 5-10 分钟甚至不看代码就能合并。7 项标准的高前置成本换来了极低的人工介入成本。这是**前置质量投资 vs 后置修复成本**的经典权衡。

---

## 反直觉洞见

### 1. 更多上下文 ≠ 更好结果

直觉上，给 AI 越多信息越好。但这篇文章证明：**刻意限制执行层的上下文**（最小必要原则）反而提高了产出质量。因为执行层不需要知道"这个客户上次为什么投诉"，它只需要知道"在这个文件里加一个函数"。

### 2. 一个人 + 系统 > 一个小团队

作者的 Git 历史看起来像"刚招了一个开发团队"，但实际上只有一个人。关键不是 AI 替代了团队成员，而是**AI 消除了团队协作中的沟通开销**——编排层承担了所有的"翻译"和"协调"工作。

### 3. 瓶颈在物理层，不在 AI 层

大家讨论 AI 系统时总关心 token 成本、API 限制、模型能力，但这套系统的真正瓶颈是 RAM。16GB 只能跑 4-5 个 Agent，作者花 $3500 升级到 128GB Mac Studio。**当 AI 编排足够好时，物理资源反而成为增长天花板**。

### 4. Claude Code Reviewer 几乎没用

三个 reviewer 中，Claude Code 被作者评为"基本没用过"——过度谨慎，总建议"考虑添加……"，大部分是过度设计。这打破了"用同一个 AI 做所有事"的幻想，**不同 AI 有明确的能力边界和适用场景**。

### 5. 静态 Prompt 是改进版 Ralph Loop 的反模式

传统 Ralph Loop 的学习改善了检索质量，但 prompt 本身是静态的。作者的改进是：**失败后 Zoe 会重写 prompt 本身**，而不只是改善下一次的上下文检索。这意味着系统学习的不是"找什么信息"，而是"怎么表达需求"。

---

## 行动清单

> 从推论推导的 10 项行动，按优先级排序

- [ ] **P0: 给 trading-system 加编排层** — 新建一个"策略编排 Agent"，持有 Polymarket 市场上下文/鲸鱼表现历史/Agent Elo 评分/历史反思知识，在每个 cycle 动态调整策略 prompt
- [ ] **P0: 实现"确定性监控 + AI 分析"分离** — trading-system 先用规则引擎过滤（价格变动/成交量/时间窗口），只有通过的候选才调 Agent，节省 token
- [ ] **P1: Git Worktree + tmux 并发开发模式** — 在下一个多任务开发场景中，用 worktree 隔离分支 + tmux 后台运行多个 Agent 并发编码
- [ ] **P1: 搭建自动化 Code Review 管道** — PR 创建后自动触发 AI review（可用 GitHub Actions + Claude API），review 通过后才通知人类
- [ ] **P1: 定义"完成"的多维标准** — 为你的项目建立类似的 checklist：PR + CI + AI review + 人工批准，减少人工审查负担
- [ ] **P2: 构建 Obsidian → Agent 的自动上下文管道** — 把会议记录/决策日志/客户信息自动同步为 Agent 可读的上下文
- [ ] **P2: 实现 Agent 路由策略** — 根据任务类型（后端逻辑/前端/文档）自动选择最合适的 Agent/模型
- [ ] **P2: 建立"成功模式记忆"系统** — 记录哪种 prompt 结构对哪类任务有效，逐步提升 prompt 质量
- [ ] **P3: 探索 OpenClaw 作为编排层的可行性** — 评估是否适合你的项目，起步 $20/月
- [ ] **P3: 资源预算规划** — 评估并发 Agent 数量 vs RAM 需求，确定是否需要升级硬件

---

## 知识网络关联

| 已有笔记 | 关系类型 | 连接点 |
|----------|----------|--------|
| [Ralph - Claude Code 自动化循环工具](../ai-ide/2026-01-16-ralph-claude-code-automation.md) | **深化** | 本文是 Ralph Loop 的进化版：从静态 prompt 到动态重写 |
| [Boris Cherny 访谈：一人军团的指挥艺术](../claude-code/2026-01-25-boris-interview-one-man-army.md) | **互补** | Boris 的单人高产出理念 + 本文的具体架构实现 |
| [Cursor 多 Agent 并发编码协调经验](../ai-ide/2026-01-16-cursor-multi-agent-coordination.md) | **应用** | 本文的 worktree + tmux 并发模式可直接迁移到 Cursor 工作流 |
| [Agent 蜂群：从单体到群体智能的进化](../agent-skill/2026-02-05-agent-swarm-research.md) | **对比** | 蜂群=对等协作，本文=层级编排；不同场景适用不同模式 |
| [Manus 上下文工程：AI Agent 的六大实战原则](../agent-architecture/2026-01-29-manus-context-engineering-for-ai-agents.md) | **理论基础** | Manus 的"上下文是 Agent 能力的上限"是本文架构的理论依据 |
| [Anthropic：有效的上下文工程实践](../agent-architecture/2026-01-29-anthropic-effective-context-engineering.md) | **理论基础** | 上下文分层设计的最佳实践参考 |
| [OpenClaw 深度使用 6 大实战技巧](../agent-skill/2026-02-05-openclaw-6-practical-tips.md) | **深化** | 本文展示了 OpenClaw 的架构级应用，超越了 6 个技巧的操作层面 |
| [Vibe Kanban - AI 编程代理编排平台](../ai-ide/2026-01-16-vibe-kanban-multi-agent-orchestration.md) | **对比** | Vibe Kanban 是另一种编排方式（看板式），本文是编程式编排 |
| [EvoMap - AI 自进化基础设施](../agent-architecture/2026-02-26-evomap-ai-self-evolution-infrastructure.md) | **互补** | EvoMap 提供自进化的基础设施层，本文展示自进化在实战中的效果 |
| [Quest 自进化自主编程 Agent](../agent-architecture/2026-01-17-quest-self-evolving-agent.md) | **互补** | Quest 是学术视角的自进化 Agent，本文是工程实践视角 |

---

## 金句摘录

> 以下为作者原话（或语义等价的表述）

1. "Codex 和 Claude Code 对你的业务几乎一无所知。它们只看到代码，看不到完整的业务图景。"
2. "上下文窗口是固定的，你只能二选一：塞满代码 → 没空间放业务上下文；塞满客户历史 → 没空间放代码库。"
3. "OpenClaw 改变了这个等式。它充当编排层，位于你和所有 AI 工具之间。"
4. "Codex/Claude Code = 专业厨师，只管做菜。OpenClaw = 主厨，知道客人口味、食材库存、菜单定位。"
5. "从管理 Claude Code，变成管理一个 AI 管家，这个管家再去管理一群 Claude Code。"
6. "Zoe 不会用同样的 prompt 重启。她会带着完整的业务上下文，分析失败原因，然后重写 prompt。"
7. "奖励信号是：CI 通过、三个 code review 通过、人工合并。任何失败都会触发循环。"
8. "不是 token 成本，不是 API 速率，而是内存（RAM）。"
9. "我们会看到大量一个人的百万美元公司从 2026 年开始出现。杠杆是巨大的，属于那些理解如何构建递归自我改进 AI 系统的人。"
10. "下一代创业者不会雇 10 个人去做一个人加一套系统就能做的事。"

---

## 延伸阅读

- [OpenClaw 官方文档](https://openclaw.com) — 编排层工具
- [Codex CLI / OpenAI Codex](https://openai.com/codex) — gpt-5.3-codex 执行层
- [Git Worktree 官方文档](https://git-scm.com/docs/git-worktree) — 并行开发隔离机制
- [tmux 终端复用器](https://github.com/tmux/tmux) — Agent 后台运行基础设施
- [Ralph Loop 原始概念](https://github.com/jikkuatwork/ralph) — 本文改进的基础框架
- [Anthropic Building Effective Agents](https://docs.anthropic.com/en/docs/build-with-claude/agent) — 上下文工程理论基础
- [你的笔记: OpenClaw 深度使用 6 大实战技巧](../agent-skill/2026-02-05-openclaw-6-practical-tips.md) — OpenClaw 操作层面的补充
