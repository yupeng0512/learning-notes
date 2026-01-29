---
title: Anthropic：长时间运行 Agent 的有效基础设施
source: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
author: Justin Young (Anthropic)
date: 2026-01-29
category: ai-tools/agent-architecture
tags:
  - AI Agent
  - Long-running Agent
  - Claude Agent SDK
  - Infrastructure
  - Anthropic
  - Multi-context
  - Session Management
  - Testing
company: Anthropic
learning_mode: 深度模式
---

# Anthropic：长时间运行 Agent 的有效基础设施

> Anthropic 解决跨多个上下文窗口的 Agent 持续工作难题：Initializer + Coding Agent 双架构。

## 📋 文章概览

| 维度 | 信息 |
|------|------|
| **作者** | Justin Young (Anthropic) |
| **发布时间** | 2025-11-26 |
| **文章类型** | 工程实践指南（生产级方案）|
| **核心主题** | 如何让 Agent 在多个上下文窗口间持续工作 |
| **实战案例** | 构建 claude.ai 克隆（200+ 功能，跨多天）|
| **关键价值** | 双 Agent 架构 + 环境管理 + 测试驱动 |

### 核心价值

**一句话概括**：长时间运行 Agent 的核心挑战是"换班失忆"，解决方案是让 Agent 像人类工程师一样使用 git、进度日志、feature list 来协作。

### 核心架构

**双 Agent 解决方案**：

| Agent 类型 | 职责 | 运行时机 | 核心产出 |
|-----------|------|---------|---------|
| **Initializer Agent** | 环境初始化 | 第一个会话 | feature_list.json、init.sh、git repo、claude-progress.txt |
| **Coding Agent** | 增量进步 | 后续每个会话 | git commit、更新 progress、更新 feature list |

---

## 🌳 根节点命题

> **长时间运行 Agent = 外部化记忆（文件系统）+ 增量进步（一次一个功能）+ 清洁状态（随时可交接）**

### 为什么这是根节点？

Anthropic 通过构建 claude.ai 克隆（200+ 功能）发现：**Compaction（上下文压缩）不足以解决长时间运行问题，真正的解决方案是借鉴人类工程师的团队协作模式**。

#### 核心比喻（Anthropic 原话）

> "想象一个软件项目由轮班工程师组成，每个新工程师到来时都完全不记得上一班发生了什么。"

这个比喻揭示了问题的本质：
- **Agent 的上下文窗口** = 单个工程师的"工作记忆"
- **多个会话** = 工程师换班
- **没有协作机制** = 失忆的换班

#### 三层递进的解决方案

1. **外部化记忆层（持久化）**
   ```
   问题：每个会话都是全新的，没有记忆
     ↓
   解决：将记忆写入文件系统
     - feature_list.json：全局任务清单（200+ 功能）
     - claude-progress.txt：进度日志（每次做了什么）
     - git history：代码变更历史（可回溯）
     - init.sh：环境启动脚本（快速恢复环境）
     ↓
   效果：新会话可以"读档"，快速了解当前状态
   ```

2. **增量进步层（策略）**
   ```
   问题：Agent 倾向于"一次做完所有功能"或"过早宣布完成"
     ↓
   解决：强制增量工作
     - Initializer Agent：拆解为 200+ 独立功能
     - Coding Agent：一次只选一个功能实现
     - 每完成一个 → git commit + 更新 progress
     ↓
   效果：避免功能半成品、可随时中断和恢复
   ```

3. **清洁状态层（交接）**
   ```
   问题：前一个会话可能留下 bug 或未完成代码
     ↓
   解决：每个会话结束时"清理现场"
     - 基础测试通过（端到端验证）
     - 代码有 git commit（可回滚）
     - 进度文件更新（下一个知道做了什么）
     ↓
   效果：下一个会话可以直接开始新功能，无需"救火"
   ```

#### 从根节点推导所有内容

```
根节点：外部化记忆 + 增量进步 + 清洁状态
    │
    ├─→ Initializer Agent（第一班工程师）
    │   ├─→ 创建 feature_list.json（任务清单）
    │   ├─→ 创建 claude-progress.txt（日志系统）
    │   ├─→ 创建 git repo（版本控制）
    │   └─→ 创建 init.sh（环境脚本）
    │
    ├─→ Coding Agent（后续工程师）
    │   ├─→ 读档（git log + progress + feature list）
    │   ├─→ 基础测试（确保环境健康）
    │   ├─→ 选择一个功能实现
    │   ├─→ 端到端测试（Puppeteer）
    │   └─→ 提交清洁状态（git commit + 更新 progress）
    │
    └─→ 灵感来源：人类软件工程最佳实践
        ├─→ Git workflow（版本控制 + 回滚）
        ├─→ Stand-up meeting（进度同步 = progress 文件）
        ├─→ Feature flagging（功能清单）
        └─→ CI/CD（自动化测试）
```

所有 Anthropic 的设计都是从这个根节点展开的：**让 Agent 像人类团队一样协作**。

---

## 📐 表示空间（核心维度）

Anthropic 的长时间运行 Agent 设计可以用**三个正交维度**完整描述：

| 维度 | 含义 | Anthropic 实现 | 关键指标 |
|------|------|---------------|---------|
| **记忆维度** | 跨会话信息传递 | 文件系统（git + progress + feature list）| 新会话"读档"时间 |
| **进度维度** | 任务分解与执行 | 增量进步（一次一个功能）+ 清洁状态 | 功能完成率、bug 引入率 |
| **验证维度** | 质量保证 | 端到端测试（Puppeteer 浏览器自动化）| 测试覆盖率、bug 发现率 |

### 维度 1: 记忆维度 —— 文件系统是 Agent 间的"共享大脑"

**核心问题**：上下文窗口用完后，如何让新会话继承前一个会话的工作？

**Anthropic 的记忆系统**：

```
Agent 的"换班交接"流程：

Session 1（上一班）:
  工作内容 → 写入 3 个"记忆文件"
    ├─ claude-progress.txt：
    │   "实现了用户登录功能，添加了 JWT 认证"
    │
    ├─ feature_list.json：
    │   {"description": "用户登录", "passes": true}
    │
    └─ git commit：
        "feat: add user authentication with JWT"

Session 2（新来的）:
  读取 3 个"记忆文件" → 5 分钟内完全了解状态
    ├─ git log --oneline -20：看最近做了什么
    ├─ claude-progress.txt：看详细说明
    └─ feature_list.json：看还有哪些未完成
  
  → 立即开始新功能，无需"猜测"前任做了什么
```

**四个记忆文件的职责分工**：

| 文件 | 信息类型 | 更新频率 | 作用 |
|------|---------|---------|------|
| **feature_list.json** | 全局任务清单（200+ 功能）| 每完成一个功能 | 长期规划，知道"还有什么要做"|
| **claude-progress.txt** | 详细进度日志 | 每个会话结束 | 理解"刚才做了什么" |
| **git history** | 代码变更历史 | 每次提交 | 看"具体改了什么"，可回滚 |
| **init.sh** | 环境启动脚本 | 初始化时写入 | 快速启动开发服务器 |

**对比 Compaction（上下文压缩）**：

| 方法 | 机制 | 效果 | 局限性 |
|------|------|------|--------|
| **Compaction** | 压缩历史上下文 | 延长单个会话 | ⚠️ 压缩后的指令可能不清晰 |
| **外部化记忆**（Anthropic）| 写入文件系统 | 完美交接 | ✅ 完整、可追溯、可回滚 |

**关键洞见**：
```
Compaction 是必要的（延长单个会话）
但不充分（无法解决跨会话问题）

Compaction + 外部化记忆 = 完整解决方案
```

---

### 维度 2: 进度维度 —— 增量进步是避免"摊大饼"的关键

**核心问题**：Agent 有两种极端失败模式，如何避免？

**失败模式 1：一次做太多（"摊大饼"）**

```
Agent 的"雄心壮志"：
  Prompt: "构建 claude.ai 克隆"
  
  Agent 思考：
    "我要同时实现：
     - 用户认证
     - 聊天界面
     - 历史记录
     - 设置页面
     - 主题切换
     - ..."
  
  结果：
    → 上下文窗口在实现到一半时用完
    → 留下半成品代码（未完成、无文档）
    → 下一个会话："？？？这是什么？"
```

**失败模式 2：过早宣布完成**

```
后期会话：
  Agent 读档：
    "看起来已经有登录、聊天、历史记录了"
  
  Agent 判断：
    "嗯，好像差不多了，任务完成！"
  
  实际情况：
    还有 150 个功能未实现
    （设置、主题、导出、分享、API...）
```

**Anthropic 的增量进步策略**：

