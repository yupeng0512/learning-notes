---
title: Pathways - Google's New ML System
source: https://www.bodunhu.com/blog/posts/pathways-googles-new-ml-system/
author: Bodun Hu
date: 2026-02-05
category: ai-research
subcategory: deep-research
tags: [ML系统, 分布式系统, Pathways, Google, 控制器架构, 死锁, Gang-Scheduling]
---

# Pathways: Google's New ML System

> 📖 原文：[Pathways: Google's New ML System](https://www.bodunhu.com/blog/posts/pathways-googles-new-ml-system/)
> 📅 学习日期：2026-02-05
> 🏷️ 分类：AI Research / Deep Research

---

## 根节点命题

> **ML系统控制器架构的本质矛盾：通信拓扑的对称性决定控制模式**

**根节点公式**：

```
控制器数量 = f(通信模式对称性, 设备调度粒度)

• 对称通信 (All-Reduce/All-Gather) → 多控制器 (SPMD) ✅ 高效
• 非对称通信 (Pipeline, Point-to-Point) → 单控制器 (MPMD) ✅ 必需
• 非抢占式硬件 (TPU) → Gang-Scheduling ✅ 防死锁
```

**为什么这是根节点**：

文章所有论述都在回答"为什么Pathways要回到单控制器"。答案归结为一个底层约束：

**当设备执行非对称工作负载 (MPMD) 且硬件不支持抢占时 (TPU单线程)，集中式调度是避免死锁的最优解。**

TensorFlow v1 → JAX → Pathways 的演进，本质是在"通信效率"与"调度灵活性"之间寻找最优平衡点。

---

## 表示空间

> 描述ML系统架构需要哪几个核心维度？

| 维度 | 含义 | 本文位置 |
|------|------|----------|
| **控制器数量** | 单点调度 (集中) vs 多点协作 (分布) | TF1单控 → JAX多控 → Pathways回归单控 |
| **执行模式** | SPMD (对称程序) vs MPMD (非对称程序) | All-Reduce是SPMD → Pipeline是MPMD |
| **硬件特性** | 可抢占 (GPU多流) vs 非抢占 (TPU单线程FIFO) | TPU单线程强制需要Gang-Scheduling |
| **通信拓扑** | 对称 (点对点等价) vs 非对称 (流水线依赖) | 对称用多控 → 非对称用单控 |
| **延迟来源** | DCN跨数据中心 vs PCIe/NVLink本地 | 单控走DCN慢 → 多控走PCIe快 |

---

## 推论展开

> 从根节点推导出的核心结论（树状结构）

```
根节点：通信拓扑对称性 → 决定控制器架构
│
├─ 推论1：多控制器优势来自"对称性假设"
│   ├─ SPMD要求所有worker执行相同代码逻辑
│   ├─ All-Reduce/All-Gather是天然对称通信
│   └─ 应用：数据并行训练 (每个GPU处理不同batch但执行相同操作)
│
├─ 推论2：Pipeline Parallelism打破对称性
│   ├─ 不同stage执行不同计算 (conv → encoding → decoding)
│   ├─ 不同batch在不同stage流转 (时间+空间维度非对称)
│   ├─ Point-to-Point通信无法用SPMD单一代码描述
│   └─ 应用：大模型分层部署 (Transformer各层分散到不同节点)
│
├─ 推论3：非抢占式硬件强化死锁风险
│   ├─ TPU是单线程FIFO队列 (任务入队后无法抢占)
│   ├─ 如果设备A等待B的任务X，但B先入队了任务Y等待A
│   │   → 形成循环等待 → 死锁
│   ├─ GPU多CUDA流可部分缓解 (但NCCL通信仍可能死锁)
│   └─ 应用：NCCL通信死锁排查 (检查通信算子入队顺序)
│
├─ 推论4：Gang-Scheduling是单控的核心价值
│   ├─ 集中调度器全局可见所有设备状态
│   ├─ 保证相互依赖的任务以一致顺序入队
│   ├─ 消除"互相等待"的循环依赖条件
│   └─ 应用：分布式训练调度系统设计 (K8s Job调度器)
│
└─ 推论5：架构选择是"效率-灵活性"权衡
    ├─ 多控制器：牺牲灵活性换通信效率 (适合数据并行)
    ├─ 单控制器：牺牲DCN开销换调度灵活性 (适合Pipeline/MoE)
    └─ 应用：根据训练范式选择框架 (数据并行用JAX，模型并行用Pathways)
```

---

## 泛化模式

> 这个洞见可以迁移到哪些其他场景？

| 原场景 | 迁移场景 | 如何应用 |
|--------|----------|----------|
| ML训练调度 | **微服务编排** | 当服务间依赖非对称时（如支付流→库存→物流），需要集中式编排器（如Temporal/Cadence）而非各自独立决策 |
| TPU死锁问题 | **数据库事务管理** | 非抢占式锁机制下，必须用全局死锁检测器或Two-Phase Locking保证事务顺序一致性 |
| Gang-Scheduling | **容器调度 (K8s Gang Scheduling)** | 多Pod必须同时启动的场景（如MPI Job、Spark Executor），需要 `schedulingGates` 或 Volcano 调度器 |
| SPMD vs MPMD | **并行计算范式** | MapReduce是SPMD（所有worker执行相同map/reduce逻辑），但Dataflow系统（如Flink）支持MPMD（不同算子异构执行） |
| DCN通信开销 | **跨区域系统设计** | 跨可用区通信代价高时，优先设计本地自治架构（如多写数据库的本地优先写入） |

---

## 关键概念

> 只保留理解根节点必需的概念

| 概念 | 定义 | 与根节点的关系 |
|------|------|----------------|
| **SPMD** | Single Program Multiple Data - 所有worker执行相同代码，处理不同数据 | 多控制器的前提假设，要求通信对称性 |
| **MPMD** | Multiple Program Multiple Data - 不同worker执行不同代码 | Pipeline并行导致需要MPMD，打破多控假设 |
| **Gang-Scheduling** | 一组相互依赖的任务必须同时调度，保证入队顺序全局一致 | 单控制器的核心价值，防止死锁 |
| **Data Center Network (DCN)** | 跨机架/数据中心的网络，延迟远高于PCIe/NVLink | 单控制器的主要性能瓶颈（控制消息走DCN） |
| **非抢占式执行** | 任务入队后无法被打断，必须执行完才能处理下一个 | TPU单线程特性，强化了死锁风险 |
| **Pipeline Parallelism** | 将模型分层部署到不同设备，数据像流水线流转 | 典型的MPMD场景，需要非对称通信 |
| **All-Reduce** | 分布式归约操作，所有节点对等参与通信 | 典型的对称通信，适合多控制器 |
| **Deadlock四要素** | 互斥、持有并等待、非抢占、循环等待 | TPU单线程FIFO满足所有条件，需Gang-Scheduling打破循环等待 |

## 推论深度展开

### 推论1：多控制器优势来自"对称性假设"

**Q1：为什么多控制器系统在数据并行场景下性能更好？**

<details>
<summary>点击展开答案</summary>

核心原因是**消除了DCN通信瓶颈**：

1. **TensorFlow v1 的控制流**：
   - Client定义图 → 提交给Master
   - Master编译 → 分割子图 → 下发给Worker
   - 每个step都需要跨DCN通信（橙色线）
   - Worker空闲时间长（等待控制消息）

2. **JAX/PyTorch 多控的改进**：
   - 每个Worker自己就是入口（无需中心Master）
   - 控制逻辑本地决策（通过PCIe/NVLink通信）
   - 设备间通信走高速互联（黑色虚线）
   - 消除了lock-step等待所有worker完成的开销

**关键洞察**：多控不是"更分布式"，而是"把控制权下放到靠近计算的地方"。

</details>

---

**Q2：SPMD对称性假设隐含了什么限制？**

<details>
<summary>点击展开答案</summary>

**隐含约束**：所有worker必须执行"结构相同但参数不同"的代码。

```python
# SPMD 示例 (All-Reduce)
# 所有 worker 执行相同逻辑，只是 data 不同
for worker_id in range(num_workers):
    local_gradient = compute_gradient(local_data[worker_id])
    all_reduce(local_gradient)  # 对称通信
```

**为什么 Pipeline 打破这个假设**：

```python
# MPMD 示例 (Pipeline Parallelism)
if worker_id == 0:
    stage1_output = conv_layer(input)
    send_to(worker_1, stage1_output)
elif worker_id == 1:
    stage1_output = recv_from(worker_0)
    stage2_output = encoding_layer(stage1_output)
    send_to(worker_2, stage2_output)
elif worker_id == 2:
    stage2_output = recv_from(worker_1)
    final_output = decoding_layer(stage2_output)
```

**无法用单一SPMD代码描述**：worker 0、1、2 执行完全不同的函数，通信也是非对称的点对点。

</details>

---

### 推论2：Pipeline Parallelism打破对称性

**Q3：为什么说Pipeline Parallelism是"时间+空间"双重非对称？**

<details>
<summary>点击展开答案</summary>

**空间非对称**：不同stage执行不同计算
- Stage 1: Convolution
- Stage 2: Encoding
- Stage 3: Decoding

**时间非对称**：同一时刻不同stage处理不同batch

```
Time   | Stage 1   | Stage 2   | Stage 3
-------|-----------|-----------|----------
t=1    | Batch 1   | Idle      | Idle
t=2    | Batch 2   | Batch 1   | Idle
t=3    | Batch 3   | Batch 2   | Batch 1
```

**关键问题**：如何用单一程序描述"Stage 1处理Batch 3的同时，Stage 2处理Batch 2"？

答案是**无法用SPMD描述**，必须有中心调度器协调谁在何时处理哪个batch。

</details>

---

**Q4：为什么作者说"multi-controller doesn't fit into this kind of workload"？**

<details>
<summary>点击展开答案</summary>

多控制器依赖**对等协商**，但Pipeline需要**异构协调**。

**假设用MPI风格实现Pipeline**：

```python
# 伪代码：试图用多进程实现 Pipeline
if rank == 0:
    output = stage1(input)
    MPI.Send(output, dest=1)
elif rank == 1:
    data = MPI.Recv(source=0)
    output = stage2(data)
    MPI.Send(output, dest=2)
```

**问题1**：如何处理多batch流水？每个进程需要维护复杂的状态机（当前处理batch X，同时等待batch X+1）

**问题2**：如何决定何时发送/接收？（没有全局视角，容易死锁）

**单控优势**：Master知道全局状态，可以统一编排"Rank 0发送Batch 1给Rank 1，同时Rank 0开始Batch 2"。

</details>

---

### 推论3：非抢占式硬件强化死锁风险

**Q5：TPU为什么比GPU更容易死锁？**

<details>
<summary>点击展开答案</summary>

**TPU的致命限制**：单线程 + 非抢占式

```
TPU = 单个FIFO队列
│
├─ Task A 入队 → 正在执行
├─ Task B 等待 Task A 完成
└─ 无法插队执行 Task C（即使Task C不依赖Task A）
```

**GPU的缓解机制**：多CUDA Stream

```
GPU = 多个并发队列
│
├─ Stream 1: Task A 执行中
├─ Stream 2: Task C 可以并发执行
└─ 即使 Task A 阻塞，Task C 仍可进行
```

**死锁示例**：

```
设备1: [Task A] → 等待设备2的Task A
设备2: [Task B] → 等待设备1的Task B

TPU: 死锁！（两个队列头部互相等待）
GPU: 可能避免（如果Task A/B在不同Stream，可能并发）
```

**但注意**：文章提到"NCCL通信仍可能死锁"——即使GPU有多流，如果通信算子本身是阻塞的，依然会死锁。

</details>

---

**Q6：死锁的四个必要条件在TPU场景下如何满足？**

<details>
<summary>点击展开答案</summary>

经典死锁四要素：

| 条件 | TPU场景 |
|------|---------|
| **互斥 (Mutual Exclusion)** | 通信资源（网络带宽、接收buffer）同一时间只能被一个任务占用 |
| **持有并等待 (Hold and Wait)** | Task A持有设备1的执行权，同时等待设备2的消息 |
| **非抢占 (No Preemption)** | **TPU单线程FIFO，任务入队后无法被打断** ← 核心问题 |
| **循环等待 (Circular Wait)** | 设备1等设备2，设备2等设备3，设备3等设备1 |

**单控+Gang-Scheduling打破哪个条件？**

**打破"循环等待"**：通过全局调度保证任务入队顺序一致，避免形成环路依赖。

</details>

---

### 推论4：Gang-Scheduling是单控的核心价值

**Q7：Gang-Scheduling如何防止死锁？**

<details>
<summary>点击展开答案</summary>

**关键思想**：相互依赖的任务必须"原子性"地同时入队到所有设备。

**无Gang-Scheduling的死锁场景**：

```
时刻1：
  设备1先入队 Task A（等待设备2）
  设备2先入队 Task B（等待设备1）
  → 循环等待 → 死锁
```

**有Gang-Scheduling的正确调度**：

```
Master协调：
  1. 等待所有设备就绪
  2. 同时向设备1和设备2发送Task A
  3. 保证Task A在两个设备上都排在Task B之前
  → 无循环等待 → 无死锁
```

**类比操作系统的Two-Phase Locking**：事务先获取所有锁再执行，避免持有部分锁时等待其他锁。

</details>

---

**Q8：为什么说"centralized coordinator"是Gang-Scheduling的前提？**

<details>
<summary>点击展开答案</summary>

**核心问题**：如何让分布式设备"同步"入队顺序？

**方案1：分布式协商**（多控尝试）
- 每个设备自己决定任务顺序
- 通过消息传递协商全局一致性
- **问题**：网络延迟导致时序不确定，容易出现race condition

**方案2：中心化调度**（单控Pathways）
- Master维护全局任务依赖图
- Master统一下发任务到所有设备
- **优势**：全局可见性 + 确定性顺序

**Trade-off**：牺牲了DCN通信开销，换取了调度正确性。

</details>

---

### 推论5：架构选择是"效率-灵活性"权衡

**Q9：为什么不能"两全其美"——既用多控又支持Pipeline？**

<details>
<summary>点击展开答案</summary>

**根本矛盾**：多控依赖SPMD的对称性假设，而Pipeline天然是MPMD。

**理论上的"混合方案"**：
- 在Pipeline各stage内部用多控（例如每个stage内数据并行）
- Stage之间用单控协调

**实际实现挑战**：
1. **复杂度爆炸**：需要两套调度逻辑，边界情况难处理
2. **性能开销**：频繁切换控制模式引入额外开销
3. **调试困难**：死锁/性能问题更难定位

**Pathways的选择**：专注做好单控，通过优化DCN通信（如使用RDMA）缓解性能问题。

</details>

---

## 反直觉洞见

### 1. "回到单控"不是倒退，而是螺旋上升

**常识误区**：分布式系统演进应该是"越来越分布化"。

**反直觉真相**：架构演进是"根据约束条件选择最优解"，而非单向进化。

- TF1单控 → 适应当时硬件（通信慢、计算简单）
- JAX多控 → 适应数据并行（对称通信占主导）
- Pathways回归单控 → 适应模型并行（非对称通信+TPU约束）

**启示**：不存在"银弹架构"，只有"适合当前约束的架构"。

---

### 2. GPU的多流并非万能死锁解药

**常识误区**：GPU有多CUDA Stream，所以不会死锁。

**反直觉真相**：NCCL通信层仍是单线程阻塞，依然可能死锁。

**原因**：
- CUDA Stream只解决计算并发
- 但NCCL的`AllReduce`等通信原语是阻塞调用
- 如果两个Stream同时发起相互依赖的通信 → 依然死锁

**工程建议**：PyTorch DDP使用单Stream避免此问题，但用户自定义通信时需注意。

---

### 3. DCN慢不是单控的本质问题

**常识误区**：单控慢是因为控制消息走DCN。

**反直觉真相**：单控的价值在于"全局可见性"，DCN开销是可优化的代价。

**优化方向**：
- 使用RDMA绕过TCP/IP栈
- 控制消息批处理减少RTT
- 预调度减少实时控制需求

**类比**：微服务的API Gateway也引入延迟，但提供了统一鉴权/路由的价值。

---

### 4. Pipeline不仅是模型并行，更是"时间维度的并行"

**常识误区**：Pipeline就是把模型切分到多个设备。

**反直觉真相**：Pipeline的核心是"让不同设备在同一时刻处理不同batch"。

**时间并行的本质**：
```
传统串行: [Batch1 Stage1→2→3] → [Batch2 Stage1→2→3]
Pipeline:  Batch1 Stage1 → Batch2 Stage1 → Batch3 Stage1
              ↓                ↓                ↓
           Batch1 Stage2 → Batch2 Stage2
              ↓                ↓
           Batch1 Stage3
```

**启示**：Pipeline是"空间切分+时间流水"的组合优化，而非单纯的空间切分。

## 行动清单

### 立即可行动作 (Top 3)

1. **排查分布式训练代码的通信顺序**
   - [ ] 检查现有PyTorch/JAX训练代码中是否存在非对称通信
   - [ ] 验证NCCL通信是否在同一CUDA Stream内（避免死锁）
   - [ ] 记录通信算子调用顺序，确认是否存在循环依赖

2. **评估训练范式与框架的匹配度**
   - [ ] 分析当前模型是数据并行主导还是模型并行主导
   - [ ] 数据并行 → 优先JAX/PyTorch DDP（多控优势）
   - [ ] 模型并行/Pipeline → 考虑支持集中调度的框架

3. **学习Gang-Scheduling在K8s中的应用**
   - [ ] 阅读Volcano调度器文档（K8s的Gang-Scheduling实现）
   - [ ] 实验：创建需要多Pod同时启动的Job（如MPI训练）
   - [ ] 对比默认调度器与Volcano在资源碎片化时的表现

### 进阶行动 (4-10)

4. **实验：复现TPU死锁场景**
   - [ ] 在TPU模拟器上构造两设备互相等待的通信模式
   - [ ] 对比GPU多Stream下的表现差异
   - [ ] 记录死锁发生的临界条件

5. **深入研究Pathways论文原文**
   - [ ] 阅读Google Pathways官方论文（2022）
   - [ ] 关注：Sharding API、资源分配策略、编译优化
   - [ ] 对比Pathways与DeepSpeed/Megatron的架构差异

6. **优化现有单控系统的DCN开销**
   - [ ] 如使用TensorFlow，启用gRPC压缩减少控制消息体积
   - [ ] 评估RDMA网络是否可用（绕过TCP/IP栈）
   - [ ] 批量提交任务减少RTT次数

7. **分析开源框架的死锁防护机制**
   - [ ] 阅读PyTorch DDP源码：为何单Stream能避免通信死锁
   - [ ] 阅读NCCL源码：通信原语的阻塞语义
   - [ ] 总结最佳实践文档

8. **将Gang-Scheduling应用到微服务编排**
   - [ ] 调研Temporal/Cadence的Saga模式（分布式事务编排）
   - [ ] 设计实验：对比无编排器的服务调用链 vs Workflow编排
   - [ ] 记录在异步场景下编排器的价值

9. **学习并行计算理论基础**
   - [ ] 阅读教材：《分布式系统原理与范型》（死锁章节）
   - [ ] 学习Two-Phase Locking、Wait-Die、Wound-Wait算法
   - [ ] 对比数据库与ML系统的死锁场景异同

10. **撰写技术博客：ML系统架构演进**
    - [ ] 总结TF1 → JAX → Pathways的架构权衡
    - [ ] 绘制决策树：如何根据场景选择单控/多控
    - [ ] 分享给团队，收集实际生产环境的反馈

---

## 知识网络关联

### 互补关系

| 相关笔记 | 关联类型 | 连接点 |
|----------|----------|--------|
| `programming/2026-01-04-celery-task-scheduling-design.md` | 互补 | 任务调度系统设计 - Celery的分布式调度与Pathways的Gang-Scheduling可类比 |
| `ai-tools/agent-architecture/*` | 互补 | Agent架构中的记忆/调度系统 - 可借鉴单控/多控的权衡思路 |

### 对比关系

**Pathways (单控) vs JAX (多控)**
- 适用场景：模型并行 vs 数据并行
- 通信模式：MPMD vs SPMD
- 死锁风险：通过Gang-Scheduling防护 vs 通过对称性避免

### 深化方向

- **分布式系统理论** → 学习死锁检测算法（如银行家算法）
- **操作系统调度** → 了解进程调度中的Gang-Scheduling历史
- **ML框架源码** → 深入PyTorch/JAX的调度器实现

### 应用场景

- **大模型训练** → 选择合适的并行策略和框架
- **K8s Job调度** → 配置Volcano处理MPI/Spark等并行任务
- **微服务架构** → 决策何时使用编排器（如Temporal）

---

## 金句摘录

1. "Gang-scheduling is essential in the case of TPUs, since they are single-threaded and only run non-preemptible kernels, so the system will deadlock if communicating computations are not enqueued in a consistent order."

2. "We can think of a computing device as a FIFO task queue (e,g. CUDA streams, TPU, or CPU…)."

3. "This is a classical example of deadlock in operating systems."

4. "Using gang-scheduling helps preventing deadlocks, because it enforces a global enqueueing order across multiple FIFO queues."

5. "If each devices allows concurrency executions (each device has multiple queues), then the task on one queue can be preemptied to allow the other task start executing, thus no deadlock."

6. "Dispatching computations in a single-controller system requires communnication across (data center network) DCN."

7. "Under multi-controller systems, each worker shares the same code and executes different stage/branch of the code. This is why they are called SPMD systems (single-program-multiple-data)."

8. "Obviously, multi-controller doesn't fit into this kind of workload [Pipeline Parallelism]. How do you write a single copy of code that does all these irregular communications under multi-process scenarios?"

9. "Pathways proposes we should go back to single-controller, so that we can let the master node handle all these nasty communication patterns."

10. "Allowing device (e.g. GPUs) to execute tasks concurrently can prevent deadlocks. This is because concurrency eliminates the non-preemption property which is required for deadlocks to happen."

---

## 延伸阅读

### 论文

- **Pathways原始论文** (Google, 2022)  
  https://arxiv.org/abs/2203.12533

- **OneFlow系统论文** (Jinhui Yuan)  
  https://arxiv.org/abs/2110.15032

### 博客

- **Jinhui Yan的Pathways解读**（中文）  
  https://zhuanlan.zhihu.com/p/450656463  
  *(本文参考的核心资源，讲解更清晰)*

- **PyTorch DDP死锁排查指南**  
  https://pytorch.org/docs/stable/notes/ddp.html

### 源码

- **NCCL通信库源码**  
  https://github.com/NVIDIA/nccl

- **Volcano调度器（K8s Gang-Scheduling）**  
  https://github.com/volcano-sh/volcano

### 书籍

- 《分布式系统原理与范型》（Tanenbaum）- 死锁检测章节
- 《Operating System Concepts》（恐龙书）- 进程同步与死锁

### 相关技术

- **Megatron-LM** - NVIDIA的模型并行训练框架
- **DeepSpeed** - Microsoft的分布式训练框架
- **Ray** - 分布式Python框架（类似Pathways的调度理念）

## 个人思考

---
