---
title: Skill From Masters 三条路径方法论
source: https://github.com/GBSOSS/skill-from-masters
date: 2026-02-03
tags: [skill-creation, methodology, decision-framework]
type: knowledge
---

# Skill From Masters 三条路径方法论

> 学习时间：2026-02-03  
> 实践场景：创建技术架构评审 Skill  
> 核心价值：站在巨人肩膀上创建 Skill，而非从零开始

---

## 根节点命题

**创建 Skill 的难点不在格式，而在方法论**。

三条路径（市场搜索 → GitHub 学习 → 专家方法论）构成一个**成本递增但质量可控的决策树**：
- 先复用市场现成的（最快）
- 再从开源项目学习知识（学习 > 包装工具）
- 最后从专家方法论创造（最深入）

核心公式：**ROI = 单次收益 / 创建成本 × 预期使用次数**

---

## 表示空间

| 维度 | 路径 1: search-skill | 路径 2: skill-from-github | 路径 3: skill-from-masters |
|------|---------------------|--------------------------|---------------------------|
| **时间成本** | 10-30 分钟 | 1-2 小时 | 3-5 小时 |
| **适用场景** | 常见通用任务 | 技术类 Skill | 非技术类 Skill |
| **产出质量** | ★★★★★（如果找到） | ★★★★ | ★★★★★ |
| **盈亏平衡点** | 0.1 次回本 | 0.5 次回本 | 2 次回本 |
| **核心能力** | 搜索和筛选 | 提取原理 | 融合方法论 |

---

## 推论展开

### 推论 1：成本递增原则

**「先看轮子，再造轮子」**

```
决策流程：
市场搜索（10分钟）→ 找到 → 直接用 ✓
                   → 没找到 ↓
判断类型：技术类 → skill-from-github（1-2h）
         非技术类 → skill-from-masters（3-5h）
```

**ROI 阈值**：
- 通用高频（>10 次/年）→ 路径 1
- 技术中频（3-10 次/年）→ 路径 2
- 专业低频（<3 次/年）→ 路径 3

### 推论 2：知识提取 > 工具包装

**路径 2 的核心不是调用工具，而是学习原理**

- ❌ 错误：创建调用 `kubectl` 的 Skill
- ✅ 正确：学习 K8s 运维最佳实践，编码为诊断 Skill

**可迁移 vs 依赖**：
- 工具会升级、API 会变化 → 包装脆弱
- 原理和模式相对稳定 → 知识持久

### 推论 3：任务收窄到「专家级」

**路径 3 的 5 层收窄框架**

| 层级 | 问题 | 停止条件 |
|------|------|---------|
| L1 | 哪个领域？ | 明确主题 |
| L2 | 谁用？输出什么？ | 清晰用户和交付物 |
| L3 | 具体场景？ | 举出真实案例 |
| L4 | 不包含什么？ | 划清边界 |
| L5 | 真实案例？ | **专家会有独特建议** ✓ |

停止收窄的信号：专家对这个任务有独特的方法论，而非泛泛而谈。

### 推论 4：三层方法论搜索

**深度优先，直到「非常清晰」**

```
Layer 1: 本地数据库 → 快速定位领域专家
Layer 2: Web 搜索专家 → "[domain] best practices framework"
Layer 3: 深挖一手资源 → "[expert] white paper PDF"
```

停止条件：
- ✅ 找到 2-3 个专家方法论
- ✅ 方法论足够具体（有步骤、有案例）
- ✅ 能清楚解释每个步骤

### 推论 5：融合优于单一

**实践案例：技术架构评审 Skill**

融合三个框架：
1. **Azure 5 大支柱**（结构化维度）→ 评审框架
2. **ATAM 风险分析**（深度方法）→ 识别风险/敏感点/权衡点
3. **Martin Fowler 演进视角**（长期视角）→ 技术债务评估

结果：用时 5 小时，创建融合世界级方法论的 Skill

---

## 泛化模式

### 模式 1：三层决策树（适用于任何创造性工作）

```
创建任何知识资产时：
1. 先复用（搜索现成的）
2. 再学习（从优质案例提取）
3. 最后创造（从第一性原理）
```

**应用场景**：
- 写技术文档 → 先找模板、再学优秀案例、最后原创
- 设计系统架构 → 先看参考架构、再学经典案例、最后定制
- 制定流程规范 → 先看业界标准、再学标杆企业、最后结合实际

### 模式 2：ROI 驱动的投入决策

**公式**：投入 = f(使用频率, 质量要求, 复用性)

| 场景 | 频率 | 投入策略 |
|------|------|---------|
| 一次性任务 | 1 次 | 最小化投入（快速复用） |
| 低频专项 | <3 次/年 | 中等投入（学习精髓） |
| 高频通用 | >10 次/年 | 高质量投入（深度定制） |

### 模式 3：知识收窄漏斗

**从宽到窄的五层收窄**（非技术类工作适用）

```
领域（宽） → 约束条件 → 具体场景 → 边界 → 真实案例（窄）
```

停止信号：**专家会有独特建议**（而非常识性建议）

---

## 关键概念

### 盈亏平衡点（Break-even Point）

创建成本 / 单次收益 = 最少使用次数

