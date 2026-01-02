---
title: A Guide to Claude Code 2.0 and getting better at using coding agents
source: https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/
author: Sankalp
date: 2025-12-29
category: ai-tools
tags: [claude-code, coding-agents, context-engineering, subagents, ai-tools]
---

# Claude Code 2.0 完全使用指南

> 📖 原文：[A Guide to Claude Code 2.0](https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/)
> 📅 学习日期：2025-12-29
> 🏷️ 分类：AI 工具与效率

---

## 一句话总结

掌握**上下文工程**和**子代理系统**，是高效使用编程代理的关键。

---

## 核心要点

1. **上下文工程是关键**：有效窗口只有 50-60%，要主动管理
2. **善用子代理系统**：Explore 从零开始，Plan/General 继承上下文
3. **技能按需加载**：避免系统提示膨胀
4. **工具组合使用**：Claude 执行 + GPT 审查
5. **迭代优化**：采用"废弃初稿"方法，先实现再优化

---

## 关键概念

| 概念 | 定义 | 应用场景 |
|------|------|----------|
| 上下文窗口 | 模型能"看到"的文本范围（Opus 4.5 为 200K tokens） | 决定理解能力边界 |
| 上下文衰减 | 随着 token 增加，检索性能下降的现象 | 有效窗口只有 50-60% |
| 子代理系统 | 将任务分解给专门代理（Explore/Plan/General） | 提高效率，隔离上下文 |
| 技能系统 | 按需加载的领域专业知识 | 避免系统提示膨胀 |

---

## 实用技巧

| 技巧 | 操作方法 | 效果 |
|------|----------|------|
| 历史搜索 | `Ctrl+R` | 快速找到之前的命令 |
| 检查点回滚 | `Esc+Esc` 或 `/rewind` | 代码实验后回退 |
| 上下文压缩 | `/compact` | 达到 60% 使用率时压缩 |
| 深度思考 | `/ultrathink` | 复杂问题需要更多推理 |
| 技能加载 | Skills 系统 | 按需加载领域知识 |

---

## 子代理系统详解

| 代理 | 角色 | 特点 |
|------|------|------|
| **Explore** | 只读代码库搜索专家 | 从零开始，不继承上下文 |
| **Plan** | 软件架构师 | 用于实现规划，继承上下文 |
| **General** | 通用代理 | 处理复杂多步骤任务 |
| **claude-code-guide** | 文档查询代理 | 查询官方文档 |

---

## 上下文管理策略

```
┌─────────────────────────────────────┐
│  上下文使用率监控                    │
├─────────────────────────────────────┤
│  0-40%   ✅ 正常工作                │
│  40-60%  ⚠️ 考虑使用 /compact       │
│  60%+    🚨 进行上下文交接           │
└─────────────────────────────────────┘
```

---

## 行动清单

- [ ] 尝试 `Ctrl+R` 历史搜索功能
- [ ] 了解 `/rewind` 检查点功能
- [ ] 创建一个自定义命令，自动化常见任务
- [ ] 设置一个领域技能文件
- [ ] 监控上下文使用率，练习使用 `/compact`
- [ ] 尝试"废弃初稿"方法开发复杂功能

---

## 个人思考

{留空，供后续补充}

---

## 延伸阅读

- Claude Code 官方文档
- 上下文工程最佳实践
- AI 编程助手对比评测
