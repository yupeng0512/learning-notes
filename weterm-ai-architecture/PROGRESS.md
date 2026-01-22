# WeTERM AI 功能架构设计 - 精读进度

> 源文档：https://iwiki.woa.com/p/4017283697
> 开始时间：2026-01-22
> 状态：进行中

---

## 文档目录（共 23 篇）

### 00 架构总览
- [x] `4017283707` - 00-architecture-overview ✅

### 01 客户端 (Client)
- [x] `4017283708` - 01-client (目录页) ✅
- [ ] `4017283709` - weterm-js ⛔ 无权限
- [x] `4017283721` - command-block ✅
- [ ] `4017283731` - universal-input ⛔ 无权限
- [ ] `4017283737` - shell-init ⛔ 无权限

### 02 AI Agent
- [x] `4017283742` - 02-ai-agent (目录页) ✅
- [x] `4017283750` - overview ✅
- [x] `4017283751` - command-generation ✅
- [x] `4017283754` - error-diagnosis ✅
- [x] `4017283759` - observation-ai ✅
- [x] `4017283762` - tui-interaction ✅

### 03 服务端 (Server)
- [x] `4017283764` - overview ✅
- [x] `4017283797` - config-reference ✅
- [x] `4017283799` - prompt-management ✅

### 04 AI 网关 (AI Gateway)
- [x] `4017283804` - overview ✅
- [x] `4017283805` - channel-management ✅
- [x] `4017283807` - model-routing ✅
- [x] `4017283809` - quota-token ✅
- [x] `4017283811` - business-flows ✅

### 05 协议 (Protocols)
- [x] `4017283815` - acp ✅
- [x] `4017283817` - mcp-integration ✅

### 06 附录 (Appendix)
- [x] `4017283851` - 实施路线图 ✅
- [x] `4017283852` - tech-stack ✅
- [x] `4017283858` - evaluation ✅

### 其他
- [x] `4017283870` - PPT-大纲 ✅

---

## 精读记录

| 序号 | 文档 | 状态 | 笔记文件 | 完成时间 |
|------|------|------|----------|----------|
| 1 | 00-architecture-overview | ✅ 完成 | 00-architecture-overview.md | 2026-01-22 |
| 2 | 01-client (目录页) | ✅ 完成 | - | 2026-01-22 |
| 3 | command-block | ✅ 完成 | 01-command-block.md | 2026-01-22 |
| 4 | weterm-js | ⛔ 无权限 | - | - |
| 5 | universal-input | ⛔ 无权限 | - | - |
| 6 | shell-init | ⛔ 无权限 | - | - |
| 7 | 02-ai-agent (5篇合并) | ✅ 完成 | 02-ai-agent.md | 2026-01-22 |
| 8 | 03-server (3篇) | ✅ 完成 | 03-backend-infrastructure.md | 2026-01-22 |
| 9 | 04-gateway (5篇) | ✅ 完成 | 03-backend-infrastructure.md | 2026-01-22 |
| 10 | 05-protocol (2篇) | ✅ 完成 | 03-backend-infrastructure.md | 2026-01-22 |
| 11 | 06-appendix (3篇) | ✅ 完成 | 03-backend-infrastructure.md | 2026-01-22 |
| 12 | PPT-大纲 | ✅ 完成 | - | 2026-01-22 |

---

## 核心洞见汇总

（精读过程中逐步填充）

### #1 架构总览
- **根节点**：远程环境零依赖约束下，通过 Shell Hook + 客户端增强实现完整 AI Agent
- **关键设计**：四层分离（Client → Agent → Server → Gateway）、双协议扩展（ACP + MCP）、渐进式降级
- **演进路线**：终端助手 → 完整 Agent → 多 Agent 协同

### #2 Command Block
- **根节点**：通过 Shell Hook 捕获命令完整生命周期，为 AI Agent 提供结构化上下文
- **三阶段 Hook**：Preexec（命令）→ CommandFinished（退出码）→ Precmd（环境信息）
- **智能裁剪**：最近 10 条 + 所有失败命令，合并去重按时间排序
- **环境感知**：pwd/git_branch/virtual_env/node_version/kube_context

### #3 AI Agent 层（5 篇合并）
- **根节点**：远程零部署约束下，通过 Command Block 上下文 + Bash 工具集实现 AI Agent
- **五大能力**：命令生成 + 错误诊断 + 旁路观测 + TUI 交互 + 任务规划
- **演进路线**：终端助手(2026.06) → 完整 Agent(2026.12) → 多 Agent 协同(2027.06)
- **安全设计**：danger/warning/safe 三级分类 + 密码不自动填充
- **工具实现**：全部基于 Bash 标准命令（cat/grep/git/kubectl）

### #4 后端基础设施（12 篇合并）
- **根节点**：分层服务端 + 统一 AI 网关 + 双协议扩展，为 Agent 提供可靠后端支撑
- **服务端三组件**：Prompt Manager（版本控制+热更新）+ Config（特性开关+黑名单）+ Conversation（对话持久化）
- **AI Gateway**：统一接入 + 智能路由 + 故障转移 + 配额管理（支持 6+ 提供商）
- **双协议**：ACP（Agent 通信）+ MCP（工具调用）
- **路线图**：2 人团队 18 个月完成三期（v1.0→v2.0→v3.0）
- **评估基准**：命令准确率(40%) + 上下文理解(25%) + 安全性(20%) + 响应速度(15%)