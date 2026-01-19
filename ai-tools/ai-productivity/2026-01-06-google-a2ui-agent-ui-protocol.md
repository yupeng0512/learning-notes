---
title: "Google A2UI - Agent 生成 UI 的声明式协议"
date: 2026-01-06
tags: [AI, Frontend, UI Generation, Google, Protocol, A2A]
category: ai-tools
subcategory: ai-productivity
source: https://github.com/google/A2UI
related: [2026-01-19-json-render-ai-ui-generation.md]
---

# Google A2UI - Agent 生成 UI 的声明式协议

> 📖 来源：[Google A2UI GitHub](https://github.com/google/A2UI)
> 📅 学习日期：2026-01-06
> 🏷️ 分类：AI 工具与效率 / AI 生产力
> 🔗 相关：[json-render](./2026-01-19-json-render-ai-ui-generation.md)

---

## 根节点命题

> **"声明式 JSON + 组件白名单 + 客户端原生渲染"：将 AI 的"执行权"转移到客户端，从架构层面消除安全风险**

**为什么这是根节点**：A2UI 所有设计都可以从这个架构决策推导出来——
- 为什么安全？因为 AI 只生成 JSON 描述，不生成可执行代码，执行权在客户端
- 为什么跨平台？因为同一份 JSON 可以被不同平台的渲染器解释
- 为什么支持流式？因为邻接表结构（扁平化）让 LLM 可以逐个输出组件
- 为什么能富交互？因为 Action 只是字符串标识符，由客户端映射到安全函数

---

## 表示空间

> **理解 AI 生成 UI 问题的最小维度**

| 维度 | 传统 AI 生成 | A2UI 方案 |
|------|-------------|-----------|
| **生成对象** | 直接生成代码（HTML/JS） | 生成 JSON 描述 |
| **执行位置** | 服务端/客户端都可能执行 AI 生成的代码 | 仅客户端解释 JSON |
| **安全边界** | 无（可能 XSS 攻击） | 有（组件白名单 + 无代码执行） |
| **平台适配** | 需要针对每个平台生成不同代码 | 同一份 JSON，不同渲染器 |
| **数据结构** | 树形（嵌套）| 邻接表（扁平化，LLM 友好）|

---

## 推论展开

```
根节点：声明式 JSON + 组件白名单 + 客户端原生渲染
│
├─ 推论1：AI 不生成代码，只生成 JSON 描述 → 杜绝代码注入
│   └─ 应用：传输的是纯数据，未知组件类型降级为 Text 或忽略
│
├─ 推论2：客户端预定义组件白名单 → AI 只能"点菜"不能"自创"
│   └─ 应用：Button/Card/TextField 等预定义，AI 无法请求白名单外的组件
│
├─ 推论3：Action 是字符串标识符 → 由客户端映射到安全函数
│   └─ 应用：AI 说 `action: "confirm_booking"`，客户端决定这个动作具体做什么
│
├─ 推论4：邻接表结构（扁平化）→ 流式生成 + 增量更新
│   └─ 应用：LLM 可以逐个输出组件，不需要等待完整树；只需发送修改部分
│
└─ 推论5：同一份 JSON，不同渲染器 → 真正的跨平台
    └─ 应用：Web (Lit/Angular)、移动端 (Flutter)、桌面端共用同一协议
```

---

## 泛化模式

> **"执行权转移到可信客户端"这个模式可以迁移到哪些场景？**

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| AI 生成 UI → 客户端渲染 | **AI 生成数据库操作** | AI 生成操作描述 JSON，由可信的数据库中间件解释执行 |
| AI 生成 UI → 客户端渲染 | **AI 控制智能家居** | AI 生成设备控制意图，由本地网关解释执行 |
| AI 生成 UI → 客户端渲染 | **AI 编排微服务** | AI 生成调用链描述，由可信的编排引擎执行 |

**泛化公式**：任何需要 AI 触发"有风险动作"的场景，都可以通过"AI 生成意图描述 + 可信客户端解释执行"来实现安全控制

---

## 关键概念

> **只保留理解根节点必需的概念**

| 概念 | 定义 | 与根节点的关系 |
|------|------|----------------|
| **声明式 JSON** | 描述"要什么"而非"怎么做" | 核心载体——AI 输出的中间层 |
| **组件白名单** | 客户端预定义的可用组件列表 | 约束来源——AI 只能选择这些组件 |
| **邻接表结构** | 扁平化的组件列表 + ID 引用 | 实现细节——让流式生成成为可能 |
| **Action** | 用户操作的字符串标识符 | 安全桥梁——AI 声明意图，客户端决定执行 |
| **Renderer** | 将 JSON 解释为原生组件的引擎 | 消费端——跨平台的关键 |

---

## 核心架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      A2UI 架构                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ① Agent 层（LLM）                                               │
│     └─ 生成 A2UI JSON 消息（声明式 UI 描述）                       │
│                                                                 │
│  ② 传输层（WebSocket / SSE / A2A）                               │
│     └─ 将 JSON 传输到客户端（支持流式）                           │
│                                                                 │
│  ③ 渲染层（Lit / Flutter / Angular）                             │
│     ├─ 解析 JSON，校验组件白名单                                  │
│     ├─ 映射到原生组件                                            │
│     └─ 处理 Action（映射到安全函数）                              │
│                                                                 │
│  ④ 用户交互                                                      │
│     └─ UserAction 消息回传给 Agent                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

在 Agent 协议栈中的位置：
┌───────────────────────────────────────────┐
│ MCP  → Agent 获取能力（工具/数据）         │
│ A2A  → Agent 之间协作                     │
│ AG-UI → 通信管道（怎么传）                 │
│ A2UI → UI 内容（传什么）  ← 我们讨论的这个  │
└───────────────────────────────────────────┘
```

---

## 行动清单

| 推论 | 行动 | 预计时间 |
|------|------|----------|
| 声明式 JSON 杜绝代码注入 | 了解 A2UI JSON 格式规范 | ⏱️ 30 分钟 |
| 邻接表结构支持流式 | 对比树形结构和邻接表结构的优劣 | ⏱️ 30 分钟 |
| 与 json-render 对比 | 分析两者的设计差异和适用场景 | ⏱️ 1 小时 |
| 跨平台渲染 | 评估团队是否有多平台 AI UI 需求 | ⏱️ 持续 |

---

## 个人思考

{留空，供后续补充}

---

## 延伸阅读

- [A2UI GitHub](https://github.com/google/A2UI) - 项目源码
- [A2UI 协议规范](https://github.com/google/A2UI/blob/main/spec.md) - 完整协议文档
- [json-render](./2026-01-19-json-render-ai-ui-generation.md) - Vercel 的同类方案（相关笔记）

---

## 补充材料

<details>
<summary>📋 邻接表 vs 树形结构对比</summary>

**传统树形结构（对 LLM 不友好）：**
```json
{
  "Card": {
    "children": {
      "Column": {
        "children": [
          { "Text": { "text": "标题" } }
        ]
      }
    }
  }
}
```

**A2UI 邻接表结构（LLM 友好）：**
```json
{
  "components": [
    { "id": "card", "Card": { "child": "content" } },
    { "id": "content", "Column": { "children": ["title", "body"] } },
    { "id": "title", "Text": { "text": { "literalString": "标题" } } },
    { "id": "body", "Text": { "text": { "literalString": "内容" } } }
  ],
  "root": "card"
}
```

| 优势 | 说明 |
|------|------|
| **流式友好** | LLM 可以一个接一个输出组件，不需要等待完整树 |
| **增量更新** | 只需发送修改部分的 ID，不用重传整棵树 |
| **容错性强** | 即使某个组件出错，其他组件仍可渲染 |
| **低出错率** | 不需要匹配括号/标签，减少 LLM 生成错误 |

</details>

<details>
<summary>🔧 完整示例：餐厅预订界面</summary>

```json
{
  "surfaceUpdate": {
    "surfaceId": "restaurant-booking",
    "components": [
      { "id": "card", "Card": { "child": "form" } },
      { "id": "form", "Column": { "children": ["title", "date", "time", "submit"] } },
      { "id": "title", "Text": { "text": { "literalString": "餐厅预订" }, "usageHint": "h1" } },
      { "id": "date", "TextField": { "placeholder": "选择日期", "path": "/booking/date" } },
      { "id": "time", "TextField": { "placeholder": "选择时间", "path": "/booking/time" } },
      { "id": "submit", "Button": { 
          "child": "submit-text", 
          "action": { "name": "confirm_booking" },
          "style": "primary"
        } 
      },
      { "id": "submit-text", "Text": { "text": { "literalString": "确认预订" } } }
    ]
  }
}
```

</details>

<details>
<summary>📊 与其他方案对比</summary>

| 方案 | 安全性 | 跨平台 | 流式支持 | 可编辑 |
|------|--------|--------|----------|--------|
| **直接生成 HTML/JS** | ❌ | ❌ | ❌ | ✅ |
| **Streamlit** | ✅ | ❌ (仅 Web) | ❌ | ❌ |
| **Gradio** | ✅ | ❌ (仅 Web) | ❌ | ❌ |
| **CopilotKit** | ✅ | ❌ (仅 React) | ✅ | ✅ |
| **A2UI** | ✅ | ✅ | ✅ | ✅ |
| **json-render** | ✅ | ❌ (仅 React) | ✅ | ✅ |

</details>

---

## 金句摘录

> "如果说 ChatGPT 让 AI 学会了说话，那么 A2UI 就是给了 AI 一支画笔，让它能够用界面与人类交流。"

> "像数据一样安全，像代码一样强大——这是 A2UI 的核心设计理念。"

> "邻接表模型的精髓：LLM 可以一个接一个输出组件，不需要等待完整树。"
