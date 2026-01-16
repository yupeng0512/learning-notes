# OpenCode - 终端图形界面的 AI 编程 Agent

> 学习日期：2026-01-16
> 来源：https://github.com/anomalyco/opencode
> 类型：AI 工具学习

---

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **工具名称** | OpenCode |
| **GitHub** | anomalyco/opencode |
| **Stars** | 6 万+ |
| **核心价值** | 在命令行实现图形界面，比 Claude Code 更有极客感 |

> 💡 **一句话总结**：OpenCode 是一个拥有 6 万 Star 的终端 AI 编程 Agent，最大特点是在命令行里实现了"图形界面"——独立窗口、缓冲区管理、内置 Vim 模式，比 Claude Code 更有极客感。

---

## 🗺️ 知识地图

### 与 Claude Code 的核心区别

| 维度 | Claude Code | OpenCode |
|------|-------------|----------|
| **交互方式** | 流式对话输出（类 ChatGPT） | 终端图形界面（TUI） |
| **窗口管理** | 无 | 独立缓冲区、可滚动历史 |
| **文件编辑** | 调用外部编辑器或输出 diff | 内置 Vim 模式 |
| **视觉体验** | 传统终端 | 像操作简易编辑器 |
| **极客感** | 中等 | 极高 |

**类比**：如果 Claude Code 是"在终端里聊天"，OpenCode 就是"在终端里操作一个迷你 IDE"。

---

## 🔑 核心能力

```
OpenCode 能力矩阵
├── 对话能力
│   └── 直接在终端里与 AI 对话
│
├── 执行能力
│   ├── Shell 命令执行
│   ├── 文件搜索
│   └── 内置 Vim 模式直接修改代码
│
├── 智能体模式
│   ├── 开发模式（Development）
│   │   └── 适合日常编码、Bug 修复
│   └── 规划模式（Planning）
│       └── 适合复杂架构分析
│
├── 模型支持
│   ├── Claude
│   ├── Gemini
│   └── 本地模型
│
└── 扩展能力
    └── 完整支持 MCP 协议
```

---

## 🛠️ 安装方式

```bash
# 方式一：官方脚本
curl -fsSL https://opencode.ai/install | bash

# 方式二：Homebrew（macOS）
brew install anomalyco/tap/opencode
```

---

## 📊 两种智能体模式

| 模式 | 适用场景 | 特点 |
|------|----------|------|
| **开发模式** | 日常编码、简单 Bug 修复 | 快速响应、直接执行 |
| **规划模式** | 复杂架构分析、大型任务 | 先思考后行动、拆解步骤 |

---

## 💡 关键洞见

### 1. TUI（终端用户界面）的价值

**传统命令行工具的痛点**：
- 输出滚过去就找不到了
- 没法同时看多个内容
- 编辑需要切换工具

**OpenCode 的 TUI 解决方案**：
- **独立缓冲区**：对话历史可随时滚动查看
- **窗口管理**：像操作编辑器一样分区浏览
- **内置 Vim**：无需跳出即可编辑

### 2. MCP 协议的战略意义

> 全面支持 MCP 协议，可以根据自己的需求轻松扩展能力边界。

MCP = 模型上下文协议，是让 AI Agent 能调用外部工具的标准。支持 MCP 意味着：
- 可以接入任何 MCP 工具（数据库、API、浏览器等）
- 能力不受限于 OpenCode 本身
- 社区生态可以无限扩展

### 3. 多模型支持的灵活性

| 场景 | 推荐模型 |
|------|----------|
| 复杂推理 | Claude |
| 多模态任务 | Gemini |
| 隐私敏感 | 本地模型 |

---

## ⚠️ 与 oh-my-opencode 的关系

| 问题 | 答案 |
|------|------|
| 是同一个项目吗？ | **不是** |
| oh-my-opencode 能用在这个 OpenCode 上吗？ | **不能**（oh-my-opencode 是为 opencode-ai/opencode 设计的） |
| 这两个项目冲突吗？ | **不冲突**，完全独立 |

**项目对比**：

| 项目 | GitHub 地址 | 性质 |
|------|-------------|------|
| OpenCode (本文) | `anomalyco/opencode` | 独立终端 AI Agent |
| oh-my-opencode | `code-yeongyu/oh-my-opencode` | opencode-ai/opencode 的插件 |
| OpenCode (旧) | `opencode-ai/opencode` | 已归档 |

---

## 📝 金句摘录

> "你可以在终端里像操作一个简易编辑器一样去查看文件、滚动浏览历史记录。"

> "内置了开发和规划两种智能体模式，能够应对从简单 Bug 修复到复杂架构分析的各种任务。"

> "全面支持了 MCP 协议，你可以根据自己的需求轻松扩展它的能力边界。"

---

## ✅ 行动清单

### 立即可做（今天）
- [ ] 安装 OpenCode 体验 TUI 界面

### 短期实践（本周）
- [ ] 对比 Claude Code 和 OpenCode 的工作流差异
- [ ] 探索两种智能体模式的切换

### 长期探索
- [ ] 尝试接入 MCP 工具扩展能力
- [ ] 评估是否适合日常工作流

---

## 个人思考

{留空，供后续补充}

---

## 📚 延伸阅读

- [OpenCode GitHub](https://github.com/anomalyco/opencode) - 项目源码
- [MCP 协议](https://modelcontextprotocol.io/) - 模型上下文协议
- [oh-my-opencode 笔记](./2026-01-06-oh-my-opencode-multi-model-agent.md) - 相关但不同的项目

---

*笔记生成时间：2026-01-16*
