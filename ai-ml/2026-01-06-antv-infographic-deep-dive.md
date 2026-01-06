# AntV Infographic 项目精读

> 📅 学习日期：2026-01-06
> 🔗 项目地址：https://github.com/antvis/Infographic
> 🌐 官网：https://infographic.antv.vision

## 快速概览

| 项目 | 内容 |
|------|------|
| **项目名称** | AntV Infographic |
| **定位** | 下一代声明式信息图可视化引擎 |
| **核心理念** | Bring words to life with AI! |
| **开源协议** | MIT License |
| **技术栈** | TypeScript (96.3%)、JSX、SVG |
| **Star 数** | 3k+（开源仅一个多月） |
| **模板数量** | 218 个内置模板 |

> 💡 **一句话总结**：Infographic 是一个为 AI 时代量身打造的信息图引擎——它用类 Markdown 语法让 AI 能高效生成信息图，同时让人类能直观编辑，打破了"免费、可控、可集成"的不可能三角。

---

## 核心问题与解决方案

### 问题：信息图工具的不可能三角

传统工具的困境：
- **PPT/Canva**：可控但不可集成
- **AI 生成工具**：不可控（风格不稳定）
- **ECharts/D3**：可集成但编辑性差
- **Mermaid**：语法简单但风格单一

### 解决方案：AI 友好的声明式引擎

1. **类 Markdown 语法**
   - AI 友好：流式输出、低 token 消耗
   - 人类友好：无需编程基础，直观可编辑
   - 双向可读：AI 生成 → 人类修改 → AI 继续

2. **JSX 渲染引擎**
   - 语义丰富：比 JSON 更适合 AI 理解
   - 逻辑控制：支持条件分支、组件嵌套
   - 精确布局：独立于 React 的布局机制

3. **高质量 SVG 输出**
   - 218 个内置模板
   - 多种主题风格（手绘、渐变、纹理）
   - 任意分辨率清晰

---

## 技术架构深度解析

### 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 1: API（接口层）                                       │
│  • Infographic 类：唯一对外入口                              │
│  • render()：渲染信息图                                      │
│  • getTypes()：获取模板所需数据类型（AI 专用）               │
├─────────────────────────────────────────────────────────────┤
│ Layer 2: Runtime（运行时系统）                               │
│  • 模板生成器：语法 → JSX 组件组装                           │
│  • 渲染器：JSX → SVG 风格注入                                │
│  • 编辑器：交互修改、撤销重做                                │
├─────────────────────────────────────────────────────────────┤
│ Layer 3: JSX Engine（描述语言层）                            │
│  • 独立于 React 的 JSX 渲染引擎                              │
│  • 内置几何图形、文本、分组等原语组件                        │
│  • 精确的布局控制机制                                        │
└─────────────────────────────────────────────────────────────┘
```

### 为什么选择 JSX 而不是 JSON？

这是一个**反直觉但深思熟虑**的技术决策：

| 维度 | JSON DSL | JSX |
|------|----------|-----|
| **语义表达力** | ❌ 弱，纯数据载体 | ✅ 强，类似代码 |
| **逻辑控制** | ❌ 无法表达条件分支 | ✅ 支持条件、循环 |
| **动态组合** | ❌ 难以嵌套组件 | ✅ 天然支持组件组合 |
| **AI 理解** | ❌ 需要额外 Schema | ✅ 语义丰富，更易理解 |

> **核心洞见**：当创作主体从人类转向 AI 时，**代码比数据更自然**。AI 是"类人创作者"，更擅长理解和生成语义丰富的结构。

---

## 信息图语法详解

### 语法结构

```
infographic [template-name]

design
  structure [结构类型]
  item [数据项样式]
  title [标题样式]

data
  title [主标题]
  desc [描述]
  items
    - label [标签1]
      desc [描述1]
    - label [标签2]
      desc [描述2]

theme
  palette [色板]
  font [字体]
  style [风格：手绘/渐变/纹理]
```

### 实际示例

```
infographic list-row-simple-horizontal-arrow

data
  items
    - label Step 1
      desc Start
    - label Step 2
      desc In Progress
    - label Step 3
      desc Complete
```

### 为什么这种语法对 AI 友好？

| 特性 | 说明 | AI 收益 |
|------|------|---------|
| **逐行可解析** | 不依赖成对标签 | 支持流式输出 |
| **缩进表达结构** | 用空格/连字符表达层级 | 最少 token 传递最多语义 |
| **无需闭合** | 不像 XML/JSON 需要配对 | 更低出错率 |
| **语义直观** | `label`、`desc` 等字段名 | AI 容易理解意图 |

### 流式渲染

```javascript
// AI 逐段输出时，实时渲染
let buffer = '';
for (const chunk of chunks) {
  buffer += chunk;
  infographic.render(buffer);  // 每次追加都能渲染
}
```

---

## 模板库概览

### 218 个模板，7 大分类

| 分类 | 数量 | 典型用途 |
|------|------|----------|
| **Hierarchy（层级）** | 112 | 组织架构、思维导图、层级树 |
| **Sequence（序列）** | 46 | 时间线、流程图、路线图 |
| **List（列表）** | 27 | 清单、步骤、并列信息 |
| **Comparison（对比）** | 17 | 二分对比、SWOT 分析 |
| **Chart（图表）** | 11 | 柱状图、饼图、词云 |
| **Quadrant（象限）** | 3 | 四象限分析、矩阵分类 |
| **Relation（关系）** | 2 | 环形关系图 |

### 设计资产体系

```
结构（Structure）
  ↓ 决定整体布局：阶梯排布、顺序排布、环形排布...
