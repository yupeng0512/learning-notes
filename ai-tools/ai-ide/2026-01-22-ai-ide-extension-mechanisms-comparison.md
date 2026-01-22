# AI IDE 扩展机制对比：Rules/Skills/Commands/Agents

> **学习日期**：2026-01-22
> **来源**：多平台官方文档与社区实践
> **标签**：#AI-IDE #Cursor #Claude-Code #Gemini-CLI #GitHub-Copilot #扩展机制

---

## TL;DR 速览

| 工具 | 配置/Rules | 命令扩展 | Agent/SubAgent | MCP 支持 |
|------|-----------|----------|----------------|----------|
| **Cursor** | `.cursor/rules/*.mdc` | 无原生 | Agent Mode | ✅ |
| **Claude Code** | `CLAUDE.md` | Slash Commands | SubAgent | ✅ 原生 |
| **Gemini CLI** | `GEMINI.md` | 无原生 | Agent Mode | 计划中 |
| **GitHub Copilot** | `copilot-instructions.md` | @mentions | Agent Mode | ✅ |
| **Windsurf** | Project Rules | Cascade | Flow Action | ✅ |
| **Cline** | Custom Instructions | 无原生 | 单体 Agent | ✅ |
| **Aider** | `.aider.conf.yml` | 无原生 | Architect Mode | ❌ |
| **CodeBuddy** | Skills | Commands | SubAgent | ✅ |

---

## 根节点命题

> **AI IDE 的扩展机制 = 上下文工程的产品化实现**

所有 AI IDE 的 Rules/Skills/Commands/Agents 本质上都是在解决同一个问题：**如何高效地向 LLM 注入正确的上下文**。不同产品的设计差异反映了它们对"谁来决定注入什么上下文"这个问题的不同回答。

---

## 第一部分：概念术语表

### 1.1 核心概念定义

| 术语 | 定义 | 类比 |
|------|------|------|
| **Rules** | 静态配置文件，定义 AI 行为规范和约束 | 操作系统的配置文件 |
| **Skills** | 可复用的能力包，包含知识 + 工作流 | 插件/技能包 |
| **Commands** | 用户触发的快捷操作，执行预定义任务 | Shell 脚本/快捷指令 |
| **Agent** | 自主决策的 AI 实体，可调用工具完成任务 | 自动驾驶模式 |
| **SubAgent** | 主 Agent 派生的专门子任务执行者 | 团队成员/下属 |
| **Hooks** | 事件驱动的自动化脚本，在特定时机触发 | Git Hooks |
| **MCP** | Model Context Protocol，标准化工具调用协议 | API 标准 |

### 1.2 三层架构模型

```
┌─────────────────────────────────────────────────────────────────┐
│                      AI IDE 扩展机制三层架构                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    第一层：静态配置                       │   │
│   │   Rules / Instructions / CLAUDE.md / GEMINI.md          │   │
│   │   → 始终注入的基础上下文                                  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              ▼                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    第二层：动态触发                       │   │
│   │   Commands / Skills / Hooks / @mentions                 │   │
│   │   → 按需激活的专业能力                                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              ▼                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    第三层：自主执行                       │   │
│   │   Agent Mode / SubAgent / Flow Action                   │   │
│   │   → AI 自主决策调用工具                                   │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 第二部分：各 AI IDE 详解

### 2.1 Cursor

#### 2.1.1 Rules 系统

**配置位置**：
- 全局：`Settings > General > Rules for AI`
- 项目：`.cursor/rules/*.mdc` （推荐）
- 旧版：`.cursorrules`（根目录，已废弃）

**四种 Rule 类型**：

| 类型 | 触发方式 | 适用场景 |
|------|----------|----------|
| **Always** | 始终应用 | 通用编码规范、语言偏好 |
| **Auto Attached** | 匹配 glob 模式时自动附加 | 特定文件类型规则（如 `*.tsx`） |
| **Agent Requested** | AI 自主判断是否需要 | 复杂的领域知识（需提供描述） |
| **Manual** | 用户 `@规则名` 显式触发 | 特殊场景规则 |

**Rule 文件格式 (.mdc)**：
```markdown
---
description: "数据库操作规范"
globs: ["**/db/**/*.ts", "**/models/**/*.ts"]
alwaysApply: false
---

