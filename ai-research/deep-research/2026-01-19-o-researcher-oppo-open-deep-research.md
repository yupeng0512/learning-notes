---
title: "O-Researcher: OPPO 开源深度研究模型"
date: 2026-01-19
category: ai-research/deep-research
tags: [deep-research, multi-agent, RLAIF, trajectory-distillation, OPPO, open-source]
source: https://arxiv.org/pdf/2601.03743
code: https://github.com/OPPO-PersonalAI/O-Researcher
---

# O-Researcher: OPPO 开源深度研究模型

> 📅 学习日期：2026-01-19
> 📄 论文：O-Researcher: An Open Ended Deep Research Model via Multi-Agent Distillation and Agentic RL
> 🔗 链接：https://arxiv.org/pdf/2601.03743
> 💻 代码：https://github.com/OPPO-PersonalAI/O-Researcher

---

## 📖 概览

| 项目 | 内容 |
|------|------|
| **作者** | OPPO AI Agent 团队 |
| **类型** | 学术论文（AI Agent / 深度研究） |
| **核心主题** | 通过多智能体蒸馏 + Agentic RL 训练开源深度研究模型 |
| **阅读价值** | ⭐⭐⭐⭐⭐ 开源 SOTA 方法论，可迁移性强 |

> **一句话总结**：用「并行多智能体生成高质量轨迹 → SFT → RLAIF」的两阶段范式，让开源模型在深度研究任务上超越闭源模型。

---

## 🎯 根节点命题

> **开源模型性能 = 高质量推理轨迹(SFT) × Agentic RL 对齐(RLAIF) × 并行执行效率**

### 推论树

```
根节点：开源性能 = 高质量轨迹 × RL对齐 × 并行效率
│
├─ 推论1：传统蒸馏只学答案，需要「轨迹级蒸馏」
│   └─ 方案：并行执行工作流生成完整推理轨迹
│
├─ 推论2：SFT只能模仿，需要RL「自主探索」
│   └─ 方案：GRPO + 复合奖励函数(质量+工具+格式)
│
├─ 推论3：顺序执行导致上下文竞争，需要「并行隔离」
│   └─ 方案：子任务独立上下文 + Summarizer聚合
│
└─ 推论4：更多推理步≠更好，存在「效率甜点」
    └─ 方案：10步工作流（性能-成本最优）
```

---

## 📚 核心技术创新

### 1. 并行轨迹生成（解决数据问题）

```
传统蒸馏：Query → Teacher Model → Final Answer（丢失推理过程）

O-Researcher：
Query → [Agent1: 子任务1] ─┐
     → [Agent2: 子任务2] ─┼→ Summarizer → 最终报告
     → [Agent3: 子任务3] ─┘
         ↓
收集所有Agent轨迹 → 合并为单一SFT训练数据
```

**关键设计**：
- 每个Agent执行「计划-执行-观察」循环（≥10步 + ≥5次工具调用）
- 三层质量过滤：规则硬拒绝 → LLM-as-Judge → 人工抽查

### 2. RLAIF 奖励函数设计

```
R_total = 0.6×R_base + 0.2×R_tool + 0.2×R_format

R_base  = LLM Judge评分（全面性+洞察力+指令遵循+可读性）
R_tool  = min(search次数, crawl次数) ∈ [2,8]  ← 防止只搜不读
R_format = XML标签闭合 + 必须有<suggested_answer>
```

### 3. 并行执行架构

| 对比维度 | 顺序执行 | 并行执行 |
|----------|----------|----------|
| GPT-5 总分 | 42.92 | **49.60** (+6.68) |
| 上下文竞争 | 严重 | 轻微（各自独立） |

---

## 📊 关键数字速记

| 指标 | 数值 | 意义 |
|------|------|------|
| **48.48** | DeepResearch Bench 总分 | 开源新 SOTA |
| **26.01 vs 8.96** | RL后 vs RL前引用数 | RL 带来的工具能力提升 |
| **64k** | 上下文窗口 | 技术约束 |
| **10步** | 最优推理步数 | 性能-成本甜点 |
| **6:2:2** | 奖励权重 | 质量:工具:格式 |

---

## ✅ 行动清单

| 推论 | 行动 | 预计时间 |
|------|------|----------|
| 轨迹级蒸馏 | 复现并行数据合成 Pipeline | 2-3天 |
| RLAIF 复合奖励 | 设计任务奖励函数：质量(60%)+工具(20%)+格式(20%) | 1天 |
| 并行执行架构 | 将 RAG/Agent 改造为并行子任务 + 聚合架构 | 1-2天 |
| 推理步数甜点 | 测试 5/10/15/20 步找最优推理长度 | 半天 |

---

## 💡 核心洞察

1. **轨迹 > 答案**：蒸馏「思考过程」比蒸馏「最终答案」更有价值
2. **并行 > 顺序**：隔离上下文避免竞争，Summarizer 负责聚合
3. **复合奖励**：质量为主(60%)，工具和格式为辅(各20%)
4. **10步甜点**：推理步数存在性能-成本最优点

---

## 🏷️ 标签

`#deep-research` `#multi-agent` `#RLAIF` `#trajectory-distillation` `#OPPO` `#open-source`
