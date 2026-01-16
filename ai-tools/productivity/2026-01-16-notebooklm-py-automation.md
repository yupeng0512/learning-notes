# notebooklm-py：Google NotebookLM 的命令行/API 工具

> 学习日期：2026-01-16
> 来源：GitHub - teng-lin/notebooklm-py
> 标签：#NotebookLM #自动化 #Python #CLI #RAG #知识管理

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **名称** | notebooklm-py |
| **作者** | teng-lin（社区开发者） |
| **类型** | 开源工具库（Python + CLI） |
| **核心主题** | 将 Google NotebookLM 从网页界面解放到命令行和代码中 |
| **目标读者** | 需要自动化知识管理的开发者、研究者、内容创作者 |
| **许可证** | MIT License |

> **一句话总结**：这是一把打开 NotebookLM「后门」的钥匙——让你用代码和命令行驾驭 Google 最强的 RAG 引擎。

---

## 🗺️ 项目架构

```
┌─────────────────────────────────────────────────────────────┐
│                    notebooklm-py 架构                        │
├─────────────────────────────────────────────────────────────┤
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐    │
│   │ Claude Code │  │   CLI 命令  │  │   Python API    │    │
│   │   Skills    │  │   notebooklm│  │ NotebookLMClient│    │
│   └──────┬──────┘  └──────┬──────┘  └────────┬────────┘    │
│          │                │                   │             │
│          └────────────────┼───────────────────┘             │
│                           ▼                                 │
│              ┌────────────────────────┐                     │
│              │  非官方 Google API 层   │                     │
│              └────────────┬───────────┘                     │
│                           ▼                                 │
│              ┌────────────────────────┐                     │
│              │   Google NotebookLM    │                     │
│              │   (Gemini + RAG)       │                     │
│              └────────────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 核心问题：为什么需要这个项目？

### NotebookLM 是好产品，但它是个「孤岛」

```
传统工作流（痛点）：
┌──────┐    手动     ┌──────────┐    手动     ┌──────┐
│ 资料 │ ──────────> │NotebookLM│ ──────────> │ 产出 │
└──────┘   上传点击   │  网页版  │   下载导出   └──────┘
              ↑                        ↑
           重复劳动               重复劳动

自动化工作流（解法）：
┌──────┐   脚本批量   ┌──────────┐   脚本下载   ┌──────┐
│ 资料 │ ──────────> │notebooklm│ ──────────> │ 产出 │
└──────┘    导入      │   -py    │     导出    └──────┘
              ↑                        ↑
           一次编写               自动执行
```

### 背景

如果说最近半年 AI 圈有什么"现象级"的产品，Google 的 NotebookLM 绝对榜上有名。凭借着超强的 Gemini 3 Pro 模型能力及 Nano Banana Pro 的生图能力，思维导图、PPT、播客轻松实现。

之前我们需要打开 NotebookLM 网页，慢慢点慢慢磨。但现在社区开发者把完整的 NotebookLM 搬到了命令行里。只需一行命令，生成思维导图、音频播客、PPT，乃至解析在线 YouTube 视频，统统不在话下。

---

## 🔧 三种使用方式对比

| 方式 | 适合场景 | 技术门槛 | 灵活性 |
|------|----------|----------|--------|
| **CLI 命令行** | 快速任务、Shell 脚本、一次性操作 | ⭐ 低 | ⭐⭐ 中 |
| **Python API** | 复杂工作流、异步处理、应用集成 | ⭐⭐⭐ 高 | ⭐⭐⭐ 高 |
| **Claude Skills** | AI Agent 自动化、自然语言操控 | ⭐ 低 | ⭐⭐⭐ 高 |

---

## 🚀 快速上手（5 分钟）

### 安装

```bash
# Step 1: 安装库（带浏览器支持）
pip install "notebooklm-py[browser]"

# Step 2: 安装浏览器引擎
playwright install chromium

# Step 3: 登录认证（会弹出浏览器窗口）
notebooklm login

# Step 4: 验证安装
notebooklm notebooks list
```

### 第一个实战：生成播客

```bash
# 创建笔记本
notebooklm create "我的第一个测试"

# 设置当前笔记本
notebooklm use <notebook_id>

# 添加素材
notebooklm source add "https://zh.wikipedia.org/wiki/人工智能"

# 生成并下载播客
notebooklm generate audio --wait
notebooklm download audio ./my-first-podcast.mp3
```

---

## 📝 三种使用方式详解

### 方式一：Claude Skills 方式

```bash
# 通过命令行界面安装，或者让 Claude Code 来安装
notebooklm skill install

# 然后使用自然语言：
# "制作一个关于量子计算的播客"
# "将测验下载为 markdown 格式"
# "/notebooklm 生成视频"
```

### 方式二：命令行界面

```bash
# 1. 首次运行先要登录认证（会打开浏览器）
notebooklm login

# 2. 创建一个 notebook
notebooklm create "My Research"
notebooklm use <notebook_id>

# 3. 添加源
notebooklm source add "https://en.wikipedia.org/wiki/Artificial_intelligence"
notebooklm source add "./paper.pdf"

