# MCP Server 与 Client 概念精读

> 精读日期：2026-01-05
> 原文链接：
> - https://modelcontextprotocol.io/docs/learn/server-concepts
> - https://modelcontextprotocol.io/docs/learn/client-concepts

## 一、文档定位

这两篇文档回答了：
- **Server 文档**：MCP 服务器能提供什么能力？（供给侧）
- **Client 文档**：MCP 客户端能提供什么能力？（需求侧）

```
┌─────────────────────────────────────────────────────────┐
│                    MCP 能力全景图                        │
├─────────────────────────┬───────────────────────────────┤
│   Server 提供（给 AI）   │   Client 提供（给 Server）     │
├─────────────────────────┼───────────────────────────────┤
│  Tools    - 可执行函数   │  Elicitation - 请求用户输入   │
│  Resources - 只读数据    │  Roots       - 文件系统边界   │
│  Prompts  - 交互模板     │  Sampling    - 借用 LLM 能力  │
└─────────────────────────┴───────────────────────────────┘
```

---

## 第一部分：Server 概念深度解析

### 1. 三大核心原语对比

| 维度 | Tools（工具） | Resources（资源） | Prompts（提示） |
|------|--------------|------------------|----------------|
| **本质** | 可执行的函数 | 只读的数据源 | 预设的模板 |
| **控制方** | 模型决定 | 应用程序决定 | 用户决定 |
| **读/写** | 可读可写 | 只读 | 只读 |
| **触发方式** | AI 自主调用 | 应用自动加载 | 用户显式调用 |
| **典型场景** | 发邮件、查航班 | 读文档、查日历 | "/plan-vacation" |

**思考题**：为什么要区分 Tools 和 Resources？

> 答案：**安全性**。Tools 可以产生副作用（写数据库、发消息），需要更严格的控制；Resources 只读，风险更低。这种区分让权限管理更精细。

---

### 2. Tools 详解

**定义结构**（JSON Schema）：

```json
{
  "name": "searchFlights",
  "description": "搜索可用航班",
  "inputSchema": {
    "type": "object",
    "properties": {
      "origin": { "type": "string", "description": "出发城市" },
      "destination": { "type": "string", "description": "到达城市" },
      "date": { "type": "string", "format": "date" }
    },
    "required": ["origin", "destination", "date"]
  }
}
```

**协议操作**：

| 方法 | 作用 |
|------|------|
| `tools/list` | 发现可用工具 |
| `tools/call` | 执行特定工具 |

**用户交互模式**：

```
用户请求 → AI 决定用工具 → [可能需要用户批准] → 执行 → 返回结果
```

**安全机制**：
- UI 显示可用工具列表
- 敏感操作需用户批准
- 可预批准安全操作
- 活动日志记录所有执行

---

### 3. Resources 详解

**两种发现模式**：

| 模式 | 说明 | URI 示例 |
|------|------|----------|
| **直接资源** | 固定 URI，指向特定数据 | `calendar://events/2024` |
| **资源模板** | 带参数的动态 URI | `travel://activities/{city}/{category}` |

**协议操作**：

| 方法 | 作用 |
|------|------|
| `resources/list` | 列出直接资源 |
| `resources/templates/list` | 发现资源模板 |
| `resources/read` | 读取资源内容 |
| `resources/subscribe` | 订阅资源变化 |

**参数补全**：

```
输入: weather://forecast/Par
建议: ["Paris", "Park City"]
```

---

### 4. Prompts 详解

**定义结构**：

```json
{
  "name": "plan-vacation",
  "title": "规划假期",
  "description": "指导假期规划过程",
  "arguments": [
    { "name": "destination", "type": "string", "required": true },
    { "name": "duration", "type": "number", "description": "天数" },
    { "name": "budget", "type": "number", "required": false }
  ]
}
```

**触发方式**：
- 斜杠命令：`/plan-vacation`
- 命令面板
- UI 按钮
- 右键菜单

**与 Tools 的区别**：

| 维度 | Tools | Prompts |
|------|-------|---------|
| 谁触发 | AI 自主决定 | 用户显式调用 |
| 何时用 | 运行时动态 | 开始时选择 |
| 作用 | 执行操作 | 引导对话 |

---

## 第二部分：Client 概念深度解析

### 1. 三大核心能力

| 能力 | 作用 | 方向 |
|------|------|------|
| **Elicitation** | Server 向用户请求输入 | Server → User |
| **Roots** | 定义文件系统边界 | Client → Server |
| **Sampling** | Server 借用 LLM 能力 | Server → Client → LLM |

