# WeTERM 后端基础设施精读笔记

> 精读时间：2026-01-22
> 覆盖文档：03-server (3篇) + 04-gateway (5篇) + 05-protocol (2篇) + 06-appendix (2篇)

---

## 根节点命题

> **通过分层的服务端架构 + 统一的 AI 网关 + 双协议扩展（ACP + MCP），为远程 AI Agent 提供可靠、可扩展、可观测的后端支撑**

---

## 1. 服务端层 (03-server)

### 核心组件

| 组件 | 职责 | 关键特性 |
|------|------|----------|
| **Prompt Manager** | 管理所有 Prompt 模板 | 版本控制 + A/B 测试 + 热更新 |
| **Config** | 特性开关 + 安全规则 | Feature Flags + 黑名单 |
| **Conversation** | 对话历史存储 | 上下文持久化 |
| **MCP Server** | 工具扩展协议 | Model Context Protocol |

### 配置结构

```yaml
# 功能开关
ai_features:
  command_generation: true
  error_diagnosis: true
  observation_ai: true

# 安全配置
command_generation:
  blacklist:
    - "rm -rf /"
    - ":(){ :|:& };:"    # Fork 炸弹
  require_confirmation:
    - "rm -rf"
    - "sudo rm"

# Command Block 配置
command_block:
  max_blocks: 50
  max_output_length: 100000
  recent_blocks_for_ai: 10
```

---

## 2. AI Gateway (04-ai-gateway)

### 核心能力

| 能力 | 说明 |
|------|------|
| **统一接入** | OpenAI 兼容的 API 接口 |
| **智能路由** | 成本优先 / 性能优先 / 质量优先 |
| **故障转移** | 主渠道失败自动切换备用 |
| **配额管理** | Token 级别 + 用户级别 |
| **用量统计** | 完整的请求追踪和使用记录 |

### 支持的提供商

| 类型 | 名称 |
|------|------|
| 1 | BKAIDEV（内部） |
| 2 | OpenAI |
| 3 | Anthropic |
| 4 | Azure |
| 5 | Gemini |
| 10 | DeepSeek |
| 99 | Custom（自定义兼容接口）|

### 路由策略

| 策略 | 适用场景 |
|------|---------|
| 成本优先 | 简单命令生成 → 轻量模型 |
| 质量优先 | 复杂任务规划 → 强大模型 |
| 性能优先 | 实时补全 → 响应最快模型 |

### 技术栈

- API 框架：Go Gin
- 数据存储：MySQL / Redis
- 可观测：OpenTelemetry

---

## 3. 协议层 (05-protocol)

### ACP (Agent Client Protocol)

**定位**：WeTERM 与本地 Agent（Claude Code、gemini-cli）的通信协议

**基于**：JSON-RPC 2.0 + stdin/stdout

**核心消息类型**：

| 方法 | 方向 | 说明 |
|------|------|------|
| `agent/sendMessage` | Client → Agent | 发送用户消息 |
| `agent/streamChunk` | Agent → Client | 流式响应片段 |
| `agent/toolCall` | Agent → Client | 请求执行工具 |
| `agent/taskComplete` | Agent → Client | 任务完成通知 |

**启动流程**：

1. WeTerm 启动 CLI Agent 进程（`--acp-mode`）
2. Agent 初始化就绪
3. WeTerm 发送 `agent/sendMessage`
4. Agent 流式返回 `agent/streamChunk`
5. Agent 发送 `agent/taskComplete`

### MCP (Model Context Protocol)

**定位**：Agent 与工具的通信协议

**内置工具**：

| 工具 | 说明 | 参数 |
|------|------|------|
| `bash` | 执行 Shell 命令 | `command` |
| `read_file` | 读取文件内容 | `path` |
| `write_file` | 写入文件 | `path`, `content` |
| `list_dir` | 列出目录 | `path` |
| `search` | 搜索文件内容 | `pattern`, `path` |

