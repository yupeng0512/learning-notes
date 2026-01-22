# Skill 生态全攻略：从发现到构建，把开源智慧变成你的 AI 弹药库

> 团队周会分享 | 2026-01-22
> 
> **一句话**：掌握 Skill 发现、安装、构建的完整技能栈，让 AI 助手从「能用」变成「超能用」

---

## TL;DR（给没时间的同学）

| 关键信息 | 说明 |
|----------|------|
| **问题** | AI 助手能力受限于 Prompt，Skill 生态碎片化、发现难、安装繁 |
| **方案** | Skill 生态四件套：发现 + 安装 + 上下文准备（GitIngest）+ 构建（Skill Seekers） |
| **效果** | 一句话搜索安装 Skill；30 分钟把任意开源项目 Skill 化 |
| **适用场景** | 需要频繁使用 AI 编程助手的开发者、想提升团队 AI 效率的 TL |

**今日行动**：会后 5 分钟，运行 `npx skills-installer search` 体验一下

---

## 议程

```
┌─────────────────────────────────────────────────────────────┐
│  Part 1：为什么需要 Skill？（3 分钟）                        │
│  Part 2：Skill 发现与安装（8 分钟）                          │
│  Part 3：Skill 构建方法论（10 分钟）                         │
│  Part 4：团队 Skill 最佳实践（4 分钟）                       │
│  Q&A（5 分钟）                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 1：为什么需要 Skill？

### 开源世界的「可及性危机」

> **用户价值 = 工具能力 × 可及性系数**

**案例**：`yt-dlp` —— 143k stars 的视频下载神器

```bash
# 99.99% 的人在看到这行命令后选择了关闭页面
yt-dlp -f "bestvideo[height<=1080]+bestaudio" --cookies-from-browser chrome \
  --embed-metadata --embed-thumbnail "https://www.youtube.com/watch?v=xxx"
```

| 工具 | 能力值 | 可及性 | 用户价值 |
|------|--------|--------|----------|
| yt-dlp（原生） | 100 | ≈ 0（命令行） | ≈ 0 |
| yt-dlp + Skill | 100 | ≈ 1（自然语言） | ≈ 100 |

**Skill 的本质**：让 AI 当翻译官，把命令行翻译成自然语言

### Skill 对团队的价值

| 场景 | 没有 Skill | 有 Skill |
|------|-----------|----------|
| **新人上手** | 学 3 天 CLI 工具 | 说一句话就能用 |
| **经验沉淀** | 每个人重复踩坑 | 写入 Skill，团队共享 |
| **工具切换** | 重新学习成本 | Skill 屏蔽底层差异 |

---

## Part 2：Skill 发现与安装

### 生态全景图

```
┌─────────────────────────────────────────────────────────────┐
│                      Skill 生态三层架构                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  L3 构建层：Skill Seekers（自动化）/ 手动 Skill 化           │
│      ↑ 生产 Skill                                           │
│                                                             │
│  L2 分发层：Skills Installer + Skill Lookup                 │
│      ↑ 发现 & 安装                                          │
│                                                             │
│  L1 使用层：Claude Code / Cursor / Windsurf / Gemini CLI    │
│      ↑ 调用 Skill                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 两种发现与安装方式

#### 方式 A：Skills Installer（CLI 工具）

**适合**：开发者、需要跨客户端、跨仓库的灵活安装

```bash
# 安装方式
npx skills-installer search          # 交互式搜索
npx skills-installer install @owner/repo/skill  # 一键安装

# 实际演示
npx skills-installer search "code review"       # 搜索代码审查相关
npx skills-installer install @anthropic/skills/ui-ux-pro-max  # 安装 UI 专家技能
npx skills-installer install @owner/repo/skill --client cursor --local  # 指定客户端 + 项目级
```

**支持 15 种客户端**：

