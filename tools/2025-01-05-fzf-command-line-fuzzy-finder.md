# fzf - 命令行模糊查找器精读笔记

> 学习日期：2026-01-05
> 来源：https://github.com/junegunn/fzf
> 类型：技术工具学习

---

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **工具名称** | fzf (command-line fuzzy finder) |
| **作者** | Junegunn Choi |
| **GitHub Stars** | 7.6万+ |
| **开发语言** | Go |
| **核心价值** | 用模糊匹配彻底改变命令行查找体验 |

> 💡 **一句话总结**：fzf 通过模糊匹配算法，将"找东西"这件琐事的时间成本降到几乎为零，让命令行操作回归心流状态。

---

## 🗺️ 知识地图

### 核心功能架构

```
┌─────────────────────────────────────────────────┐
│              fzf 核心功能                        │
├─────────────────────────────────────────────────┤
│  1. 历史命令搜索 (Ctrl+R)                        │
│     └── 模糊匹配所有历史命令                      │
│  2. 文件快速定位 (Ctrl+T)                        │
│     └── 递归搜索当前目录所有文件                  │
│  3. 管道组合能力                                 │
│     └── 与任意命令组合（ps、git、docker等）       │
│  4. 编辑器集成                                   │
│     └── Vim/Neovim 插件支持                      │
└─────────────────────────────────────────────────┘
```

---

## 🔑 关键概念

| 概念 | 定义 | 类比理解 |
|------|------|----------|
| **模糊匹配** | 不要求精确输入，根据部分字符推断目标 | 像搜索引擎一样"猜你想要什么" |
| **子序列匹配** | 只要目标按顺序包含查询字符即匹配 | `cfg` 匹配 `config.yml` |
| **管道哲学** | 程序只做一件事，通过管道组合实现复杂功能 | 乐高积木，单块简单，组合无限 |
| **心流状态** | 完全沉浸在任务中，不被琐事打断的专注状态 | 开车时的"自动驾驶"感觉 |

---

## 🛠️ 实用技巧速查

### 安装配置

```bash
# Mac
brew install fzf
$(brew --prefix)/opt/fzf/install

# Linux (Debian/Ubuntu)
sudo apt install fzf

# Shell 集成（添加到 ~/.bashrc 或 ~/.zshrc）
eval "$(fzf --bash)"   # Bash
source <(fzf --zsh)    # Zsh
```

### 核心快捷键

| 快捷键 | 功能 | 数据源 |
|--------|------|--------|
| `Ctrl+R` | 搜索历史命令 | `~/.bash_history` |
| `Ctrl+T` | 搜索文件路径 | 当前目录递归 |
| `Alt+C` | 搜索并进入目录 | 当前目录递归 |

### 搜索语法

| 语法 | 含义 | 示例 |
|------|------|------|
| `abc` | 模糊匹配 | 包含 a、b、c（按顺序） |
| `'abc` | 精确匹配 | 必须包含连续的 "abc" |
| `^abc` | 前缀匹配 | 以 "abc" 开头 |
| `abc$` | 后缀匹配 | 以 "abc" 结尾 |
| `!abc` | 排除匹配 | 不包含 "abc" |
| `a \| b` | 或匹配 | 包含 a 或 b |

### 常用组合命令

```bash
# 1. 进程管理：选择并杀死进程
ps -ef | fzf | awk '{print $2}' | xargs kill -9

# 2. Git 分支切换
git branch | fzf | xargs git checkout

# 3. Git 提交浏览（带预览）
git log --oneline | fzf --preview 'git show {1}'

# 4. Docker 容器进入
docker ps | fzf | awk '{print $1}' | xargs -I {} docker exec -it {} bash

# 5. 文件预览（需安装 bat）
fzf --preview 'bat --color=always {}'

# 6. 编辑选中文件
vim $(fzf)
```

### 环境变量配置

```bash
# 默认选项
export FZF_DEFAULT_OPTS='--height 40% --layout=reverse --border'

# 使用 fd 替代 find（更快）
export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'

# Ctrl+T 使用 fd
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
```

---

## 💡 核心洞见

### 1. 效率的本质是消除"减速带"

找文件、找命令、找代码这些琐事，是隐形的生产力杀手。fzf 的价值不在于"快"，而在于"不打断思路"。

### 2. 好工具应该是"无形"的

- 不抢戏、不臃肿、界面简陋但功能精准
- 成为思维的延伸，而非工作的负担

### 3. 允许模糊是一种"人味"设计

- 传统工具要求精确输入，对人不友好
- fzf 允许犯懒、允许记错，符合人类认知习惯

---

## ⚠️ 常见误区

| 误区 | 正确做法 |
|------|----------|
| 认为 fzf 只能搜历史命令 | 它可以搜任何列表（文件、进程、Git 记录等） |
| 不配置 Shell 集成就使用 | 必须运行 `eval "$(fzf --bash)"` 等命令启用快捷键 |
| 在大目录下用默认 find | 配合 `fd` 或 `rg` 可获得更快速度 |

---

## 📚 延伸工具链

| 工具 | 用途 | 安装 |
|------|------|------|
| [fd](https://github.com/sharkdp/fd) | 更快的 find 替代品 | `brew install fd` |
| [ripgrep](https://github.com/BurntSushi/ripgrep) | 更快的 grep 替代品 | `brew install ripgrep` |
| [bat](https://github.com/sharkdp/bat) | 带语法高亮的 cat | `brew install bat` |
| [fzf.vim](https://github.com/junegunn/fzf.vim) | Vim/Neovim 集成 | Plug 'junegunn/fzf.vim' |

---

## 📖 参考资料

- [fzf 官方 GitHub](https://github.com/junegunn/fzf)
- [fzf 官方文档](https://junegunn.github.io/fzf/)
- [fzf Wiki 示例集](https://github.com/junegunn/fzf/wiki/Examples)

---

## 💬 金句摘录

> "真正的效率工具，应该是无形的。不应该成为你工作的负担，而应该成为你思维的延伸。"

> "它允许你犯懒，允许你模糊。这就是'人味'。"

> "这种快，不是那种'我不卡'的快，而是'心流不被打断'的快。"

---

*笔记生成时间：2026-01-05*
