---
title: Vibe Kanban - AI 编程代理编排平台实操指南
source: https://www.vibekanban.com/docs
author: Vibe Kanban 官方
date: 2026-01-16
category: ai-tools
subcategory: ai-ide
tags: [vibe-kanban, multi-agent, claude-code, cursor, mcp, ai-orchestration]
---

# Vibe Kanban：AI 编程代理编排平台实操指南

> 📖 原文：[Vibe Kanban 官方文档](https://www.vibekanban.com/docs)
> 📅 学习日期：2026-01-16
> 🏷️ 分类：AI 工具与效率 / AI IDE

---

## 一句话总结

Vibe Kanban 是一个 AI 编程代理编排平台，通过隔离的 git worktree 环境让多个 AI Agent 并行处理编码任务，并提供可视化审查和统一管理界面。

---

## 核心概念

### 什么是 Vibe Kanban？

Vibe Kanban 是一个面向 **AI 编程代理的编排平台**，帮助开发者规划、审查并安全地执行 AI 辅助的编码任务。

**核心特点：**
- **隔离执行**：每个任务在独立的 git worktree 中运行，互不干扰
- **多代理支持**：支持 Claude Code、Cursor Agent、OpenCode、Codex、Gemini 等多种 AI 代理
- **可视化审查**：逐行对比代码变更，支持添加评论反馈

### 为什么需要 Vibe Kanban？

| 痛点 | Vibe Kanban 解决方案 |
|------|---------------------|
| AI 代理可能破坏主分支 | 隔离的 git worktree 环境 |
| 不同 AI 工具切换成本高 | 统一的多代理管理界面 |
| AI 代码变更难以审查 | 可视化 diff + 评论反馈 |
| 任务执行缺乏追踪 | 看板式任务管理 |

## 安装与启动

### 快速启动

```bash
# 使用 npx 直接运行（推荐）
npx vibe-kanban

# 或全局安装
npm install -g vibe-kanban
vibe-kanban
```

启动后访问：`http://127.0.0.1:59683`

### 配置文件位置

- **macOS**: `~/Library/Application Support/ai.bloop.vibe-kanban/`
- **Linux**: `~/.local/share/vibe-kanban/`

主要配置文件：
- `profiles.json` - 代理配置
- `config.json` - 全局设置
- `db.sqlite` - 项目数据

## 代理配置实践

### 支持的代理类型

| 代理 | 说明 | 认证方式 |
|------|------|----------|
| CLAUDE_CODE | Anthropic Claude Code | 浏览器 OAuth |
| CURSOR_AGENT | Cursor IDE CLI | `cursor-agent login` |
| OPENCODE | OpenCode 终端助手 | 环境变量 API Key |
| CODEX | OpenAI Codex | API Key |
| GEMINI | Google Gemini | API Key |
| COPILOT | GitHub Copilot | GitHub 登录 |

### Claude Code 配置示例

在 `profiles.json` 中添加：

```json
{
  "executors": {
    "CLAUDE_CODE": {
      "DEFAULT": {
        "CLAUDE_CODE": {
          "append_prompt": null,
          "dangerously_skip_permissions": true
        }
      },
      "OPUS": {
        "CLAUDE_CODE": {
          "append_prompt": null,
          "model": "opus",
          "dangerously_skip_permissions": true
        }
      },
      "INTERNAL": {
        "CLAUDE_CODE": {
          "append_prompt": null,
          "base_command_override": "claude-internal",
          "dangerously_skip_permissions": true
        }
      }
    }
  }
}
```

### 关键配置参数

| 参数 | 适用代理 | 说明 |
|------|----------|------|
| `dangerously_skip_permissions` | CLAUDE_CODE | 跳过权限确认，自动执行 |
| `plan` | CLAUDE_CODE | 启用规划模式处理复杂任务 |
| `force` | CURSOR_AGENT | 强制执行无需确认 |
| `yolo` | GEMINI | 无需确认直接运行 |
| `auto_approve` | OPENCODE | 自动批准操作 |
| `base_command_override` | 通用 | 覆盖底层 CLI 命令 |
| `model` | 多种 | 指定使用的模型 |

### 使用 claude-internal 替代 OpenCode

如果本地已安装 `claude-internal`（腾讯内部 Claude Code 封装），可以用它替代需要 API Key 的 OpenCode：

```bash
# 检查是否已安装
which claude-internal

# 启动命令
claude-internal
```

配置方式：在 CLAUDE_CODE 下添加 `INTERNAL` 配置，设置 `base_command_override: "claude-internal"`。

## 工作流程

### 1. 创建项目

1. 访问 Vibe Kanban 首页
2. 点击「创建项目」
3. 关联本地 Git 仓库路径

### 2. 创建任务

1. 进入项目看板
2. 创建新任务卡片
3. 选择代理和配置
4. 描述任务需求

### 3. 执行与监控

- 任务自动在独立 worktree 中执行
- 实时查看执行日志
- 可随时暂停/继续

### 4. 代码审查

- 查看逐行 diff
- 添加评论反馈给 AI
- 批准或拒绝变更

### 5. 合并完成

- 审查通过后合并到主分支
- 自动清理 worktree

## MCP 服务器集成

### 为代理添加 MCP 能力

Vibe Kanban 支持配置 MCP（Model Context Protocol）服务器扩展代理能力：

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_API_KEY"
      }
    },
    "chrome_devtools": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    }
  }
}
```

### Vibe Kanban 作为 MCP Server

Vibe Kanban 本身也提供 MCP Server，可以被其他 IDE（如 Cursor、Claude Desktop）调度。

**在 Cursor 或 Claude Desktop 的 MCP 配置中添加：**

```json
{
  "mcpServers": {
    "vibe_kanban": {
      "command": "npx",
      "args": ["-y", "vibe-kanban@latest", "--mcp"]
    }
  }
}
```

**可调度的操作：**

| 工具 | 功能 |
|------|------|
| `list_projects` | 列出所有项目 |
| `list_tasks` | 列出任务（支持状态筛选） |
| `create_task` | 创建新任务 |
| `start_workspace_session` | **启动代理执行任务** |
| `get_task` / `update_task` | 查看/更新任务 |

**典型场景 - 在 Cursor 中对话：**

> "帮我在 bk-sap-api 项目创建 3 个任务：用户认证、权限管理、日志系统，然后用 Claude Code 开始处理第一个任务"

Cursor 会自动调用：
1. `list_projects` → 找到项目 ID
2. `create_task` × 3 → 创建三个任务
3. `start_workspace_session` → 启动 Claude Code 执行

### 工作原理

```
┌─────────────────┐     ┌─────────────────┐
│  Vibe Kanban    │     │  Cursor/Claude  │
│  Web UI         │     │  Desktop        │
│  (端口 59683)    │     │                 │
└────────┬────────┘     └────────┬────────┘
         │                       │
         │   npx vibe-kanban     │   npx vibe-kanban --mcp
         │                       │
         └───────────┬───────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │  共享本地数据库        │
         │  ~/Library/Application│
         │  Support/ai.bloop.    │
         │  vibe-kanban/db.sqlite│
         └───────────────────────┘
