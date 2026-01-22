---
title: MemOS - AI 记忆操作系统
source: https://github.com/MemTensor/MemOS
author: MemTensor 团队
date: 2026-01-22
category: ai-tools
subcategory: agent-architecture
tags: [AI-Memory, Agent-Infrastructure, RAG-Alternative, LLM]
type: practical-project
---

# MemOS：为 AI Agent 装上海马体

> 项目：[MemOS](https://github.com/MemTensor/MemOS)
> 学习日期：2026-01-22
> 分类：ai-tools / agent-architecture
> 版本：v2.0（3700+ Stars）

---

## 快速上手实践

> **实践类项目学习的关键**：先跑起来，再深入原理

### 1. 安装方式

**推荐方式（云服务）**：
```python
import os
import json

os.environ["MEMOS_API_KEY"] = "your_key"
os.environ["MEMOS_BASE_URL"] = "https://memos.memtensor.cn/api/openmem/v1"
os.environ["KNOWLEDGE_BASE_IDS"] = json.dumps(["your_kb_id"])
```

**其他平台**：
| 平台 | 安装命令 |
|------|----------|
| 本地部署 | `git clone https://github.com/MemTensor/MemOS.git && pip install -e .` |
| Docker | `docker-compose up -d` |

### 2. 基础使用流程

```
┌─────────────────────────────────────────────────────────┐
│                  MemOS 核心使用流程                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   1. 配置 API Key / 启动本地服务                          │
│              │                                          │
│              ▼                                          │
│   2. store() ──────────────────────────────────────┐    │
│      存储记忆（user_id, type, content）              │    │
│              │                                      │    │
│              ▼                                      │    │
│   3. recall() ─────────────────────────────────────┤    │
│      检索记忆（user_id, query）                     │    │
│              │                                      │    │
│              ▼                                      │    │
│   4. 记忆自动调度（热/温/冷/归档）                    │    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 3. 核心功能速览

| 功能 | 操作方式 | 说明 |
|------|----------|------|
| 存储记忆 | `memos.store(user_id, type, content)` | 支持多种记忆类型 |
| 检索记忆 | `memos.recall(user_id, query)` | 语义检索相关记忆 |
| 添加消息 | `add_message(user_id, role, content)` | 自动提取记忆 |
| 搜索记忆 | `search_memory(user_id, query)` | 精确搜索 |
| **知识库管理** | `kb.create() / kb.delete()` | v2.0 新增：知识库级别管理 |
| **记忆反馈** | `feedback(memory_id, action)` | v2.0 新增：反馈和修正记忆 |
| **精确删除** | `delete(memory_id)` | v2.0 新增：删除指定记忆 |

### 3.1 v2.0 新增：MCP 协议支持

MemOS v2.0 原生支持 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)，可通过 MCP 进行记忆操作：

```python
# MCP 工具调用示例
mcp_tools = [
    "memos_add_memory",      # 添加记忆
    "memos_search_memory",   # 搜索记忆
    "memos_delete_memory",   # 删除记忆（v2.0）
    "memos_feedback_memory"  # 记忆反馈（v2.0）
]
```

### 3.2 v2.0 新增：多模态记忆

| 类型 | 说明 |
|------|------|
| 文本记忆 | 传统对话内容 |
| **图像记忆** | 原生支持图像作为记忆存储 |
| **图表记忆** | 支持结构化图表数据 |

### 3.3 API 端点示例

**添加记忆**：
```bash
curl -X POST "https://memos.memtensor.cn/api/openmem/v1/memories/add" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_123",
    "content": "用户喜欢 Python 和分布式系统",
    "type": "preference"
  }'
```

**搜索记忆**：
```bash
curl -X POST "https://memos.memtensor.cn/api/openmem/v1/memories/search" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_123",
    "query": "用户的技术偏好",
    "top_k": 5
  }'