---

### 2. Elicitation（引导）详解

**场景**：Server 执行过程中需要用户确认或补充信息。

**请求结构**：

```json
{
  "method": "elicitation/requestInput",
  "params": {
    "message": "请确认您的预订详情：",
    "schema": {
      "type": "object",
      "properties": {
        "confirmBooking": {
          "type": "boolean",
          "description": "确认预订（机票+酒店=3000美元）"
        },
        "seatPreference": {
          "type": "string",
          "enum": ["靠窗", "过道", "无偏好"]
        }
      },
      "required": ["confirmBooking"]
    }
  }
}
```

**用户交互**：

```
Server 请求 → Client 展示 UI → 用户填写 → 返回给 Server
```

**安全原则**：
- 清晰显示哪个 Server 在询问
- 用户可拒绝提供信息
- **永不请求密码或 API 密钥**

---

### 3. Roots（根目录）详解

**作用**：告诉 Server 可以操作哪些目录。

**结构**：

```json
{
  "uri": "file:///Users/agent/travel-planning",
  "name": "旅行规划工作区"
}
```

**重要理解**：

> Roots 是**协调机制**，不是**安全边界**。它帮助 Server 理解工作范围，防止意外行为，但不能阻止恶意行为。

**配置方式**：
- 自动检测：用户打开文件夹时自动设置
- 手动配置：高级用户指定多个根目录

---

### 4. Sampling（采样）详解

**场景**：Server 需要 AI 帮忙分析，但自己没有 LLM。

**请求结构**：

```json
{
  "messages": [
    {
      "role": "user",
      "content": "分析这些航班选项并推荐最佳选择：\n[47个航班详情]"
    }
  ],
  "modelPreferences": {
    "hints": [{"name": "claude-sonnet-4-20250514"}],
    "costPriority": 0.3,
    "speedPriority": 0.2,
    "intelligencePriority": 0.9
  },
  "systemPrompt": "您是旅行专家",
  "maxTokens": 1500
}
```

**工作流程**：

```
Server 需要 AI 分析
    ↓
发送 Sampling 请求给 Client
    ↓
Client 调用 Host 的 LLM
    ↓
返回结果给 Server
```

**为什么这样设计？**
- Server 无需集成或付费 AI 模型
- Client 保持完全控制（权限、安全）
- 上下文边界清晰

---

## 第三部分：多服务器协作示例

以"旅行规划"为例，展示三个 Server 如何协作：

```
┌─────────────────────────────────────────────────────────┐
│                      MCP Host                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │Travel Client│ │Weather Client│ │Calendar Client│     │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘       │
└─────────┼───────────────┼───────────────┼──────────────┘
          │               │               │
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────┐
   │Travel Server│  │Weather Server│ │Calendar Server│
   │  航班、酒店  │  │  天气预报   │  │  日程管理   │
   └────────────┘  └────────────┘  └────────────┘
```

**完整流程**：

1. 用户调用 `/plan-vacation destination=Barcelona`（Prompt）
2. AI 读取日历资源，了解用户空闲时间（Resource）
3. AI 调用航班搜索工具（Tool）
4. AI 调用天气查询工具（Tool）
5. Server 请求用户确认预订（Elicitation）
6. AI 调用预订工具完成预订（Tool）

---

## 第四部分：关键设计原则

| 原则 | Server 体现 | Client 体现 |
|------|-------------|-------------|
| **用户控制** | Tools 可能需批准 | Elicitation 可拒绝 |
| **安全优先** | 区分读写操作 | Roots 限定范围 |
| **灵活性** | 动态工具列表 | 动态根目录 |
| **透明度** | 活动日志 | 显示请求来源 |

---

## 第五部分：速记要点

### Server 三原语

1. **Tools** = 可执行函数（模型控制，可读写）
2. **Resources** = 只读数据（应用控制，URI 寻址）
3. **Prompts** = 交互模板（用户控制，斜杠命令）

### Client 三能力

1. **Elicitation** = Server 向用户要信息
2. **Roots** = 告诉 Server 可操作的目录
3. **Sampling** = Server 借用 LLM 能力

### 协议方法速查

| 类别 | 方法 |
|------|------|
| Tools | `tools/list`, `tools/call` |
| Resources | `resources/list`, `resources/read`, `resources/subscribe` |
| Prompts | `prompts/list`, `prompts/get` |
| Elicitation | `elicitation/requestInput` |

---

## 相关文档

- [MCP 架构概述](./2026-01-05-mcp-architecture.md)
