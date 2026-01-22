---
title: WeTERM AI 架构总览
source: https://iwiki.woa.com/p/4017283707
date: 2026-01-22
category: weterm-ai-architecture
tags: [AI Agent, 终端, 架构设计, Shell Hook]
type: architecture-doc
---

# WeTERM AI 架构总览

> 精读日期：2026-01-22
> 文档版本：v2.0 (2026-01-19)

---

## 根节点命题

> **在远程环境零依赖的约束下，通过 Shell Hook + 客户端增强实现完整 AI Agent 能力**

**为什么这是根节点**：整个架构的所有设计决策都围绕「远程主机无需安装二进制 CLI」这一核心约束展开。

---

## 表示空间

| 维度 | 含义 | 架构选择 |
|------|------|----------|
| **部署位置** | AI 能力在哪运行 | 客户端 + 服务端，非远程主机 |
| **能力注入方式** | 如何让远程感知 AI | Shell Hook（零二进制） |
| **演进阶段** | 功能完整度 | 三期递进（助手→Agent→多Agent协同） |
| **扩展机制** | 如何集成外部能力 | ACP + MCP 双协议 |

---

## 核心架构

### 四层分离

```
Client (weterm.js) → AI Agent → Server → AI Gateway → LLM
```

| 层级 | 核心组件 | 职责 |
|------|----------|------|
| **Client** | weterm.js, Command Block, Universal Input, Shell Init | 终端增强、命令捕获、交互入口 |
| **AI Agent** | WeTERM AI Agent, Observation AI | 命令生成、错误诊断、旁路观测 |
| **Server** | Prompt Manager, Config, Conversation | Prompt 编排、配置管理、对话存储 |
| **AI Gateway** | Model Router, Quota Manager, Channel Manager | 模型路由、配额管理、多渠道调度 |

### 双协议扩展

| 协议 | 方向 | 用途 |
|------|------|------|
| **ACP** | 与本地 Agent 通信 | 复用 Claude Code/gemini-cli 能力 |
| **MCP** | 工具扩展 | 让 AI 调用外部工具 |

---

## 推论展开

```
根节点：远程零依赖 + 完整 AI Agent
│
├─ 推论1：必须有 Shell Hook 机制
│   └─ 实现：shell-init 组件，通过 SSH 会话注入初始化脚本
│
├─ 推论2：命令上下文必须在客户端捕获
│   └─ 实现：Command Block 捕获命令生命周期
│
├─ 推论3：AI 计算必须在服务端/网关
│   └─ 实现：WeTERM Server + AI Gateway 分层
│
├─ 推论4：需要渐进式降级
│   └─ 实现：Feature Flag + Bootstrap 检测 + 自动回退 xterm.js
│
└─ 推论5：要支持多 Agent 协同
    └─ 实现：ACP 协议 + 本地 Agent 集成
```

---

## 能力演进路线

| 阶段 | 时间 | 核心能力 | 交互模式 |
|------|------|----------|----------|
| **第一期** | 2026.1-6 | 命令生成 + 错误诊断 + 补全 | 人主导，AI 建议 |
| **第二期** | 2026.6-12 | Prompt 管理 + TUI 识别 + 任务自动化 | AI 主导，人监督 |
| **第三期** | 2027.1-6 | 旁路观测 + ACP + 多 Agent 协同 | AI 团队，人统筹 |

---

## 架构亮点

1. **远程零依赖**：通过 Shell Hook 而非二进制实现能力注入
2. **渐进式降级**：完整功能 → Bootstrap 失败 → 回退 xterm.js
3. **四层关注点分离**：各层可独立演进、故障隔离
4. **双协议扩展**：ACP（Agent 协同）+ MCP（工具扩展）

---

## 技术栈

| 层级 | 技术选型 |
|------|----------|
| 前端 | xterm.js / weterm.js, React, WebSocket |
| 后端 | Go Gin, MySQL, Redis, OpenTelemetry |
| AI Gateway | bkaidev |
| 远程环境 | Shell Hook（Zsh/Bash），无额外依赖 |

---

## 思考

1. **Shell Hook vs 远程 Agent**：选择 Shell Hook 是安全合规 + 运维成本的权衡
2. **三期演进的本质**：从「AI 作为工具」到「AI 作为同事」
3. **与 Claude Code 的关系**：不是竞争，而是通过 ACP 复用其能力

---

## 延伸阅读

- [weterm-js](https://iwiki.woa.com/p/4017283713) - 客户端核心
- [command-block](https://iwiki.woa.com/p/4017283721) - 命令捕获
- [acp](https://iwiki.woa.com/p/4017283815) - Agent 协同协议
- [mcp-integration](https://iwiki.woa.com/p/4017283817) - 工具扩展
