---
title: "AI 生成 UI 方案对比：json-render vs A2UI"
date: 2026-01-19
tags: [AI, Frontend, UI Generation, Comparison, DSL]
category: ai-tools
subcategory: ai-productivity
---

# AI 生成 UI 方案对比：json-render vs A2UI

> 📅 学习日期：2026-01-19
> 🏷️ 分类：AI 工具与效率 / AI 生产力
> 🔗 相关笔记：
> - [json-render](./2026-01-19-json-render-ai-ui-generation.md)
> - [Google A2UI](./2026-01-06-google-a2ui-agent-ui-protocol.md)

---

## 根节点命题

> **两者的核心思想相同：用"结构化中间层"将 AI 的不可控输出转化为可控的声明式描述**

**共同的根本洞见**：
- AI 直接生成代码 = 不可控（安全风险、格式不一致、跨平台困难）
- AI 生成结构化描述 + 可信客户端渲染 = 可控（安全、一致、可移植）

---

## 核心对比

| 维度 | json-render (Vercel) | A2UI (Google) |
|------|---------------------|---------------|
| **定位** | React 生态的 AI UI 生成库 | 跨平台的 AI UI 协议规范 |
| **抽象层级** | 库/SDK | 协议/标准 |
| **平台支持** | 仅 React/Next.js | Web (Lit/Angular)、Flutter、未来更多 |
| **数据结构** | 树形 JSON | 邻接表（扁平化）|
| **约束定义** | Catalog + Zod Schema | 组件白名单 |
| **数据绑定** | `valuePath` 路径绑定 | `path` 路径绑定 |
| **动作处理** | ActionProvider + Action Schema | Action 字符串标识符 |
| **流式支持** | ✅ useUIStream | ✅ JSONL 流式传输 |
| **代码导出** | ✅ Code Export 导出 Next.js | ❌ 主要用于运行时渲染 |

---

## 设计哲学对比

### json-render：**工程实用主义**

```
目标：让 React 开发者快速上手 AI 生成 UI
     ├─ 紧贴 React 生态（hooks, context）
     ├─ Zod 类型安全（TypeScript 友好）
     ├─ Code Export（生成可维护的代码）
     └─ 实用优先，跨平台不是首要目标
```

### A2UI：**协议标准主义**

```
目标：定义 Agent 生成 UI 的通用协议
     ├─ 跨平台优先（同一份 JSON 多端渲染）
     ├─ 邻接表结构（对 LLM 流式生成友好）
     ├─ 融入 Agent 协议栈（MCP/A2A/AG-UI/A2UI）
     └─ 标准先行，生态逐步完善
```

---

## 技术细节对比

### 1. 组件定义方式

**json-render (Zod Schema)**：
```typescript
const catalog = createCatalog({
  components: {
    Metric: {
      props: z.object({
        label: z.string(),
        valuePath: z.string(),
        format: z.enum(['currency', 'percent', 'number']),
      }),
    },
  },
});
```

**A2UI (协议规范)**：
```json
{
  "id": "metric",
  "Text": {
    "text": { "path": "/data/revenue" },
    "usageHint": "h2"
  }
}
```

**差异分析**：
- json-render 用 Zod 做编译时类型检查，TypeScript 友好
- A2UI 用协议规范做运行时验证，跨语言友好

### 2. 数据结构

**json-render (树形)**：
```json
{
  "type": "Card",
  "props": { "title": "Revenue" },
  "children": [
    { "type": "Metric", "props": { "valuePath": "/revenue" } }
  ]
}
```

**A2UI (邻接表)**：
```json
{
  "components": [
    { "id": "card", "Card": { "child": "metric" } },
    { "id": "metric", "Text": { "text": { "path": "/revenue" } } }
  ],
  "root": "card"
}
```

**差异分析**：
- 树形结构：直观，但 LLM 生成时需要匹配括号
- 邻接表：LLM 可逐个输出，增量更新更容易

### 3. 动作处理

**json-render**：
```typescript
<ActionProvider actions={{
  export_report: () => downloadPDF(),
  refresh_data: () => refetch(),
}}>
```

**A2UI**：
```json
{
  "action": { "name": "confirm_booking" }
}
// 客户端映射：
const handlers = {
  confirm_booking: (params) => bookingService.confirm(params)
};
```

**差异分析**：
- 两者本质相同：AI 声明意图，客户端决定执行
- json-render 用 React Context，A2UI 用消息协议

---

## 适用场景对比

| 场景 | json-render | A2UI | 推荐 |
|------|-------------|------|------|
| **React/Next.js 项目** | ⭐⭐⭐⭐⭐ | ⭐⭐ | json-render |
| **多平台应用（Web + App）** | ⭐⭐ | ⭐⭐⭐⭐⭐ | A2UI |
| **快速原型** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | json-render |
| **多 Agent 协作** | ⭐⭐ | ⭐⭐⭐⭐⭐ | A2UI |
| **需要代码导出** | ⭐⭐⭐⭐⭐ | ⭐ | json-render |
| **已有 Flutter 客户端** | ⭐ | ⭐⭐⭐⭐⭐ | A2UI |

---

## 选型建议

### 选 json-render 当：

1. 你的项目是 **React/Next.js 技术栈**
2. 你需要 **TypeScript 类型安全**（Zod 集成）
3. 你需要 **Code Export** 功能，导出可维护的代码
4. 你想 **快速上手**，不想深入协议细节
5. 你的 AI 生成 UI 主要用于 **Web Dashboard / 管理后台**

### 选 A2UI 当：

1. 你需要 **跨平台**（Web + Flutter + 未来更多）
2. 你在构建 **多 Agent 协作系统**（需要融入 MCP/A2A 生态）
3. 你需要 **流式渲染**，且对 LLM 输出容错性要求高
4. 你在构建 **对话式 AI 应用**，需要从纯文本升级为富交互
5. 你愿意 **投入更多前期设计**，换取更好的跨平台一致性

---

## 共同的启示

1. **约束是解放**：限制 AI 的自由度，反而让它更可靠
2. **中间层是关键**：结构化 JSON 是连接"AI 不可控输出"和"可信客户端渲染"的桥梁
3. **声明式优于命令式**：AI 说"要什么"，客户端决定"怎么做"
4. **安全是设计出来的**：不是事后检查，而是架构层面消除风险

---

## 个人思考

**与 OctoCodingBench 的关联**：

两个方案都是对"AI 生成代码不可控"问题的回应：
- OctoCodingBench 的发现：ISR = CSR^N，规则多了 AI 必然违规
- json-render / A2UI 的解法：**大幅减少 N**（AI 只能选择白名单组件）+ **提高 CSR**（结构化约束让遵守更容易）

这是"主动降维"策略的两种实现：
- json-render：降维到 React 组件词汇表
- A2UI：降维到协议定义的组件白名单

---

## 延伸阅读

- [json-render GitHub](https://github.com/vercel-labs/json-render) - Vercel 方案
- [A2UI GitHub](https://github.com/google/A2UI) - Google 方案
- [OctoCodingBench](../agent-architecture/2026-01-19-octocodingbench-process-evaluation.md) - AI 过程合规评测
