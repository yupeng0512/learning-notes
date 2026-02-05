---
title: RSS 订阅自动推送飞书的完整方案
date: 2026-02-04
tags: [rss, feishu, automation, n8n]
type: practical
---

# RSS 订阅自动推送飞书的完整方案

> 目标：实现跨端 RSS 阅读 + 定时推送飞书  
> 技术栈：Inoreader + N8N + 飞书 Webhook

---

## 方案架构

```
Inoreader（订阅管理） → N8N（自动化） → 飞书（推送通知）
```

---

## 实施步骤

### Step 1: 注册 Inoreader

1. 访问 [Inoreader](https://www.inoreader.com)
2. 注册账号，选择 Pro 计划（$49.99/年）
3. 下载 Mac/iOS 客户端
4. 添加你的订阅源

**关键配置**：
- 创建标签/文件夹分类
- 设置过滤规则（关键词高亮）
- 开启 API 访问权限

---

### Step 2: 部署 N8N（自动化引擎）

**选项 A：云端托管**
```bash
# 使用 N8N Cloud
https://n8n.io/cloud
# $20/月，免运维
```

**选项 B：自建（推荐）**
```bash
# Docker 部署
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# 访问 http://localhost:5678
```

---

### Step 3: 配置飞书 Webhook

1. 打开飞书群聊 → 设置 → 群机器人
2. 添加自定义机器人
3. 复制 Webhook URL
4. 设置签名校验（可选）

示例 Webhook：
```
https://open.feishu.cn/open-apis/bot/v2/hook/xxx
```

---

### Step 4: 创建 N8N 工作流

```yaml
工作流结构：
1. Cron Trigger（定时触发）
   ↓
2. HTTP Request（获取 Inoreader 未读文章）
   ↓
3. Function（数据处理 + AI 摘要）
   ↓
4. HTTP Request（推送飞书卡片）
```

**核心节点配置**：

#### 节点 1：Cron Trigger
```javascript
// 每天早上 8:00 触发
0 8 * * *
```

#### 节点 2：获取 Inoreader 文章
```javascript
// HTTP Request 节点
URL: https://www.inoreader.com/reader/api/0/stream/contents/user/-/state/com.google/reading-list
Method: GET
Authentication: OAuth2
Query Parameters:
  - n: 10  // 获取最新 10 篇
  - xt: user/-/state/com.google/read  // 排除已读
```

**获取 API Token**：
- 访问 https://www.inoreader.com/developers/
- 创建 OAuth 应用
- 获取 Access Token

#### 节点 3：数据处理 + AI 摘要
```javascript
// Function 节点
const items = $input.all();
const articles = items[0].json.items;

const processedArticles = articles.slice(0, 5).map(article => ({
  title: article.title,
  url: article.canonical[0].href,
  summary: article.summary.content.substring(0, 200) + '...',
  published: new Date(article.published * 1000).toLocaleString('zh-CN'),
  source: article.origin.title
}));

return processedArticles.map(article => ({ json: article }));
```

#### 节点 4：推送飞书卡片
```javascript
// HTTP Request 节点
URL: https://open.feishu.cn/open-apis/bot/v2/hook/xxx
Method: POST
Body (JSON):
{
  "msg_type": "interactive",
  "card": {
    "header": {
      "title": {
        "content": "📰 今日 RSS 精选",
        "tag": "plain_text"
      }
    },
    "elements": [
      {
        "tag": "div",
        "text": {
          "content": "{{$json.title}}",
          "tag": "lark_md"
        }
      },
      {
        "tag": "div",
        "text": {
          "content": "📝 {{$json.summary}}",
          "tag": "plain_text"
        }
      },
      {
        "tag": "action",
        "actions": [
          {
            "tag": "button",
            "text": {
              "content": "阅读全文",
              "tag": "plain_text"
            },
            "url": "{{$json.url}}",
            "type": "primary"
          }
        ]
      },
      {
        "tag": "hr"
      }
    ]
  }
}
```

---

### Step 5: 增强功能（可选）

#### 功能 1：AI 摘要增强
```javascript
// 在 Function 节点中添加 OpenAI 调用
const OpenAI = require('openai');
const openai = new OpenAI({ apiKey: 'xxx' });

const summary = await openai.chat.completions.create({
  model: "gpt-4",
  messages: [{
    role: "user",
    content: `用 3 句话总结这篇文章：\n${article.content}`
  }]
});
```

#### 功能 2：关键词过滤
```javascript
// 只推送包含特定关键词的文章
const keywords = ['AI', '架构', 'Claude', 'Cursor'];
const filtered = articles.filter(article => 
  keywords.some(keyword => 
    article.title.includes(keyword) || 
    article.summary.content.includes(keyword)
  )
);
```

#### 功能 3：多时段推送
```javascript
// Cron 配置多个时间
早报：0 8 * * *   // 早上 8:00
午报：0 12 * * *  // 中午 12:00
晚报：0 20 * * *  // 晚上 8:00
```

---

## 效果预览

**飞书卡片示例**：

```
📰 今日 RSS 精选
─────────────────────

🔸 Claude 3.5 Sonnet 发布，性能提升 2 倍
📝 Anthropic 今日发布 Claude 3.5 Sonnet，在编码和数学任务上表现优异...
🔗 [阅读全文]
来源：Anthropic Blog | 2026-02-04 08:00

─────────────────────

🔸 Cursor 新功能：AI 代码审查
📝 Cursor 推出全新 AI Code Review 功能，自动检测潜在 bug...
🔗 [阅读全文]
来源：Cursor Blog | 2026-02-04 07:30
```

---

## 成本分析

| 项目 | 月成本 | 年成本 |
|------|-------|-------|
| Inoreader Pro | $4.17 | $49.99 |
| N8N 自建（腾讯云轻量） | $5 | $60 |
| OpenAI API（可选） | $10 | $120 |
| **总计** | **$19.17** | **$229.99** |

**性价比方案**（不含 AI 摘要）：$9.17/月

---

## 替代方案对比

### 方案 A：Feedly + Zapier
- 成本：$32/月（Feedly Pro+ $12 + Zapier $20）
- 优势：Leo AI 智能分析
- 劣势：Zapier 贵，推送限制多

### 方案 B：RSSHub + TTRSS + N8N（全开源）
- 成本：$5/月（仅服务器）
- 优势：完全免费，灵活性最高
- 劣势：需要技术能力，自己搭建维护

### 方案 C：ReadBot + Cubox
- 成本：$2.5/月（ReadBot + Cubox）
- 优势：国内方案，便宜
- 劣势：Mac 端无原生客户端，需通过浏览器

---

## 快速启动清单

- [ ] 注册 Inoreader，导入订阅源
- [ ] 部署 N8N（Docker 或 Cloud）
- [ ] 创建飞书群机器人，获取 Webhook
- [ ] 配置 Inoreader API OAuth
- [ ] 创建 N8N 工作流（导入模板）
- [ ] 测试推送效果
- [ ] 设置定时任务
- [ ] （可选）集成 OpenAI API

---

## 延伸阅读

- [Inoreader API 文档](https://www.inoreader.com/developers/)
- [N8N 官方文档](https://docs.n8n.io/)
- [飞书机器人开发指南](https://open.feishu.cn/document/ukTMukTMukTM/ucTM5YjL3ETO24yNxkjN)
- [RSSHub 官方文档](https://docs.rsshub.app/)

---

## ⚡ 快速上手（10 分钟）

### 方案 1：使用配置好的 n8n 工作流（推荐）

我已经为你生成了完整的 n8n 工作流配置文件，直接导入即可使用！

**文件位置**：
```bash
/data/workspace/n8n-workflows/
├── rss-to-feishu-workflow.json  # 工作流配置（导入此文件）
├── README.md                     # 详细配置指南
└── quick-start.sh                # 一键启动脚本
```

**一键启动**：
```bash
cd /data/workspace/n8n-workflows
./quick-start.sh
```

脚本会自动：
- ✅ 检查 Docker 环境
- ✅ 启动 n8n 容器
- ✅ 创建数据目录
- ✅ 提供访问地址和配置指引

**手动启动**：
```bash
# 1. 启动 n8n
docker run -d --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n

# 2. 访问 n8n
open http://localhost:5678

# 3. 导入工作流
# 点击右上角 "Import from file"
# 选择 rss-to-feishu-workflow.json
```

**配置步骤**：
1. **创建飞书机器人**（2 分钟）
   - 飞书群 → 设置 → 群机器人 → 添加「自定义机器人」
   - 复制 Webhook URL

2. **配置工作流**（5 分钟）
   - 点击「发送到飞书」节点
   - 替换 `YOUR_WEBHOOK_URL` 为你的 Webhook
   - 保存工作流

3. **测试推送**（1 分钟）
   - 点击「Execute Workflow」按钮
   - 查看飞书群是否收到消息

4. **启用定时任务**（1 分钟）
   - 点击工作流右上角「Active」开关
   - 每小时自动执行

---

### 工作流功能说明

**包含节点**：
1. ⏰ **定时触发器**：每小时自动执行（可调整）
2. 📰 **RSS 订阅源**：抓取 RSS Feed（默认：少数派）
3. 🔍 **关键词筛选**：过滤包含 AI、Claude、Cursor 等关键词
4. 📝 **数据处理**：格式化标题、摘要、添加分类标签
5. 🤖 **AI 智能摘要**：可选，调用 OpenAI 生成精炼摘要
6. 📬 **发送到飞书**：推送富文本卡片消息

**推送效果预览**：
```
┌─────────────────────────────────────┐
│ 🤖 AI Claude 3.5 Sonnet 发布        │
├─────────────────────────────────────┤
│ 📝 Anthropic 今日发布 Claude 3.5    │
│ Sonnet，在编码和数学任务上表现优    │
│ 异，推理速度提升 2 倍...            │
├─────────────────────────────────────┤
│ 来源：少数派                        │
│ 发布时间：2026-02-04 08:30          │
├─────────────────────────────────────┤
│           [📖 阅读全文]             │
└─────────────────────────────────────┘
```

---

### 自定义配置

**1. 添加更多 RSS 订阅源**
```javascript
// 推荐订阅源
少数派：https://sspai.com/feed
阮一峰博客：https://www.ruanyifeng.com/blog/atom.xml
V2EX：https://www.v2ex.com/index.xml
Hacker News：https://news.ycombinator.com/rss
GitHub Trending：https://mshibanami.github.io/GitHubTrendingRSS/daily/all.xml
```

复制「RSS 订阅源」节点，修改 URL 即可。

**2. 修改关键词过滤**
- 点击「关键词筛选」节点
- 添加/删除关键词
- 支持 AND/OR 逻辑组合

**3. 调整推送频率**
- 每 3 小时：`"hoursInterval": 3`
- 每天早上 8 点：`"cronExpression": "0 8 * * *"`
- 每天早晚 2 次：`"cronExpression": "0 8,20 * * *"`

**4. 启用 AI 摘要（可选）**
- 点击「AI 智能摘要（可选）」节点
- 取消勾选「Disabled」
- 配置 OpenAI API Key
- 成本约 $0.30/月（每天 10 篇文章）

---

## 总结

**最佳实践**：
1. 用 Inoreader 管理订阅（跨端体验好）
2. 用 n8n 实现定时推送（灵活可控）
3. 用飞书接收通知（工作场景友好）
4. 可选 AI 摘要增强（降低信息过载）

**ROI 分析**：
- 投入：$5/月（n8n 自建）+ 10 分钟配置
- 产出：每天节省 30 分钟信息筛选时间
- 回本周期：< 1 周

---

## 🎁 附赠工作流

已为你生成完整的 n8n 工作流配置，查看：
```bash
/data/workspace/.codebuddy/n8n-workflows/
```

包含：
- ✅ 可导入的工作流 JSON 文件
- ✅ 详细配置指南（README.md）
- ✅ 一键启动脚本（quick-start.sh）
- ✅ 飞书卡片模板
- ✅ AI 摘要集成（可选）