### ACP vs MCP

| 协议 | 定位 | 方向 |
|------|------|------|
| **ACP** | Agent Client Protocol | WeTERM ↔ 本地 Agent |
| **MCP** | Model Context Protocol | Agent ↔ 工具 |

---

## 4. 实施路线图 (06-appendix)

### 时间线

```
2026 年
├── Q1 (1-3月)：命令生成 + 错误诊断
├── Q2 (4-6月)：命令补全建议
├── Q3 (7-9月)：TUI 状态识别 (vim & k9s)
└── Q4 (10-12月)：任务自动化

2027 年
├── Q1-Q2 (1-6月)：旁路观测 + ACP + 多 Agent 协同
└── 6月：v3.0 上线
```

### 团队配置

| 角色 | 人数 | 职责 |
|------|------|------|
| 前端开发 | 1 | weterm.js、GUI、ACP Client |
| 后端开发 | 1 | Server、Gateway、Prompt、AI 逻辑 |

### 版本规划

| 版本 | 时间 | 核心功能 |
|------|------|----------|
| **v1.0** | 2026 Q1-Q2 | Bootstrap + 命令生成 + 错误诊断 |
| **v2.0** | 2026 Q3-Q4 | vim TUI + 任务自动化 |
| **v3.0** | 2027 Q1-Q2 | Observation AI + ACP + 多 Agent |

### 关键风险

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|---------|
| 远程 Shell 不兼容 | 中 | 高 | 支持 Zsh/Bash，提供降级方案 |
| TUI 识别难度高 | 高 | 中 | 仅支持 vim，k9s 后续迭代 |
| 第三方 Agent 无 ACP 支持 | 中 | 高 | Q4 提前验证，准备 Adapter |

---

## 5. AI 能力评估基准

### 评估维度

| 评估项 | 权重 | 说明 |
|-------|------|------|
| 命令准确率 | 40% | 生成命令的正确性 |
| 上下文理解 | 25% | 对环境的感知能力 |
| 安全性 | 20% | 危险命令识别 |
| 响应速度 | 15% | 生成延迟 |

### 测试用例规模

| 类别 | 数量 |
|------|------|
| 文件操作 | 50 |
| Git 操作 | 30 |
| 系统管理 | 40 |
| 网络操作 | 20 |
| 文本处理 | 30 |
| **合计** | **170+** |

### 长期目标

| 指标 | 目标 |
|------|------|
| 命令生成准确率 | > 85% |
| 命令生成延迟 | < 2s |
| 实时补全延迟 | < 500ms |
| 危险命令拦截率 | 100% |
| 系统可用性 | > 99.9% |

---

## 核心洞见

### 设计亮点

1. **AI Gateway 统一抽象**：一套 API 适配所有 LLM 提供商，故障自动转移
2. **双协议扩展**：ACP（Agent 通信）+ MCP（工具调用）分层清晰
3. **渐进式演进**：2 人团队，18 个月完成三期目标
4. **安全第一**：危险命令黑名单 + 确认机制 + 100% 拦截率目标

### 关键约束

1. **零部署**：远程环境无需安装二进制，全部通过 Shell 内置命令
2. **2 人团队**：前后端各 1 人，需要高度复用和优先级管理
3. **多 Shell 兼容**：Zsh/Bash 优先，其他 Shell 降级处理

---

## 思考题

### Q1: 为什么需要 AI Gateway？

**核心价值**：
- 统一接口：一套 API 适配所有 LLM
- 故障转移：主渠道挂了自动切换
- 成本控制：Token 配额 + 智能路由
- 可观测性：所有调用有迹可循
- 安全隔离：API Key 不暴露给前端

### Q2: ACP 和 MCP 的关系？

- ACP 是「对话协议」：定义 Agent 之间怎么交流
- MCP 是「工具协议」：定义 Agent 怎么调用工具

第三期目标：WeTERM → ACP → Claude Code → MCP → 工具 → 执行结果
