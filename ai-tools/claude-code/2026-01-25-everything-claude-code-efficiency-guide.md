# Everything Claude Code 精读笔记

> **根节点命题**：AI 编程助手的效率瓶颈不在模型能力，而在「配置体系」——通过专业分工的 Agent 团队 + 自动化 Hooks + 结构化工作流，把「实习生」变成「高级工程师团队」。

## 项目概览

| 维度 | 内容 |
|------|------|
| **项目名称** | Everything Claude Code |
| **作者** | Affaan Mustafa（Anthropic 黑客松获奖者） |
| **核心价值** | 10 个月高强度实战打磨的生产级 Claude Code 配置 |
| **开源内容** | 9 个专业 Agent、11 个 Skill、15 个 Command、8 条 Rule、完整 Hooks 系统 |
| **GitHub** | https://github.com/affaan-m/everything-claude-code |

## 问题本质

```
裸用 Claude Code 的问题
    │
    ├─→ 无分工：一个 AI 干所有事，质量参差不齐
    ├─→ 无记忆：关掉电脑，上下文全丢
    ├─→ 无规范：乱创建文件、留下 console.log、不写测试
    ├─→ 无流程：想到哪写到哪，缺乏系统性
    └─→ 结果：AI 只能当「实习生」级别的助手
```

## 架构全景图

```
Everything Claude Code 架构
│
├── 🤖 Agents（9 个专业子代理）
│   ├── planner        需求拆解，规划路径
│   ├── architect      系统设计，架构决策
│   ├── tdd-guide      测试驱动开发引导
│   ├── code-reviewer  代码质量审查
│   ├── security-reviewer  安全漏洞分析
│   ├── build-error-resolver  构建错误修复
│   ├── e2e-runner     Playwright E2E 测试
│   ├── refactor-cleaner  死代码清理
│   └── doc-updater    文档同步更新
│
├── 🎯 Skills（11 个工作流知识库）
│   ├── continuous-learning  自动从会话中提取可复用模式
│   ├── strategic-compact    智能上下文压缩建议
│   ├── tdd-workflow         TDD 方法论
│   ├── security-review      安全检查清单
│   └── ...
│
├── ⌨️ Commands（15 个快捷命令）
│   ├── /plan          自动出实现方案
│   ├── /tdd           测试驱动开发
│   ├── /code-review   代码审查
│   ├── /build-fix     修复构建错误
│   └── ...
│
├── 📏 Rules（8 条强制规则）
│   ├── security.md      禁止硬编码密钥
│   ├── coding-style.md  不可变性、文件组织
│   ├── testing.md       TDD、80% 覆盖率
│   └── ...
│
└── 🪝 Hooks（自动化触发器）
    ├── PreToolUse    阻止不规范行为
    ├── PostToolUse   自动格式化、类型检查
    ├── SessionStart  加载上次会话上下文
    └── SessionEnd    保存会话状态 + 提取模式
```

## 核心提效机制

### 机制 1：专业 Agent 分工

**核心洞见**：每个 Agent 有专门的 System Prompt 和工具限制

| Agent | 职责 | 模型 | 工具限制 |
|-------|------|------|----------|
| planner | 需求拆解、实现规划 | Opus | Read, Grep, Glob（只读） |
| architect | 架构设计、技术决策 | Opus | Read, Grep, Glob（只读） |
| tdd-guide | TDD 引导 | Sonnet | Read, Grep, Glob, Bash, Edit |
| code-reviewer | 质量审查 | Opus | Read, Grep, Glob（只读） |
| security-reviewer | 安全分析 | Opus | Read, Grep, Glob, Bash（检测） |

**关键原则**：
- **规划类 Agent 禁止编辑**：防止「想到哪写到哪」
- **审查类 Agent 用 Opus**：深度推理发现问题
- **执行类 Agent 用 Sonnet**：平衡成本和能力

### 机制 2：记忆持久化

**问题**：Claude Code 默认没有跨会话记忆

**解法**：SessionStart/SessionEnd Hooks

```
会话结束时
    ├── session-end.js → 保存：任务状态、打开的文件、当前进度
    └── evaluate-session.js → 提取：错误解决方案、用户纠正 → 保存为 Skill

新会话开始时
    └── session-start.js → 加载：最近 7 天的会话状态
```