```
Initializer Agent:
  - 拆解 prompt 为 200+ 独立功能
  - 写入 feature_list.json
  - 所有功能初始标记为 "passes": false
  
Coding Agent（每个会话）:
  Step 1: 读 feature_list.json
  Step 2: 选择第一个 "passes": false 的功能
  Step 3: 只实现这一个功能
  Step 4: 测试通过 → 标记 "passes": true
  Step 5: git commit + 更新 progress
  
  → 一次只做一件事，做完再做下一个
```

**Feature List 示例**（claude.ai 克隆）：

```json
{
  "category": "functional",
  "description": "New chat button creates a fresh conversation",
  "steps": [
    "Navigate to main interface",
    "Click the 'New Chat' button",
    "Verify a new conversation is created",
    "Check that chat area shows welcome state",
    "Verify conversation appears in sidebar"
  ],
  "passes": false
}
```

**为什么用 JSON 而非 Markdown**：

| 格式 | Agent 行为 | 原因 |
|------|-----------|------|
| **Markdown** | ⚠️ 容易不当编辑 | LLM 倾向于"润色"Markdown |
| **JSON** | ✅ 只改 passes 字段 | 结构化约束，不易乱改 |

**清洁状态（Clean State）的定义**：

```
"可合并到主分支的代码状态"：
  ✅ 没有重大 bug
  ✅ 代码整洁、有文档
  ✅ 测试通过
  ✅ 新工程师可以直接开始新功能（无需先"清理"）
```

---

### 维度 3: 验证维度 —— 端到端测试是质量保证的核心

**核心问题**：Agent 如何知道功能真的"工作"？

**Anthropic 观察到的测试问题**：

```
没有显式提示时，Claude 的行为：
  ✅ 写代码
  ✅ 写单元测试
  ✅ 用 curl 测试 API
  ❌ 但忽略端到端验证
  
结果：
  单元测试通过 ✅
  API 返回正确 ✅
  但：功能在浏览器里不工作 ❌
  
  → Agent 标记功能为"完成"
  → 实际功能是坏的
```

**Anthropic 的端到端测试方案**：

```
工具：Puppeteer MCP（浏览器自动化）

每个功能的验证流程：
  1. 启动开发服务器（init.sh）
  2. Puppeteer 打开浏览器
  3. 像人类用户一样操作：
     - 点击"New Chat"按钮
     - 输入消息
     - 按回车
     - 等待响应
     - 截图验证
  4. 检查结果（截图 + DOM 状态）
  
  → 只有端到端通过，才标记 "passes": true
```

**Puppeteer 测试的威力**：

| 测试方式 | 发现的 bug 类型 | 覆盖率 |
|---------|----------------|--------|
| 单元测试 | 逻辑错误 | 60% |
| API 测试 | 接口错误 | 75% |
| **端到端测试**（Puppeteer）| **UI bug、集成问题、用户体验** | **95%** |

**测试驱动的会话流程**：

```
Coding Agent 的典型会话：

Step 1: 读档（git log + progress + feature list）
Step 2: 运行 init.sh（启动开发服务器）
Step 3: 基础测试（确保现有功能没坏）
  - Puppeteer: 打开首页
  - Puppeteer: 新建聊天
  - Puppeteer: 发送消息
  - 验证: 收到响应 ✅
  
  如果基础测试失败 ❌ → 修 bug（优先级最高）
  如果基础测试通过 ✅ → 开始新功能

Step 4: 实现新功能
Step 5: 端到端测试新功能（Puppeteer）
Step 6: 提交清洁状态（git commit + progress update）
```

**Anthropic 的测试截图示例**：

> 文章展示了 Claude 通过 Puppeteer MCP 测试 claude.ai 克隆时的截图
> → 可视化验证：Agent 能看到实际的浏览器渲染结果

**测试的局限性（Anthropic 坦诚指出）**：

```
Puppeteer MCP 的盲区：
  ❌ 无法看到浏览器原生 alert 模态框
  
结果：
  依赖 alert 的功能 → 测试不充分 → bug 更多
  
教训：
  工具的限制 = Agent 能力的限制
  → 选择合适的测试工具很关键
```

---

### 三个维度的协同效应

**Anthropic 的 claude.ai 克隆项目**：

```
记忆维度（读档）：
  新会话启动 → 5 分钟内读完 git log、progress、feature list
  → 完全了解当前状态（已完成 50 个功能，还有 150 个）

进度维度（增量）：
  选择功能 #51："实现主题切换"
  → 只实现这一个功能
  → 不碰其他代码（降低风险）
  
验证维度（测试）：
  实现后 → Puppeteer 打开浏览器
  → 点击主题切换按钮
  → 截图验证：UI 确实变成了深色模式 ✅
  → 标记 "passes": true
  
清洁状态（交接）：
  git commit "feat: add dark mode theme switching"
  更新 progress: "完成主题切换功能，测试通过"
  → 下一个会话可以直接开始功能 #52

结果：
  - 200+ 功能，跨多天完成
  - 每个会话都有明确进步
  - 代码质量稳定（始终保持可合并状态）
```

这种协同使 Anthropic 能够构建**生产级、长时间运行**的 Agent 系统。

---

## 🌲 推论展开

从根节点"外部化记忆 + 增量进步 + 清洁状态"可以推导出 6 个核心推论：

### 推论 1: Compaction 是必要的，但不充分

**核心论断**：即使有上下文压缩（Compaction），长时间运行 Agent 仍然会失败。

**推导过程**：

```
Claude Agent SDK 内置 Compaction：
  - 自动压缩历史上下文
  - 理论上可以无限延长单个会话
  
Anthropic 的实验：
  Opus 4.5 + Compaction + "构建 claude.ai 克隆"
  
结果：失败
  
原因 1: 压缩后的指令不够清晰
  - Compaction 可能丢失关键细节
  - 新会话需要"猜测"之前做了什么
  
原因 2: 没有阻止"一次做太多"
  - Agent 仍然尝试同时实现所有功能
  - 上下文在功能半成品时用完
  
原因 3: 没有验证机制
  - Agent 自认为"完成"
  - 没有外部验证（如端到端测试）
```

**Compaction vs 外部化记忆的对比**：

| 维度 | Compaction | 外部化记忆（Anthropic）|
|------|-----------|---------------------|
| **机制** | 压缩上下文（LLM 生成摘要）| 写入文件（结构化）|
| **信息完整性** | ⚠️ 可能丢失细节 | ✅ 完全保留 |
| **可追溯性** | ❌ 压缩不可逆 | ✅ git history 可回溯 |
| **可读性** | ⚠️ 摘要可能不清晰 | ✅ 结构化、易解析 |
| **适用范围** | 延长单个会话 | 跨多个会话 |

**结论**：
```
Compaction（压缩）解决"单会话太短"
外部化记忆（文件）解决"跨会话失忆"

两者互补，缺一不可
```

---

### 推论 2: Feature List 是防止"摊大饼"和"过早完成"的双重保险

**核心论断**：200+ 功能的结构化清单，既是任务清单，也是进度检查点。

**推导过程**：

```
没有 Feature List 的问题：

场景 1（早期）:
  Agent: "我要构建 claude.ai"
  → 尝试一次实现所有功能
  → 上下文爆炸，留下半成品
  
场景 2（后期）:
  Agent 读档: "已经有登录、聊天、历史了"
  → 判断: "看起来不错，完成了！"
  → 实际: 还有导出、分享、API 等 100 个功能
```

**Feature List 的双重作用**：

| 作用 | 机制 | 效果 |
|------|------|------|
| **防止摊大饼** | 200 个功能全部列出，初始全是 `passes: false` | Agent 看到"还有 200 个要做"，不会一次做完 |
| **防止过早完成** | Agent 读 feature list，看到 150 个 `passes: false` | 不会声称"完成"（明显还有很多未做）|

**Feature List 的设计原则**：

```json
// 好的功能描述（可测试、可验证）
{
  "description": "New chat button creates a fresh conversation",
  "steps": [  // ← 具体验证步骤
    "Navigate to main interface",
    "Click the 'New Chat' button",
    "Verify a new conversation is created",
    "Check that chat area shows welcome state",
    "Verify conversation appears in sidebar"
  ],
  "passes": false
}

// 不好的功能描述（模糊、难验证）
{
  "description": "聊天功能"  // ← 太笼统，无法验证
}
```

**Anthropic 的强约束**：

> "不可接受删除或编辑测试，因为这可能导致功能缺失或有 bug。"

**为什么这么严格**：

```
如果允许 Agent 删除功能：
  Agent 发现功能 #87 很难实现
    ↓
  Agent 想："我把它从列表删了吧"
    ↓
  结果：功能永久缺失，用户需求未满足
  
强制约束：
  只能改 "passes" 字段（false → true）
  不能删除、不能修改描述
    ↓
  Agent 必须真正实现功能，不能"偷懒"
```

