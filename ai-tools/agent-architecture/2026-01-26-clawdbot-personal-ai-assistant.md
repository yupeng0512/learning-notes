---
title: Clawdbot - 自托管个人 AI 助手
source: https://github.com/clawdbot/clawdbot
author: Peter Steinberger (@steipete)
date: 2026-01-26
category: ai-tools
subcategory: agent-architecture
tags: [ai-agent, self-hosted, chatbot, automation, typescript, nodejs]
type: practical-project
stars: 35824
---

# Clawdbot 🦞：去中心化的 AI Agent 运行时

> 🔗 项目：[Clawdbot](https://github.com/clawdbot/clawdbot) | [官网](https://clawd.bot) | [文档](https://docs.clawd.bot)
> 📅 学习日期：2026-01-26
> 🏷️ 分类：AI Tools / Agent Architecture
> ⭐ GitHub Stars: 35,824（爆发式增长中）
> 🚀 版本：v2026.x（持续更新）

---

## 项目概览

Clawdbot 是一个**自托管的个人 AI 助手**，运行在你自己的设备上（Mac/Windows/Linux），通过你日常使用的聊天工具（WhatsApp/Telegram/Discord/Slack/iMessage）进行控制。

**核心亮点**：
- 🦞 **有"手"的 AI**：不只会聊天，能执行真实操作（浏览器、文件、Shell）
- 🧠 **自我进化**：遇到没有的功能，自己写代码安装 Skill
- 🏠 **完全自主**：数据留在本地，不依赖云端 SaaS
- 📱 **去中心化交互**：通过 WhatsApp/Telegram 等已有工具控制，无需专门 App

---

## 快速上手实践

### 1. 安装方式

**推荐方式（一键安装）**：
```bash
# 自动安装脚本（macOS/Linux）
curl -fsSL https://clawd.bot/install.sh | bash

# 或使用 npm
npm install -g clawdbot@latest

# 引导式配置（推荐）
clawdbot onboard --install-daemon
```

**其他平台**：
| 平台 | 安装命令 |
|------|----------|
| **macOS/Linux** | `curl -fsSL https://clawd.bot/install.sh \| bash` |
| **Windows (WSL2)** | 在 WSL2 中运行上述命令 |
| **从源码构建** | `git clone && pnpm install && pnpm build` |
| **Docker** | `docker run -d clawdbot/clawdbot` |
| **Nix** | 参考 [Nix 安装文档](https://docs.clawd.bot/install/nix) |

### 2. 基础使用流程

```
用户（手机/电脑）
    │
    ├─ WhatsApp 发送消息："帮我查明天去上海的机票"
    │           ↓
    │   ┌─────────────────────────┐
    │   │   Gateway (ws://本地)    │
    │   │   - 接收消息             │
    │   │   - 路由到 Pi Agent       │
    │   └─────────────────────────┘
    │           ↓
    │   ┌─────────────────────────┐
    │   │   Pi Agent (RPC 模式)    │
    │   │   - 调用工具：browser    │
    │   │   - 访问携程/飞猪        │
    │   │   - 提取航班信息          │
    │   └─────────────────────────┘
    │           ↓
    └─ WhatsApp 接收回复："明天上海航班共 12 班..."
```

### 3. 核心功能速览

| 功能 | 操作方式 | 说明 |
|------|----------|------|
| **浏览器控制** | 自动调用 `browser` 工具 | 访问网页、填表单、截图 |
| **文件操作** | `read_file` / `write_file` | 读写本地文件、整理文件夹 |
| **Shell 执行** | `bash` 工具 | 执行命令（可沙箱隔离） |
| **多渠道接入** | 配置 WhatsApp/Telegram/Discord | 任意聊天工具控制 |
| **自我进化** | AI 自动生成 `SKILL.md` | 缺少功能时自动编写代码 |
| **多 Agent 路由** | 配置 `agents.*.routing` | 不同渠道/账号用不同 Agent |
| **定时任务** | Cron Jobs | 定时提醒、数据同步 |
| **语音交互** | Voice Wake (macOS/iOS) | 语音唤醒、连续对话 |

### 4. 配置文件

| 用途 | 路径 | 格式 |
|------|------|------|
| **主配置** | `~/.clawdbot/clawdbot.json` | JSON |
| **凭证存储** | `~/.clawdbot/credentials/` | 二进制 |
| **工作区** | `~/clawd/` | 目录（可自定义） |
| **Skills** | `~/clawd/skills/{name}/SKILL.md` | Markdown |
| **日志** | `~/.clawdbot/logs/` | 日志文件 |

**最小配置示例**：
```json
{
  "agent": {
    "model": "anthropic/claude-opus-4-5"
  },
  "channels": {
    "whatsapp": {
      "allowFrom": ["+8613800138000"]
    }
  }
}
```

### 5. 常见问题

| 问题 | 解决方案 |
|------|----------|
| **WhatsApp 连接失败** | 重新扫码：`clawdbot channels login` |
| **权限不足（macOS）** | 授予终端「完全磁盘访问」权限 |
| **Gateway 无法启动** | 检查端口占用：`lsof -i :18789` |
| **陌生人能用我的 Bot** | 配置 `dmPolicy="pairing"` + `allowFrom` 白名单 |
| **消息延迟** | 检查 Gateway 运行状态：`clawdbot status` |
| **Docker 沙箱启动失败** | 确保 Docker daemon 运行 |

### 6. 本地开发（二次开发）

**环境要求**：
- Node.js ≥22
- pnpm（推荐）或 npm
- Docker（可选，用于沙箱隔离）

**开发流程**：
```bash
# 克隆仓库
git clone https://github.com/clawdbot/clawdbot.git
cd clawdbot

# 安装依赖
pnpm install

# 构建 UI
pnpm ui:build

# 构建项目
pnpm build

# 开发模式（自动重载）
pnpm gateway:watch

# 运行测试
pnpm test
```

---

## 根节点命题

> **Clawdbot = 去中心化的 AI Agent 运行时**
> 
> 将 AI Agent 从云端 SaaS 解放到本地设备，通过**反向控制**（你控制 AI）+ **工具能力**（AI 有"手"）+ **自我进化**（能写代码扩展自己），构建真正的个人数字助理。

**为什么这是根节点**：
- **反向控制** → 数据隐私 + 完全自主 → 自托管架构
- **工具能力** → 不只聊天，能执行 → Tools/Skills 系统
- **自我进化** → 适应用户需求 → Self-Expanding Skills

所有设计决策（Gateway 架构、多渠道接入、沙箱隔离）都从这个根节点推导而来。

---

## 表示空间

> 描述「个人 AI 助手」这个领域需要哪几个核心维度？

| 维度 | 含义 | Clawdbot 的选择 |
|------|------|-----------------|
| **控制权归属** | 谁拥有 AI 的控制权和数据？ | 用户完全拥有（Self-hosted） |
| **能力边界** | AI 能做什么？只能聊天还是能执行？ | 有"手"（Exec, Browser, File）+ 自我进化 |
| **交互界面** | 如何与 AI 对话？专门 App 还是已有工具？ | 去中心化（利用已有聊天工具） |
| **安全隔离** | 如何防止 AI 失控或滥用？ | Sandbox + Tool Policy + DM Pairing |
| **扩展性** | 如何适应新需求？预装 vs 按需安装？ | 自我进化（AI 自己写 Skills） |

---

## 架构分析

### 技术栈

| 层级 | 技术选型 | 选择理由 |
|------|----------|----------|
| **运行时** | Node.js 22+ | 跨平台、生态丰富 |
| **语言** | TypeScript | 类型安全、开发效率 |
| **AI Agent** | Pi (RPC 模式) | 工具流式调用、性能优化 |
| **WhatsApp** | Baileys | 开源 WhatsApp Web 协议实现 |
| **Telegram** | grammY | 现代化 Bot 框架 |
| **Discord** | discord.js | 官方推荐库 |
| **浏览器控制** | Puppeteer/Playwright | Chrome DevTools Protocol |
| **沙箱** | Docker | 进程隔离、安全防护 |
| **通信** | WebSocket | Gateway 控制平面 |

### 核心设计模式

#### 1. Gateway 模式（中心化控制平面）

```
传统架构（分散式）：
WhatsApp Bot ──> AI Service A
Telegram Bot ──> AI Service B
Discord Bot ──> AI Service C
❌ 问题：重复部署、会话不同步、配置分散

Gateway 架构（中心化）：
WhatsApp ┐
Telegram ├──> Gateway (WS) ──> Pi Agent
Discord  ┘
✅ 优势：单点控制、会话统一、配置集中
```

**实现要点**：
- Gateway 运行在 `ws://127.0.0.1:18789`
- 所有渠道连接到同一个 WebSocket
- 支持远程连接（Tailscale/SSH Tunnel）

#### 2. Tools Registry 模式（工具注册表）

```typescript
// 工具定义
interface Tool {
  name: string;
  description: string;
  parameters: JSONSchema;
  execute: (params: any) => Promise<any>;
}

// 工具注册
registry.register({
  name: "browser",
  description: "Control a Chrome browser",
  execute: async ({ action, url }) => {
    // Puppeteer 实现
  }
});

// AI 调用
const result = await agent.useTool("browser", {
  action: "navigate",
  url: "https://example.com"
});
```

#### 3. Session Isolation 模式（会话隔离）

```
主会话（个人 DM）：
- 运行环境：主机直接执行
- 工作区：~/clawd/
- 权限：完全信任

群组会话：
- 运行环境：Docker 沙箱
- 工作区：/tmp/agent-{session-id}/
- 权限：受限（Tool Policy）

实现：
if (session.type === "main") {
  executor = new HostExecutor();
} else {
  executor = new DockerExecutor({
    image: "clawdbot/sandbox",
    allowTools: ["bash", "read", "write"]
  });
}
```

#### 4. Self-Expanding Skills 模式（自我进化）

```
用户请求：「把视频转成 GIF」
    ↓
AI 分析：发现缺少 video-to-gif Skill
    ↓
AI 生成代码：
    ~/clawd/skills/video-to-gif/SKILL.md
    ├─ description: 视频转 GIF 工具
    ├─ dependencies: ffmpeg
    └─ execute: ffmpeg -i input.mp4 output.gif
    ↓
热加载 Skill → 执行任务
    ↓
下次遇到类似请求 → 直接使用该 Skill
```

---

## 推论展开

> 从根节点推导出的核心设计决策

```
根节点：去中心化的 AI Agent 运行时
│
├─ 推论1：架构设计 → Gateway 单点控制 + 多渠道连接
│   ├─ 实现：WebSocket 控制平面（ws://127.0.0.1:18789）
│   ├─ 优势：避免重复部署、会话统一管理
│   └─ 应用：一台设备运行 Gateway，所有渠道通过它连接
│
├─ 推论2：工具能力 → Tools/Skills 架构
│   ├─ 实现：核心工具（bash, browser, file）+ 可扩展 Skills
│   ├─ 优势：标准化工具调用、灵活扩展
│   └─ 应用：AI 可以调用浏览器、执行命令、读写文件
│
├─ 推论3：自我进化 → Skills 自动生成
│   ├─ 实现：AI 通过 write_file 创建 SKILL.md
│   ├─ 优势：适应千变万化的用户需求
│   └─ 应用：用户请求新功能 → AI 自动编写并安装
│
├─ 推论4：安全隔离 → Sandbox + Tool Policy
│   ├─ 实现：主会话（主机）vs 群组会话（Docker 沙箱）
│   ├─ 优势：防止误操作或恶意攻击
│   └─ 应用：个人使用高权限，群组使用受限权限
│
├─ 推论5：多 Agent 路由 → 工作区隔离
│   ├─ 实现：不同渠道/账号路由到不同 Agent
│   ├─ 优势：不同场景使用不同 AI 人格
│   └─ 应用：个人 WhatsApp（家庭助理）vs 公司 Slack（工作助理）
│
└─ 推论6：去中心化交互 → 利用已有聊天工具
    ├─ 实现：Baileys(WhatsApp) + grammY(Telegram) + discord.js
    ├─ 优势：不强迫用户改变习惯
    └─ 应用：通过 WhatsApp/Telegram 发消息即可控制
```

---

## 泛化模式

> 这些设计可以迁移到哪些其他场景？

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| **Gateway 模式** | Zapier/n8n 集成 | 统一 Webhook 入口 → 多服务分发 |
| **Gateway 模式** | API Gateway | 多微服务 → 统一网关 → 路由/鉴权 |
| **Self-Hackable** | VS Code Extension API | 开发者可编写插件扩展功能 |
| **Self-Hackable** | Obsidian Plugin | 用户自定义工作流 |
| **Agent Workspace** | Cursor 项目根目录 | 注入 `.cursor/rules` 影响 AI 行为 |
| **Session Isolation** | Jupyter Notebook Kernel | 不同 Notebook → 隔离运行环境 |
| **Tool Registry** | LangChain Tools | 标准化工具接口 + 动态加载 |

---

## 核心优势（与竞品对比）

### vs ChatGPT/Claude (SaaS)

| 维度 | ChatGPT/Claude | Clawdbot |
|------|---------------|----------|
| **数据隐私** | 上传到云端 | 留在本地 |
| **自定义能力** | 受限于平台 | 完全可扩展 |
| **工具能力** | 有限（Browse, Code Interpreter） | 无限（bash, browser, 自定义 Skills） |
| **成本** | 订阅费 + 用量计费 | 一次性配置 + 自己的 API Key |
| **可靠性** | 依赖服务商 | 自己掌控（24/7 在线） |

### vs Zapier/n8n (自动化)

| 维度 | Zapier/n8n | Clawdbot |
|------|-----------|----------|
| **交互方式** | 预设 Workflow | 自然语言对话 |
| **灵活性** | 需提前配置 | 临时需求即时响应 |
| **学习成本** | 需学习节点编排 | 像和人聊天一样 |
| **智能程度** | 规则引擎 | LLM 驱动（理解意图） |

### vs HomeAssistant (智能家居)

| 维度 | HomeAssistant | Clawdbot |
|------|--------------|----------|
| **专注领域** | 智能家居 | 通用个人助理 |
| **设备支持** | 家居设备集成 | 计算机任务（文件、浏览器、Shell） |
| **交互方式** | Dashboard + 语音 | 多聊天渠道（WhatsApp/Telegram） |

---

## 安全模型（重要）

### 分层防护

```
┌─────────────────────────────────────────┐
│ Level 4: DM Pairing（陌生人防护）        │
│ - 默认策略：dmPolicy="pairing"          │
│ - 陌生人发消息 → 返回配对码 → 手动批准   │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│ Level 3: Channel Allowlist（渠道白名单） │
│ - channels.whatsapp.allowFrom           │
│ - channels.telegram.allowFrom           │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│ Level 2: Sandbox Mode（沙箱隔离）        │
│ - 主会话：直接在主机执行                 │
│ - 群组会话：Docker 沙箱执行              │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│ Level 1: Tool Policy（工具白名单）       │
│ - Allowlist: bash, read, write          │
│ - Denylist: browser, gateway, cron      │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│ Level 0: Elevated Mode（提权控制）       │
│ - 默认禁用                               │
│ - 需手动 /elevated on 才能执行敏感命令   │
└─────────────────────────────────────────┘
```

### 配置示例

```json
{
  "gateway": {
    "bind": "loopback",  // 只监听本地
    "auth": {
      "mode": "password",  // 远程访问需密码
      "password": "your-secure-password"
    }
  },
  "channels": {
    "whatsapp": {
      "allowFrom": ["+8613800138000"],  // 白名单
      "groups": {
        "*": { "requireMention": true }  // 群组需 @mention
      }
    },
    "telegram": {
      "dmPolicy": "pairing"  // DM 需配对
    }
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // 非主会话使用沙箱
        "image": "clawdbot/sandbox"
      },
      "tools": {
        "allowlist": ["bash", "read", "write"],
        "denylist": ["browser", "gateway"]
      }
    }
  }
}
```

---

## 行动清单

### 即时行动（上手实践）

- [ ] **P0**: 在闲置电脑/服务器上安装 Clawdbot
  - 执行：`curl -fsSL https://clawd.bot/install.sh | bash`
  - 验证：`clawdbot gateway` 成功运行
  
- [ ] **P0**: 配置一个聊天渠道（推荐 Telegram）
  - 创建 Telegram Bot：`@BotFather`
  - 配置 `channels.telegram.botToken`
  - 验证：发消息收到回复
  
- [ ] **P1**: 尝试让 AI 自动生成一个 Skill
  - 请求：「帮我写一个检查天气的 Skill」
  - 验证：`~/clawd/skills/check-weather/SKILL.md` 生成
  
- [ ] **P1**: 配置安全策略（白名单 + DM Pairing）
  - 设置 `allowFrom` 白名单
  - 启用 `dmPolicy="pairing"`
  - 验证：陌生人无法直接使用

### 深度学习（架构理解）

- [ ] **P1**: 研究 Gateway 配置（多 Agent 路由）
  - 阅读：[Configuration 文档](https://docs.clawd.bot/gateway/configuration)
  - 理解：`agents.*.routing` 配置规则
  
- [ ] **P2**: 阅读 Pi Agent 源码
  - 仓库：https://github.com/badlogic/pi-mono
  - 理解：RPC 模式、工具流式调用
  
- [ ] **P2**: 部署到云服务器（24/7 在线）
  - 平台：Hetzner / Fly.io / Railway
  - 配置：Tailscale Serve 远程访问
  
- [ ] **P2**: 集成你的工作流（Gmail/Calendar/Notion）
  - 编写自定义 Skills
  - 自动化日常任务

### 进阶探索

- [ ] **P3**: 研究浏览器控制机制
  - 阅读 `tools/browser.ts`
  - 理解：Puppeteer CDP 控制
  
- [ ] **P3**: 开发自定义渠道（企业微信/钉钉）
  - 参考 `channels/` 目录实现
  - 贡献回社区
  
- [ ] **P3**: 参与社区贡献
  - Discord：https://discord.gg/clawd
  - GitHub Issues/PR

---

## 社区评价（精选）

> 从 35,000+ Star 背后看用户真实反馈

### 技术视角

> "A megacorp like Anthropic or OpenAI could not build this. Literally impossible with how corpo works."  
> — @Dimillian

> "The gap between 'what I can imagine' and 'what actually works' has never been smaller."  
> — @tobi_bsf

### 产品视角

> "A smart model with eyes and hands at a desk with keyboard and mouse. You message it like a coworker and it does everything a person could do with that Mac mini."  
> — @nathanclark_

> "Everything Siri was supposed to be. And it goes so much further."  
> — @crossiBuilds

### 使用体验

> "It's running my company."  
> — @therno

> "From nervous 'hi what can you do?' to full throttle - design, code review, taxes, PM, content pipelines... AI as teammate, not tool."  
> — @lycfyi

> "I'm literally building a whole website on a Nokia 3310 by calling @clawdbot right now."  
> — @youbiak（😂 夸张但生动）

---

## 延伸阅读

### 官方资源
- [官方文档](https://docs.clawd.bot) - 完整使用指南
- [GitHub 仓库](https://github.com/clawdbot/clawdbot) - 源码 + Issue
- [更新日志](https://github.com/clawdbot/clawdbot/blob/main/CHANGELOG.md)
- [Discord 社区](https://discord.gg/clawd) - 实时讨论

### 技术栈相关
- [Pi Agent](https://github.com/badlogic/pi-mono) - 底层 AI Agent 引擎
- [Baileys](https://github.com/WhiskeySockets/Baileys) - WhatsApp Web 协议
- [grammY](https://grammy.dev/) - Telegram Bot 框架
- [Puppeteer](https://pptr.dev/) - 浏览器自动化

### 类似项目对比
- [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) - 自主 AI Agent（云端）
- [LangChain](https://github.com/langchain-ai/langchain) - AI 应用开发框架
- [n8n](https://github.com/n8n-io/n8n) - 自动化工作流（无 AI）
- [Home Assistant](https://github.com/home-assistant/core) - 智能家居（专注硬件）

---

## 个人思考

### 为什么 Clawdbot 如此火爆？

1. **时机成熟**：
   - LLM 能力足够强（Claude Opus 4.5、GPT-4）
   - 工具调用成熟（Function Calling）
   - 开发者对 SaaS 的「数据焦虑」积累已久

2. **产品定位精准**：
   - 不是「AI 聊天工具」，而是「有手的 AI 助理」
   - 不是「企业级 SaaS」，而是「个人级 Self-hosted」
   - 不是「预设工作流」，而是「自然语言交互」

3. **极客友好**：
   - 完全开源（MIT 协议）
   - 可深度定制（Self-Hackable）
   - 社区驱动（快速迭代）

### 对比 Cursor/Claude Code

| 维度 | Cursor/Claude Code | Clawdbot |
|------|-------------------|----------|
| **定位** | 代码编辑器中的 AI | 通用个人助理 |
| **主要场景** | 写代码 | 任何需要 AI 的场景 |
| **交互方式** | IDE 内对话 | 任意聊天工具 |
| **工具能力** | 受限于 IDE API | 完全控制操作系统 |
| **互补关系** | ✅ 可以同时使用 | Clawdbot 可以启动 Cursor |

**结论**：Cursor 是「写代码的 AI」，Clawdbot 是「生活中的 AI」。两者互补。

### 适合人群

✅ **强烈推荐**：
- 极客/开发者
- 注重隐私的用户
- 需要自动化日常任务
- 有闲置电脑/服务器

⚠️ **需谨慎**：
- 非技术用户（需一定命令行基础）
- 没有稳定设备（手机不适合）
- 只想聊天不需要执行能力（ChatGPT 更合适）

---

## 质量自检

| 检查项 | 通过标准 | 状态 |
|--------|----------|------|
| **可上手** | 用户能否按照笔记直接安装和使用？ | ✅ 提供详细安装步骤 |
| **流程清晰** | 核心使用流程是否用流程图/步骤展示？ | ✅ ASCII 流程图 + 表格 |
| **常见问题覆盖** | 首次使用可能遇到的坑是否列出？ | ✅ 6 个常见问题 + 解决方案 |
| **根节点明确** | 项目的核心设计理念是否提炼清楚？ | ✅ 去中心化 AI Agent 运行时 |
| **可迁移** | 设计模式能否迁移到其他项目？ | ✅ 6 个泛化模式示例 |
| **安全意识** | 是否强调安全配置？ | ✅ 专门章节 + 配置示例 |

---

## 更新日志

- 2026-01-26：初版学习笔记（基于 v2026.1.26 版本）
- 待更新：后续版本的重要特性变更
