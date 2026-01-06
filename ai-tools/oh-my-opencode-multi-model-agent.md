# oh-my-opencode - 多模型协作的 AI 编程 Agent 插件

> 学习日期：2026-01-05
> 来源：https://github.com/code-yeongyu/oh-my-opencode
> 类型：AI 工具学习

---

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **工具名称** | oh-my-opencode |
| **作者** | code-yeongyu |
| **GitHub Stars** | 8.8k+ |
| **开发语言** | TypeScript |
| **核心价值** | 让 Claude、GPT、Gemini 多模型"组队干活"，发挥 1+1+1>3 的效果 |

> 💡 **一句话总结**：oh-my-opencode 是 OpenCode 的"超级插件"，通过智能代理编排，让多个 AI 模型各司其职、并行协作，将你的多个 AI 订阅从"轮流用"变成"一起用"。

---

## 🗺️ 知识地图

### 核心架构

```
┌─────────────────────────────────────────────────────┐
│         oh-my-opencode 多模型协作系统                │
├─────────────────────────────────────────────────────┤
│  Sisyphus（主代理）- Claude Opus 4.5                 │
│     ├── 统筹全局、任务分解、结果整合                  │
│     │                                               │
│     ├── Oracle (GPT-5.2)                            │
│     │   └── 架构设计、代码审查、策略制定              │
│     │                                               │
│     ├── Frontend Engineer (Gemini 3 Pro)            │
│     │   └── UI/UX 开发、前端代码                     │
│     │                                               │
│     ├── Librarian (Claude Sonnet / Gemini Flash)    │
│     │   └── 多仓库分析、文档查找                     │
│     │                                               │
│     ├── Explore (Grok Code / Gemini Flash)          │
│     │   └── 快速代码库探索                          │
│     │                                               │
│     └── Document Writer (Gemini Flash)              │
│         └── 技术文档编写                            │
└─────────────────────────────────────────────────────┘
```

---

## 🔑 关键概念

| 概念 | 定义 | 类比理解 |
|------|------|----------|
| **Agent 编排** | 让多个 AI 模型按分工协作 | 足球队战术配合，前锋、中场、后卫各司其职 |
| **并行执行** | 多个 Agent 同时工作，不互相等待 | 流水线作业，不是串行排队 |
| **兼容层** | 让原有配置（如 Claude Code）无缝迁移 | 手机换代，数据一键迁移 |
| **LSP** | 语言服务器协议，让 Agent 能跳转定义、查引用 | 给 AI 装上 IDE 的"眼睛" |
| **ultrawork** | 触发并行编排 + 深度探索 + 强制完成的魔法词 | 全员出动模式 |

---

## 🛠️ 实用技巧速查

### 安装配置

```bash
# 1. 安装 OpenCode（前置依赖）
brew install opencode-ai/tap/opencode
# 或
curl -fsSL https://raw.githubusercontent.com/opencode-ai/opencode/refs/heads/main/install | bash

# 2. 安装 oh-my-opencode
bunx oh-my-opencode install
# 或
npx oh-my-opencode install

# 3. 根据订阅情况安装（非交互式）
bunx oh-my-opencode install --no-tui --claude=yes --chatgpt=yes --gemini=yes
```

### 认证配置

```bash
# Claude 认证
opencode auth login  # 选择 Anthropic → Claude Pro/Max

# Gemini 认证（需安装插件）
# 安装 opencode-antigravity-auth 插件

# OpenAI 认证（需安装插件）
# 安装 opencode-openai-codex-auth 插件
```

### 配置文件

```json
// .opencode/oh-my-opencode.json 或 ~/.config/opencode/oh-my-opencode.json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "google_auth": false,
  "agents": {
    "oracle": { "model": "openai/gpt-5.2" },
    "explore": { "model": "opencode/grok-code" },
    "frontend": { "model": "google/gemini-3-pro" }
  },
  "disabled_agents": [],
  "disabled_hooks": [],
  "disabled_mcps": []
}
```

### 代理权限控制

```json
{
  "agents": {
    "explore": {
      "permission": {
        "edit": "deny",
        "bash": "ask",
        "webfetch": "allow"
      }
    }
  }
}
```

### 核心使用方式