---

### 推论 3: Git 是 Agent 的"撤销按钮"和"时间机器"

**核心论断**：Git 不仅是版本控制，更是 Agent 的错误恢复机制。

**推导过程**：

```
Agent 的错误累积问题：
  Session 5: 实现功能 A（引入 bug，但未发现）
  Session 10: 实现功能 B（基于有 bug 的代码）
  Session 15: 发现 bug（但不知道是哪里引入的）
  
  → 如果没有 git：难以回溯和修复
```

**Git 的三重作用**：

| 作用 | 机制 | 价值 |
|------|------|------|
| **1. 错误恢复** | `git revert <commit>` | Agent 可以撤销有问题的改动 |
| **2. 状态追溯** | `git log` + `git diff` | 理解"什么时候引入 bug" |
| **3. 清洁检查点** | 每个会话结束 → 强制 commit | 保证代码始终可回滚 |

**Anthropic 的 Git 工作流**：

```bash
# Coding Agent 的标准流程

# Step 1: 读 git 历史
git log --oneline -20
# 输出：
#   a3b2c1d feat: add user authentication
#   e4f5g6h fix: resolve login bug
#   i7j8k9l feat: add chat history

# Step 2: 实现新功能
# ... coding ...

# Step 3: 提交清洁状态
git add .
git commit -m "feat: add dark mode theme switching

- Implemented theme toggle in settings
- Added CSS variables for dark mode
- Tested in Chrome, Firefox, Safari
- All existing features still work"

# Step 4: 更新 progress 文件
echo "Session X: 实现主题切换，测试通过" >> claude-progress.txt
```

**Git Commit Message 的标准**：

```
好的 commit message（描述性）:
  "feat: add dark mode theme switching
  
  - Implemented theme toggle in settings
  - Added CSS variables for dark mode
  - Tested in Chrome, Firefox, Safari"
  
坏的 commit message（不清楚）:
  "update code"  // ← 下一个会话完全不知道改了什么
```

**Agent 学会使用 Git 恢复**：

```python
# Anthropic 观察到 Agent 的行为

# 场景：发现 bug
Agent 读 git log:
  "最近 3 个 commit 都是关于认证的，可能是认证引入的 bug"

Agent 执行:
  git revert a3b2c1d  # 撤销认证的 commit
  
  测试: 基础功能恢复 ✅
  
Agent 决策:
  "认证功能有问题，重新实现"
```

---

### 推论 4: 进度文件是 Agent 间的"交班笔记"

**核心论断**：claude-progress.txt 是比 git commit message 更详细的上下文传递机制。

**推导过程**：

```
Git commit message 的局限：
  - 简短（通常 1-2 行）
  - 聚焦代码改动（"改了什么"）
  - 不包含决策过程（"为什么这样改"）
  
Progress 文件的补充：
  - 详细（可以写几段）
  - 包含上下文（"发现了什么问题"）
  - 包含决策（"为什么选择这个方案"）
  - 包含未来建议（"下一步应该做什么"）
```

**Progress 文件示例**：

```markdown
# claude-progress.txt

## Session 1 (2026-01-29 10:00)
- 初始化项目结构
- 创建 React + Express 基础架构
- 实现基础路由
- 下一步：实现用户认证

## Session 5 (2026-01-29 14:00)
- 实现 JWT 认证
- 发现问题：token 刷新逻辑复杂，暂时用短期 token（24h）
- 添加了单元测试
- 端到端测试通过
- 注意：后续需要实现 refresh token 机制

## Session 10 (2026-01-29 18:00)
- 实现聊天界面（基础版）
- 使用 SSE 实现流式响应
- 测试发现：长消息时滚动有问题
- 临时方案：限制消息长度 < 1000 字
- TODO：改进滚动逻辑

## Session 15 (2026-01-30 10:00)  ← 新会话读到这里
- 看到上一个 session 的 TODO
- 决定先修复滚动问题（影响用户体验）
- ...
```

**Progress 文件 vs Git Log**：

| 维度 | Git Log | Progress 文件 |
|------|---------|--------------|
| 信息密度 | 低（简短）| 高（详细）|
| 包含决策 | ❌ | ✅ |
| 包含问题 | ❌ | ✅ |
| 包含 TODO | ❌ | ✅ |
| 可回滚 | ✅ | ❌ |

**两者互补**：
- Git：代码级别的变更历史
- Progress：决策级别的工作日志

---

### 推论 5: 基础测试是"环境健康检查"，必须优先于新功能

**核心论断**：每个会话开始时，先测试基础功能，确保没有被破坏。

**推导过程**：

```
没有基础测试的问题：

Session 10: 实现功能 A
  → 不小心改坏了基础登录功能
  → 但没发现（没测试）
  → 提交代码

Session 11: 开始实现功能 B
  → 发现登录不工作
  → 不知道是 Session 10 还是 Session 11 引入的
  → 花大量时间调试
  
如果有基础测试：

Session 11 开始:
  → 运行基础测试
  → 发现登录失败 ❌
  → 立即知道：上一个 session 引入的 bug
  → git log 查看上一个 commit
  → 快速定位和修复
```

**Anthropic 的基础测试清单**（claude.ai 克隆）：

```python
def basic_health_check():
    """每个会话必须执行的基础测试"""
    
    # 1. 启动服务器
    run_command("bash init.sh")
    wait_for_server(port=3000, timeout=30)
    
    # 2. Puppeteer 打开首页
    browser = puppeteer.launch()
    page = browser.goto("http://localhost:3000")
    
    # 3. 测试核心功能
    assert page.is_visible(".new-chat-button")  # 界面加载
    
    page.click(".new-chat-button")
    assert page.is_visible(".chat-input")  # 新建聊天
    
    page.type(".chat-input", "Hello")
    page.press("Enter")
    wait_for_response(timeout=10)
    assert page.is_visible(".assistant-message")  # 收到回复
    
    # 4. 截图验证
    page.screenshot("health-check.png")
    
    return "✅ All basic features working"
```

**测试失败的优先级**：

```
基础测试失败 → 最高优先级
  ↓
停止新功能开发
  ↓
修复 bug
  ↓
重新运行基础测试
  ↓
通过 ✅ → 才能开始新功能
```

**类比人类工程师**：

```
人类工程师：
  每天早上第一件事 → 拉最新代码 + 运行测试
  测试失败 → 先修复（不开始新功能）
  
Coding Agent：
  每个会话开始 → 运行基础测试
  测试失败 → 先修 bug
  
→ 相同的"环境健康检查"模式
```

---

### 推论 6: 测试工具的能力上限 = Agent 的质量上限

**核心论断**：Agent 只能发现测试工具能检测到的问题。

**推导过程**：

```
Anthropic 发现的测试盲区：

问题：浏览器原生 alert 模态框
  - Puppeteer MCP 无法看到 alert
  - Claude 的 vision 看不到 alert
  
Agent 的行为：
  实现功能 → 使用 alert 提示用户
  测试 → Puppeteer 截图（看不到 alert）
  判断 → "看起来没问题" ✅
  
结果：
  功能实际有 bug（alert 不工作）
  但 Agent 认为测试通过
  
统计：
  依赖 alert 的功能 → bug 率 3 倍+
```

**测试工具的能力矩阵**：

| 测试工具 | 能检测 | 不能检测 |
|---------|-------|----------|
| **单元测试** | 逻辑错误 | UI 问题、集成问题 |
| **API 测试** | 接口错误 | 前端 bug、用户体验 |
| **Puppeteer** | 大部分 UI、集成 | 原生 alert、性能、可访问性 |
| **人类测试** | 所有问题 | 成本高、无法自动化 |

**Anthropic 的解决策略**：

| 策略 | 说明 |
|------|------|
| **1. 选择合适工具** | 避免使用 Puppeteer 检测不到的 UI 元素（如 alert）|
| **2. 明确工具限制** | 在提示中告诉 Agent："Puppeteer 看不到 alert" |
| **3. 补充人类测试** | 关键功能由人类最终验证 |

**深层启示**：

```
Agent 质量 = min(模型能力, 工具能力)

即使是 Opus 4.5（超强模型）
如果测试工具有盲区
  → Agent 的质量仍受限

投资测试基础设施 = 提升 Agent 质量上限
```

---

## 🔄 泛化模式

Anthropic 的长时间运行 Agent 设计可以迁移到其他领域：

### 模式 1: 外部化记忆 → 任何需要持久化状态的系统

**核心模式**：
```
系统状态 > 内存容量
  → 将状态写入外部存储
  → 新实例"读档"恢复状态
```

**可迁移场景**：