| 客户端 | 全局路径 | 本地路径 |
|--------|----------|----------|
| Claude Code | `~/.claude/skills/` | `./.claude/skills/` |
| Cursor | ❌ | `./.cursor/skills/` |
| Windsurf | `~/.codeium/windsurf/skills/` | `./.windsurf/skills/` |
| CodeBuddy | `~/.codebuddy/skills/` | `./.codebuddy/skills/` |

**核心设计亮点**：

```typescript
// 配置表驱动 —— 新增客户端只需加 3 行
export const CLIENT_CONFIGS = {
  "claude-code": { globalDir: "~/.claude/skills", localDir: "./.claude/skills" },
  "cursor": { localDir: "./.cursor/skills" },  // 无全局
  // ... 15 种客户端
};
```

#### 方式 B：Skill Lookup（对话式）

**适合**：支持 MCP 协议的 AI 客户端（Claude Desktop、Cursor、CodeBuddy 等），想要「说句话就安装」的体验

```
用户："Find me skills for code review"
AI：[自动调用 search_skills MCP 工具]
    找到 3 个相关 Skill：
    1. code-review-expert - 专业代码审查
    2. pr-reviewer - PR 审查助手
    ...
        
用户："Install the code-review-expert"
AI：[自动调用 get_skill + 写入文件]
    ✅ 已安装到 .codebuddy/skills/code-review-expert/
```

**底层机制**：

```
┌─────────────────────────────────────────────────────────────┐
│              Skill Lookup 工作原理                           │
├─────────────────────────────────────────────────────────────┤
│  ① 意图识别：frontmatter description 定义触发条件           │
│  ② 工具调用：search_skills / get_skill MCP 工具            │
│  ③ 文件落地：安装到 .claude/skills/{slug}/                  │
└─────────────────────────────────────────────────────────────┘
```

#### 对比选择

| 维度 | Skills Installer | Skill Lookup |
|------|------------------|--------------|
| **交互方式** | 命令行 | 自然对话 |
| **Skill 来源** | 任意 GitHub 仓库 | prompts.chat 平台 |
| **客户端支持** | 15 种 | 仅 Claude |
| **使用门槛** | 需要终端 | 说话就行 |
| **灵活性** | 高 | 低 |

**推荐**：先用 Skill Lookup 快速上手，再用 Skills Installer 管理复杂场景

### 元技能：教 AI 找技能

```bash
# 安装「发现技能」的技能 —— 递归设计的精髓
npx skills-installer install @Kamalnrf/claude-plugins/skills-discovery
```

安装后，AI 会主动在任务开始前思考：

```markdown
## When to search for skills
Before starting any non-trivial task:
1. Do I have a skill for this? → Use it
2. Might one exist that I don't have? → Search the registry
```

---

## Part 3：Skill 构建方法论

### 两条路径

| 维度 | 手动 Skill 化 | Skill Seekers（自动化） |
|------|---------------|-------------------------|
| **适用场景** | 单个项目深度定制 | 批量生成、标准化项目 |
| **投入时间** | 30-60 分钟 | 5-10 分钟 |
| **定制灵活度** | 高 | 中 |
| **推荐人群** | 想理解原理 | 追求效率 |

### 路径 A：手动 Skill 化流程

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub → Skill 完整流程                   │
├─────────────────────────────────────────────────────────────┤
│  1. 需求 → "我想下载 YouTube 视频"                           │
│       ↓                                                     │
│  2. 搜索 → yt-dlp (143k stars)                              │
│       ↓                                                     │
│  3. Skill 化 → 编写 SKILL.md                                │
│       ↓                                                     │
│  4. 调试 → 处理 Cookie/依赖问题                              │
│       ↓                                                     │
│  5. 沉淀 → 踩坑经验写回 Skill                                │
│       ↓                                                     │
│  6. 固化 → 后续调用只需十几秒                                │
└─────────────────────────────────────────────────────────────┘
```

**Skill 文件结构**：

```markdown
# {Skill 名称}

