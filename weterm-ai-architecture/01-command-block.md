# Command Block 精读笔记

> 精读时间：2026-01-22 | 源文档：4017283721

## 根节点命题

**通过 Shell Hook 捕获命令的完整生命周期，为 AI Agent 提供结构化的上下文信息**

Command Block 是 Agent 的「眼睛和记忆」——没有它，Agent 就是个瞎子。

---

## 核心设计

### 三阶段 Hook 机制

```
用户输入命令
    │
    ▼
┌─────────┐
│ Preexec │ ← 捕获：command（即将执行什么）
└────┬────┘
     │
     ▼  [命令执行中，捕获输出流]
     │
┌────────────────┐
│ CommandFinished │ ← 捕获：exit_code（成功还是失败）
└────────┬───────┘
         │
         ▼
┌────────┐
│ Precmd │ ← 捕获：环境信息（pwd/git/venv/node/k8s）
└────────┘
```

### 环境信息价值

| 字段 | 价值 |
|------|------|
| `pwd` | 知道用户在哪个目录操作 |
| `git_branch` | 知道在哪个分支，避免建议影响 main |
| `virtual_env` | 知道 Python 环境，避免包冲突建议 |
| `node_version` | 知道 Node 版本，避免语法兼容问题 |
| `kube_context` | 知道 K8s 集群，避免误操作生产环境 |

### 智能裁剪策略

```javascript
getContextForAI() {
  const recent = this.getRecentBlocks(10);  // 最近 10 条
  const failed = this.getFailedBlocks();     // 所有失败的
  // 合并去重，按时间排序
  return [...new Set([...recent, ...failed])]
    .sort((a, b) => a.startTime - b.startTime);
}
```

**设计意图**：
- 失败命令优先保留 → AI 需要诊断错误
- 限制数量 → 控制 Token 消耗
- 按时间排序 → 保持因果关系

---

## 数据结构

### CommandBlock

```typescript
interface CommandBlock {
  id: string;           // 格式: <hook_type>-<timestamp>-<sequence>
  command: string;
  exit_code: number;
  output: string;       // 最大 10KB，超过截断
  env: Environment;
  start_time: timestamp;
  end_time: timestamp;
}

interface Environment {
  pwd: string;
  git_branch: string;
  virtual_env: string;
  conda_env: string;
  node_version: string;
  kube_context: string;
}
```

### 存储配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `maxBlocks` | 50 | 最大存储 Block 数量 |
| `maxOutputLength` | 100KB | 单个 Block 最大输出长度 |
| `recentBlocksForAI` | 10 | 传递给 AI 的最近 Block 数量 |

---

## 状态机

```
[*] --> Pending: Preexec
Pending --> Running: 开始执行
Running --> Completed: CommandFinished (exit_code == 0)
Running --> Failed: CommandFinished (exit_code != 0)
Failed --> Diagnosed: AI 诊断
Completed/Diagnosed --> [*]: 归档
```

---

## 思考与洞见

### 为什么输出截断设置为 10KB？

**权衡**：
- Token 成本：10KB ≈ 2500 Token
- 信息密度：大部分错误信息在前几 KB
- 响应速度：更大上下文 = 更慢响应

**改进方向**：智能截断——保留头尾，中间省略

### 失败命令优先的代价

**问题**：大量失败命令会挤掉有价值的成功命令上下文

**改进**：
- 失败命令也做滚动窗口
- 按「诊断价值」评分

---

## 关联文档

- Shell Init：上游，注入 Hook
- AI Agent：下游，消费 Command Block
- 错误诊断：exit_code != 0 时触发