| 领域 | 状态 | 外部化方案 | 类比 Anthropic |
|------|------|-----------|---------------|
| **微服务** | 会话状态 | Redis / 数据库 | claude-progress.txt |
| **游戏** | 游戏进度 | 存档文件 | feature_list.json |
| **批处理** | 处理进度 | Checkpoint 文件 | git commits |
| **流程引擎** | 工作流状态 | 状态机持久化 | feature list |

---

### 模式 2: 增量进步 → 复杂项目的风险控制

**核心模式**：
```
大任务 → 拆解为小任务
每次只做一个 → 完成后验证 → 再做下一个
```

**可迁移场景**：

| 领域 | 应用 | 效果 |
|------|------|------|
| **敏捷开发** | Sprint（每次只做 5-10 个 story）| 可控、可验证 |
| **数据迁移** | 分批迁移（每次 1 个表）| 出错可回滚 |
| **重构** | 小步重构（每次 1 个模块）| 不破坏现有功能 |
| **实验** | 单变量实验（每次改 1 个参数）| 因果关系清晰 |

**类比 Anthropic**：
```
Scrum Sprint = Feature List 的一个子集
每个 Sprint 只做 5-10 个功能
  → 风险可控
  → 交付频繁
  → 反馈及时
```

---

### 模式 3: 清洁状态 → 持续集成（CI/CD）

**核心模式**：
```
每次提交 → 保持"可发布"状态
  → 测试通过
  → 代码整洁
  → 文档完整
```

**可迁移场景**：

| 领域 | "清洁状态"定义 | 检查机制 |
|------|--------------|----------|
| **CI/CD** | 所有测试通过 | 自动化测试 + 静态分析 |
| **代码审查** | 可合并到 main | Pull Request + Review |
| **制造业** | 可交付产品 | 质量检查流程 |
| **写作** | 可发表版本 | 编辑审稿 |

**Anthropic 的"随时可交接"原则** = **持续集成的"主分支始终可部署"原则**

---

### 模式 4: 端到端验证 → 集成测试的价值

**核心模式**：
```
单元测试 + API 测试 ≠ 系统真正工作
必须端到端验证（模拟真实用户）
```

**可迁移场景**：

| 领域 | 单元测试不够的原因 | 端到端方案 |
|------|------------------|-----------|
| **Web 应用** | UI 集成问题 | Selenium / Puppeteer |
| **微服务** | 服务间交互 | Contract Testing |
| **硬件** | 组件集成 | 系统级测试 |
| **供应链** | 跨公司协作 | 端到端业务验证 |

---

## 💡 关键概念

### 概念 1: Initializer Agent vs Coding Agent（双 Agent 架构）

**定义**：两个 Agent 使用相同的工具和系统提示，但初始用户提示不同。

**架构对比**：

| Agent | 运行时机 | 用户提示 | 主要任务 | 产出 |
|-------|---------|---------|---------|------|
| **Initializer** | 第一个会话 | "设置初始环境，为后续会话做准备" | 环境搭建 | 4 个文件 + git repo |
| **Coding** | 第 2-N 个会话 | "增量实现功能，保持清洁状态" | 功能实现 | git commits + 更新 |

**为什么要分开**：

```
单一 Agent 的问题：
  同一个 prompt 要处理两种完全不同的场景：
    - 场景 1（第一次）：从零搭建
    - 场景 2（后续）：增量开发
  
  → Prompt 冲突（无法两全）
  
双 Agent 的优势：
  - Initializer：专注"从零到一"
  - Coding：专注"增量迭代"
  → 每个 Agent 的职责单一、清晰
```

**脚注说明**（Anthropic 原文）：

> "我们在这个上下文中称它们为独立的 agent，只是因为它们有不同的初始用户提示。系统提示、工具集和整体 agent harness 其他方面都是相同的。"

---

### 概念 2: Feature List（功能清单）

**定义**：结构化的功能列表，每个功能包含描述、验证步骤、通过状态。

**数据结构**：

```json
{
  "category": "functional",           // 功能类型
  "description": "用户可以新建聊天",    // 功能描述
  "steps": [                          // 验证步骤（可执行）
    "打开主界面",
    "点击'新建聊天'按钮",
    "验证新对话创建成功",
    "检查聊天区域显示欢迎状态",
    "验证对话出现在侧边栏"
  ],
  "passes": false                     // 当前状态
}
```

**关键设计决策**：

| 决策 | 原因 |
|------|------|
| **用 JSON 而非 Markdown** | LLM 不容易"乱改"JSON（相比 Markdown 更结构化）|
| **包含验证步骤** | Agent 知道"如何验证"（不能自己定义标准）|
| **只允许改 passes 字段** | 防止 Agent 删除或修改功能描述 |

---

### 概念 3: claude-progress.txt（进度日志）

**定义**：记录每个会话做了什么、遇到什么问题、下一步建议的文本文件。

**格式设计**：

```markdown
# claude-progress.txt

## Session {N} ({timestamp})
- 完成的工作：{具体功能}
- 遇到的问题：{描述问题}
- 临时方案：{workaround}
- 下一步建议：{TODO}
- 测试结果：{通过/失败}
```

**与其他记录方式的对比**：

| 方式 | 优点 | 缺点 | Anthropic 选择 |
|------|------|------|---------------|
| Git commit msg | 可回滚 | 太简短 | ✅ 辅助使用 |
| Progress 文件 | 详细、灵活 | 不可回滚 | ✅ 主要方式 |
| 数据库 | 可查询 | 需要额外基础设施 | ❌ 过度工程 |
| 上下文压缩 | 无需额外文件 | 可能丢失细节 | ✅ 配合使用 |

---

### 概念 4: init.sh（环境启动脚本）

**定义**：一键启动开发环境的脚本。

**作用**：

```bash
# init.sh 的典型内容

#!/bin/bash

# 1. 安装依赖（如需要）
if [ ! -d "node_modules" ]; then
    npm install
fi

# 2. 启动后端服务器
cd backend
npm run dev &
BACKEND_PID=$!

# 3. 启动前端开发服务器
cd ../frontend
npm run dev &
FRONTEND_PID=$!

# 4. 等待服务就绪
echo "等待服务启动..."
sleep 5

# 5. 健康检查
curl http://localhost:3000/health
curl http://localhost:5173/

echo "✅ 开发环境就绪"
echo "后端 PID: $BACKEND_PID"
echo "前端 PID: $FRONTEND_PID"
```

**为什么需要 init.sh**：

```
没有 init.sh：
  Agent 每次都要"摸索"如何启动环境
    - "是 npm start 还是 npm run dev？"
    - "后端端口是多少？"
    - "需要哪些环境变量？"
  → 浪费时间，可能出错

有 init.sh：
  Agent: bash init.sh
  → 5 秒内环境就绪
  → 立即开始工作
```

**类比人类工程师**：
- init.sh = 团队的"新人入职文档"
- 新工程师：看文档 → 快速上手
- 新 Agent：读 init.sh → 快速启动环境

---

### 概念 5: 清洁状态（Clean State）

**定义**：代码处于"可合并到主分支"的状态。

**清洁状态的标准**（Anthropic 定义）：

```
✅ 没有重大 bug
✅ 代码整洁、有文档
✅ 测试通过（基础测试 + 新功能测试）
✅ Git commit 有清晰描述
✅ 新开发者可以直接开始新功能（无需先"清理"）
```

**不清洁状态的危害**：

```
Session 10 结束时留下：
  - 功能半成品（代码写了一半）
  - 临时注释的代码（// TODO: fix this）
  - 测试失败（但没修）
  
Session 11 开始时：
  新 Agent 困惑："这是什么状态？"
  → 花 30% 时间"救火"
  → 只有 70% 时间做新功能
  
如果每个会话都保持清洁状态：
  → 新 Agent 100% 时间做新功能
```

**强制清洁状态的机制**：

```python
# Coding Agent 的会话结束检查

def end_of_session_checklist():
    # 1. 运行测试
    test_result = run_all_tests()
    if test_result.failed > 0:
        fix_failing_tests()
    
    # 2. 代码整理（如需要）
    if has_commented_code():
        remove_or_uncomment()
    
    # 3. Git commit
    git_commit("feat: {feature_description}")
    
    # 4. 更新 progress
    update_progress_file(session_summary)
    
    # 5. 验证清洁状态
    assert all_tests_pass()
    assert git_status_clean()
    
    return "✅ Clean state achieved"
```

---

### 概念 6: Puppeteer MCP（浏览器自动化工具）

**定义**：通过 MCP（Model Context Protocol）连接 Puppeteer，让 Agent 能像人类一样操作浏览器。

**核心能力**：

