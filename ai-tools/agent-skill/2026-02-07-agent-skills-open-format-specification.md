---
title: Agent Skills：AI Agent 的开放能力扩展格式
source: https://agentskills.io/home
author: Anthropic（开放标准）
date: 2026-02-07
category: ai-tools
subcategory: agent-skill
tags: [Agent Skills, Skill Format, 开放标准, 能力扩展, 互操作性]
---

# Agent Skills：AI Agent 的开放能力扩展格式

> 📖 来源：[Agent Skills Official Site](https://agentskills.io/home) | [GitHub Repo](https://github.com/agentskills/agentskills)  
> 📅 学习日期：2026-02-07  
> 🏷️ 分类：AI 工具 / Agent 技能  
> ⭐ GitHub：9.2k stars | 490 forks

---

## 根节点命题

> **Agent Skills = 渐进式披露（Progressive Disclosure）+ 可移植格式（Portable Format）+ 开放生态（Open Ecosystem）**
>
> 通过「只在需要时加载上下文」的设计，解决了 AI Agent 能力扩展的三大核心矛盾：**性能 vs 能力**、**通用 vs 专业**、**封闭 vs 开放**。

**为什么这是根节点**：

1. **渐进式披露**：决定了架构设计（三层加载机制：元数据 → 指令 → 资源）
2. **可移植格式**：决定了采用率（30+ 工具支持，包括 Cursor/Claude Code/GitHub Copilot）
3. **开放生态**：决定了长期价值（写一次，到处运行；社区贡献；企业沉淀）

从这三个要素可推导出：
- 所有技术设计（SKILL.md 格式、目录结构、frontmatter 规范）
- 所有使用场景（领域专业知识包、新能力扩展、可重复工作流）
- 所有生态价值（互操作性、版本控制、审计能力）

---

## 表示空间

> 理解 Agent Skills 需要在 3 个核心维度上定位：

| 维度 | 含义 | Agent Skills 的位置 |
|------|------|-------------------|
| **上下文管理策略** | 如何处理 AI 上下文限制 | 渐进式披露（按需加载），而非全量加载或完全外部化 |
| **能力扩展方式** | 如何给 Agent 增加新能力 | 文件系统格式（SKILL.md），而非 API/Plugin/Hard-coded |
| **生态开放度** | 供应方与需求方的耦合度 | 开放标准（30+ 工具支持），而非单一平台专属 |

**坐标系可视化**：

```
上下文管理：全量加载 ←──────────→ 渐进式披露
                           ↑
                      Agent Skills

能力扩展：Hard-coded ←──────────→ 文件格式
                           ↑
                      Agent Skills

生态开放度：封闭 ←──────────────→ 开放标准
                           ↑
                      Agent Skills
```

---

## 推论展开

> 从根节点（渐进式披露 + 可移植格式 + 开放生态）推导出的核心结论

```
根节点：Agent Skills = 渐进式披露 + 可移植格式 + 开放生态

├─ 推论 1：渐进式披露解决性能瓶颈
│   ├─ 第1层（启动时）：只加载 name + description（~100 tokens）
│   ├─ 第2层（激活时）：加载完整 SKILL.md（<5000 tokens）
│   ├─ 第3层（执行时）：按需加载 scripts/references/assets
│   └─ 效果：Agent 可访问数百个 Skill，但只为当前任务付出上下文成本
│   应用：设计 Skill 时，把详细内容拆到 references/ 而非塞进 SKILL.md
│
├─ 推论 2：可移植格式带来网络效应
│   ├─ 格式简单：YAML frontmatter + Markdown（任何编辑器可编辑）
│   ├─ 自我描述：人类可阅读，便于审计和改进
│   ├─ 版本控制：Git 友好，支持协作和回滚
│   └─ 效果：写一次 → 30+ 工具支持 → 网络效应增长
│   应用：企业内部 Skill 可跨 Cursor/Claude Code 复用
│
├─ 推论 3：开放生态降低供需双方成本
│   ├─ Skill 作者：无需为每个平台重写
│   ├─ Agent 产品：用户自带 Skill 生态
│   ├─ 企业团队：知识沉淀为可审计的资产
│   └─ 效果：正向循环（更多 Skill → 更多工具支持 → 更多 Skill）
│   应用：优先采用已支持 Agent Skills 的工具
│
├─ 推论 4：标准化格式提高质量下限
│   ├─ 强制元数据：name（命名规范）+ description（激活条件）
│   ├─ 验证工具：skills-ref validate 检查合规性
│   ├─ 最佳实践：官方文档 + 示例库
│   └─ 效果：新手也能写出基本可用的 Skill
│   应用：用 skills-ref 工具验证 Skill 质量
│
└─ 推论 5：互操作性是长期护城河
    ├─ 对比：单平台 Plugin（被锁定）
    ├─ 优势：跨平台 Skill（可迁移）
    ├─ 证据：Anthropic 主导但开放标准，GitHub/Cursor 等跟进
    └─ 效果：形成事实标准，先发优势难以撼动
    应用：投资时间学习 Agent Skills，而非单一平台 Plugin API
```

---

## 泛化模式

> Agent Skills 的设计原则可迁移到哪些其他场景？

| 原场景 | 迁移场景 | 如何应用根节点 |
|--------|----------|----------------|
| **Agent Skills** | **VS Code Extensions** | 渐进式披露（按需激活扩展）+ 可移植（VSIX 格式）+ 开放（Marketplace）|
| **渐进式披露** | **前端代码分割** | 首屏只加载核心 → 路由时加载页面 → 交互时加载组件 |
| **SKILL.md 格式** | **README.md 规范** | 元信息（frontmatter）+ 使用说明（Markdown）+ 自我描述 |
| **开放生态** | **容器镜像（Docker）** | 标准格式 → 多平台支持 → 社区贡献 |
| **三层加载** | **游戏资源管理** | 启动加载列表 → 进入关卡加载场景 → 触发事件加载特效 |
| **skills-ref 验证** | **OpenAPI Spec** | 标准化格式 + 自动验证 + 生成文档 |

---

## 关键概念

> 理解根节点必需的概念

| 概念 | 定义 | 与根节点的关系 |
|------|------|----------------|
| **Progressive Disclosure（渐进式披露）** | 信息按需加载，避免一次性暴露全部内容 | 根节点的性能保证机制 |
| **SKILL.md** | 包含 YAML frontmatter + Markdown 指令的核心文件 | 可移植格式的具体实现 |
| **Frontmatter** | YAML 格式的元数据（name, description, license 等） | 第1层加载的内容，用于发现和激活 |
| **Discovery → Activation → Execution** | 三阶段加载流程 | 渐进式披露的工作流程 |
| **Self-documenting** | 文件本身就是文档，人类可读 | 可移植格式的审计能力 |
| **Portable Format** | 格式独立于具体实现，可跨平台使用 | 开放生态的基础 |
| **Network Effect（网络效应）** | 使用者越多，价值越大 | 开放生态的增长飞轮 |
| **skills-ref** | 官方参考库，提供验证和 XML 生成 | 标准化的质量保证工具 |
| **Interoperability（互操作性）** | 同一 Skill 可在多个 Agent 产品中使用 | 开放生态的核心价值 |

---

## 反直觉洞见

### 💡 洞见 1：简单 ≠ 弱，复杂 ≠ 强

**直觉**：格式越复杂，功能越强大  
**真相**：Agent Skills 只用 YAML + Markdown，却比复杂的 Plugin API 更强大

**原因**：
- 简单格式 → 易于采用 → 更多工具支持 → 网络效应
- 复杂 API → 学习成本 → 少数工具支持 → 生态碎片化

**证据**：
- Agent Skills：YAML + Markdown → 30+ 工具支持
- 各平台 Plugin：各自 API → 互不兼容

---

### 💡 洞见 2："按需加载"不是优化，而是架构决策

**直觉**：渐进式披露是性能优化手段  
**真相**：渐进式披露是能力扩展的根本前提

**推理**：
```
假设没有渐进式披露：
- Agent 启动 = 加载所有 Skill 的完整内容
- 100 个 Skill × 5000 tokens = 500,000 tokens
- 超出上下文窗口，无法启动

有渐进式披露：
- 启动 = 100 个 Skill × 100 tokens = 10,000 tokens
- 激活 1 个 = +5000 tokens = 15,000 tokens（可控）
```

**启示**：不是"能优化就优化"，而是"不这样设计就活不了"

---

### 💡 洞见 3：开放标准的"先行者劣势"

**直觉**：Anthropic 开放标准会失去竞争优势  
**真相**：开放标准是 Anthropic 的护城河

**逻辑**：
1. Anthropic 主导规范定义 → 深度理解标准
2. 其他工具跟进实现 → 扩大 Agent Skills 生态
3. 生态壮大 → Claude Code 作为"标准参考实现"受益最大
4. 标准演进 → Anthropic 仍是主导方

**类比**：
- Google 开放 Kubernetes → K8s 成为标准 → GKE 受益
- Meta 开放 React → React 成为标准 → Meta 技术影响力受益

---

### 💡 洞见 4：文件夹 > API 的深层原因

**直觉**：API 更现代，文件夹太传统  
**真相**：文件夹是最优的知识载体

**对比**：

| 维度 | API Plugin | 文件夹 Skill |
|------|------------|-------------|
| **可读性** | 代码（需要理解 API） | Markdown（人类可读）|
| **版本控制** | 困难（二进制/复杂状态） | 自然（Git 友好）|
| **审计性** | 黑盒（不知道内部逻辑） | 白盒（所见即所得）|
| **协作性** | PR 复杂（代码审查） | PR 简单（文档审查）|
| **迁移性** | 绑定平台 | 跨平台复制粘贴 |

**本质**：文件夹是「可读的代码」和「可执行的文档」的完美结合

---

### 💡 洞见 5：description 字段是"激活函数"

**直觉**：description 只是说明文字  
**真相**：description 决定 Skill 能否被正确激活

**机制**：
```python
# Agent 决策流程
for skill in skills:
    if skill.description 匹配 user_query:
        load_full_skill(skill)
        break
```

**启示**：
- ✅ Good description：包含关键词 + 使用场景 + 触发条件
  - `"Extract text from PDFs. Use when user mentions PDFs, forms, or document extraction."`
- ❌ Bad description：过于简短或模糊
  - `"Helps with PDFs."`

**类比**：description 就像函数的 docstring，但它是给 AI 看的"API 文档"

---

## 行动清单

基于根节点推导的 10 个可执行行动：

### 🚀 立即可行（Top 3）

- [ ] **创建第一个 Skill**：选择一个你经常用的工作流（如"生成 PR 描述"），48 小时内写成 SKILL.md 格式（<100 行）
- [ ] **验证工具安装**：克隆 `github.com/agentskills/agentskills`，运行 `skills-ref validate` 验证你的 Skill
- [ ] **测试跨平台**：在 2 个支持 Agent Skills 的工具（如 Cursor + Claude Code）中测试同一 Skill

### 📚 学习深化（4-7）

- [ ] **研读示例库**：浏览 `github.com/anthropics/skills`，找到 3 个与你工作相关的 Skill，分析其结构
- [ ] **优化 description**：检查你的历史 Skill，确保每个 description 包含「做什么 + 何时用 + 关键词」
- [ ] **渐进式拆分**：将一个 >500 行的 SKILL.md 拆分为：核心指令（SKILL.md）+ 详细参考（references/）
- [ ] **泛化练习**：将企业内部文档（操作手册、审批流程）转换为 Agent Skills 格式

### 🏗️ 系统建设（8-10）

- [ ] **建立企业 Skill 库**：在 Git 仓库中创建 `/skills` 目录，制定团队 Skill 编写规范
- [ ] **集成监控**：记录 Skill 激活次数、执行成功率、平均耗时，优化高频 Skill
- [ ] **贡献开源**：将通用性强的 Skill 提交到社区（github.com/agentskills），建立个人品牌

---

## 知识网络关联

> 与历史笔记的关系

| 笔记 | 关系类型 | 关联说明 |
|------|---------|---------|
| `2026-02-05-agent-swarm-research.md` | **互补** | Agent Swarm 解决"多 Agent 协作"，Agent Skills 解决"单 Agent 能力扩展" |
| `2026-02-05-openclaw-6-practical-tips.md` | **应用** | OpenClaw 是支持 Agent Skills 的工具之一，可实践本笔记 |
| `2026-01-25-anthropic-building-agents-with-skills.md` | **深化** | Anthropic 官方博客，详细讲解 Skill 设计最佳实践 |
| `2026-01-25-remotion-skill-ai-video-generation.md` | **案例** | Remotion Skill 是 Agent Skills 格式的实际应用 |
| `2026-01-22-skill-seekers-automated-skill-generator.md` | **工具** | 自动生成 Agent Skills 的工具 |
| `2026-01-20-skills-installer-cli.md` | **工具** | Agent Skills 的包管理器 |
| `2026-01-16-claude-scientific-skills.md` | **案例** | 科研类 Agent Skills 的标杆实现 |

**知识网络图**：

```
        Agent Skills（本笔记）
              ↓
    ┌─────────┼─────────┐
    ↓         ↓         ↓
  理论基础  工具生态  实战案例
    ↓         ↓         ↓
Anthropic  OpenClaw  Remotion
 博客               Skill
```

---

## 金句摘录

1. > "A simple, open format for giving agents new capabilities and expertise."  
   — Agent Skills 官网（定义）

2. > "Write once, use everywhere."  
   — Agent Skills 核心价值（可移植性）

3. > "Skills use progressive disclosure to manage context efficiently."  
   — 核心机制说明

4. > "At startup, agents load only the name and description of each available skill."  
   — 第1层加载策略

5. > "This approach keeps agents fast while giving them access to more context on demand."  
   — 性能 vs 能力的平衡

6. > "Skills are just files, so they're easy to edit, version, and share."  
   — 可移植格式的优势

7. > "Self-documenting: A skill author or user can read a SKILL.md and understand what it does."  
   — 审计能力

8. > "Portable: Skills can range in complexity from just text instructions to executable code."  
   — 可扩展性

9. > "Build capabilities once and deploy them across multiple agent products."  
   — Skill 作者的价值

10. > "Support for skills lets end users give agents new capabilities out of the box."  
    — Agent 产品的价值

---

## 延伸阅读

### 📚 官方文档

- [Agent Skills Official Site](https://agentskills.io/) - 完整文档和规范
- [Agent Skills Specification](https://agentskills.io/specification) - 格式规范详解
- [Integrate Skills Guide](https://agentskills.io/integrate-skills) - 为 Agent 添加 Skills 支持
- [GitHub Repo](https://github.com/agentskills/agentskills) - 9.2k stars 开源仓库

### 🛠️ 工具与资源

- [Example Skills](https://github.com/anthropics/skills) - Anthropic 官方示例库
- [skills-ref Library](https://github.com/agentskills/agentskills/tree/main/skills-ref) - 验证工具和 XML 生成器
- [Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) - Anthropic 最佳实践指南

### 🎓 相关笔记

- [Building Agents with Skills](./2026-01-25-anthropic-building-agents-with-skills.md) - Anthropic 技术博客
- [Skill From Masters](./2026-01-25-skill-from-masters-trinity-skill-generator.md) - 自动化 Skill 生成
- [OpenClaw 实战技巧](./2026-02-05-openclaw-6-practical-tips.md) - 支持 Agent Skills 的工具

### 🌐 生态工具

**支持 Agent Skills 的 30+ 工具**：
- AI IDE: Cursor, Claude Code, VS Code, OpenCode
- AI CLI: Gemini CLI, Autohand Code, Goose
- 平台: GitHub Copilot, Databricks, Spring AI
- 企业: Letta, Factory, Piebald

---

## 个人思考

_(供后续补充)_

---

## 附录：技术规范详解

### A1. SKILL.md 完整格式

#### 最小示例

```markdown
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill

Use this skill when the user needs to work with PDF files...

## How to extract text

1. Use pdfplumber for text extraction...

## How to fill forms

...
```

---

#### 完整示例（所有字段）

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
license: Apache-2.0
compatibility: Requires pdfplumber, PyPDF2. Designed for Claude Code.
metadata:
  author: example-org
  version: "1.0"
  created_at: "2026-01-15"
allowed-tools: Bash(python:*) Read Write
---

# PDF Processing Skill

{Markdown 指令内容...}
```

---

### A2. Frontmatter 字段规范

#### name 字段（必需）

**约束**：
- 长度：1-64 字符
- 字符：只能包含小写字母、数字、连字符（`a-z`, `0-9`, `-`）
- 规则：不能以连字符开头或结尾，不能有连续连字符

**示例**：

```yaml
✅ Good:
  name: pdf-processing
  name: data-analysis
  name: code-review

❌ Bad:
  name: PDF-Processing  # 大写不允许
  name: -pdf            # 不能以连字符开头
  name: pdf--processing # 连续连字符
```

**重要**：name 必须与父目录名一致

```
pdf-processing/
└── SKILL.md  # name: pdf-processing ✅
```

---

#### description 字段（必需）

**约束**：
- 长度：1-1024 字符
- 内容：描述「做什么」+「何时用」+「关键词」

**Good Example**：

```yaml
description: >
  Extracts text and tables from PDF files, fills PDF forms, 
  and merges multiple PDFs. Use when working with PDF documents 
  or when the user mentions PDFs, forms, or document extraction.
```

**Poor Example**：

```yaml
description: Helps with PDFs.  # 太简短，缺少关键词
```

**设计技巧**：
- 包含触发关键词（PDFs, forms, document extraction）
- 描述具体能力（extract, fill, merge）
- 说明使用场景（when working with...）

---

#### license 字段（可选）

**推荐**：简短（许可证名称或文件引用）

```yaml
✅ Good:
  license: Apache-2.0
  license: MIT
  license: Proprietary. See LICENSE.txt

❌ Too verbose:
  license: |
    Licensed under the Apache License, Version 2.0...
    {完整许可证文本}
```

---

#### compatibility 字段（可选）

**约束**：1-500 字符（如提供）

**使用场景**：仅当有特定环境要求时

```yaml
✅ Good:
  compatibility: Designed for Claude Code (or similar products)
  compatibility: Requires git, docker, jq, and access to the internet

❌ Unnecessary:
  compatibility: Works everywhere  # 大多数 Skill 不需要此字段
```

---

#### metadata 字段（可选）

**用途**：存储自定义属性

```yaml
metadata:
  author: example-org
  version: "1.0"
  created_at: "2026-01-15"
  category: "data-processing"
  difficulty: "beginner"
```

**注意**：
- 键名要合理唯一，避免冲突
- 客户端可用于自定义功能（如分类、过滤）

---

#### allowed-tools 字段（实验性）

**用途**：预批准工具列表（空格分隔）

```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read Write
```

**状态**：实验性，支持情况因 Agent 而异

---

### A3. 目录结构规范

#### 最小结构

```
skill-name/
└── SKILL.md  # 必需
```

---

#### 完整结构

```
skill-name/
├── SKILL.md           # 必需：核心指令
├── scripts/           # 可选：可执行代码
│   ├── extract.py
│   └── process.sh
├── references/        # 可选：详细文档
│   ├── REFERENCE.md
│   ├── FORMS.md
│   └── advanced.md
└── assets/            # 可选：静态资源
    ├── templates/
    ├── images/
    └── data/
```

---

#### 各目录用途

**scripts/**：
- 包含可执行代码（Python, Bash, JavaScript 等）
- 应自包含或明确声明依赖
- 包含有用的错误消息
- 优雅处理边缘情况

**references/**：
- 包含按需加载的详细文档
- 推荐文件：
  - `REFERENCE.md` - 技术参考
  - `FORMS.md` - 表单模板/结构化数据格式
  - 领域文件（`finance.md`, `legal.md`）
- **保持单个文件聚焦**（Agent 按需加载，小文件 = 少消耗上下文）

**assets/**：
- 包含静态资源
  - 模板（文档模板、配置模板）
  - 图片（流程图、示例截图）
  - 数据文件（查找表、Schema）

---

### A4. 渐进式披露机制详解

#### 三层加载模型

```
┌─────────────────────────────────────────┐
│ Layer 1: Discovery (启动时)             │
│ - 加载：name + description              │
│ - Token 消耗：~100 tokens/skill         │
│ - 目的：知道「有哪些 Skill」            │
└─────────────┬───────────────────────────┘
              ↓ (任务匹配 description)
┌─────────────────────────────────────────┐
│ Layer 2: Activation (激活时)            │
│ - 加载：完整 SKILL.md body              │
│ - Token 消耗：<5000 tokens (推荐)       │
│ - 目的：知道「如何使用这个 Skill」      │
└─────────────┬───────────────────────────┘
              ↓ (指令引用其他文件)
┌─────────────────────────────────────────┐
│ Layer 3: Execution (执行时)             │
│ - 加载：scripts/references/assets      │
│ - Token 消耗：按需（引用时才加载）      │
│ - 目的：执行具体操作                    │
└─────────────────────────────────────────┘
```

---

#### 性能对比

**无渐进式披露**：
```
100 个 Skill × 5000 tokens = 500,000 tokens
→ 超出上下文窗口（大多数模型 <200k）
→ 无法启动
```

**有渐进式披露**：
```
启动：100 个 Skill × 100 tokens = 10,000 tokens
激活 1 个：+5000 tokens = 15,000 tokens
执行：按需加载 reference = +2000 tokens = 17,000 tokens
→ 可控且高效
```

---

#### 设计建议

| 层级 | 推荐大小 | 优化策略 |
|------|---------|---------|
| **name + description** | <100 tokens | 精炼关键词，避免冗余 |
| **SKILL.md body** | <500 行 / <5000 tokens | 详细内容拆到 references/ |
| **references/** | 每个文件 <2000 tokens | 单一职责，聚焦主题 |

**反模式警告**：
- ❌ SKILL.md 超过 1000 行（应拆分）
- ❌ 嵌套引用链（`SKILL.md` → `ref1.md` → `ref2.md` → `ref3.md`）
- ❌ description 过于宽泛（"Helps with everything"）

---

### A5. 文件引用规范

#### 相对路径引用

**从 SKILL.md 引用其他文件**：

```markdown
See [the reference guide](references/REFERENCE.md) for details.

Run the extraction script:
scripts/extract.py
```

**规则**：
- ✅ 相对路径从 Skill 根目录开始
- ✅ 保持引用深度为 1 层（避免链式引用）
- ❌ 不要使用绝对路径

---

#### 引用最佳实践

```markdown
✅ Good:
  For detailed API reference, see [API.md](references/API.md)
  
  Execute the setup script:
  ```bash
  python scripts/setup.py
  ```

❌ Bad:
  See references/detailed/subsection/specific-case.md
  # 引用层级过深
```

---

### A6. 验证工具使用

#### 安装 skills-ref

```bash
git clone https://github.com/agentskills/agentskills.git
cd agentskills/skills-ref
```

---

#### 验证 Skill

```bash
# 验证单个 Skill
skills-ref validate ./my-skill

# 输出示例
✅ Validation passed
  - name: pdf-processing (valid)
  - description: 145 characters (valid)
  - SKILL.md: frontmatter valid

❌ Validation failed
  - name: PDF-Processing (invalid: uppercase not allowed)
  - description: missing
```

---

#### 生成 Prompt XML

```bash
# 生成适合 LLM 的 XML 格式
skills-ref generate-xml ./my-skill

# 输出
<skill name="pdf-processing">
  <description>Extract text and tables...</description>
  <instructions>
    {SKILL.md 内容...}
  </instructions>
</skill>
```

---

### A7. 支持的 Agent 产品（30+）

#### AI IDE 类

| 产品 | 支持情况 | 特点 |
|------|---------|------|
| **Cursor** | ✅ 全面支持 | 主流 AI IDE |
| **Claude Code** | ✅ 原生支持 | Anthropic 官方 |
| **VS Code** | ✅ 通过扩展 | 最大生态 |
| **OpenCode** | ✅ 原生支持 | 终端图形界面 |

#### AI CLI 类

| 产品 | 支持情况 | 特点 |
|------|---------|------|
| **Gemini CLI** | ✅ 支持 | Google 生态 |
| **Autohand Code** | ✅ 支持 | CLI 自动化 |
| **Goose** | ✅ 支持 | Block (Square) 开源 |

#### 企业/平台类

| 产品 | 支持情况 | 特点 |
|------|---------|------|
| **GitHub Copilot** | ✅ 支持 | GitHub 官方 |
| **Databricks** | ✅ 支持 | 数据平台 |
| **Spring AI** | ✅ 支持 | Java 生态 |
| **Letta** | ✅ 支持 | 记忆系统 |

**完整列表见官网**：https://agentskills.io/home

---

### A8. 实战案例：创建第一个 Skill

#### 场景：生成 PR 描述

**1. 创建目录结构**

```bash
mkdir -p pr-description-generator
cd pr-description-generator
touch SKILL.md
```

---

**2. 编写 SKILL.md**

```markdown
---
name: pr-description-generator
description: Generate high-quality pull request descriptions from git diff. Use when user asks to create PR description, summarize changes, or write commit message.
license: MIT
metadata:
  author: your-name
  version: "1.0"
---

# PR Description Generator

## When to use this skill

Use this skill when the user:
- Asks to create a pull request description
- Wants to summarize code changes
- Needs to write a commit message
- Mentions "PR", "pull request", "git diff"

## How it works

1. Get the git diff of current branch
2. Analyze changed files and code
3. Generate structured PR description

## Steps

### 1. Get git diff

```bash
git diff main...HEAD
```

### 2. Analyze changes

Group changes by:
- New features
- Bug fixes
- Refactoring
- Documentation

### 3. Generate description

Use this template:

\`\`\`markdown
## Summary
{1-2 sentences overview}

## Changes
- Feature: {what was added}
- Fix: {what was fixed}
- Refactor: {what was improved}

## Test Plan
{how to test these changes}
\`\`\`

## Tips

- Keep summary concise (<100 words)
- Focus on "why" not "what"
- Include test instructions
```

---

**3. 验证**

```bash
# 安装验证工具
git clone https://github.com/agentskills/agentskills.git
cd agentskills/skills-ref

# 验证 Skill
./validate ../pr-description-generator

# 期望输出
✅ Validation passed
```

---

**4. 使用**

将 `pr-description-generator/` 放到：
- Cursor: `.cursor/skills/`
- Claude Code: `.claude/skills/`
- VS Code: `.vscode/skills/`

然后在 Agent 中说：
```
"帮我生成这个 PR 的描述"
```

Agent 会自动：
1. 发现 Skill（匹配 "PR" 关键词）
2. 激活 Skill（加载完整指令）
3. 执行工作流（运行 git diff + 分析 + 生成）

---

### A9. 与其他能力扩展方式对比

| 维度 | Agent Skills | VS Code Extension | OpenAI Plugin | MCP Server |
|------|-------------|-------------------|--------------|------------|
| **格式** | 文件夹 | JavaScript 包 | API Manifest | JSON-RPC |
| **安装** | 复制文件夹 | Marketplace | OAuth 授权 | npm install |
| **可移植性** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐ |
| **审计性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ | ⭐⭐⭐ |
| **学习曲线** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **执行能力** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **生态支持** | 30+ 工具 | VS Code 专属 | OpenAI 专属 | 10+ 工具 |

**选择指南**：
- **纯指令类**：Agent Skills（最优）
- **复杂 UI**：VS Code Extension
- **外部 API**：OpenAI Plugin / MCP Server
- **本地工具**：MCP Server / Agent Skills

---

## 🏁 学习总结

### 核心收获

✅ **根节点理解**：Agent Skills = 渐进式披露 + 可移植格式 + 开放生态  
✅ **三层加载**：元数据（~100 tokens）→ 指令（<5000 tokens）→ 资源（按需）  
✅ **设计原则**：简单格式（YAML + Markdown）→ 易采用 → 网络效应  
✅ **生态价值**：写一次，30+ 工具支持，跨平台复用  
✅ **质量保证**：强制元数据 + skills-ref 验证 + 最佳实践文档  

### 反直觉洞见总结

💡 简单格式 > 复杂 API（可移植性带来网络效应）  
💡 渐进式披露不是优化，而是架构前提（否则无法启动）  
💡 开放标准是 Anthropic 的护城河（主导标准演进）  
💡 文件夹 > API（可读 + 可版本控制 + 可审计）  
💡 description 是"激活函数"（决定 Skill 能否被正确匹配）

### 立即行动（Top 3）

**本周行动**：
1. [ ] 创建第一个 Skill（PR 描述生成器），48h 内完成
2. [ ] 安装 skills-ref 验证工具，验证你的 Skill
3. [ ] 在 2 个工具中测试跨平台（Cursor + Claude Code）

**关键指标**：
- 第1个 Skill：<100 行 SKILL.md
- description：包含 3+ 关键词
- 验证通过：`skills-ref validate` ✅

---

**学习完成日期**：2026 年 2 月 7 日  
**笔记版本**：v1.0（符合 article-tutor 规范）  
**下一步**：创建你的第一个 Agent Skill！🚀
