---
title: Claude Scientific Skills - 科研类 AI 提示词工程标杆
source: https://github.com/K-Dense-AI/claude-scientific-skills
author: K-Dense Inc.
date: 2026-01-16
category: ai-tools
subcategory: agent-skill
tags: [Claude, Skill, 科研, 提示词工程, 学术写作]
---

# Claude Scientific Skills - 科研类 AI 提示词工程标杆

> 📖 原文：[K-Dense-AI/claude-scientific-skills](https://github.com/K-Dense-AI/claude-scientific-skills)
> 📅 学习日期：2026-01-16
> 🏷️ 分类：AI 工具与效率 / Agent & Skill

---

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **名称** | Claude Scientific Skills |
| **作者** | K-Dense Inc.（AI 科研平台公司） |
| **类型** | 针对 Claude 优化的科研类系统提示词集合 + Skill 框架 |
| **核心价值** | 140+ 即用型科研技能，让通用 AI 变身专业科研助手 |
| **阅读价值** | ⭐⭐⭐⭐⭐ 顶级的提示词工程范例，垂直领域 Skill 设计的标杆 |

> **一句话总结**：将「领域知识结构化」转化为「AI 可执行专业能力」的系统工程，每个 Skill 都是一部完整的科研方法论手册。

---

## 🗺️ 知识架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude Scientific Skills                      │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  科学数据库   │  │  Python 包   │  │  研究方法论   │          │
│  │  (28+ APIs)  │  │   (55+ 库)   │  │   (20+ 流程) │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         ↓                 ↓                 ↓                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              SKILL.md 文件 (标准格式)                        ││
│  │  ┌─────────┬────────────┬────────────┬─────────┐           ││
│  │  │YAML头   │ Overview   │ Workflows  │References│           ││
│  │  │元数据   │ 应用场景   │ 操作流程   │ 参考资料 │           ││
│  │  └─────────┴────────────┴────────────┴─────────┘           ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Skill 三层结构

1. **元数据层**：name, description, allowed-tools, license
2. **指导层**：Overview, When to Use, Core Capabilities
3. **资源层**：references/, assets/ 目录下的补充材料

---

## 📚 五大核心洞见

### 1. "正确的废话"克星

通过极其详尽的操作指南、禁止事项、具体示例，强制 AI 产出可执行、专业的回复。

| 普通提示词 | Scientific Skill |
|-----------|------------------|
| "写一篇论文" | "用 IMRAD 结构，Introduction 需包含 5 要素..." |
| 无禁止项 | "❌ NEVER 在正文用 bullet point" |
| 质量标准模糊 | "每个假设框 ≤0.6 页，超出移到附录" |
| 输出格式自由 | LaTeX 模板 + 彩色框 + 具体环境名 |

### 2. 两阶段写作法

**这是让 AI 学术写作质量飞跃的关键技巧**：

```
Stage 1: 创建带要点的章节大纲
┌────────────────────────────────────────┐
│ - 背景：AI 在药物发现中的应用             │
│   * 引用近期综述 (Smith 2023, Jones 2024)│
│   * 传统方法慢且昂贵                     │
│ - 空白：罕见病应用有限                   │
│   * 仅 2 项前期研究                      │
└────────────────────────────────────────┘
           ↓ 转换
Stage 2: 转为流畅的段落
┌────────────────────────────────────────┐
│ 人工智能方法在过去十年中在药物发现          │
│ 管线中获得了显著关注（Smith, 2023；        │
│ Jones, 2024）。尽管这些计算方法...        │
└────────────────────────────────────────┘
```

**关键约束**：
```markdown
**Critical Principle: Always write in full paragraphs with flowing prose. 
Never submit bullet points in the final manuscript.**

- ❌ **Never** leave bullet points in the final manuscript
- ❌ **Never** submit lists where paragraphs should be
- ✅ Bullet points are for planning only
```

### 3. 量化的质量门槛

用数值边界而非模糊描述约束输出：

```markdown
**Critical Overflow Prevention:**
- Insert `\newpage` before each hypothesis box to start it on a fresh page
- Keep each complete hypothesis box to ≤0.6 pages (approximately 15-20 lines of content)
- If content exceeds this, move additional details to Appendix A
- Never let boxes overflow off page boundaries - this creates unreadable PDFs
```

**最低图表要求**：

| 文档类型 | 最低 | 推荐 |
|---------|------|------|
| 研究论文 | 5 | 6-8 |
| 文献综述 | 4 | 5-7 |
| 市场研究 | 20 | 25-30 |
| 演示文稿 | 1/页 | 1-2/页 |

### 4. 领域术语规范

详细定义各学科的专业用语规范，避免 AI 混用不同领域的表达：

```markdown
**Molecular Biology and Genetics:**
- Use italics for gene symbols (e.g., *TP53*), regular font for proteins (e.g., p53)
- Follow species-specific gene nomenclature:
  - uppercase for human: *BRCA1*
  - sentence case for mouse: *Brca1*
- Organism names: full at first mention, then abbreviations
  (e.g., *Escherichia coli*, then *E. coli*)
```

| 写法 | 专业度判断 |
|------|-----------||
| "TP53 gene" | ❌ 外行（基因名应斜体） |
| "*TP53* gene" | ✅ 专业 |
| "the TP53 protein" | ❌ 外行（蛋白名不斜体，且应是 p53） |
| "the p53 protein" | ✅ 专业 |

### 5. 单一职责 + 可组合

多 Skill 编排（Skill Orchestration）完成复杂科研任务：

```
药物发现工作流：

┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ ChEMBL  │───▶│  RDKit  │───▶│ datamol │───▶│DiffDock │
│ 数据库  │    │ 分析包  │    │ 生成包  │    │ 对接包  │
│         │    │         │    │         │    │         │
│查询抑制剂│    │SAR 分析 │    │生成类似物│    │虚拟筛选 │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
      │                                            │
      └────────────────────────────────────────────┘
              整个流程由一个 prompt 驱动
```

**设计理念**（来自项目 FAQ）：
> "We believe good science in the age of AI is inherently interdisciplinary. Bundling all skills into a single plugin makes it trivial for you (and Claude) to bridge across fields."

---

## 💡 可借鉴的设计模式

| 模式 | 用途 | 示例 |
|------|------|------|
| **两阶段写作** | 长文本生成 | 先列点 → 再成段 |
| **强制输出声明** | 确保不遗漏 | "⚠️ MANDATORY" |
| **正反示例** | 明确边界 | "❌ Never... ✅ Always..." |
| **分层引用** | 控制主文长度 | 主文精简 + 附录详尽 |
| **场景触发器** | 精准匹配 | "When to Use This Skill" |
| **质量检查清单** | 系统化评审 | 表格化的检查项 |

---

## 🛠️ 实用技巧速查

| 技巧 | 来源 Skill | 应用场景 | 效果 |
|------|-----------|----------|------|
| 两阶段写作法 | scientific-writing | 所有长文本生成 | 避免杂乱、提升逻辑性 |
| 强制图表生成 | 所有研究类 | 报告/文档生成 | 增强可读性和专业感 |
| 质量检查清单 | scientific-critical-thinking | 评审类任务 | 系统化、不遗漏 |
| 场景触发器 | 所有 Skill | Skill 选择 | 精准匹配任务需求 |
| 分层引用系统 | hypothesis-generation | 复杂输出 | 主文精简、附录详尽 |

---

## ⚠️ 常见误区

- ❌ 在学术写作中使用 bullet point → ✅ 必须转为完整段落
- ❌ 泛泛的学术指导 → ✅ 具体到字段、格式、字数的精确要求
- ❌ 单一 Skill 包罗万象 → ✅ 职责单一、可组合的 Skill 设计
- ❌ 忽视参考资料目录 → ✅ 用 references/ 存放深度内容，Skill 主体保持精炼

---

## ✅ 行动清单

### 立即可做
- [ ] 在 Claude Code 中尝试安装：`/plugin marketplace add K-Dense-AI/claude-scientific-skills`
- [ ] 阅读 3 个代表性 Skill：scientific-writing、hypothesis-generation、scientific-critical-thinking
- [ ] 将"两阶段写作法"应用到下一个 AI 写作任务

### 短期实践
- [ ] 为熟悉领域设计一个迷你 SKILL.md
- [ ] 对比不同类型 Skill（数据库类、方法论类、工具类）的结构差异
- [ ] 测试多 Skills 组合完成复杂任务

### 长期提升
- [ ] 建立 Skill 设计模板库
- [ ] 为团队业务场景设计垂直 Skills
- [ ] 关注 [Agent Skills Specification](https://agentskills.io/specification) 标准演进

---

## 📋 核心收获

1. **Skill 本质是结构化的领域知识** — 不是简单的提示词，而是完整的方法论手册
2. **两阶段写作法是关键技巧** — 先列要点规划结构，再转换为流畅段落
3. **量化的质量门槛** — 用页数、字数、行数等硬性指标约束输出
4. **领域术语规范是专业性的保障** — 每个学科有自己的表达习惯，必须显式定义
5. **单一职责 + 可组合** — 设计小而精的 Skills，通过组合完成复杂任务

---

## 🔗 延伸阅读

- [Agent Skills Specification](https://agentskills.io/specification) - 官方 Skill 规范
- [K-Dense Web](https://k-dense.ai) - 该团队的完整科研平台
- [Claude Code Quickstart](https://docs.claude.com/en/docs/claude-code/quickstart) - Claude Code 快速入门