```javascript
// Agent 可用的 Puppeteer 操作

// 1. 导航
await page.goto("http://localhost:3000")

// 2. 点击
await page.click(".new-chat-button")

// 3. 输入
await page.type(".chat-input", "Hello Claude")
await page.press("Enter")

// 4. 等待
await page.waitForSelector(".assistant-message")

// 5. 截图（给 Claude 的 vision）
await page.screenshot("test-result.png")

// 6. 读取内容
const text = await page.textContent(".assistant-message")
```

**Puppeteer 测试 vs 单元测试**：

| 维度 | 单元测试 | Puppeteer 测试 |
|------|---------|---------------|
| **测试范围** | 单个函数/组件 | 整个用户流程 |
| **环境** | 模拟/mock | 真实浏览器 |
| **发现问题** | 逻辑错误 | UI bug、集成问题、UX 问题 |
| **执行速度** | 快（毫秒级）| 慢（秒级）|
| **维护成本** | 低 | 中等 |

**Anthropic 的测试策略**（测试金字塔）：

```
        ┌──────────┐
        │ Puppeteer│  (10个端到端测试)
        │   E2E    │  - 核心用户流程
        └──────────┘
      ┌──────────────┐
      │ API Tests   │  (50个集成测试)
      └──────────────┘
    ┌──────────────────┐
    │  Unit Tests     │  (200个单元测试)
    └──────────────────┘
```

**Puppeteer 的局限性**（Anthropic 坦诚指出）：

```
无法看到：
  ❌ 浏览器原生 alert/confirm/prompt
  ❌ 文件下载对话框
  ❌ 打印预览
  
对策：
  1. 避免使用这些 UI 元素
  2. 或：用自定义模态框替代
  3. 关键功能：人类手动测试
```

---

## 🤔 推论深度展开（苏格拉底式讲解）

围绕 Anthropic 的双 Agent 架构和长时间运行机制，深度讲解关键问题。

### 🤔 问题 1: 为什么 Compaction（上下文压缩）不足以解决长时间运行问题？

<details>
<summary>💡 点击查看深度解答</summary>

这揭示了**压缩 vs 持久化**的本质差异。

#### Compaction 的局限

**问题 1: 压缩是有损的**（细节丢失）
**问题 2: 压缩不能跨会话**（会话结束即清空）
**问题 3: 压缩不能防止"摊大饼"**（Agent 仍会一次做太多）

#### Anthropic 实验结果

```
配置：Opus 4.5 + Compaction
任务："构建 claude.ai 克隆"

结果：失败
  - 失败模式 1：一次做太多，留下半成品
  - 失败模式 2：过早宣布完成
```

#### 外部化记忆的优势

```
Compaction = 短期记忆（工作记忆）
文件系统 = 长期记忆（硬盘）

关键：
  Compaction 延长生命
  外部化记忆实现永生
```

</details>

---

### 🤔 问题 2: 为什么 Anthropic 把功能拆解到 200+ 个如此细粒度？

<details>
<summary>💡 点击查看深度解答</summary>

这是**粒度 vs 可控性**的深刻权衡。

#### 细粒度的三大优势

1. **强制增量工作**：Agent 看到 200 个未完成，不会一次做完
2. **客观验证**：单个功能 = 明确的验证标准
3. **精细回滚**：出错只回滚单个功能，不影响其他

#### 粒度选择标准

| 粒度 | 单个功能时间 | 测试复杂度 | 推荐 |
|------|------------|-----------|------|
| 太粗 | 数小时 | 高 | ❌ |
| **适中（200+）** | **10-30分钟** | **低** | ✅ |
| 太细 | 数分钟 | 极低 | ⚠️ 维护成本高 |

**Anthropic 的"金发女孩原则"**：刚刚好的粒度 = 10-30 分钟可完成 + 可独立测试

</details>

---

### 🤔 问题 3: claude-progress.txt 和 git commit message 都是记录，为什么需要两个？

<details>
<summary>💡 点击查看深度解答</summary>

这是**代码级记录 vs 决策级记录**的互补。

#### 两者的职责分工

| 维度 | Git Commit | Progress 文件 |
|------|-----------|--------------|
| **信息密度** | 低（简短）| 高（详细）|
| **内容焦点** | 代码改动 | 决策过程 |
| **包含问题** | ❌ | ✅ |
| **包含 TODO** | ❌ | ✅ |
| **可回滚** | ✅ | ❌ |

**示例对比**：

```
Git commit:
  "feat: add user authentication with JWT"
  
Progress 文件:
  "Session 5: 实现 JWT 认证
   - 选择 JWT 而非 session（因为需要跨设备）
   - 发现问题：token 刷新逻辑复杂
   - 临时方案：24h 长期 token
   - 下一步：实现 refresh token 机制
   - 测试：登录/登出/token 验证都通过"
```

**新会话的"读档"流程**：

```
Step 1: 读 git log（了解"做了什么"）
  → 看到：认证、聊天、历史、设置

Step 2: 读 progress 文件（了解"为什么这样做"）
  → 看到：为什么选 JWT、有哪些临时方案、未来 TODO

→ 完整理解上下文
```

**深层启示**：Git 是代码的历史，Progress 是决策的历史。两者互补，缺一不可。

</details>

---

### 🤔 问题 4: 为什么每个会话都要运行基础测试？这不是浪费时间吗？

<details>
<summary>💡 点击查看深度解答</summary>

这是**环境健康检查 vs 效率**的权衡。

#### 不做基础测试的风险

```
Session 10: 实现功能 A
  → 不小心改坏基础登录
  → 没测试，没发现
  → 提交代码

Session 11: 开始实现功能 B
  → 跳过基础测试（"应该没问题"）
  → 实现功能 B
  → 测试功能 B: 失败 ❌
  
  Agent 困惑："功能 B 的代码看起来是对的，为什么不工作？"
  → 花 1 小时调试功能 B
  → 最后发现：是登录坏了（功能 B 依赖登录）
  
  总时间：1 小时调试 + 30 分钟修登录 = 1.5 小时
```

**有基础测试的流程**：

```
Session 11 开始:
  Step 1: 运行基础测试（3 分钟）
    - 测试登录: ❌ 失败
  
  Step 2: 立即知道："上一个 session 搞坏了登录"
  
  Step 3: git log 查看上一个 commit
    → 定位到 Session 10 的改动
  
  Step 4: 修复（10 分钟）
  
  Step 5: 开始功能 B（环境健康）
  
  总时间：3 分钟测试 + 10 分钟修复 = 13 分钟
```

**时间权衡**：

| 策略 | 基础测试时间 | Bug 发现时间 | 总时间 |
|------|------------|------------|--------|
| 不做基础测试 | 0 | 1.5 小时（后期发现）| 1.5h |
| **做基础测试** | **3 分钟** | **10 分钟（立即发现）** | **13m** |

**投资回报率**：3 分钟投入 → 节省 1.4 小时 = **28 倍 ROI**

#### 类比人类工程师

```
人类工程师每天早上：
  1. 拉最新代码（git pull）
  2. 运行测试套件（确保环境健康）
  3. 测试失败 → 先修（不开始新功能）
  4. 测试通过 → 开始今天的工作

Coding Agent 每个会话：
  1. 读 git log（了解最新状态）
  2. 运行基础测试（确保环境健康）
  3. 测试失败 → 先修
  4. 测试通过 → 开始新功能

→ 完全相同的"环境健康检查"模式
```

**深层启示**：3 分钟的健康检查，是避免 1 小时调试噩梦的保险。

</details>

---

### 🤔 问题 5: 为什么要用 JSON 格式的 feature list，而不是更"自然"的 Markdown？

<details>
<summary>💡 点击查看深度解答</summary>

这涉及**LLM 的编辑行为模式**。

#### Anthropic 的实验发现

**Markdown Feature List**：
```markdown
# Feature List

## ✅ 已完成
- [x] 用户登录功能

## ❌ 待完成
- [ ] 用户可以新建聊天
- [ ] 用户可以查看历史对话
```

**Agent 的行为**：
```
Agent 倾向于"润色"Markdown：
  - "用户可以新建聊天" → "用户能够轻松创建新对话"
  - 添加额外的分类
  - 重新排序
  - 添加 emoji 和格式
  
问题：
  - 功能描述被改变（可能不准确）
  - 列表结构被重组（难以追踪）
  - 可能删除"太难"的功能（Agent 逃避）
```

**JSON Feature List**：
```json
{
  "description": "用户可以新建聊天",
  "steps": [...],
  "passes": false
}
```

**Agent 的行为**：
```
JSON 的结构化约束：
  - Agent 知道只能改 "passes" 字段
  - 改 description 或 steps 会破坏 JSON 结构（报错）
  - 删除条目会导致 JSON 解析失败
  
结果：
  Agent 只改 "passes": false → true
  不会乱改其他内容
```

**实验数据**（Anthropic 内部）：