### 机制 3：自动化质量门禁

**核心**：Hooks 是硬约束，Rules 是软约束

```json
// 阻止创建随意的 .md 文件
{
  "matcher": "tool == \"Write\" && file_path matches \"\\.md$\" && !(file_path matches \"README|CLAUDE\")",
  "hooks": [{
    "type": "command",
    "command": "echo '[Hook] BLOCKED: 禁止创建随意文档' && exit 1"
  }]
}

// 编辑后警告 console.log
{
  "matcher": "tool == \"Edit\" && file_path matches \"\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command", 
    "command": "grep -n 'console.log' \"$file_path\" && echo '[Hook] 提交前请移除 console.log'"
  }]
}
```

### 机制 4：战略性上下文压缩

**问题**：自动压缩可能在任务中间触发，丢失关键上下文

**战略压缩时机**：
- ✅ 规划完成后、执行前
- ✅ 一个里程碑完成后
- ✅ 调试结束后
- ❌ 任务进行中

```
每 50 次工具调用后
    └── suggest-compact.sh → 输出：建议 /compact
用户决定是否 /compact
    ├── 是 → PreCompact Hook 先保存状态
    └── 否 → 继续工作
```

### 机制 5：模型选择策略

| 模型 | 场景 | 成本 |
|------|------|------|
| Haiku 4.5 | 轻量代理、频繁调用、Worker Agent | 最低 |
| Sonnet 4.5 | 日常开发、代码生成、多数任务 | 中等 |
| Opus 4.5 | 架构决策、深度推理、复杂分析 | 最高 |

### 机制 6：持续学习系统

**原理**：不是「模型学习」，而是「上下文注入」

```yaml
patterns_to_detect:
  - error_resolution     # 错误是如何解决的
  - user_corrections     # 用户的纠正模式
  - workarounds          # 框架/库的变通方法
  - debugging_techniques # 有效的调试技巧
  - project_specific     # 项目特定约定
```

## 标准开发流程

```
/plan（Planner Agent）
    ├── 重述需求、识别风险、分阶段规划
    └── 等待确认 ← 关键：不确认不动代码
    ▼
/tdd（TDD Guide Agent）
    ├── 先写测试（RED）→ 实现代码（GREEN）→ 重构（IMPROVE）
    ▼
/code-review（Code Reviewer Agent）
    └── 质量检查、安全审查、性能评估
    ▼
/build-fix（如果构建失败）
    └── Build Error Resolver Agent 修复
```

## 上下文窗口管理

```
⚠️ 不要同时启用所有 MCP！

200K 上下文窗口 → 启用过多工具后可能缩至 70K

经验法则：
├── 配置 20-30 个 MCP
├── 每个项目启用 < 10 个
└── 活跃工具 < 80 个
```

## 可迁移的提效经验

| 技巧 | 说明 | 迁移方式 |
|------|------|----------|
| **规划先行** | /plan 后必须确认才能写代码 | 在 Prompt 中加「不确认不执行」 |
| **Agent 只读** | Planner/Architect 禁止 Edit | 限制工具权限 |
| **Hooks 门禁** | 阻止不规范行为 | 在 settings.json 配置 |
| **战略压缩** | 在逻辑边界压缩 | 里程碑后手动 /compact |
| **模型分级** | 按任务选模型 | 配置 Agent 的 model 字段 |
| **记忆持久化** | 保存/恢复会话状态 | SessionStart/End Hooks |

## 对我的启发

| 当前做法 | 可优化点 |
|----------|----------|
| 单 Agent | 拆分为 Planner + Executor |
| 无工具限制 | 规划类 Agent 只读 |
| 无 Hooks | 添加质量门禁 |
| 无记忆 | 添加 Session Hooks |
| 无 compact 策略 | 里程碑后提示压缩 |

## 延伸思考

1. **分工 vs 上下文成本**：多 Agent 意味着更多 Prompt 注入，如何平衡？
2. **记忆的有效期**：提取的模式可能过时，如何管理 learned skills 的生命周期？
3. **Hooks 的边界**：过多 Hooks 会降低开发速度，哪些值得加？
4. **本地化适配**：这套配置面向 Next.js/TypeScript，如何适配 Python/Go 项目？