# 数据库规范

1. 所有查询必须使用参数化
2. 必须处理连接池
3. 事务操作需要 try-catch-finally
```

**最佳实践**：
- 保持单个 Rule 文件 < 500 行
- 使用 glob 精确匹配，避免过度触发
- 团队共享 Rules 提交到 Git

#### 2.1.2 Agent Mode

**定位**：Cursor 的 Agent Mode 是其核心差异化功能，允许 AI 自主：
- 遍历项目文件
- 执行终端命令
- 修改多个文件
- 调用 MCP 工具

**与 Ask 模式对比**：

| 特性 | Ask 模式 | Agent 模式 |
|------|----------|------------|
| 文件操作 | 需用户确认 | 自主执行 |
| 终端命令 | 不支持 | 自主执行 |
| 多文件修改 | 手动逐个 | 批量处理 |
| 上下文管理 | 用户控制 | AI 自主获取 |

---

### 2.2 Claude Code

#### 2.2.1 CLAUDE.md 记忆系统

**配置层级**（优先级从低到高）：

| 位置 | 作用域 | 是否提交 Git |
|------|--------|-------------|
| `~/.claude/CLAUDE.md` | 全局所有项目 | 否 |
| `父目录/CLAUDE.md` | Monorepo 共享 | 是 |
| `项目根目录/CLAUDE.md` | 当前项目 | 是（推荐） |
| `子目录/CLAUDE.md` | 特定模块 | 是 |
| `CLAUDE.local.md` | 本地覆盖 | 否（加入 .gitignore） |

**创建方式**：
```bash
# 自动生成（推荐）
/init

# 手动编辑
/memory
```

#### 2.2.2 Slash Commands

**内置命令**：

| 命令 | 功能 |
|------|------|
| `/init` | 初始化项目，生成 CLAUDE.md |
| `/compact` | 压缩对话上下文 |
| `/memory` | 编辑记忆文件 |
| `/mcp` | 管理 MCP 服务器 |
| `/agents` | 管理 SubAgent |
| `/review` | 代码审查 |

**自定义命令**：

在 `.claude/commands/` 目录创建 `.md` 文件：

```markdown
---
name: "code-review"
description: "执行代码审查"
---

请对以下代码进行审查：
1. 安全性检查
2. 性能优化建议
3. 代码风格问题