# 4. 自然语言对话
notebooklm ask "What are the key themes?"

# 5. 生成播客
notebooklm generate audio --wait
notebooklm download audio ./podcast.mp3
```

### 方式三：Python API 方式

```python
import asyncio
from notebooklm import NotebookLMClient

async def main():
    async with await NotebookLMClient.from_storage() as client:
        # List notebooks
        notebooks = await client.notebooks.list()

        # Create notebook and add source
        nb = await client.notebooks.create("Research")
        await client.sources.add_url(nb.id, "https://example.com")

        # Chat
        result = await client.chat.ask(nb.id, "Summarize this")
        print(result.answer)

        # Generate podcast
        status = await client.artifacts.generate_audio(nb.id)
        await client.artifacts.wait_for_completion(nb.id, status.task_id)

asyncio.run(main())
```

---

## 📊 功能特性列表

| 类别 | 功能描述 |
|------|----------|
| **笔记本** | 创建、列表、重命名、删除、分享 |
| **来源** | 支持 URL、YouTube、文件 (PDF/TXT/MD/DOCX)、Google Drive、粘贴文本 |
| **聊天** | 提问、对话历史、自定义人设 |
| **生成** | 生成音频播客、视频、幻灯片、测验、抽认卡、报告、信息图、思维导图 |
| **研究** | 网络和云端驱动的研究代理，支持自动导入 |
| **下载** | 支持下载音频、视频、幻灯片、信息图、报告、思维导图、数据表、测验、抽认卡 |
| **代理技能** | Claude Code 技能，用于 LLM 驱动的自动化 |

---

## 💡 自动化工作流示例

### 工作流一：YouTube 视频批量学习器

```python
import asyncio
from notebooklm import NotebookLMClient

YOUTUBE_VIDEOS = [
    "https://www.youtube.com/watch?v=xxx1",
    "https://www.youtube.com/watch?v=xxx2",
]

async def weekly_video_digest():
    async with await NotebookLMClient.from_storage() as client:
        nb = await client.notebooks.create("本周技术视频")
        
        # 批量导入视频
        for video_url in YOUTUBE_VIDEOS:
            await client.sources.add_url(nb.id, video_url)
        
        # 生成综合分析
        summary = await client.chat.ask(nb.id, "总结这些视频的核心观点")
        print(summary.answer)
        
        # 生成播客
        await client.artifacts.generate_audio(nb.id)

asyncio.run(weekly_video_digest())
```

### 工作流二：研究论文自动处理

```python
async def process_papers(paper_urls: list[str]):
    async with await NotebookLMClient.from_storage() as client:
        nb = await client.notebooks.create("论文研究")
        
        for url in paper_urls:
            await client.sources.add_url(nb.id, url)
        
        # 提取关键信息
        themes = await client.chat.ask(nb.id, "这些论文的主要研究方向是什么？")
        methods = await client.chat.ask(nb.id, "总结各论文使用的研究方法")
        
        # 生成学习指南
        await client.artifacts.generate_study_guide(nb.id)
```

---

## ⚠️ 风险与注意事项

| 风险类型 | 可能性 | 应对策略 |
|----------|--------|----------|
| **API 变更导致失效** | 🔴 高 | 关注 GitHub，及时更新版本 |
| **速率限制** | 🟡 中 | 批量操作间加延迟 |
| **账号风险** | 🟢 低 | 使用非主力 Google 账号 |

### 重要声明

⚠️ 这是一个非官方库，使用未公开的 Google API：
- **风险**：API 可能随时变更中断，不受 Google 官方支持
- **用途**：最适合原型开发、研究和个人项目
- **限制**：可能存在速率限制

**关键心态**：把它当作「实验工具」而非「生产依赖」

---

## 🎯 适用人群

特别适合这几类人：
- 做研究/调研/技术写作的
- 经常看 YouTube 技术视频的
- 想把 NotebookLM 当成「长期知识库」的
- 用 Claude Code/Gemini CLI/各种 Agent 的重度用户

---

## 📋 核心要点总结

1. **notebooklm-py = NotebookLM 的命令行/API 版本**，把网页操作变成可编程的自动化流程
2. **三种使用方式**：CLI（快速任务）→ Python API（复杂工作流）→ Claude Skills（AI 自动化）
3. **核心价值：构建知识管道**，信息自动流入 → AI 处理 → 多格式流出
4. **重要风险：非官方 API**，仅适合原型/研究/个人项目
5. **快速上手**：`pip install "notebooklm-py[browser]"` → `notebooklm login` → 开始使用

---

## 💬 金句收录

> "对于真正的 Power User 来说，**可编程性和自动化才是王道**。"

> "未来的知识管理是**一个自动化的管道**，信息从一端流入，经过 AI 处理，从另一端流出我们需要的格式。"

> "这个工具，包括类似的项目，就是搭建这个管道的一块关键积木。"

---

## 🔗 相关链接

- [GitHub 仓库](https://github.com/teng-lin/notebooklm-py)
- [Google NotebookLM 官网](https://notebooklm.google.com/)