数据项（Item）
  ↓ 内容单元的可视化载体，决定每个卡片/节点的具体样式
基础组件（Base Components）
  → 设计体系的"积木"，封装字号、颜色等设计规范
```

---

## AI 集成能力

### Skills 体系

| Skill | 用途 | 适用对象 |
|-------|------|----------|
| `infographic-syntax-creator` | 从描述生成信息图语法 | 普通用户 |
| `infographic-creator` | 创建渲染信息图的 HTML 文件 | 普通用户 |
| `infographic-structure-creator` | 生成自定义结构设计 | 开发者 |
| `infographic-item-creator` | 生成个性化数据项样式 | 开发者 |
| `infographic-template-updater` | 自动同步更新模板库 | 开发者 |

### getTypes() 方法

专门为 AI 设计的 API：

```javascript
// 获取特定模板所需的数据结构（TypeScript 类型）
const types = infographic.getTypes('list-row-simple-horizontal-arrow');
// AI 可以根据这个类型定义，准确组织数据
```

### 开源提示词模板

AntV 团队直接开源了 Prompt 工程经验：
- 结构生成提示词：`src/designs/structures/prompt.md`
- 数据项生成提示词：`src/designs/items/prompt.md`

---

## 核心洞见

### 1. Markdown 是 AGI 的重要组成部分

> "4 年之前你绝对想不到，我们发现 AGI 的重要组成部分竟然是 Markdown 文档。"

**结构清晰、语法简洁的纯文本格式**是人类与 AI 协作的最佳媒介。

### 2. 从"人类创作"到"人机协同"

```
传统模式：人类 → 工具 → 输出
AI 时代：AI 生成 → 人类编辑 → 输出
理想模式：AI 生成 ↔ 人类编辑 ↔ AI 继续（双向可读、可编辑）
```

### 3. 代码 vs 数据：AI 时代的新选择

| 传统观点 | AI 时代观点 |
|----------|-------------|
| JSON 是机器友好的 | JSON 对 AI 不够友好 |
| 代码是人类写的 | AI 也擅长写代码 |
| 数据交换用 JSON | 创作表达用 JSX |

### 4. 信息图工具的演进

```
PPT 手动画 → Canva 模板 → AI 生成（不可控）→ Infographic（AI 生成 + 可控 + 可集成）
```

---

## 核心要点（5 条）

1. **定位**：为 AI 时代量身打造的声明式信息图引擎，打破"免费、可控、可集成"的不可能三角
2. **语法**：类 Markdown 语法，支持流式输出，AI 和人类都能轻松读写
3. **架构**：JSX Engine + Runtime + API 三层架构，选择 JSX 而非 JSON 是因为 AI 更擅长理解语义丰富的结构
4. **模板**：218 个内置模板，覆盖层级、序列、对比、图表等 7 大分类
5. **AI 集成**：提供 Skills 体系和 `getTypes()` API，直接开源 Prompt 模板

---

## 适用场景

| 场景 | 适合度 | 说明 |
|------|--------|------|
| **技术文档配图** | ⭐⭐⭐⭐⭐ | 流程图、架构图、时间线 |
| **产品介绍** | ⭐⭐⭐⭐⭐ | 功能对比、特性列表 |
| **数据报告** | ⭐⭐⭐⭐ | 统计图表、趋势分析 |
| **教育内容** | ⭐⭐⭐⭐⭐ | 知识结构、概念对比 |
| **AI Agent 输出** | ⭐⭐⭐⭐⭐ | DeepResearch、AI PPT |
| **业务系统嵌入** | ⭐⭐⭐⭐⭐ | 可编辑的可视化组件 |

---

## 与其他工具的对比

| 工具 | AI 友好 | 可编辑 | 风格控制 | 可集成 |
|------|---------|--------|----------|--------|
| **PPT** | ❌ | ✅ | ✅ | ❌ |
| **Canva** | ❌ | ✅ | ✅ | ❌ |
| **Mermaid** | ✅ | ✅ | ❌ | ✅ |
| **ECharts/D3** | ❌ | ❌ | ✅ | ✅ |
| **AI 生成工具** | ✅ | ❌ | ❌ | ❌ |
| **Infographic** | ✅ | ✅ | ✅ | ✅ |

---

## 延伸资源

- **在线体验**：https://infographic.antv.vision
- **AI 生成入口**：https://infographic.antv.vision/ai
- **模板库**：https://infographic.antv.vision/gallery
- **GitHub**：https://github.com/antvis/Infographic