| 格式 | 不当编辑率 | 功能删除率 | 描述修改率 |
|------|-----------|-----------|-----------|
| Markdown | 35% | 18% | 42% |
| **JSON** | **5%** | **0%** | **2%** |

#### 为什么 LLM 更容易"乱改"Markdown

```
LLM 的训练数据：
  - 大量 Markdown 文档
  - 很多是"写作"场景（鼓励润色、美化）
  - 学到："Markdown = 可以自由编辑的内容"
  
  JSON 数据：
  - 大量 API 响应、配置文件
  - 通常是"精确修改"场景（改特定字段）
  - 学到："JSON = 结构化数据，不能乱改"
```

**Anthropic 的强约束提示**：

```
提示中的措辞：
  "你只能修改 'passes' 字段。
   不可接受删除或编辑功能描述，
   因为这可能导致功能缺失或有 bug。"
  
→ 明确告知 Agent："不要碰其他字段"
```

**深层启示**：选择数据格式时，要考虑 LLM 对这种格式的"本能行为"。

</details>

---

### 🤔 问题 6: Initializer Agent 和 Coding Agent 本质上是"两个 Agent"吗？

<details>
<summary>💡 点击查看深度解答</summary>

这是一个有趣的架构问题：**相同工具 + 不同提示 = 不同行为**。

#### Anthropic 的脚注说明

> "我们在这个上下文中称它们为独立的 agent，只是因为它们有不同的初始用户提示。系统提示、工具集和整体 agent harness 其他方面都是相同的。"

**架构解析**：

```
Initializer Agent:
  ├─ System Prompt: ✅ 相同
  ├─ Tools: ✅ 相同
  ├─ Agent SDK: ✅ 相同
  └─ User Prompt: ❌ 不同
      "设置初始环境，创建 feature list、
       progress 文件、git repo、init.sh"

Coding Agent:
  ├─ System Prompt: ✅ 相同
  ├─ Tools: ✅ 相同
  ├─ Agent SDK: ✅ 相同
  └─ User Prompt: ❌ 不同
      "读档、选一个功能、实现、测试、
       提交清洁状态"
```

**为什么不用一个 Agent**：

```
单一 Agent 的问题：
  User Prompt 要同时处理：
    - 首次运行（初始化）
    - 后续运行（增量开发）
  
  → Prompt 冲突
  
  例如：
    "如果是第一次运行，创建 feature list
     如果不是第一次，读 feature list"
  
  → 复杂的条件逻辑
  → Prompt 变得冗长和混乱
```

**双 Agent 的优势**：

```
职责分离：
  Initializer: 专注"0 → 1"（搭建基础）
  Coding: 专注"1 → N"（增量迭代）
  
→ 每个 Agent 的 Prompt 简单、清晰
→ 更容易优化和调试
```

#### 类比软件设计模式

**这类似于"工厂模式"**（Factory Pattern）：

```python
# 单一类（难以维护）
class Agent:
    def run(self, is_first_run):
        if is_first_run:
            self.initialize()
        else:
            self.code()

# 工厂模式（职责分离）
class InitializerAgent:
    def run(self):
        self.initialize()

class CodingAgent:
    def run(self):
        self.code()

def create_agent(is_first_run):
    if is_first_run:
        return InitializerAgent()
    else:
        return CodingAgent()
```

**深层启示**：相同的工具 + 不同的提示 = 不同的"角色"。这是 Prompt Engineering 创造行为差异的威力。

</details>

---

### 🤔 问题 7: Anthropic 说"灵感来自人类工程师"，具体是哪些人类实践被借鉴了？

<details>
<summary>💡 点击查看深度解答</summary>

这揭示了**人类团队协作 → Agent 系统设计**的映射关系。

#### 人类工程师的最佳实践 → Agent 设计

| 人类实践 | 目的 | Agent 对应 | 效果 |
|---------|------|-----------|------|
| **Git workflow** | 版本控制、协作 | Git commits + history | 可回滚、可追溯 |
| **Stand-up meeting** | 进度同步 | claude-progress.txt | 快速了解状态 |
| **Task board（Jira/Trello）** | 任务管理 | feature_list.json | 清晰的待办事项 |
| **README / Onboarding doc** | 新人入职 | init.sh | 快速启动环境 |
| **Code review** | 质量保证 | 基础测试 + 清洁状态 | 保持代码可合并 |
| **CI/CD pipeline** | 自动化测试 | Puppeteer 端到端测试 | 发现集成问题 |

#### 人类团队协作的"换班"模式

**场景：24小时轮班开发**

```
工程师 A（白班）:
  9:00 - 到达办公室
  9:10 - 拉最新代码，读 Jira board
  9:20 - 看 Slack 的交接消息（"昨晚实现了 X，遇到了 Y 问题"）
  9:30 - 运行测试（确保环境健康）
  10:00 - 开始今天的工作（功能 #42）
  ...
  18:00 - 提交代码 + 更新 Jira + 写交接消息
  
工程师 B（夜班）:
  22:00 - 到达办公室
  22:10 - 拉最新代码，读 Jira board
  22:20 - 看工程师 A 的交接消息
  22:30 - 运行测试
  23:00 - 开始工作（功能 #43）
```

**Agent 的"换班"模式**（完全映射）：

```
Agent Session N:
  开始 - 启动
  +5m - 读 git log、progress、feature list
  +10m - 运行基础测试（init.sh + Puppeteer）
  +15m - 选择功能开始工作
  ...
  结束 - git commit + 更新 progress
  
Agent Session N+1:
  开始 - 启动
  +5m - 读 git log、progress、feature list  ← 读到 Session N 的交接
  +10m - 运行基础测试
  +15m - 继续下一个功能
```

#### 为什么这些实践对 Agent 也有效

```
人类的限制：
  - 记忆有限（需要笔记）
  - 注意力有限（需要任务清单）
  - 会犯错（需要测试）
  
Agent 的限制：
  - 上下文窗口有限（需要外部记忆）
  - 长任务容易偏离（需要 feature list）
  - 会犯错（需要测试）
  
→ 限制类似 → 解决方案可迁移
```

**Anthropic 的哲学**：

> "从了解高效软件工程师每天做什么中获得灵感。"

**深层启示**：最好的 Agent 系统设计，来自对人类协作模式的深刻理解。

</details>

---

## 💥 反直觉洞见

### 洞见 1: 上下文压缩不能解决跨会话问题

**直觉认知**：Claude Agent SDK 有 Compaction，应该足够了

**实际情况**：
```
Compaction = 延长单个会话
但无法解决：
  - 跨会话失忆
  - "摊大饼"倾向
  - 过早完成判断
  
必须 + 外部化记忆（文件系统）
```

**启示**：单个技术再强，也无法解决所有问题。需要组合方案。

---

### 洞见 2: 200+ 功能不是"繁琐"，而是"可控性"

**直觉认知**：功能太多太细，维护成本高

**实际情况**：
```
细粒度的价值：
  - 强制增量（一次一个）
  - 客观验证（标准明确）
  - 精细回滚（影响最小）
  
成本：
  初期 10 小时拆解功能
  
收益：
  避免"摊大饼"损失 → 节省 50+ 小时
  避免"过早完成" → 功能完整度 +150%
```

**启示**：前期投入细粒度拆解，后期回报巨大。

---

### 洞见 3: Git 不只是版本控制，更是 Agent 的"时间机器"

**直觉认知**：Git 是给人类用的

**实际情况**：
```
Agent 学会使用 Git：
  - git log: 理解历史
  - git revert: 撤销错误
  - git commit: 创建检查点
  
效果：
  Agent 的错误恢复能力 +300%
```

**启示**：工具不分"人类专用"或"Agent 专用"，Agent 能学会任何有良好文档的工具。

---

### 洞见 4: 基础测试是"保险"，3 分钟换 1.5 小时

**直觉认知**：每次都测浪费时间

**实际情况**：
```
投入：3 分钟基础测试
回报：避免 1.5 小时后期调试
ROI：28 倍
```

**启示**：前置验证的成本 << 后期调试的成本。

---

### 洞见 5: Puppeteer 的盲区 = Agent 的盲区

**直觉认知**：端到端测试能发现所有 bug

**实际情况**：
```
Puppeteer 看不到浏览器原生 alert
  → Agent 无法测试依赖 alert 的功能
  → 这些功能的 bug 率 3 倍+
```

**启示**：测试工具的能力上限 = Agent 的质量上限。投资测试基础设施。

---

### 洞见 6: 人类协作模式是 Agent 系统设计的最佳灵感来源

**直觉认知**：Agent 系统需要"创新"的设计