```

### 4. 配置文件

| 用途 | 路径 | 格式 |
|------|------|------|
| 环境变量 | `.env` 或环境变量 | KEY=VALUE |
| Docker 配置 | `docker-compose.yml` | YAML |
| 依赖配置 | `pyproject.toml` | TOML |

### 5. 常见问题

| 问题 | 解决方案 |
|------|----------|
| Neo4j 连接失败 | 检查 Docker 服务是否启动：`docker-compose ps` |
| API Key 无效 | 前往 https://memos-dashboard.openmem.net/ 获取 |
| 记忆检索为空 | 确认 user_id 一致，检查记忆是否已存储 |

### 6. 本地开发

**环境要求**：Python 3.10+、Docker、Neo4j/Qdrant/Redis/MySQL

```bash
git clone https://github.com/MemTensor/MemOS.git
cd MemOS
docker-compose up -d    # 启动依赖服务
pip install -e .        # 安装开发模式
pytest                  # 运行测试
```

---

## 性能基准

> **vs OpenAI Memory**（官方基准测试数据）

| 指标 | MemOS | OpenAI Memory | 提升 |
|------|-------|---------------|------|
| 记忆准确率 | 89.2% | 62.1% | **+43.70%** |
| Token 消耗 | 1,247 | 1,925 | **-35.24%** |
| 响应延迟 | 1.2s | 1.8s | -33.3% |

---

## 根节点命题

> **记忆 = 系统资源，需要操作系统级别的生命周期管理**

**为什么这是根节点**：MemOS 的所有设计决策——三层架构、四种记忆类型、调度系统、自动遗忘——都是从「把记忆当系统资源管理」这一核心理念推导出来的。就像操作系统管理 CPU/内存/磁盘一样，MemOS 管理记忆的创建、检索、更新、遗忘。

---

## 表示空间

> **描述 AI 记忆系统的核心维度**

| 维度 | 含义 | MemOS 的选择 |
|------|------|--------------|
| 记忆持久性 | 记忆保留多久 | 动态调度 + 自动遗忘（模拟人类遗忘曲线） |
| 记忆类型丰富度 | 支持几种记忆形态 | 4 种（文本/激活/参数/工具） |
| 状态管理 | 有状态 vs 无状态 | 完整生命周期状态管理 |
| 可控性 | 黑盒 vs 可编程 | 完全开源、API 可控 |

---

## 架构分析

### 技术栈

| 层级 | 技术选型 | 选择理由 |
|------|----------|----------|
| 存储层 | Neo4j/Qdrant/Redis/MySQL | 不同数据类型针对性优化 |
| 服务层 | FastAPI | 高性能异步 API |
| 调度层 | Redis Streams | 高并发多级队列 |
| 语言 | Python 3.10+ | AI 生态兼容 |

### 核心设计模式

**三层记忆架构**：
```
┌─────────────────────────────────────────────────────────────┐
│  Layer 3: Memory Operating System（调度/生命周期/多租户）     │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Memory Types（文本/激活/参数/工具）                 │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Storage Backend（Neo4j/Qdrant/Redis/MySQL）        │
└─────────────────────────────────────────────────────────────┘
```

**四种记忆类型**：
| 类型 | 存储内容 | 类比 |
|------|----------|------|
| Textual Memory | 对话内容 | 日记本 |
| Activation Memory | KV Cache | 工作记忆 |
| Parametric Memory | LoRA 权重 | 肌肉记忆 |
| Tool Memory | 工具调用轨迹 | 经验教训 |

---

## 推论展开

> **从根节点推导出的核心结论**

```
根节点：记忆 = 系统资源，需要操作系统级别管理
│
├─ 推论1：记忆需要分类型存储
│   └─ 实现：四种记忆类型（文本/激活/参数/工具），针对性优化
│
├─ 推论2：记忆需要生命周期管理
│   └─ 实现：热/温/冷/归档分层 + 自动遗忘机制
│
├─ 推论3：记忆需要智能调度
│   └─ 实现：重要度打分 + 访问频率衰减 + 动态激活
│
└─ 推论4：记忆需要隔离保护
    └─ 实现：多租户隔离，不同用户/Agent 记忆互不干扰
```

---

## 泛化模式

> **MemOS 的设计可以迁移到哪些其他场景？**

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| AI 记忆管理 | 用户行为系统 | 用户行为也需要生命周期管理：记录→分析→遗忘过时行为 |
| AI 记忆管理 | 推荐系统 | 兴趣记忆的热/温/冷分层，长期兴趣 vs 短期兴趣 |
| Tool Memory | 任何 Agent 系统 | 记录工具调用轨迹，让 Agent 从错误中学习 |
| 调度器设计 | 缓存系统 | Redis Streams + 重要度打分的通用模式 |

---

## 行动清单

### 即时行动（上手实践）

- [ ] 注册云服务账号，获取 API Key
- [ ] 运行示例代码，体验 store/recall 核心 API
- [ ] 用自己的 Agent 项目接入 MemOS 测试

### 深度学习（架构理解）

- [ ] 阅读三层架构源码，理解调度机制
- [ ] 本地部署完整服务，理解各组件协作
- [ ] 尝试自定义记忆类型或遗忘策略

---

## 知识关联

| 历史笔记 | 关系类型 | 关联说明 |
|----------|----------|----------|
| Context Engineering 系列 | 互补 | MemOS 解决 Context 窗口限制问题的另一种思路 |
| Agent Architecture 系列 | 深化 | MemOS 是 Agent 基础设施的重要组成部分 |

---

## 个人思考

{留空，供用户后续补充}

---

## 延伸阅读

- [MemOS GitHub](https://github.com/MemTensor/MemOS)
- [MemOS API 控制台](https://memos-dashboard.openmem.net/)
- [与 Claude Memory 的对比分析](#与-claude-memory-的定位对比)
