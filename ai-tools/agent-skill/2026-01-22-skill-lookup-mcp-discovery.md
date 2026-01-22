# Skill Lookup：prompts.chat 平台的 Skill 发现与安装技能

> 📖 原文：[skill-lookup (GitHub)](https://github.com/f/awesome-chatgpt-prompts/blob/main/plugins/claude/prompts.chat/skills/skill-lookup/SKILL.md)
> 📅 学习日期：2026-01-22
> 🏷️ 标签：#Skill发现 #MCP工具 #元技能 #prompts.chat

---

## 根节点命题

> **Skill 发现的最佳实践 = 意图识别 × 工具调用 × 文件落地**

**为什么这是根节点**：skill-lookup 的设计完美体现了 Claude Skill 的三层架构：
1. **意图识别**：frontmatter 中的 `description` 定义激活条件
2. **工具调用**：通过 MCP 工具（`search_skills`、`get_skill`）实现核心能力
3. **文件落地**：将获取的 Skill 安装到 `.claude/skills/` 目录

三者缺一不可，共同构成"发现 → 获取 → 安装"的完整闭环。

---

## 表示空间

| 维度 | 含义 | skill-lookup 的位置 |
|------|------|---------------------|
| **实现方式** | 如何实现 Skill 分发 | MCP 工具 + Claude 原生能力（vs CLI 命令行） |
| **平台依赖** | 依赖哪个 Skill 仓库 | prompts.chat 平台（vs GitHub Registry） |
| **运行主体** | 谁执行安装动作 | Claude Agent 自动执行（vs 用户手动运行命令） |

---

## 核心机制解析

### 1️⃣ 激活条件设计（When to Use）

```yaml
# SKILL.md frontmatter
description: >
  Activates when the user asks about Agent Skills,
  wants to find reusable AI capabilities,
  needs to install skills,
  or mentions skills for Claude.
```

**设计要点**：
- 使用自然语言描述触发条件
- 覆盖多种用户意图表达方式
- Claude 自动匹配激活

**激活场景示例**：

| 用户说 | 是否激活 | 原因 |
|--------|----------|------|
| "Find me a code review skill" | ✅ | 关键词 "skill" + "find" |
| "What skills are available for testing?" | ✅ | 关键词 "skills" + "available" |
| "Install the documentation skill" | ✅ | 关键词 "install" + "skill" |
| "Help me write better code" | ❌ | 无明确 Skill 相关词汇 |

### 2️⃣ MCP 工具使用

skill-lookup 依赖 `prompts.chat` MCP 服务器提供的两个工具：

```typescript
// 搜索 Skill
search_skills({
  query: "code review",    // 搜索关键词
  limit: 10,               // 返回数量（max 50）
  category: "coding",      // 可选：分类过滤
  tag: "automation"        // 可选：标签过滤
})

// 获取 Skill 详情
get_skill({
  id: "skill-id"           // Skill 唯一标识
})
```

**返回内容**：
- `search_skills`：Skill 列表（标题、描述、作者、文件列表、分类标签）
- `get_skill`：完整 Skill 内容（所有文件的内容）

### 3️⃣ 安装流程

```
用户请求安装 Skill
│
├─ ① 调用 get_skill 获取完整文件
│
├─ ② 创建目标目录
│   └─ .claude/skills/{slug}/
│
└─ ③ 保存文件
    ├─ SKILL.md → .claude/skills/{slug}/SKILL.md
    ├─ reference.md → .claude/skills/{slug}/reference.md
    └─ helper.py → .claude/skills/{slug}/helper.py
```

**关键设计**：
- 使用 `slug`（URL 友好的标识符）作为目录名
- 完整保留 Skill 的所有附属文件
- 安装后需确认成功

---

## 推论展开

```
根节点：Skill 发现 = 意图识别 × 工具调用 × 文件落地
│
├─ 推论1：意图识别需要"宽松匹配 + 明确边界"
│   ├─ 实现：frontmatter description 用多种表达覆盖
│   └─ 应用：避免过度激活，但不漏掉真实需求
│
├─ 推论2：工具调用需要"查询抽象 + 结果标准化"
│   ├─ 实现：search_skills 支持 query/category/tag 组合
│   └─ 应用：用户模糊查询也能返回相关结果
│
└─ 推论3：文件落地需要"结构保持 + 位置约定"
    ├─ 实现：保留 Skill 完整目录结构
    └─ 应用：Claude 能自动识别已安装的 Skill
```

---

## 与 skills-installer 对比

| 对比维度 | skill-lookup | skills-installer |
|----------|--------------|------------------|
| **实现形式** | Claude Skill（提示词） | CLI 工具（npm 包） |
| **运行方式** | Claude 自动激活执行 | 用户手动运行命令 |
| **后端平台** | prompts.chat MCP | Val Town Registry API |
| **Skill 来源** | prompts.chat 平台 | GitHub 任意仓库 |
| **安装位置** | `.claude/skills/` only | 15 种客户端，支持全局/本地 |
| **使用门槛** | 自然对话即可 | 需要了解命令行 |
| **灵活性** | 低（固定平台） | 高（任意 GitHub） |

**选择建议**：
- **skill-lookup**：适合 Claude 用户，想要最简单的"说一句话就安装"体验
- **skills-installer**：适合开发者，需要跨客户端、跨仓库的灵活安装

---

## 泛化模式

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| **意图识别 → 工具调用** | 任何 Claude Skill | frontmatter 定义触发条件 + MCP 工具实现能力 |
| **搜索 + 获取两阶段** | API 设计模式 | 先 list/search 返回摘要，再 get 返回详情 |
| **slug 命名规范** | 文件系统设计 | URL 友好 + 唯一性 + 可读性 |

---

## Skill 结构剖析

```markdown
# skill-lookup/SKILL.md

---
name: skill-lookup
description: Activates when...
---

## When to Use This Skill
[激活条件说明]

## Available Tools
[MCP 工具列表]

## How to Search for Skills
[搜索操作指南]

## How to Get a Skill
[获取操作指南]

## How to Install a Skill
[安装操作指南]

## Skill Structure
[Skill 文件结构说明]

## Guidelines
[使用原则]
```

**结构特点**：
1. **frontmatter**：元数据（name、description）
2. **When to Use**：激活场景
3. **工具说明**：每个 MCP 工具的使用方式
4. **操作指南**：分步骤说明
5. **原则约束**：避免错误使用

---

## 关联历史笔记

| 历史笔记 | 关系类型 | 关联说明 |
|----------|----------|----------|
| [Skills Installer CLI](./2026-01-20-skills-installer-cli.md) | **对比** | 两种 Skill 分发实现：MCP 方式 vs CLI 方式 |
| [Agent Skills for Context Engineering](./2026-01-09-agent-skills-context-engineering.md) | **互补** | Context Engineering 定义 Skill 规范，skill-lookup 实现 Skill 发现 |
| [Skill Building Guide](./2026-01-22-skill-building-guide-share.md) | **应用** | 理解 Skill 结构后，可参考此指南构建自己的 Skill |

**知识网络**：

```
Skill 分发生态
│
├─ L1 规范层：Context Engineering（定义 Skill 是什么）
│
├─ L2 分发层（本笔记聚焦）
│   ├─ skill-lookup：MCP 方式，面向 Claude 用户
│   └─ skills-installer：CLI 方式，面向开发者
│
└─ L3 构建层：Skill Building Guide（教你如何写 Skill）
```

---

## 关键概念

| 概念 | 定义 | 与根节点的关系 |
|------|------|----------------|
| **MCP (Model Context Protocol)** | 让 Claude 调用外部工具的协议 | 实现"工具调用"维度 |
| **frontmatter** | Skill 文件开头的 YAML 元数据 | 实现"意图识别"维度 |
| **slug** | URL 友好的唯一标识符 | 实现"文件落地"的命名规范 |
| **prompts.chat** | Skill 托管与发现平台 | 提供 MCP 服务器和 Skill 仓库 |

---

## 金句摘录

> "Always search before suggesting the user create their own skill"
> 
> — skill-lookup/SKILL.md

**解读**：遵循 DRY 原则，避免重复造轮子。在创建新 Skill 前，先确认是否已有现成方案。

---

## 行动清单

### 即时行动

- [ ] 配置 prompts.chat MCP 服务器（如果尚未配置）
- [ ] 尝试搜索一个 Skill：对 Claude 说 "Find me skills for code review"
- [ ] 安装一个 Skill：对 Claude 说 "Install the {skill-name} skill"

### 深度学习

- [ ] 阅读 prompts.chat MCP 服务器实现，理解 `search_skills` 和 `get_skill` API
- [ ] 对比 skill-lookup 与 skills-discovery（skills-installer 的元技能）的设计差异
- [ ] 尝试向 prompts.chat 平台提交自己的 Skill

### 延伸阅读

- [prompts.chat](https://prompts.chat) - Skill 托管平台
- [awesome-chatgpt-prompts](https://github.com/f/awesome-chatgpt-prompts) - 原仓库
- [MCP 协议文档](https://modelcontextprotocol.io) - 理解 MCP 工作原理

---

## 个人思考

{留空，供后续补充}