## 能力描述
{这个 Skill 能做什么}

## 环境要求
{依赖、安装命令}

## 核心命令
{最常用的命令模板}

## 使用示例
{2-3 个具体场景}

## 常见问题
{踩过的坑和解决方案}  ← 这是最有价值的部分！
```

**经验沉淀是复利**：

```
首次调试 30 分钟（投资）
    ↓
写回 Skill
    ↓
后续每次节省 25 分钟（收益）
    ↓
第 2 次就回本，第 N 次就是纯赚
```

### 路径 B：Skill Seekers 自动化

#### 前置工具：GitIngest —— 上下文准备

**核心洞察**：AI 只能理解文本，但代码库是结构化的文件系统。要让 AI 生成高质量 Skill，必须先解决「格式翻译」问题。

```
┌─────────────────────────────────────────────────────────────┐
│                GitIngest 在 Skill 构建中的角色               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   GitHub 仓库                                               │
│       │  分散的文件、目录、配置                             │
│       ▼                                                    │
│   GitIngest 格式翻译                                        │
│       │  • 目录树（结构感知）                               │
│       │  • 文件内容（带路径标注）                           │
│       │  • Token 估算（能不能塞进去）                       │
│       ▼                                                    │
│   LLM 友好的纯文本                                          │
│       │                                                    │
│       ├──→ 直接问 AI 分析项目                              │
│       └──→ 喂给 Skill Seekers 做 AI 增强                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**快速上手**：

```bash
# 最快方式：URL 技巧
# github.com/owner/repo → gitingest.com/owner/repo

# CLI 方式
pip install gitingest
gitingest https://github.com/yt-dlp/yt-dlp -o yt-dlp-context.txt

# 智能过滤（避免 Token 超限）
gitingest https://github.com/facebook/react \
  --include-patterns "*.js,*.md" \
  --exclude-patterns "tests/*,fixtures/*"
```

**为什么重要**：
- Skill Seekers 的 `github` 命令内部也需要处理仓库内容
- 手动 Skill 化时，先用 GitIngest 让 AI 「看懂」项目，再让它帮你写 SKILL.md
- Token 估算帮你判断：整个仓库能否一次性塞给 AI，还是需要分块处理

#### Skill Seekers 主流程

```bash
# 安装
pip install skill-seekers[all]

# 一键生成（用内置预设）
skill-seekers install --config react

# 分析 GitHub 仓库
skill-seekers github --repo facebook/react

# AI 增强 + 打包
skill-seekers enhance output/react/
skill-seekers package output/react/ --format claude
```

**核心架构 —— 三流合一**：

```
┌─────────────────────────────────────────────────────────────┐
│                   Skill Seekers 三流架构                     │
├───────────────┬───────────────┬─────────────────────────────┤
│    Code 流    │    Docs 流    │       Insights 流           │
├───────────────┼───────────────┼─────────────────────────────┤
│ • AST 解析   │ • 官方文档    │ • Issues 讨论              │
│ • API 提取   │ • 示例代码    │ • PR 最佳实践              │
│ • 类型信息   │ • 配置说明    │ • 常见问题                 │
└───────────────┴───────────────┴─────────────────────────────┘
                            ↓
                    冲突检测（文档 vs 代码）
                            ↓
                    统一 Skill 输出
```

**亮点**：冲突检测 —— 自动发现文档与代码实现不一致的地方

---

## Part 4：团队 Skill 最佳实践

### 组织结构建议

```
team-skills/
├── dev-tools/           # 开发工具类
│   ├── code-review/
│   └── git-workflow/
├── data-processing/     # 数据处理类
│   └── csv-analyzer/
├── internal-systems/    # 内部系统类
│   └── deploy-helper/
└── README.md           # 索引 + 使用说明
```

### 核心原则