**示例**：
- 路径 1（0.2h）/ 单次节省（2h）= 0.1 次 → 立即回本
- 路径 3（4h）/ 单次节省（2h）= 2 次 → 用 3 次才值得

### 方法论数据库（Methodology Database）

**定义**：领域专家的核心框架和最佳实践库

**结构**：
```
领域（15+）→ 专家（80+）→ 核心框架 → 参考资源
```

**价值**：路径 3 的起点，避免从零搜索

### 质量门禁（Quality Gate）

**路径 1 筛选标准**：
- Stars < 10 → 过滤
- 6 个月未更新 → 谨慎
- 无 SKILL.md → 不用

**路径 2 筛选标准**：
- Stars < 100 → 不考虑
- 最近 12 月无更新 → 谨慎
- 无文档 → 不用

### 黄金案例 + 反模式

**黄金案例**：定义「什么叫做得好」
**反模式**：编码「千万别这么干」

**示例（PRD 写作）**：
- 黄金案例：Amazon 6-pager, Intercom PRD 模板
- 反模式：没定义问题就跳方案、缺少成功指标

---

## 行动清单

### 立即可行（Top 3）

1. **创建 Skill 前先搜索 10 分钟**
   - 搜索关键词：`[skill name] Claude skills marketplace`
   - 筛选标准：Stars > 10, 6 个月内更新
   - 找到即用，节省 3-5 小时

2. **技术类 Skill 优先路径 2**
   - 在 GitHub 搜索：`site:github.com [domain] stars:>100`
   - 阅读 README 和核心代码
   - **提取原理而非包装工具**

3. **建立个人方法论数据库**
   - 创建 `methodology-database.md`
   - 按领域收录专家和核心框架
   - 定期补充（季度审查）

### 完整清单（10 项）

4. **非技术 Skill 用 5 层收窄**
   - 从宽泛领域收窄到具体场景
   - 停止条件：专家会有独特建议

5. **用 ROI 思维决定投入**
   - 高频通用 → 低成本快速复用
   - 中频专用 → 中成本深度学习
   - 低频定制 → 高成本精细创造

6. **路径 2 重点：学知识不包装工具**
   - 理解算法原理、设计模式
   - 编码为可迁移的 Skill

7. **路径 3 搜索到「非常清晰」**
   - 三层搜索：本地 → Web → 一手资源
   - 找到 2-3 个专家方法论
   - 能清楚解释每个步骤

8. **融合优于单一**
   - 多个方法论各取所长
   - 例：Azure（结构）+ ATAM（深度）+ Fowler（长期）

9. **黄金案例 + 反模式**
   - 先找「做得好的例子」定义质量标准
   - 编码「千万别这么干」

10. **设置质量门禁**
    - 市场：Stars < 10 过滤
    - GitHub：Stars < 100, 12 月无更新谨慎
    - 方法论：无具体步骤和案例不用

---

## 知识网络关联

### 互补关系

- **[2026-01-25-skill-from-masters-trinity-skill-generator.md](2026-01-25-skill-from-masters-trinity-skill-generator.md)**
  - 关系：理论 → 实践
  - 价值：精读笔记提供理论基础，本笔记提供实践验证

- **[command-skill-creator](../../../.codebuddy/skills/command-skill-creator/)**
  - 关系：方法论 → 格式规范
  - 价值：skill-from-masters 负责内容，command-skill-creator 负责格式

### 应用关系

- **[tech-review SKILL](../../../.cursor/skills/tech-review/)**
  - 关系：方法论 → 产出
  - 价值：本次实践创建的架构评审 Skill，融合 ATAM + AWS + Fowler

### 对比关系

- **article-tutor vs skill-from-masters**
  - article-tutor：学习现有知识（输入：文章/书籍）
  - skill-from-masters：创造新 Skill（输入：方法论）
  - 协同：用 article-tutor 深度学习 → 提取方法论 → 用 skill-from-masters 生成 Skill

---

## 金句摘录

> "站在巨人肩膀上，而非从零开始"

> "质量不是写出来的，而是选择出来的"

> "任务收窄到「专家会有独特建议」的粒度"

> "学知识，不包装工具"

> "三层搜索，直到方法论「非常清晰」"

> "先市场、再开源、最后自创——成本递增原则"

> "黄金案例定义「什么叫做得好」，反模式编码「千万别这么干」"

---

## 延伸阅读

### 原版项目
- [GitHub - GBSOSS/skill-from-masters](https://github.com/GBSOSS/skill-from-masters)
- [精读笔记：2026-01-25](2026-01-25-skill-from-masters-trinity-skill-generator.md)

### 方法论来源
- [ATAM 方法论](https://sei.cmu.edu/library/atam) - CMU SEI 架构评审
- [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/) - 云架构评审框架
- [Martin Fowler - Architecture](https://martinfowler.com/architecture) - 架构原则和演进

### 实践产出
- 本次创建的 Skill：`~/.cursor/skills/tech-review/SKILL.md`
- 方法论数据库：`~/.cursor/skills/skill-from-masters/references/methodology-database.md`
- Skill 分类体系：`~/.cursor/skills/skill-from-masters/references/skill-taxonomy.md`
