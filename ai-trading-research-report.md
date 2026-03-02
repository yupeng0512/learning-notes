# AI 辅助交易深度研究报告

> 核心理念：利用 AI 的能力辅助人，提升自我认知、思维判断及决策能力
> 
> 日期：2026-03-01

---

## 目录

1. [Hedge Fund + AI Agent 落地实践](#1-hedge-fund--ai-agent-落地实践)
2. [Polymarket 自动化交易](#2-polymarket-自动化交易)
3. [期权交易组合：Sell Put & Covered Call](#3-期权交易组合sell-put--covered-call)
4. [三者的融合：AI 辅助交易系统设计思路](#4-三者的融合ai-辅助交易系统设计思路)

---

## 1. Hedge Fund + AI Agent 落地实践

### 1.1 行业现状

2026 年，AI 对冲基金已从概念验证走向实战落地。几个关键数据：

- **全球 AI 交易平台市场**：2024 年估值 $112.3 亿，预计 2030 年达 $334.5 亿（年增 20%）
- **High-Flyer（幻方量化）**：中国量化基金巨头，管理 $137.9 亿，十年投入数千万美元购买 A100 芯片建 AI 超算集群，后孵化出 DeepSeek
- **Axelrod**：AI-native 对冲基金，2025 Q2 以 $500 万 AUM 启动 Genesis Fund，全链上执行

### 1.2 落地路径：从开源项目到实战基金

#### 第一阶段：教育与原型（virattt/ai-hedge-fund）

GitHub 上最火的开源项目（⭐ 45,984），架构清晰可学习：

```
┌─────────────────────────────────────────────────┐
│              AI Hedge Fund Architecture          │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────── 投资者人格 Agent（12个）──────────┐ │
│  │ Buffett | Munger | Damodaran | Graham        │ │
│  │ Ackman  | C.Wood | M.Burry  | Pabrai        │ │
│  │ P.Lynch | Fisher | Jhunjhunwala | Druckenmiller │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│  ┌──────────── 分析 Agent（4个）────────────────┐ │
│  │ Valuation | Sentiment | Fundamentals | Technicals │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│  ┌──────────── 决策 Agent（2个）────────────────┐ │
│  │ Risk Manager ──→ Portfolio Manager            │ │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│              最终交易决策 & 订单生成               │
└─────────────────────────────────────────────────┘
```

**快速上手**：
```bash
git clone https://github.com/virattt/ai-hedge-fund.git
cd ai-hedge-fund
cp .env.example .env
# 设置 OPENAI_API_KEY / ANTHROPIC_API_KEY / FINANCIAL_DATASETS_API_KEY
poetry install
poetry run python src/main.py --ticker AAPL,MSFT,NVDA
poetry run python src/backtester.py --ticker AAPL,MSFT,NVDA
```

**学习价值**：
- 理解多 Agent 协作的交易决策流程
- AAPL/GOOGL/MSFT/NVDA/TSLA 数据免费，无需 API Key
- 支持 Ollama 本地 LLM，可完全离线运行
- 内置回测系统，可验证策略有效性

#### 第二阶段：多模型集成竞争（20-Agent Ensemble）

**核心突破**：不信任单一模型，让多个 AI 竞争决策权

| 组件 | 说明 |
|------|------|
| **多 LLM 协作** | GPT-4（结构化推理）+ Claude 3.5（模式识别与风控）+ Gemini Pro（多模态情感分析） |
| **Elo 评分系统** | 借鉴国际象棋等级分，根据历史信号准确度动态调整每个 Agent 的话语权 |
| **分层激活** | Tier 1（5 个轻量筛选）→ Tier 2（10 个中度分析）→ Tier 3（5 个重度推理），API 成本降低 60% |
| **熔断机制** | 任一 LLM 提供商宕机，自动切换到其他模型，零停机 |

**实战成果**：8 连胜交易，+90.6% 总回报，虚假信号减少 73%

**关键洞察**：
> "模型多样性比模型质量更重要。GPT-4 很出色，但单独使用很脆弱。Claude 3.5 总能捕捉到 GPT-4 遗漏的风险因素。" —— Nic Chin

#### 第三阶段：学术前沿（HedgeAgents）

Columbia 大学的 HedgeAgents 框架：
- 中央基金经理 + 多个对冲专家（分管不同资产类别）
- 通过三种会议类型协调 LLM 认知能力
- 3 年回测：年化 70%，总回报 400%

### 1.3 落地实操路线图

```
Phase 1: 学习与模拟（1-2 个月）
├── Clone virattt/ai-hedge-fund，本地跑通
├── 用你已有的 alpha-radar 系统做信号生成
├── 在 trading-learning/ 中记录策略表现
└── 用 Paper Trading 积累数据

Phase 2: 多 Agent 系统搭建（2-3 个月）
├── 引入多 LLM（OpenAI + Claude + DeepSeek）
├── 实现 Elo 评分的 Agent 竞争机制
├── 对接实时数据源（TradingView Webhook / 你已有的）
└── 建立回测与绩效评估体系

Phase 3: 实盘验证（3-6 个月）
├── 小资金实盘（$5K-$10K）
├── 建立风控规则（最大回撤、单笔亏损上限）
├── 持续优化 Agent 权重与策略
└── 积累 track record

Phase 4: 基金化运作（6-12 个月）
├── 香港 9 号牌照 / 离岸 VCC 结构
├── 准备 pitch deck + 历史业绩
├── 种子轮融资（$500K-$5M）
└── 合规与审计
```

### 1.4 香港及离岸注册路径

| 路径 | 说明 | 适合 |
|------|------|------|
| **香港 SFC 9 号牌** | 正规资管牌照，可面向专业投资者 | 传统路径，合规要求高 |
| **新加坡 VCC** | Variable Capital Company，税务优惠 | 亚洲市场，灵活结构 |
| **开曼群岛基金** | 经典离岸结构，多数国际基金采用 | 面向全球投资者 |
| **BVI + 链上基金** | 如 Axelrod 模式，全链上透明执行 | Crypto 原生基金 |

---

## 2. Polymarket 自动化交易

### 2.1 Polymarket 是什么？

Polymarket 是去中心化预测市场，用户交易真实事件的结果概率：
- 每个市场代表一个二元或多结果问题（政治、经济、体育、加密等）
- 合约价格 = 市场隐含概率（$0.65 = 65% 概率）
- 如果事件发生，赢家获得 $1.00/份；不发生获得 $0.00
- 2025 年交易量超过 **$440 亿**
- 结算使用 USDC on Polygon

### 2.2 AI Agent 自动化交易架构

```
┌─────────────────────────────────────────────────┐
│           Polymarket AI Agent 架构               │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─── 数据收集层 ──────────────────────────────┐ │
│  │ • Polymarket API（价格/成交量/订单簿）       │ │
│  │ • 新闻 & 社媒 API（情感信号）               │ │
│  │ • 历史数据（回测用）                         │ │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│  ┌─── 策略引擎 ────────────────────────────────┐ │
│  │ • 概率模型 & 回归分析                        │ │
│  │ • 情感分析（NLP + LLM）                     │ │
│  │ • 强化学习（持续策略优化）                   │ │
│  │ • 分类算法                                   │ │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│  ┌─── 风控模块 ────────────────────────────────┐ │
│  │ • 仓位管理 & 止损规则                        │ │
│  │ • 投资组合分散化                             │ │
│  │ • 滑点 & 流动性检查                          │ │
│  └──────────────────────────────────────────────┘ │
│                     ▼                             │
│  ┌─── 执行引擎 ────────────────────────────────┐ │
│  │ • Polymarket CLOB API                        │ │
│  │ • Polygon 钱包管理                           │ │
│  │ • 订单路由 & 成交确认                        │ │
│  └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### 2.3 三大可实操策略

#### 策略一：SPREAD（套利）
- **触发条件**：YES + NO > $1.05（5% 缓冲覆盖滑点和交易成本）
- **原理**：无风险利润，买入两边锁定收益
- **2024-2025 年套利者总利润**：超过 **$4000 万**
- **注意**：需要亚秒级延迟，15 分钟轮询只能观察，无法捕捉

#### 策略二：TAIL（趋势跟随）
- **触发条件**：概率 > 60% 且出现成交量激增
- **原理**：强共识 + 高参与度 → 趋势大概率持续
- **时间窗口**：数小时级别

#### 策略三：BONDING（逆向操作）
- **触发条件**：价格突然下跌 > 10%（通常由突发新闻导致）
- **原理**：市场对新信息过度反应，价格部分回归
- **风险**：需区分"过度反应"和"合理定价"

### 2.4 跟单交易落地

#### 寻找值得跟单的交易者

| 指标 | 推荐阈值 |
|------|---------|
| 胜率 | > 70% |
| 总利润 | > $200K |
| 交易次数 | > 300 |
| 持续时间 | > 6 个月 |
| 单笔量 | > $50K（鲸鱼级）|

**知名顶级交易者**：
- French Whale (Théo)：$8500 万利润
- Erasmus：$130 万+
- WindWalk3：$110 万+
- ilovecircle：2 个月 $220 万，74% 胜率

#### 开源跟单工具

| 项目 | 语言 | 特点 |
|------|------|------|
| `vladmeeros/polymarket-copytrading-bot` | Rust | 高性能 WebSocket，多种跟单策略 |
| `ducksybils/polymarket-copytrading` | Python | Dry-run 模式，状态持久化，健康检查 |
| `sloniew/PolymarketCopyTradingBOT` | Python | 快速执行，Telegram 报告 |

#### 现成服务

- **Stand.trade**：免费高级终端，一键跟单 + Discord 提醒
- **Polycule**：Telegram Bot，1% 手续费，支持按比例/固定仓位
- **PolyTrack**：高级筛选，按绩效指标和类别过滤

### 2.5 模拟账户 + 24h 强化学习落地方案

这是你提到的核心需求——**不用真金白银，让 Agent 不断迭代**：

```
┌─────────────── Paper Trading 闭环 ──────────────┐
│                                                   │
│  每 15 分钟轮询一次 Polymarket API                │
│       ▼                                           │
│  三策略并行评估（TAIL / BONDING / SPREAD）        │
│       ▼                                           │
│  模拟执行 → 记录到 PostgreSQL                     │
│       ▼                                           │
│  每日 8AM Discord 发送汇总报告                    │
│       ▼                                           │
│  积累 2-4 周数据后分析策略表现                    │
│       ▼                                           │
│  SQL 查询按策略评估胜率：                         │
│  SELECT strategy, COUNT(*),                       │
│    SUM(CASE WHEN pnl>0 THEN 1 ELSE 0 END)        │
│    / COUNT(*) as win_rate                         │
│  FROM paper_trades GROUP BY strategy              │
│       ▼                                           │
│  调整策略参数 → 继续迭代                          │
│                                                   │
└───────────────────────────────────────────────────┘
```

**数据库表设计**：
```sql
CREATE TABLE paper_trades (
  id SERIAL PRIMARY KEY,
  market_id TEXT,
  market_name TEXT,
  strategy TEXT,       -- TAIL / BONDING / SPREAD
  direction TEXT,      -- YES / NO
  entry_price DECIMAL,
  exit_price DECIMAL,
  quantity DECIMAL,
  pnl DECIMAL,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE portfolio (
  id SERIAL PRIMARY KEY,
  total_value DECIMAL,
  cash DECIMAL,
  positions JSONB,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**关键注意事项**：
1. Paper Trading 绩效 ≠ 实盘绩效（滑点 1-2%、流动性限制）
2. 至少积累 100+ 笔交易/策略后再下结论
3. SPREAD 策略在 15 分钟轮询下只能观察历史数据
4. 需注意 API 限流（100 requests/min），批量请求优化

### 2.6 与你的 Workspace 项目的关联

你的 TrendRadar 已经在监控 Polymarket 相关热点（Peter Liu 的 AI + Polymarket 内幕分析等），可以直接将 TrendRadar 的信号接入 Polymarket Agent 作为情感/信息层。

---

## 3. 期权交易组合：Sell Put & Covered Call

### 3.1 基础概念图解

```
期权四大基本操作：

         看涨（Bullish）          看跌（Bearish）
买方      Buy Call               Buy Put
(付权利金) (买入看涨期权)          (买入看跌期权)
         最大亏损=权利金           最大亏损=权利金

卖方      Sell Call              Sell Put
(收权利金) (卖出看涨期权)          (卖出看跌期权)
         最大收益=权利金           最大收益=权利金
```

### 3.2 Sell Put（卖出看跌期权 / Cash-Secured Put）

**本质**："有条件的买入" —— 你愿意以某个价格买入某只股票，同时收取权利金。

**操作流程**：
```
1. 选择你真心想拥有的股票（如 AAPL）
2. 确定你愿意买入的价格（如当前 $180，你愿意 $165 买）
3. 卖出行权价 $165 的 Put 期权
4. 收取权利金（如 $3.50 × 100 = $350）
5. 准备好 $16,500 现金（= $165 × 100 股）

结果A：到期时 AAPL > $165 → 期权作废，你白拿 $350
结果B：到期时 AAPL < $165 → 你被行权，以 $165 买入 100 股
       实际成本 = $165 - $3.50 = $161.50/股（比市场低）
```

**核心优势**：
- 即使被行权，你也是在**你愿意的价格**买入好公司
- 收取的权利金进一步降低了实际买入成本
- 如果不被行权，$350 权利金就是纯收益

**关键风险**：
- 如果股票暴跌（如从 $180 跌到 $120），你仍需以 $165 买入
- 所以**只对你真正想长期持有的优质股票操作**

### 3.3 Covered Call（备兑看涨期权）

**本质**："出租你的股票" —— 你已持有股票，通过卖出看涨期权收取"租金"。

**操作流程**：
```
1. 你已持有 100 股 AAPL（成本 $161.50/股）
2. 卖出行权价 $175 的 Call 期权
3. 收取权利金 $2.50 × 100 = $250
4. 等待到期

结果A：到期时 AAPL < $175 → 期权作废，保留股票 + $250
结果B：到期时 AAPL > $175 → 股票被以 $175 卖出
       总利润 = 股价差 ($175 - $161.50) + 权利金 ($2.50)
              = $16/股 = $1,600
```

**核心优势**：
- 持有股票期间持续收取"租金"
- 即使横盘也有收入

**核心风险**：
- 如果股票大涨超过行权价，你只能以行权价卖出，错失额外涨幅
- 所以适合**温和看涨或横盘**的市场环境

### 3.4 Wheel 策略（滚轮策略）—— 核心收入引擎

Wheel 策略将 Sell Put 和 Covered Call 组合成一个无限循环：

```
        ┌──────────────────────────────────────┐
        │                                      │
        ▼                                      │
   持有现金                                    │
        │                                      │
        ▼                                      │
   Sell Put（收权利金）                        │
        │                                      │
   ┌────┴────┐                                 │
   │         │                                 │
 不被行权   被行权                              │
 (保留现金  (买入股票)                          │
  + 权利金)      │                             │
   │             ▼                             │
   │        持有股票                            │
   │             │                             │
   │             ▼                             │
   │        Sell Covered Call（收权利金）       │
   │             │                             │
   │        ┌────┴────┐                        │
   │        │         │                        │
   │      不被行权   被行权                     │
   │      (保留股票  (卖出股票                  │
   │       + 权利金)  + 权利金)                 │
   │        │              │                   │
   └────────┘              └───────────────────┘
        ↑
     循环继续
```

**三重利润来源**：
1. Sell Put 权利金
2. Sell Call 权利金
3. 股票买卖价差

**年化收益预期**：15% ~ 30%（取决于市场条件和标的选择）

### 3.5 实战完整案例（AAPL）

```
第 1 周：Sell Put
├── AAPL 当前价格：$180
├── 卖出 $165 Put（30 天到期，Delta 0.20）
├── 收权利金：$350
├── 需要准备：$16,500 现金
└── 如果到期 AAPL > $165：保留 $350，年化约 25%

第 5 周：被行权
├── AAPL 跌到 $162，Put 被行权
├── 买入 100 股 AAPL @ $165
├── 实际成本基础：$161.50（$165 - $3.50 权利金）
└── 此时持有 100 股

第 6 周：Sell Covered Call
├── AAPL 当前 $163
├── 卖出 $175 Call（30 天到期）
├── 收权利金：$250
└── 等待到期

第 10 周：被行权
├── AAPL 涨到 $178，Call 被行权
├── 以 $175 卖出 100 股
├── 利润汇总：
│   ├── Put 权利金：+$350
│   ├── Call 权利金：+$250
│   └── 股票价差：+$1,350（$175 - $161.50）× 100
│   └── 总利润：$1,950（约 11.8% / 10 周）
└── 回到持有现金状态，循环重启
```

### 3.6 行权价选择与 Delta 指南

| 期权类型 | 推荐 Delta | 价外程度 | 被行权概率 | 权利金水平 |
|---------|-----------|---------|-----------|-----------|
| Sell Put（保守）| 0.15-0.20 | 10-15% OTM | 15-20% | 较低 |
| Sell Put（适中）| 0.25-0.30 | 5-10% OTM | 25-30% | 中等 |
| Covered Call（保守）| 0.20-0.25 | 8-12% OTM | 20-25% | 较低 |
| Covered Call（积极）| 0.30-0.40 | 3-7% OTM | 30-40% | 较高 |

### 3.7 风险管理清单

- [ ] **只选你愿意持有 10 年的好公司**
- [ ] 每个 Wheel 仓位限制在投资组合的 **2-5%**
- [ ] 同时运行 **多个 Wheel** 实现分散化
- [ ] **不在财报日前卖期权**（波动率突增后可能暴跌）
- [ ] **不追逐高权利金烂股**（高溢价 = 高风险）
- [ ] 永远不在**成本价以下卖 Covered Call**
- [ ] 预留 **现金缓冲** 用于 Rolling（转仓）
- [ ] 检查 13F 机构持仓、内部人交易动向

### 3.8 Rolling（转仓）技巧

当行情走反时，可以"滚动"期权到更远到期日或不同行权价：

```
情景：你 Sell Put $165，AAPL 跌到 $155

Rolling 操作：
1. 买回 $165 Put（亏损）
2. 同时卖出更远到期日的 $160 Put（收取新权利金）
3. 新权利金应 > 买回亏损，实现净收入
4. 给股票更多时间恢复

注意：Rolling 有交易成本，且不保证总能盈利
```

### 3.9 AI 辅助期权交易工具

| 工具 | 功能 | 价格 |
|------|------|------|
| **QuantWheel** | AI 期权筛选、最佳 Roll 推荐、经纪商集成 | 付费 |
| **Tiblio** | 自然语言交易指令、实时筛选、Wheel 策略自动化 | 付费 |
| **OptionsPilot** | 免费的 Covered Call / CSP 计算器、AI 行权价推荐 | 免费 |
| **Jenova AI** | AI 期权收入策略 Agent，系统化权利金收集 | 付费 |

---

## 4. 三者的融合：AI 辅助交易系统设计思路

### 4.1 统一框架

这三个方向本质上可以融合为一个 **AI 辅助交易决策系统**：

```
┌───────────────────────────────────────────────────────┐
│                  AI 辅助交易决策系统                    │
├───────────────────────────────────────────────────────┤
│                                                       │
│  信息层（你已有的基础设施）                             │
│  ├── TrendRadar：热点监控 & 信息流                     │
│  ├── alpha-radar：股票 AI 多 Agent 分析                │
│  ├── Quantify-Binance-Test：量化预测引擎              │
│  └── trading-learning：知识沉淀                       │
│                                                       │
│  策略层（三大方向）                                    │
│  ├── Hedge Fund 模式：多 Agent 集成决策               │
│  │   └── Elo 竞争 + 动态策略轮换                     │
│  ├── Polymarket 模式：预测市场自动交易                │
│  │   └── 跟单 + 概率模型 + 强化学习                  │
│  └── 期权模式：Sell Put / Covered Call / Wheel        │
│      └── AI 优化行权价 + 到期日 + Roll 时机           │
│                                                       │
│  执行层                                               │
│  ├── Paper Trading（模拟验证）                        │
│  ├── 小资金实盘（信号验证）                           │
│  └── 全自动执行（高度成熟后）                         │
│                                                       │
│  反馈层                                               │
│  ├── 绩效评估 & 策略复盘                              │
│  ├── Agent Elo 动态调整                               │
│  └── 强化学习持续优化                                 │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### 4.2 AI 辅助的核心价值

| 传统人工 | AI 辅助 |
|---------|---------|
| 情绪化决策 | 规则驱动，消除情绪偏差 |
| 信息处理有限 | 每分钟处理数千条新闻/推文/数据 |
| 单一视角 | 多 Agent 多视角集成判断 |
| 睡觉时错过机会 | 24/7 不间断监控和执行 |
| 事后复盘困难 | 每笔交易完整审计追踪 |
| 策略固化 | Elo 竞争实现动态策略轮换 |

### 4.3 推荐行动计划

**本周**：
1. Clone `virattt/ai-hedge-fund`，本地跑通理解架构
2. 注册 Polymarket 账户，熟悉 API 文档（docs.polymarket.com）
3. 开始学习期权基础（Wheel 策略完整逻辑）

**本月**：
4. 搭建 Polymarket Paper Trading Agent（PostgreSQL + 15 分钟 Cron）
5. 将 TrendRadar 信号接入交易决策流程
6. 在 OptionsPilot（免费）上模拟练习 Wheel 策略

**本季度**：
7. 构建多 Agent 集成交易系统原型
8. 积累 100+ 笔 Paper Trading 数据
9. 评估策略表现，准备小资金实盘

---

## 参考资源

### Hedge Fund
- [virattt/ai-hedge-fund](https://github.com/virattt/ai-hedge-fund) - ⭐ 45.9K，最流行的开源 AI 对冲基金
- [LLMQuant/Magents](https://github.com/llmquant/magents) - 多策略对冲基金回测框架
- [Axelrod AI Fund](https://axr.aixvc.io/) - AI-native 链上对冲基金
- [AI NeuroSignal Case Study](https://nicchin.com/case-studies/trading-ai) - 20-Agent 集成系统

### Polymarket
- [Polymarket API Docs](https://docs.polymarket.com/) - 官方 API 文档
- [vladmeeros/polymarket-copytrading-bot](https://github.com/vladmeeros/polymarket-copytrading-bot) - Rust 跟单机器人
- [ducksybils/polymarket-copytrading](https://github.com/ducksybils/polymarket-copytrading) - Python 跟单机器人
- [Paper Trading Guide](https://brainroad.com/polymarket-on-autopilot-let-your-ai-agent-paper-trade-prediction-markets/) - 模拟交易完整指南

### 期权
- [QuantWheel Wheel Strategy Guide](https://quantwheel.com/learn/wheel-strategy/) - Wheel 策略详解
- [OptionsPilot](https://optionspilot.app/) - 免费 Covered Call / CSP 计算器
- [Wheel Strategy Options](https://wheelstrategyoptions.com/) - 期权收入策略详解
