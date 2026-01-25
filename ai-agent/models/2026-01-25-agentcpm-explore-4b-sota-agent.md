# AgentCPM-Explore 精读笔记：4B 参数如何超越 30B 级 Agent

> **项目**：[OpenBMB/AgentCPM](https://github.com/OpenBMB/AgentCPM)
> **团队**：清华大学 THUNLP + 中国人民大学 + 面壁智能 + OpenBMB
> **日期**：2026-01-25

---

## 快速概览

| 维度 | 内容 |
|------|------|
| **模型基础** | Qwen3-4B-Thinking-2507 深度后训练 |
| **核心价值** | 4B 参数在 8 个高难度智能体任务上达到 SOTA，比肩 30B 级模型 |
| **开源内容** | 模型权重 + AgentDock + AgentRL + AgentToLeaP 全套基础设施 |
| **协议** | Apache-2.0 |

---

## 根节点命题

> **端侧智能体的性能瓶颈不在参数量，而在「持续深度探索」能力——4B 模型通过 100+ 轮交互和类人思考逻辑，可以解决 95% 以上的复杂任务。**

---

## 问题本质

### 传统小模型的 Agent 困境

```
传统小模型问题
├── 短视：只能做 1-2 步规划，无法处理长程任务
├── 脆弱：工具调用失败后无法恢复
├── 死记硬背：依赖训练数据模式匹配，缺乏真正推理
└── 结果：GAIA 这类任务准确率 < 30%
```

### AgentCPM 的突破

```
AgentCPM 解法
├── 100+ 轮持续交互：不放弃，持续探索直到完成
├── 多源交叉验证：质疑工具返回，追求原始数据
├── 动态策略调整：一条路走不通，换另一条
└── 结果：GAIA 95%+ 可解决（允许重试）
```

---

## 三层开源基础设施

```
AgentCPM 开源生态
│
├── 🔧 AgentDock（工具层）
│   ├── 统一 MCP 协议接口
│   ├── Docker 容器化部署
│   │   ├── agentdock-manager (8080)
│   │   ├── agentdock-mongodb (27017)
│   │   ├── agentdock-node-full (8004)
│   │   └── agentdock-node-explore (8014)
│   └── 一键启动：docker compose up -d
│
├── 🎓 AgentRL（训练层）
│   ├── 多轮交互训练：支持 Agent 训练中多轮工具调用
│   ├── 全异步训练-采样解耦
│   ├── 前缀合并加速：复用共享上下文
│   └── 支持算法：MINIRL / GRPO
│
└── 📊 AgentToLeaP（评测层）
    └── 8 个基准测试一键评测
        ├── GAIA：通用 AI 助手
        ├── HLE：人类级别评估
        ├── BrowseComp：网页理解
        ├── Frames：框架理解
        ├── WebWalkerQA：网页导航
        ├── Seal-0：安全评估
        ├── xbench-DeepSearch：深度搜索
        └── BrowseComp-ZH：中文网页
```

---

## 核心技术解析

### 1. 100+ 轮持续交互

**问题**：为什么大多数小模型只能做 3-5 轮工具调用？

**原因**：
- 上下文溢出：每轮交互增加上下文长度，小模型窗口有限
- 策略坍塌：多轮后模型开始重复之前动作，陷入死循环

**AgentCPM 解法**：
- 前缀合并加速减少冗余计算
- 训练时模拟多轮交互，让模型学会「如何不重复」

### 2. 类人思考逻辑

**五种类人特征**：

| 特征 | 说明 | 示例行为 |
|------|------|----------|
| 质疑工具 | 不盲信返回结果 | 搜索结果不完整时，换关键词重搜 |
| 追求原始数据 | 不满足于二手信息 | 找到引用来源，去看原始文档 |
| 灵活调整策略 | 一条路走不通就换 | 网页打不开，尝试其他来源 |
| 执着寻找信源 | 不轻易放弃 | 多次尝试不同搜索方式 |
| 多源验证 | 交叉确认信息 | 用多个独立来源验证答案 |

### 3. MCP 协议统一工具调用

**价值**：
- 统一接口：所有工具用同一种方式调用
- 容器化部署：工具作为独立服务运行
- 热插拔：新增工具不需要改训练代码

```python
# 添加自定义工具只需 3 步
# 1. 创建 MCP 服务
@server.list_tools()
async def list_tools():
    return [Tool(name="my_tool", description="...")]

# 2. 注册到配置
[mcpServers.my_custom_tool]
command = "python"
args = ["mcp_servers/my_tool/server.py"]

# 3. 重启服务
docker compose restart agentdock-node-explore
```

### 4. 全异步训练-采样解耦

**Agent 训练的特殊性**：
- 采样耗时长：一个任务可能需要 100 轮工具调用，每轮都有网络延迟
- 工具不稳定：网页可能打不开，API 可能超时
- 资源异构：采样主要是 IO 密集，训练主要是计算密集

**解耦架构**：
```
┌─────────────┐     MongoDB     ┌─────────────┐
│  采样集群    │ ───────────→   │  训练集群    │
│ (IO 密集)   │    任务队列     │ (计算密集)   │
└─────────────┘                └─────────────┘
```

---

## 性能数据

| 测试集 | AgentCPM-4B | MiroThinker-8B | Tongyi-30B | Claude-3.5-Sonnet |
|--------|-------------|----------------|------------|-------------------|
| GAIA (text) | 63.9% | 66.4% | 70.9% | 71.2% |
| BrowseComp | 24.1% | 31.1% | 43.4% | 19.6% |
| HLE | 19.1% | 21.5% | 32.9% | 24.5% |
| Frames | **82.7%** | 80.6% | 90.6% | 85.0% |
| WebWalkerQA | **68.1%** | 60.6% | 72.2% | / |
| xbench-Deep | **70.0%** | 60.6% | 75.0% | 66.0% |

**关键洞见**：
- 4B 在 Frames、WebWalkerQA、xbench-DeepSearch 上超越 8B 模型
- xbench-DeepResearch 上表现优于 Claude-3.5-Sonnet（70% vs 66%）

---

## 可迁移洞见

### 对 Agent 开发的启示

| 洞见 | 说明 |
|------|------|
| **参数不是瓶颈** | 4B 足够，关键是训练方法和工具调用能力 |
| **持续探索 > 单次准确** | 允许重试，95% 的问题都能解决 |
| **工具标准化至关重要** | MCP 协议让工具生态可扩展 |
| **训练采样解耦** | Agent 训练的必然趋势 |

### 对端侧部署的启示

| 场景 | 策略 |
|------|------|
| 手机/边缘设备 | 4B 模型 + 本地工具沙盒，实现本地 AI 助手 |
| 企业私有化 | AgentCPM-Report (8B) 可比肩 Gemini Deep Research |
| 二次开发 | 全套基础设施开源，支持自定义训练 |

---

## 快速体验

```bash
# 1. 克隆项目
git clone https://github.com/OpenBMB/AgentCPM.git
cd AgentCPM/AgentCPM-Explore

# 2. 启动工具沙盒
cd AgentDock && cp .env.example .env
docker compose up -d

# 3. 拉取评测环境
docker pull yuyangfu/agenttoleap-eval:v2.0
docker run -dit --name agenttoleap --gpus all --network host \
  -v $(pwd):/workspace yuyangfu/agenttoleap-eval:v2.0
docker exec -it agenttoleap /bin/bash

# 4. 配置并运行
python quickstart.py

# 5. 查看结果
cat outputs/quickstart_results/dialog.json
```

---

## 延伸思考

1. **端侧 vs 云端权衡**：4B 模型本地运行 vs API 调用大模型，哪种方案更实用？
2. **工具生态建设**：MCP 协议能否成为 Agent 工具调用的事实标准？
3. **训练数据来源**：如何获取高质量的 100+ 轮交互数据用于训练？
4. **安全边界**：Agent 具备「持续探索」能力后，如何防止越权行为？