```

**关键点：**

1. **不是连接到已有服务** - MCP 客户端自己启动 `vibe-kanban --mcp` 进程
2. **共享数据库** - Web UI 和 MCP 进程读写同一个 SQLite 数据库
3. **无需端口** - MCP 模式通过 stdin/stdout 通信，不监听网络端口

**这意味着：**
- 在 Cursor 里创建的任务 → 会出现在 Web UI 看板上
- 在 Web UI 创建的任务 → Cursor 也能查到并调度
- 可以用 Cursor/Claude Desktop 作为"指挥中心"，让 Vibe Kanban 调度多个 AI Agent 并行处理不同任务

## 最佳实践

### 代理选择建议

| 场景 | 推荐代理 | 理由 |
|------|----------|------|
| 复杂重构任务 | CLAUDE_CODE (OPUS) | 推理能力强 |
| 日常编码任务 | CLAUDE_CODE (DEFAULT) | 性价比高 |
| 省 Token 场景 | INTERNAL (claude-internal) | 免费额度 |
| 需要最新模型 | CURSOR_AGENT | 支持多种最新模型 |

### 安全建议

1. 生产环境慎用 `dangerously_skip_permissions`
2. 重要变更前先备份
3. 定期审查 AI 生成的代码
4. 敏感仓库注意模型选择限制

## 常见问题

### Q: 保存配置报错 "Built-in executor cannot be deleted"

A: 系统内置的 executor（如 CODEX、CLAUDE_CODE 等）不能删除，只能修改或添加新配置。

### Q: OpenCode auth login 卡住不动

A: OpenCode 通过环境变量配置 API Key，不是通过 `auth login` 命令。设置 `ANTHROPIC_API_KEY` 或 `OPENAI_API_KEY` 环境变量即可。

### Q: 如何使用自定义 CLI 命令？

A: 使用 `base_command_override` 参数覆盖默认命令，如设置为 `claude-internal` 使用内部版本。

---

## 关键概念

| 概念 | 定义 | 应用场景 |
|------|------|----------|
| **Git Worktree** | 独立的工作目录，与主分支隔离 | 让 AI 代理安全执行任务，不影响主分支 |
| **Executor** | 代理执行器配置 | 定义不同 AI 代理的参数和行为 |
| **Profile** | 代理配置方案 | 为同一代理定义多种使用场景（如 DEFAULT、OPUS） |
| **MCP Server** | 模型上下文协议服务 | 让外部工具（Cursor）调度 Vibe Kanban |
| **base_command_override** | 命令覆盖参数 | 使用自定义 CLI（如 claude-internal）替代默认命令 |

---

## 实用技巧

| 技巧 | 操作方法 | 效果 |
|------|----------|------|
| 省 Token | 使用 `claude-internal` 替代 Claude Code | 使用免费内部额度 |
| 自动执行 | 开启 `dangerously_skip_permissions` | 跳过权限确认，任务自动运行 |
| 复杂任务 | 开启 `plan` 模式 | 启用规划模式，更好处理复杂任务 |
| 多代理并行 | 配置 MCP Server | 通过 Cursor 统一调度多个任务 |
| 安全审查 | 使用可视化 diff | 逐行审查 AI 生成代码 |

---

## 行动清单

### 立即可做（今天）
- [ ] 安装 Vibe Kanban：`npx vibe-kanban`（⏱️ 2分钟）
- [ ] 配置 `profiles.json` 添加常用代理（⏱️ 5分钟）

### 短期实践（本周）
- [ ] 创建一个项目并尝试任务执行流程（⏱️ 30分钟）
- [ ] 在 Cursor 中配置 Vibe Kanban MCP Server（⏱️ 15分钟）
- [ ] 尝试通过 Cursor 调度 Vibe Kanban 创建多任务（⏱️ 1小时）

### 长期提升（持续）
- [ ] 总结适合不同任务的代理选择策略
- [ ] 建立团队协作的代码审查工作流

---

## 个人思考

{留空，供后续补充}

---

## 延伸阅读

- [Vibe Kanban 官方文档](https://www.vibekanban.com/docs)
- [Claude Code 官方文档](https://docs.claude.com/en/docs/claude-code/quickstart)
- [Agent Profiles 配置指南](https://www.vibekanban.com/docs/configuration-customisation/agent-configurations)
- [MCP 架构概述](../mcp/2026-01-05-mcp-architecture.md) - 同仓库相关笔记
