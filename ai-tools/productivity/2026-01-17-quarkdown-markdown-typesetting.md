# Quarkdown - Markdown 增强排版神器精读

> 精读时间：2026-01-17
> 原文来源：[GitHub - iamgio/quarkdown](https://github.com/iamgio/quarkdown)
> 标签：#Markdown #排版 #LaTeX替代 #幻灯片 #可编程文档

---

## 一、项目定位

**一句话总结**：Quarkdown 是 Markdown 和 LaTeX 的中间地带，保留 Markdown 的易读性，同时引入图灵完备的脚本系统和专业排版能力。

**Slogan**: "Markdown with superpowers: from ideas to papers, presentations, websites, books, and knowledge bases."

| 维度 | 说明 |
|------|------|
| **解决的痛点** | Markdown 排版能力弱，LaTeX 语法太复杂 |
| **核心方案** | 扩展 Markdown 语法 + 内置脚本系统 + 多输出格式 |
| **目标用户** | 写作者、研究人员、教育工作者、开发者 |
| **开源协议** | GPL-3.0（CLI/LSP 为 AGPL-3.0） |

---

## 二、核心架构解析

### 2.1 技术栈

```
┌─────────────────────────────────────────────────────────────┐
│                     Quarkdown CLI/LSP                        │
│                    (Kotlin/JVM 17)                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Lexer     │→ │   Parser    │→ │     Renderer        │ │
│  │  词法分析    │  │   语法解析   │  │  (HTML/PDF/Slides)  │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    Standard Library                          │
│         布局构建器 / I/O / 数学运算 / 数据处理                │
├─────────────────────────────────────────────────────────────┤
│                    Output Engines                            │
│  ┌──────────┐  ┌───────────────┐  ┌──────────────────────┐ │
│  │  HTML    │  │   paged.js    │  │     reveal.js        │ │
│  │ (Plain)  │  │  (分页/PDF)    │  │     (幻灯片)          │ │
│  └──────────┘  └───────────────┘  └──────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 处理流水线

```
.qd 源文件 → 词法分析 → 语法解析 → AST → 脚本执行 → 渲染 → 后处理 → 输出
              │          │         │       │         │
              └─ 函数调用 └─ 变量   └─ 条件  └─ 循环   └─ HTML/PDF/Slides
                 识别        绑定     判断     展开
```

### 2.3 Gradle 项目结构

```
quarkdown/
├── quarkdown-core/          # 核心解析和渲染逻辑
├── quarkdown-cli/           # 命令行工具 (AGPL-3.0)
├── quarkdown-lsp/           # 语言服务器协议 (AGPL-3.0)
├── quarkdown-test/          # 测试和示例
└── lib/qd/                  # 标准库 (.qd 文件)
```

**关键依赖**：
- **Kotlin 2.3.0** — 主要开发语言
- **JVM 17** — 运行时要求
- **Dokka 2.0** — 文档生成
- **paged.js** — PDF 分页渲染
- **reveal.js** — 幻灯片演示
- **Puppeteer** — PDF 导出（需 Node.js）

---

## 三、语法系统深度剖析

### 3.1 函数调用语法

Quarkdown 的核心创新是引入了**函数调用语法**：

```markdown
.functionname {arg1} {arg2}
    Body argument (多行内容)
```

**与其他语法对比**：

| 工具 | 居中文本语法 |
|------|-------------|
| **LaTeX** | `\begin{center} ... \end{center}` |
| **Typst** | `#align(center)[...]` |
| **Quarkdown** | `.center\n    ...` |
| **Markdown** | ❌ 不支持 |

### 3.2 自定义函数

```markdown
.function {greet}
    to from:
    **Hello, .to** from .from!

.greet {world} from:{iamgio}
```
输出：**Hello, world** from iamgio!

**函数系统特性**：
- 位置参数和命名参数
- Body 参数（多行内容块）
- 递归调用
- 作用域（块级 vs 内联）

### 3.3 控制流

```markdown
# 变量
.var {count} {10}

# 条件判断
.if {.count > 5}
    Count is greater than 5

# 循环
.foreach {1..5}
    Item number .1

# 数学运算
.sum {.count} {20}
```

### 3.4 数据处理

```markdown
# 读取 CSV
.csv {data.csv}

# 表格生成
.table
    .foreach {.data}
        | .1.name | .1.value |

# 字符串操作
.uppercase {hello world}
```

---

## 四、输出格式详解

### 4.1 三种文档类型

| 类型 | 指令 | 输出引擎 | 适用场景 |
|------|------|----------|----------|
| **Plain** | `.doctype {plain}` | 纯 HTML | 博客、笔记、Wiki |
| **Paged** | `.doctype {paged}` | paged.js | 论文、书籍、报告 |
| **Slides** | `.doctype {slides}` | reveal.js | 演讲、教学、会议 |

### 4.2 分页文档能力

```markdown
.doctype {paged}

# 页面设置
.pagesize {A4}
.margin {2cm}

# 页眉页脚
.header
    Chapter: .chapter

.footer
    Page .pagenumber of .totalpages

# 分页控制
.pagebreak

# 目录
.toc
```

### 4.3 幻灯片能力

```markdown
.doctype {slides}

---

# Slide 1: Introduction

Content here...

---

# Slide 2: Details

- Point 1
- Point 2

.notes
    Speaker notes go here

---
```

**幻灯片特性**：
- `---` 分隔幻灯片
- 支持演讲者备注
- 交互片段（渐进显示）
- 自定义主题

---

## 五、与同类工具对比

### 5.1 排版工具对比矩阵

| 特性 | Quarkdown | LaTeX | Typst | Markdown | MDX |
|------|-----------|-------|-------|----------|-----|
| **语法易读性** | ✅ 高 | ❌ 低 | ✅ 中高 | ✅ 高 | ✅ 高 |
| **学习曲线** | 🟢 低 | 🔴 高 | 🟠 中 | 🟢 低 | 🟢 低 |
| **脚本系统** | ✅ 图灵完备 | ⚠️ 有限 | ✅ 完整 | ❌ 无 | ✅ JSX |
| **数学公式** | ✅ LaTeX 语法 | ✅ 原生 | ✅ 原生 | ⚠️ 插件 | ⚠️ 插件 |
| **分页排版** | ✅ paged.js | ✅ 原生 | ✅ 原生 | ❌ | ❌ |
| **幻灯片** | ✅ reveal.js | ⚠️ Beamer | ⚠️ Polylux | ❌ | ⚠️ 第三方 |
| **静态网站** | ✅ | ❌ | ⚠️ 实验性 | ✅ | ✅ |
| **主要语言** | Kotlin | TeX | Rust | - | JS/TS |
| **输出格式** | HTML/PDF | PDF/PS | PDF | HTML | HTML |

### 5.2 Quarkdown vs Typst

| 维度 | Quarkdown | Typst |
|------|-----------|-------|
| **定位** | Markdown 增强 | LaTeX 替代 |
| **语法基础** | CommonMark/GFM | 自定义标记语言 |
| **编译速度** | 快（JVM） | 极快（Rust + 增量） |
| **生态成熟度** | 新兴 | 快速成长 |
| **学术支持** | 有限 | 日益完善 |
| **IDE 支持** | VS Code 扩展 | Tinymist LSP |

**选择建议**：
- **熟悉 Markdown** → Quarkdown
- **学术写作为主** → Typst
- **需要幻灯片** → Quarkdown（原生支持）

### 5.3 Quarkdown vs Marp

| 维度 | Quarkdown | Marp |
|------|-----------|------|
| **专注领域** | 全能（文档+幻灯片+网站） | 专注幻灯片 |
| **脚本能力** | 图灵完备 | 无 |
| **导出格式** | HTML/PDF | HTML/PDF/PPTX |
| **主题系统** | CSS + 内置 | 多种内置主题 |
| **学习曲线** | 稍高（有脚本） | 极低 |

**选择建议**：
- **只需要幻灯片** → Marp（更简单）
- **需要文档+幻灯片统一** → Quarkdown

---

## 六、苏格拉底式深度问答

### Q1: 为什么 Quarkdown 选择 Kotlin/JVM 而非 Rust？

**A**: 这涉及开发效率与运行性能的权衡。

| 对比维度 | Kotlin/JVM | Rust |
|----------|------------|------|
| 开发效率 | 高（GC、空安全、协程） | 中等（手动内存管理） |
| 运行性能 | 中等（JIT 优化后） | 高（零成本抽象） |
| 启动时间 | 较慢（JVM 启动） | 快 |
| 生态系统 | 丰富（Java 生态） | 快速成长 |

**Quarkdown 选择 JVM 的可能原因**：
1. 开发者熟悉度（作者 iamgio 可能是 Java/Kotlin 背景）
2. 丰富的库支持（HTML 解析、PDF 生成）
3. 跨平台部署简单
4. 对于文档编译场景，启动时间不是关键瓶颈

### Q2: 图灵完备的脚本系统有什么实际价值？

**A**: 让文档成为"可执行的程序"，而非静态文本。

**实际应用场景**：

```markdown
# 1. 自动生成目录
.toc

# 2. 从数据生成表格
.table
    .foreach {.csv {students.csv}}
        | .1.name | .1.grade |

# 3. 条件内容
.if {.draft}
    ⚠️ This is a draft version

# 4. 模板复用
.function {warning}
    message:
    ⚠️ **Warning:** .message

.warning {Do not delete this file!}

# 5. 动态计算
Total: .sum {.map {.data} {.1.price}}
```

**与传统 Markdown 对比**：
- Markdown：写死的静态内容
- Quarkdown：数据驱动的动态文档

### Q3: paged.js 和 Puppeteer 在 PDF 生成中的角色？

**A**: 

```
.qd 源文件 → Quarkdown → HTML (paged.js) → Puppeteer → PDF
                              │
                              └─ 负责分页布局、页眉页脚、页码等
```

**paged.js**：
- W3C CSS Paged Media 的 polyfill
- 在浏览器中模拟打印分页
- 处理页眉、页脚、页码、孤行/寡行控制

**Puppeteer**：
- 无头 Chrome 浏览器
- 将渲染好的 HTML 打印为 PDF
- 保证字体、布局与浏览器预览一致

### Q4: 为什么需要 VS Code 扩展？

**A**: 提供实时预览和语言智能，提升写作体验。

**扩展能力**：
1. **语法高亮** — 函数调用、变量、控制流
2. **自动补全** — 内置函数、自定义函数
3. **实时预览** — 编译即预览，所见即所得
4. **错误诊断** — 语法错误、类型错误提示
5. **代码导航** — 跳转到函数定义

---

## 七、使用场景分析

### 7.1 适合的场景

| 场景 | 为什么适合 |
|------|------------|
| **技术博客** | Markdown 易写，代码块友好 |
| **课程讲义** | 文档和幻灯片统一源文件 |
| **产品文档** | 模板复用、数据驱动 |
| **个人知识库** | 多格式输出、便于迁移 |
| **技术书籍** | 分页排版、目录、索引 |

### 7.2 不太适合的场景

| 场景 | 为什么不适合 | 替代方案 |
|------|--------------|----------|
| **顶会论文** | 模板支持不如 LaTeX/Typst | Typst、LaTeX |
| **复杂数学** | 公式能力有限 | LaTeX |
| **团队协作** | 缺乏在线协作 | Notion、Overleaf |
| **极简幻灯片** | 学习成本高于 Marp | Marp |

---

## 八、快速上手指南

### 8.1 安装

```bash
# macOS/Linux (Homebrew)
brew install quarkdown-labs/quarkdown/quarkdown

# Windows (Scoop)
scoop bucket add java
scoop bucket add quarkdown https://github.com/quarkdown-labs/scoop-quarkdown
scoop install quarkdown
```

### 8.2 创建项目

```bash
quarkdown create my-project
cd my-project
```

### 8.3 编译和预览

```bash
# 编译
quarkdown c main.qd

# 实时预览（推荐）
quarkdown c main.qd -p -w

# 导出 PDF
quarkdown c main.qd --pdf
```

### 8.4 最小示例

```markdown
.doctype {paged}
.docname {My First Document}
.docauthor {Your Name}

# Introduction

This is a **Quarkdown** document.

.center
    This text is centered.

## Features

.foreach {1..3}
    - Feature number .1

.pagebreak

# Conclusion

.var {greeting} {Thank you for reading!}

.greeting
```

---

## 九、与 AI 工具的关联

### 9.1 AI 辅助写作场景

```
用户口述 → Voquill 转录 → AI 整理 → Quarkdown 排版 → PDF/幻灯片
```

### 9.2 AI Agent 应用

- **内容生成 Agent**：生成 Quarkdown 源文件
- **排版优化 Agent**：根据内容推荐文档类型和布局
- **数据可视化 Agent**：将数据转换为 Quarkdown 表格/图表

### 9.3 与 Memvid 的结合

知识库内容（Memvid 存储）→ Quarkdown 生成 → 多格式输出

---

## 十、总结

### 核心价值主张

| 传统 Markdown | Quarkdown |
|---------------|-----------|
| 静态文本 | 可编程文档 |
| 单一 HTML 输出 | HTML/PDF/幻灯片 |
| 无排版控制 | 专业分页排版 |
| 手动重复 | 模板和循环 |

### 技术亮点

1. **图灵完备脚本** — 变量、条件、循环、函数
2. **多输出格式** — paged.js + reveal.js
3. **Markdown 兼容** — CommonMark/GFM 基础
4. **VS Code 集成** — LSP 智能提示

### 适用人群

- ✅ 喜欢 Markdown 但需要更强排版能力
- ✅ 需要统一源文件生成多种格式
- ✅ 厌倦 LaTeX 复杂语法
- ⚠️ 学术顶会论文（考虑 Typst）
- ⚠️ 极简需求（考虑 Marp）

### 与生态系统关系

```
Markdown (基础)
    │
    ├── 静态网站生成 → Hugo, Jekyll, Astro
    │
    ├── 幻灯片 → Marp, Slidev, reveal.js
    │
    ├── 排版增强 → Quarkdown ← 你在这里
    │
    └── 学术写作 → Typst, LaTeX
```

---

## 参考资料

1. [Quarkdown GitHub](https://github.com/iamgio/quarkdown)
2. [Quarkdown 官网](https://quarkdown.com)
3. [Quarkdown Wiki](https://github.com/iamgio/quarkdown/wiki)
4. [Typst](https://typst.app)
5. [Marp](https://marp.app)
6. [paged.js](https://pagedjs.org)
7. [reveal.js](https://revealjs.com)