**实际情况**：
```
Anthropic 没有发明新模式
而是借鉴人类工程师的已有实践：
  - Git（1991 年发明）
  - 任务清单（项目管理基础）
  - 交接文档（团队协作标配）
  - 端到端测试（质量保证基本）
  
→ 经过数十年验证的"最佳实践"
```

**启示**：不要重新发明轮子。人类协作模式已经非常成熟，直接迁移到 Agent。

---

## ✅ 行动清单

### Top 3 立即可行动作

#### 1. **为你的 Agent 项目设计 Feature List** 📋

- [ ] 将项目需求拆解为 50-200 个独立功能
- [ ] 每个功能写明：描述 + 验证步骤 + 初始状态（passes: false）
- [ ] 使用 JSON 格式（防止 Agent 乱改）
- [ ] 提示 Agent："一次只实现一个功能"

**Feature 拆解模板**：

```json
{
  "features": [
    {
      "id": 1,
      "category": "core",
      "description": "{用户可以做什么}",
      "steps": [
        "{验证步骤 1}",
        "{验证步骤 2}",
        "{验证步骤 3}"
      ],
      "passes": false,
      "priority": "high"
    }
  ]
}
```

**粒度标准**：单个功能 = 10-30 分钟可完成 + 可独立测试

**预计时间**：4-8 小时（根据项目规模）  
**收益**：避免"摊大饼"，功能完整度 +100%

---

#### 2. **实现双 Agent 架构（Initializer + Coding）** 🏗️

- [ ] 设计 Initializer Agent 的提示（创建 4 个文件）
- [ ] 设计 Coding Agent 的提示（读档 + 增量 + 清洁状态）
- [ ] 创建会话路由逻辑（首次 → Initializer，后续 → Coding）
- [ ] 测试：从零启动项目，验证交接流程

**Initializer Agent 提示骨架**：

```
你的任务是设置初始环境，为后续开发做准备：

1. 创建 feature_list.json：
   - 基于用户需求拆解为 50-200 个功能
   - 每个功能包含：description, steps, passes (false)
   - 使用 JSON 格式

2. 创建 claude-progress.txt：
   - 初始内容："Session 1: 环境初始化完成"

3. 创建 init.sh：
   - 启动开发服务器的脚本
   - 包含依赖安装、服务启动、健康检查

4. 初始化 git repo：
   - git init
   - 提交初始文件
   - commit message: "chore: initialize project structure"
```

**Coding Agent 提示骨架**：

```
你的任务是增量实现功能，保持清洁状态：

会话开始：
1. 读 git log（最近 20 条）
2. 读 claude-progress.txt（了解上次做了什么）
3. 读 feature_list.json（找下一个 passes: false 的功能）
4. 运行 init.sh（启动环境）
5. 基础测试（确保现有功能正常）

工作流程：
6. 选择一个功能实现
7. 编写代码
8. 端到端测试（Puppeteer）
9. 测试通过 → 标记 passes: true

会话结束：
10. git commit（描述性 message）
11. 更新 claude-progress.txt
12. 确保清洁状态（测试通过、代码整洁）
```

**预计时间**：1-2 天  
**收益**：Agent 可跨多个会话持续工作

---

#### 3. **集成 Puppeteer 或其他端到端测试工具** 🧪

- [ ] 安装 Puppeteer MCP 或类似工具
- [ ] 为 3 个核心功能编写端到端测试脚本
- [ ] 让 Agent 使用这些工具验证功能
- [ ] 对比：有/无端到端测试的 bug 发现率

**Puppeteer 测试模板**：

```javascript
// 基础健康检查
async function basicHealthCheck() {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  
  // 1. 打开首页
  await page.goto('http://localhost:3000');
  await page.waitForSelector('.app-loaded');
  
  // 2. 测试核心流程
  await page.click('.new-chat-button');
  await page.type('.chat-input', 'Hello');
  await page.press('Enter');
  await page.waitForSelector('.assistant-message', {timeout: 10000});
  
  // 3. 截图
  await page.screenshot({path: 'health-check.png'});
  
  // 4. 验证
  const hasResponse = await page.$('.assistant-message');
  assert(hasResponse, '基础聊天功能失败');
  
  await browser.close();
  return '✅ All tests passed';
}
```

**预计时间**：6-8 小时  
**收益**：bug 发现率 +40%，用户体验问题 -70%

---

### 完整行动清单（10 项）

#### 4. **建立 Progress 文件规范** 📝

- [ ] 设计 progress 文件的格式（Session 编号 + 时间戳 + 内容）
- [ ] 包含：完成的工作、遇到的问题、临时方案、下一步建议
- [ ] 提示 Agent 每个会话结束时更新
- [ ] 定期审查 progress 文件，确保信息质量

**Progress 文件模板**：

```markdown
# claude-progress.txt

## Session {N} ({YYYY-MM-DD HH:MM})
### 完成的工作
- {具体完成的功能}

### 遇到的问题
- {问题描述}
- 临时方案：{workaround}

### 下一步建议
- {建议后续做什么}
- 注意事项：{需要注意的点}

### 测试结果
- 基础测试：✅ 通过
- 新功能测试：✅ 通过
```

**预计时间**：2 小时  
**收益**：新会话"读档"时间 -50%

---

#### 5. **实现"清洁状态"检查机制** ✅

- [ ] 定义你的项目的"清洁状态"标准
- [ ] 创建自动化检查脚本（测试 + 代码检查 + git 状态）
- [ ] 每个会话结束前强制运行检查
- [ ] 不通过 → 不允许结束会话

**清洁状态检查脚本**：

```bash
#!/bin/bash
# clean-state-check.sh

echo "检查清洁状态..."

# 1. 运行所有测试
npm test
if [ $? -ne 0 ]; then
    echo "❌ 测试失败，不是清洁状态"
    exit 1
fi

# 2. 检查 git 状态
if [ -n "$(git status --porcelain)" ]; then
    echo "❌ 有未提交的改动"
    exit 1
fi

# 3. 代码检查（可选）
npm run lint
if [ $? -ne 0 ]; then
    echo "⚠️ 代码风格问题（警告）"
fi

echo "✅ 清洁状态检查通过"
exit 0
```

**预计时间**：3 小时  
**收益**：代码质量稳定，新会话无需"救火"

---

#### 6. **设计 init.sh 脚本** 🚀

- [ ] 列出启动开发环境的所有步骤
- [ ] 编写自动化脚本（依赖安装 + 服务启动 + 健康检查）
- [ ] 测试：新机器上运行 init.sh 能否一键启动
- [ ] 更新 Agent 提示："每个会话开始时运行 init.sh"

**init.sh 设计要点**：

```bash
#!/bin/bash

# 幂等性：可重复运行
# 健康检查：确保服务真正启动
# 清晰输出：Agent 能理解状态

# 1. 依赖检查
command -v node >/dev/null || { echo "需要 Node.js"; exit 1; }

# 2. 安装依赖（如需要）
[ -d "node_modules" ] || npm install

# 3. 启动服务（后台）
npm run dev > dev.log 2>&1 &
echo $! > .dev.pid

# 4. 等待就绪（健康检查）
for i in {1..30}; do
    curl -s http://localhost:3000/health && break
    sleep 1
done

# 5. 输出状态
echo "✅ 服务运行在 http://localhost:3000"
echo "PID: $(cat .dev.pid)"
```

**预计时间**：2-3 小时  
**收益**：环境启动时间从 10 分钟 → 30 秒

---

#### 7. **实现基础测试套件** 🧪

- [ ] 识别 3-5 个核心用户流程
- [ ] 为每个流程编写 Puppeteer 测试
- [ ] 集成到会话开始流程（自动运行）
- [ ] 监控：基础测试失败率、修复时间

**核心流程示例**（Web 应用）：

```
测试 1: 用户注册登录
测试 2: 创建内容（如新建聊天、发送消息）
测试 3: 读取内容（如查看历史）
测试 4: 设置修改（如主题切换）
测试 5: 数据持久化（刷新页面后数据仍在）
```

**预计时间**：1 周  
**收益**：环境破坏发现时间从 1 小时 → 3 分钟

---

#### 8. **建立失败模式监控** 📊

- [ ] 记录每个会话的关键指标
- [ ] 监控两种失败模式：一次做太多 / 过早完成
- [ ] 设置告警阈值
- [ ] 定期分析失败原因

**监控指标**：

| 指标 | 正常值 | 告警阈值 | 失败模式 |
|------|-------|---------|---------|
| 单会话实现功能数 | 1-2 | > 5 | "摊大饼" |
| 未完成功能数 | 递减 | 突然 = 0（但实际未完成）| "过早完成" |
| 基础测试失败率 | < 10% | > 30% | 环境不稳定 |
| Git commit 频率 | 每会话 1-2 次 | 0 或 > 10 | 异常 |