| 操作 | 方法 | 效果 |
|------|------|------|
| **ultrawork 模式** | 提示词加 `ultrawork` 关键词 | 并行编排 + 深度探索 + 强制完成 |
| **@代理调用** | `@oracle 审查架构` | 精准调用专业代理 |
| **Ralph Loop** | `/ralph-loop "构建 REST API"` | 自动循环直到任务完成 |
| **并行搜索** | 自动触发 | 多个 Explore 代理同时搜索 |

### 关键词检测

| 关键词 | 触发效果 |
|--------|----------|
| `ultrawork` | 全力模式：并行编排 + 深度探索 |
| `search` | 触发搜索代理 |
| `analyze` | 触发分析代理 |
| `look` | 触发多模态分析 |

---

## 💡 核心洞见

### 1. 多订阅的真正价值是"组合拳"

单独用 Claude/GPT/Gemini 都很强，但组合起来才是 1+1+1>3。每个模型有擅长领域，编排后各展所长。

### 2. $24000 Token 换来的最佳配置

| 代理 | 模型 | 职责 | 原因 |
|------|------|------|------|
| Sisyphus | Claude Opus 4.5 | 统筹全局 | 推理能力最强 |
| Oracle | GPT-5.2 | 架构调试 | 逻辑严密 |
| Frontend | Gemini 3 Pro | 前端开发 | 多模态能力强 |

### 3. 并行执行的威力

```
传统串行：A完成 → B完成 → C完成
总时间 = A + B + C

并行执行：A、B、C 同时进行
总时间 ≈ max(A, B, C)
```

### 4. LSP 让 Agent 真正"看懂"代码

- 从"文本匹配"升级到"语义理解"
- 支持跳转定义、查找引用、代码诊断
- 支持 25 种编程语言

---

## ⚠️ 常见误区

| 误区 | 正确做法 |
|------|----------|
| 只订阅一家 AI 服务就装这个 | 意义不大，这是多订阅用户的福音 |
| 以为装完就自动最优 | 需要根据订阅情况配置参数 |
| 不配置认证就使用 | 每个模型都需要单独认证 |

---

## 📚 代理角色详解

| 代理名称 | 模型 | 职责 | 类比 |
|----------|------|------|------|
| **Sisyphus** | Claude Opus 4.5 | 统筹全局、任务分解 | 项目经理 |
| **Oracle** | GPT-5.2 | 架构设计、代码审查 | 架构师 |
| **Frontend Engineer** | Gemini 3 Pro | UI/UX 开发 | 前端工程师 |
| **Librarian** | Claude Sonnet / Gemini Flash | 文档查找 | 文档管理员 |
| **Explore** | Grok Code / Gemini Flash | 代码库探索 | 侦察兵 |
| **Document Writer** | Gemini Flash | 技术文档 | 技术写作 |
| **Multimodal Looker** | Gemini Flash | 视觉分析 | 视觉分析师 |

---

## 🔧 Claude Code 兼容层

oh-my-opencode 完全兼容 Claude Code 配置：

| 功能 | 兼容性 |
|------|--------|
| Subagent | ✅ 完全兼容 |
| Skills | ✅ 完全兼容 |
| MCP | ✅ 完全兼容 |
| Hooks | ✅ 完全兼容 |
| 自定义命令 | ✅ 完全兼容 |

迁移步骤：安装 oh-my-opencode 后，原有配置自动识别，无需额外操作。

---

## 📖 参考资料

- [oh-my-opencode GitHub](https://github.com/code-yeongyu/oh-my-opencode)
- [OpenCode 主项目](https://github.com/opencode-ai/opencode)（已归档，新项目为 Crush）
- [MCP（模型上下文协议）](https://modelcontextprotocol.io/)
- [LSP 官方文档](https://microsoft.github.io/language-server-protocol/)

---

## 💬 金句摘录

> "这是烧了 24000 刀的 Token 换来的最佳配置。"

> "GPT 在调试的时候，Claude 已经在换思路了，Gemini 写前端的时候，Claude 可以干后端，相互之间不用等。"

> "以前这些东西只有人能用，现在 Agent 也可以跳转定义、查引用，代码 debug 了。"

---

## ✅ 行动清单

- [ ] 确认 AI 订阅情况（Claude/GPT/Gemini）
- [ ] 安装 OpenCode 和 oh-my-opencode
- [ ] 配置多模型认证
- [ ] 体验 ultrawork 模式
- [ ] 尝试 @代理调用

---

*笔记生成时间：2026-01-05*