| 原则 | 说明 | 反例 |
|------|------|------|
| **Skill 原子化** | 一个 Skill 做一件事 | 万能 Skill 塞 10 个功能 |
| **示例覆盖 80%** | 常见场景都有示例 | 只写定义不给例子 |
| **经验必沉淀** | 踩坑立即写回 Skill | 下次还得重新踩 |
| **版本锁定** | 依赖版本写死 | "装最新版就行" |

### 分层使用模型

```
构建 Skill：用强模型（Opus/GPT-4）—— 质量第一
运行 Skill：用便宜模型（Sonnet/Haiku）—— 成本优先
```

### 避坑指南

| 坑点 | 表现 | 解决方案 |
|------|------|----------|
| Skill 太泛 | AI 不知道何时调用 | 明确触发场景和边界 |
| 缺少错误处理 | 一出错就卡住 | 加入常见问题 + 兜底 |
| 依赖不固定 | 换环境就跑不通 | 锁定依赖版本 |
| 认证问题 | Token 过期 | 加入刷新逻辑 |

---

## 行动建议

### 今天（5 分钟）

- [ ] 运行 `npx skills-installer search` 体验交互式搜索
- [ ] 安装元技能：`npx skills-installer install @Kamalnrf/claude-plugins/skills-discovery`

### 本周（1 小时）

- [ ] 选一个团队常用的 CLI 工具，手动 Skill 化
- [ ] 把调试经验写入 Skill，分享给组内同学

### 长期

- [ ] 建立团队 Skill 仓库
- [ ] 制定 Skill 贡献规范（PR 模板 + Review 流程）
- [ ] 每月 Skill 分享会（轮流贡献）

---

## 总结：Skill 生态公式

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│    AI 助手战斗力 = 基础模型 × Skill 数量 × Skill 质量       │
│                                                             │
│    基础模型：厂商决定，你改变不了                            │
│    Skill 数量：发现 + 安装解决（Installer / Lookup）         │
│    Skill 质量：构建 + 沉淀解决（Seekers / 手动）             │
│                                                             │
│    → 你能控制的是后两项，这才是差异化竞争力                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**记住一句话**：

> Skills encode best practices, tools, and techniques you wouldn't otherwise have.
> 
> —— Skill 把你本来不具备的最佳实践、工具和技巧，编码进了 AI 助手的能力边界。

---

## Q&A

<details>
<summary><strong>Q: Skill 和 MCP Tool 有什么区别？</strong></summary>

A: **Skill** = Prompt + 脚本，教 AI「做什么」；**MCP Tool** = 标准化工具调用协议，定义「如何执行」。两者可结合：Skill 描述能力，MCP Tool 提供执行层。

</details>

<details>
<summary><strong>Q: 自动生成的 Skill 质量如何保证？</strong></summary>

A: Skill Seekers 用冲突检测对比文档和代码。但最终仍需人工 Review，特别是涉及安全和权限的部分。

</details>

<details>
<summary><strong>Q: 怎么让 AI 更主动使用 Skill？</strong></summary>

A: 安装 skills-discovery 元技能，它会教 AI 在任务开始前主动搜索是否有现成 Skill。另外 Skill 的 description 写清楚触发场景也很关键。

</details>

<details>
<summary><strong>Q: 团队 Skill 放哪里合适？</strong></summary>

A: 
- **项目级**：`.codebuddy/skills/`（仅当前项目用）
- **团队级**：内部 Git 仓库 + skills-installer 安装
- **个人级**：`~/.codebuddy/skills/`（跨项目复用）

</details>

---

## 参考资源

- [Skills Installer](https://github.com/Kamalnrf/claude-plugins) - CLI 安装工具
- [Skill Seekers](https://github.com/yusufkaraaslan/Skill_Seekers) - 自动化生成工具
- [prompts.chat](https://prompts.chat) - Skill 托管平台
- [agentskills.io](https://agentskills.io) - Agent Skills 规范
- [llms.txt 协议](https://llmstxt.org/) - 让网站对 LLM 更友好