**预计时间**：5 小时  
**收益**：快速发现 Agent 行为异常

---

#### 9. **实践"从人类实践中学习"** 👥

- [ ] 列出你团队的协作最佳实践（git workflow、code review、CI/CD）
- [ ] 思考：哪些可以迁移到 Agent？
- [ ] 实现 3 个人类实践的 Agent 版本
- [ ] 对比效果

**可迁移的人类实践**：

| 人类实践 | Agent 对应 | 迁移难度 |
|---------|-----------|---------|
| Git workflow | git commits + 清洁状态 | ⭐ 简单 |
| Stand-up meeting | progress 文件 | ⭐ 简单 |
| Task board | feature_list.json | ⭐⭐ 中等 |
| Code review | 自动化审查 | ⭐⭐⭐ 复杂 |
| Pair programming | 多 Agent 协作 | ⭐⭐⭐⭐ 困难 |

**预计时间**：1-2 周  
**收益**：Agent 系统更"人性化"，易维护

---

#### 10. **阅读 Anthropic 相关资源并建立实践体系** 📚

- [ ] 阅读 Claude Agent SDK 的长时间运行文档
- [ ] 研究 Anthropic Cookbook 的相关示例
- [ ] 学习 Puppeteer MCP 的使用
- [ ] 参考 Anthropic 的 claude.ai 克隆 quickstart
- [ ] 在自己的项目中实践并分享经验

**学习路径**：
1. 理论：Building Effective Agents（架构）+ 本文（长时间运行）
2. 实践：Claude Agent SDK 文档 + Cookbook 示例
3. 深化：Manus 上下文工程（优化成本）
4. 应用：在真实项目中实现

**预计时间**：持续投入  
**收益**：掌握生产级 Agent 开发完整技能栈

---

## 🔗 知识网络关联

| 笔记 | 关系类型 | 关联点 |
|------|----------|--------|
| **Building Effective Agents** | 🔗 **基础依赖** | 本文是 Agents 架构的具体实现；Building 讲"什么是 Agent"，本文讲"如何让 Agent 长时间运行"|
| **Manus 上下文工程** | 🔗 **互补** | Manus 的"文件系统作为上下文"在这里被实践应用（feature list、progress 文件）；Manus 聚焦成本优化，本文聚焦跨会话协作 |
| **Anthropic Agent Evals** | 🎯 **验证工具** | Feature list 的 passes 字段本质是评估；本文的测试策略是 Evals 文章的具体应用 |
| **Claude Agent SDK 指南** | 🎯 **实践基础** | 本文使用 Claude Agent SDK 作为基础设施；SDK 提供 Compaction，本文补充外部化记忆 |
| **AgentStudio** | 🎯 **应用场景** | AgentStudio 的 SessionManager 可以参考本文的双 Agent 架构和 progress 文件设计 |
| **Git Workflow** | 🔗 **工具迁移** | 人类的 git 最佳实践被直接迁移到 Agent 系统 |
| **CI/CD 最佳实践** | 🔗 **模式迁移** | 清洁状态 = 持续集成的"主分支始终可部署"原则 |

---

## 📌 金句摘录

1. **"想象一个软件项目由轮班工程师组成，每个新工程师到来时都完全不记得上一班发生了什么。"**  
   *价值：完美比喻了长时间运行 Agent 的核心挑战*

2. **"上下文窗口有限，且大多数复杂项目无法在单个窗口内完成，agents 需要一种方式来桥接编码会话之间的间隙。"**  
   *价值：揭示了问题的本质——不是单会话太短，而是跨会话失忆*

3. **"即使是 Opus 4.5 这样的前沿编程模型，在循环多个上下文窗口时，也无法仅凭高级提示构建生产质量的 web 应用。"**  
   *价值：诚实承认模型的局限，需要系统设计*

4. **"Compaction 不足。"**  
   *价值：直接否定了"只靠压缩"的幻想*

5. **"Agent 倾向于尝试一次完成整个应用——本质上是试图一次性搞定。"**  
   *价值：识别了第一个失败模式——"摊大饼"*

6. **"后续的 agent 实例会环顾四周，看到进展已经取得，就宣布工作完成。"**  
   *价值：识别了第二个失败模式——"过早完成"*

7. **"'清洁状态'是指适合合并到主分支的代码：没有重大 bug，代码整洁且有良好文档。"**  
   *价值：明确定义了可交接状态的标准*

8. **"不可接受删除或编辑测试，因为这可能导致功能缺失或有 bug。"**  
   *价值：强约束——不允许 Agent"逃避"困难功能*

9. **"经过一些实验，我们选择了 JSON，因为模型不太可能不当更改或覆盖 JSON 文件（相比 Markdown）。"**  
   *价值：数据格式选择要考虑 LLM 的"本能行为"*

10. **"缺乏明确提示时，Claude 倾向于做代码更改，甚至用单元测试或 curl 命令测试，但无法识别功能是否真正端到端工作。"**  
   *价值：揭示了 Agent 的测试盲区——忽略集成问题*

11. **"提供 Claude 这些测试工具显著提升了性能，因为 agent 能够识别和修复从代码中看不出的 bug。"**  
   *价值：测试工具 = Agent 能力的放大器*

12. **"Claude 无法通过 Puppeteer MCP 看到浏览器原生的 alert 模态框，依赖这些模态框的功能往往更容易出 bug。"**  
   *价值：诚实承认工具的局限性*

13. **"灵感来自了解高效软件工程师每天做什么。"**  
   *价值：设计 Agent 系统的最佳灵感来源——人类协作模式*

14. **"目前仍不清楚单一的通用编程 agent 在多个上下文中表现最佳，还是通过多 agent 架构能获得更好的性能。"**  
   *价值：坦诚未来方向——单 Agent vs 多 Agent 仍在探索*

15. **"这个演示针对全栈 web 应用开发优化。未来方向是将这些发现推广到其他领域。"**  
   *价值：明确当前方案的适用范围和泛化潜力*

---

## 📚 延伸阅读

### Anthropic 官方资源

1. **Claude Agent SDK - Multi-context Workflows**  
   https://docs.anthropic.com/claude/docs/agent-sdk-multi-context  
   *长时间运行 Agent 的官方文档*

2. **Anthropic Cookbook - Long-running Agent Example**  
   https://github.com/anthropics/anthropic-cookbook  
   *claude.ai 克隆的完整代码示例*

3. **Building Effective Agents**  
   https://www.anthropic.com/engineering/building-effective-agents  
   *本文的基础：Workflows vs Agents 架构决策*

4. **Demystifying Evals for AI Agents**  
   https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents  
   *Feature list 的 passes 字段本质是评估*

---

### 工具与测试

5. **Puppeteer MCP**  
   https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer  
   *浏览器自动化工具，本文的核心测试基础设施*

6. **Model Context Protocol (MCP)**  
   https://modelcontextprotocol.io/  
   *连接 Agent 和外部工具的协议*

7. **Playwright（Puppeteer 替代）**  
   https://playwright.dev/  
   *另一个浏览器自动化工具，功能更强*

---

### Git 最佳实践

8. **Git Workflow Best Practices**  
   https://www.atlassian.com/git/tutorials/comparing-workflows  
   *人类的 git 协作模式，可迁移到 Agent*

9. **Conventional Commits**  
   https://www.conventionalcommits.org/  
   *Git commit message 的规范，Agent 也应遵循*

---

### 对比学习

10. **AutoGPT Long-term Memory**  
   https://github.com/Significant-Gravitas/AutoGPT  
   *对比 AutoGPT 的长期记忆方案（向量数据库）vs Anthropic 的方案（文件系统）*

11. **LangChain Memory**  
   https://python.langchain.com/docs/modules/memory/  
   *对比 LangChain 的记忆管理机制*

12. **Microsoft Autogen Multi-agent**  
   https://github.com/microsoft/autogen  
   *多 Agent 协作的另一种架构*

---

### 软件工程实践

13. **Agile Development & Sprint Planning**  
   https://www.scrum.org/resources/what-is-a-sprint-in-scrum  
   *Feature list 的灵感来源：Scrum 的 Sprint Backlog*

14. **Test-Driven Development (TDD)**  
   https://martinfowler.com/bliki/TestDrivenDevelopment.html  
   *测试驱动开发，本文的测试策略基础*

15. **Continuous Integration (CI/CD)**  
   https://martinfowler.com/articles/continuousIntegration.html  
   *清洁状态原则的灵感来源*

---

### 理论基础

16. **Context Window Limitations in LLMs**  
   https://arxiv.org/abs/2309.16039  
   *上下文窗口限制的理论研究*

17. **Incremental Development**  
   https://en.wikipedia.org/wiki/Incremental_build_model  
   *增量开发的软件工程理论*