$ARGUMENTS
```

#### 2.2.3 SubAgent 机制

**设计理念**：解决长对话上下文退化问题

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude Code SubAgent 架构                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   用户请求                                                       │
│      │                                                          │
│      ▼                                                          │
│   ┌─────────────┐                                               │
│   │  主 Agent   │ ← 持有完整上下文                               │
│   └─────┬───────┘                                               │
│         │                                                       │
│         │ Task 工具调用                                          │
│         ▼                                                       │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              SubAgent Pool (并发执行)                    │   │
│   ├─────────────┬─────────────┬─────────────────────────────┤   │
│   │ SubAgent A  │ SubAgent B  │ SubAgent C                  │   │
│   │ 搜索代码    │ 分析依赖    │ 生成测试                     │   │
│   │ (隔离上下文) │ (隔离上下文) │ (隔离上下文)                 │   │
│   └─────────────┴─────────────┴─────────────────────────────┘   │
│         │             │              │                          │
│         └─────────────┴──────────────┘                          │
│                       │                                         │
│                       ▼                                         │
│                  结果聚合回主 Agent                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**核心优势**：
1. **上下文隔离**：SubAgent 拥有独立上下文，不污染主对话
2. **并发执行**：多个 SubAgent 可并行工作
3. **工具选择精准**：SubAgent 只看到相关工具，减少幻觉

**管理命令**：
```bash
/agents           # 列出所有 SubAgent
/agents add       # 添加自定义 SubAgent
/agents remove    # 移除 SubAgent
```

#### 2.2.4 Hooks 系统

**事件类型**：

| 事件 | 触发时机 | 典型用途 |
|------|----------|----------|
| `PreToolUse` | 工具执行前 | 权限控制、参数修改 |
| `PostToolUse` | 工具执行后 | 自动格式化、运行测试 |
| `UserPromptSubmit` | 用户提交后 | 添加上下文、内容审核 |
| `Stop` | Agent 完成响应 | 判断是否继续 |
| `SubagentStop` | SubAgent 完成 | 检查子任务 |
| `SessionStart` | 会话开始 | 环境初始化 |
| `SessionEnd` | 会话结束 | 清理资源 |

**配置示例**：
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

---

### 2.3 GitHub Copilot

#### 2.3.1 Custom Instructions

**配置文件**：

| 文件 | 作用域 |
|------|--------|
| `.github/copilot-instructions.md` | 项目全局规则 |
| `.github/instructions/*.instructions.md` | 任务级规则 |

**示例**：
```markdown
# Copilot Instructions

## 编码规范
- 使用 TypeScript 严格模式
- 函数必须有 JSDoc 注释
- 错误处理使用 Result 模式

## 命名约定
- 组件：PascalCase
- 函数：camelCase
- 常量：SCREAMING_SNAKE_CASE
```

#### 2.3.2 Agent Mode

**功能**：
- 自动拆分任务
- 遍历项目文件
- 执行终端命令
- 修改多文件

**MCP 配置**：
```json
{
  "servers": {
    "GitHub": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your-token"
      }
    }
  }
}
```

#### 2.3.3 @mentions 系统

| 提及 | 功能 |
|------|------|
| `@workspace` | 项目全局感知 |
| `@terminal` | 终端上下文 |
| `@vscode` | 编辑器状态 |

---

### 2.4 Gemini CLI

#### 2.4.1 GEMINI.md

**配置位置**：
- 项目根目录：`GEMINI.md`
- 全局配置：`~/.gemini/GEMINI.md`

**特性**：
- 100 万 Token 上下文窗口
- 每天 1000 次免费请求
- 原生 Agent 模式

#### 2.4.2 Agent 模式

**定位**：开源的命令行 AI Agent

**核心能力**：
- 文件读写
- 终端命令执行
- 代码搜索
- 内容生成

---

### 2.5 Windsurf (Codeium)

#### 2.5.1 Cascade

**定位**：深度上下文感知的 AI 助手

**特性**：
- 实时感知代码变化
- 自动理解项目结构
- 支持多文件编辑

#### 2.5.2 Flow Action

**定位**：类似 Claude Code 的 SubAgent，执行复杂任务流

---

### 2.6 Cline

#### 2.6.1 Custom Instructions

**配置方式**：设置面板中直接编辑

**Memory Bank**：
- 动态上下文存储
- 记录开发决策和进度
- 跨会话持久化

**与 Cline Rules 区别**：

| 特性 | Cline Rules | Memory Bank |
|------|-------------|-------------|
| 类型 | 静态配置 | 动态存储 |
| 内容 | 编码规范 | 决策、进度、状态 |
| 更新 | 手动 | 自动 |

---

### 2.7 Aider

#### 2.7.1 配置系统

**配置文件**：`.aider.conf.yml`

**Architect Mode**：
- 先规划后执行
- 分离设计与实现
- 适合大型重构

---

## 第三部分：横向对比

### 3.1 配置机制对比

| 工具 | 配置文件 | 层级支持 | 动态触发 | Git 友好 |
|------|----------|----------|----------|----------|
| Cursor | `.cursor/rules/*.mdc` | ✅ 多层 | ✅ 4种类型 | ✅ |
| Claude Code | `CLAUDE.md` | ✅ 多层 | ✅ Hooks | ✅ |
| Copilot | `copilot-instructions.md` | ⚠️ 2层 | ❌ | ✅ |
| Gemini CLI | `GEMINI.md` | ⚠️ 2层 | ❌ | ✅ |

### 3.2 Agent 能力对比

| 工具 | 自主执行 | SubAgent | 并发 | MCP |
|------|----------|----------|------|-----|
| Cursor | ✅ | ❌ | ⚠️ | ✅ |
| Claude Code | ✅ | ✅ 原生 | ✅ | ✅ |
| Copilot | ✅ | ❌ | ❌ | ✅ |
| Gemini CLI | ✅ | ❌ | ❌ | 计划中 |

### 3.3 扩展性对比

| 工具 | 自定义命令 | 自定义工具 | Hooks | 插件系统 |
|------|-----------|-----------|-------|----------|
| Cursor | ❌ | MCP | ❌ | VS Code 扩展 |
| Claude Code | ✅ 原生 | MCP | ✅ 原生 | ✅ 插件目录 |
| Copilot | ❌ | MCP | ❌ | VS Code 扩展 |

---

## 第四部分：最佳实践

### 4.1 选型建议

| 场景 | 推荐工具 | 原因 |
|------|----------|------|
| 快速原型 | Cursor | Agent Mode 开箱即用 |
| 企业级项目 | Claude Code | SubAgent + Hooks 可控性强 |
| 开源贡献 | GitHub Copilot | 与 GitHub 深度集成 |
| 终端用户 | Gemini CLI | 免费 + 大上下文 |

### 4.2 配置策略

**层级规划**：
```
~/.tool/config.md          # 个人全局偏好
project/CONFIG.md          # 团队共享规范
project/module/CONFIG.md   # 模块特定规则
project/CONFIG.local.md    # 本地覆盖（不提交）
```

**内容组织**：
```markdown
# 项目配置

## 技术栈
- 语言：TypeScript 5.x
- 框架：React 18 + Next.js 14
- 状态管理：Zustand

## 编码规范
- 优先使用函数式组件
- 错误处理使用 Result 模式
- 测试覆盖率 > 80%

## 禁止事项
- 不使用 any 类型
- 不直接修改 state
- 不使用 var 声明
```

### 4.3 团队协作

**Git 策略**：
```gitignore
# .gitignore
*.local.md      # 本地配置不提交
.claude/settings.local.json
```

**代码审查清单**：
- [ ] Rules 文件是否更新
- [ ] 新增 Commands 是否文档化
- [ ] Hooks 是否经过测试

---

## 第五部分：未来趋势

### 5.1 趋同方向

1. **MCP 标准化**：所有主流工具都在拥抱 MCP
2. **多 Agent 协作**：SubAgent 模式将成为标配
3. **Hooks 生态**：事件驱动的自动化成为主流

### 5.2 差异化方向

| 工具 | 差异化重点 |
|------|-----------|
| Cursor | IDE 体验 + 模型切换 |
| Claude Code | 终端体验 + SubAgent 架构 |
| Copilot | GitHub 生态整合 |
| Gemini CLI | 开源 + 大上下文 |

---

## 参考资料

- [Cursor Rules 官方文档](https://docs.cursor.com/context/rules-for-ai)
- [Claude Code 官方文档](https://docs.anthropic.com/zh-CN/docs/claude-code)
- [GitHub Copilot 文档](https://docs.github.com/en/copilot)
- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Windsurf 官网](https://windsurf.com/)

---

## 学习反思

**核心洞察**：
1. AI IDE 的扩展机制本质是"上下文工程"的产品化
2. 静态配置 → 动态触发 → 自主执行是演进路径
3. SubAgent 架构是解决长对话退化的关键

**行动项**：
- [ ] 为常用项目配置标准 Rules
- [ ] 尝试 Claude Code SubAgent 进行并发任务
- [ ] 探索 Hooks 实现自动化工作流
