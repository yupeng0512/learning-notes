# Remotion Skill：让 AI 成为视频制作专家

> 学习日期：2026-01-25
> 来源：remotion-dev/skills（Remotion 官方）+ 社区分享
> 标签：#Skill #Remotion #AI视频 #AgentSkill #领域知识
> 关联笔记：[Remotion 框架入门](../dev-tools/2026-01-06-remotion-react-video-framework.md)

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **名称** | Remotion Skill |
| **定位** | AI Agent 的视频制作领域知识包 |
| **核心价值** | 让 AI 从「懂一点 Remotion」变成「视频专家」 |
| **规模** | 30+ 规则文件，覆盖动画、音频、字幕、3D、转场等 |
| **安装方式** | `npx skills add remotion-dev/skills` |
| **兼容工具** | Claude Code / Cursor / Gemini CLI / Trae / Antigravity |
| **GitHub** | https://github.com/remotion-dev/skills |

> 💡 **一句话总结**：Remotion Skill 把专业视频知识「编码」成 AI 可理解的规则，实现「自然语言描述 → AI 生成视频代码 → 预览渲染导出」的完整闭环。

> 🎯 **Remotion 原理**：先按照你的需求做一个网站（React 组件），然后把网站的每一帧截图合并成视频。**视频 = 网站的延时摄影**。

---

## 🗺️ Skill 架构全景

```
remotion-dev/skills/
│
├── SKILL.md                    # 入口文件：触发条件 + 规则索引
│   ├── When to use: 处理 Remotion 代码时加载
│   └── How to use: 指向各规则文件
│
└── rules/                      # 30+ 领域规则
    │
    ├── 🎬 核心基础
    │   ├── animations.md       # 动画（禁用 CSS！）
    │   ├── timing.md           # 插值曲线（spring/easing）
    │   ├── sequencing.md       # 时间线排列
    │   └── compositions.md     # 视频结构定义
    │
    ├── 🎨 视觉效果
    │   ├── transitions.md      # 场景转场（fade/slide/wipe）
    │   ├── text-animations.md  # 文字动画
    │   ├── charts.md           # 数据可视化
    │   └── 3d.md               # Three.js 集成
    │
    ├── 🎵 多媒体素材
    │   ├── audio.md            # 音频处理
    │   ├── videos.md           # 视频嵌入
    │   ├── images.md           # 图片处理
    │   ├── gifs.md             # GIF 同步
    │   └── lottie.md           # Lottie 动画
    │
    ├── 📝 字幕系统
    │   ├── display-captions.md      # TikTok 风格字幕
    │   ├── import-srt-captions.md   # SRT 导入
    │   └── transcribe-captions.md   # 语音转字幕
    │
    ├── 🔧 高级功能
    │   ├── parameters.md       # Zod 参数化
    │   ├── calculate-metadata.md    # 动态元数据
    │   ├── fonts.md            # 字体加载
    │   └── tailwind.md         # Tailwind 集成
    │
    └── 📊 媒体处理（Mediabunny）
        ├── get-video-duration.md
        ├── get-audio-duration.md
        ├── get-video-dimensions.md
        ├── extract-frames.md
        └── can-decode.md
```

---

## 🔑 核心规则精华

### 规则 1：动画必须帧驱动

**Skill 的「红线规则」**：

> CSS transitions or animations are **FORBIDDEN** - they will not render correctly.
> Tailwind animation class names are **FORBIDDEN** - they will not render correctly.

```tsx
// ❌ 错误：CSS 动画不会被渲染
<div className="animate-fade-in">Hello</div>
<div style={{ transition: 'opacity 0.5s' }}>Hello</div>

// ✅ 正确：帧驱动动画
import { useCurrentFrame, useVideoConfig, interpolate } from "remotion";

export const FadeIn = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(frame, [0, 2 * fps], [0, 1], {
    extrapolateRight: 'clamp',
  });
 
  return <div style={{ opacity }}>Hello World!</div>;
};
```

**原理**：Remotion 是「逐帧渲染」，每一帧都是静态画面；CSS 动画是「时间驱动」，需要浏览器实时播放。两者机制不兼容。

---

### 规则 2：Spring 配置速查表

| 配置名 | 参数 | 效果描述 | 适用场景 |
|--------|------|----------|----------|
| **smooth** | `{damping: 200}` | 平滑无弹跳 | 字幕淡入、标题显示 |
| **snappy** | `{damping: 20, stiffness: 200}` | 快速微弹 | UI 元素、按钮出现 |
| **bouncy** | `{damping: 8}` | 明显弹跳 | 趣味动画、卡通风格 |
| **heavy** | `{damping: 15, stiffness: 80, mass: 2}` | 缓慢厚重 | Logo 落地、重物效果 |

```tsx
import { spring, useCurrentFrame, useVideoConfig } from 'remotion';

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

// 推荐：无弹跳的自然运动
const scale = spring({
  frame,
  fps,
  config: { damping: 200 },
});
```

---

### 规则 3：转场时长计算

**关键公式**：

```
总时长 = 各片段时长之和 - 各转场时长之和
```

**可视化理解**：

```
不带转场：
|--- Scene A (60帧) ---|--- Scene B (60帧) ---|
总长 = 60 + 60 = 120 帧

带 15 帧转场：
|--- Scene A (60帧) ---|
              |====|    ← 15 帧重叠
              |--- Scene B (60帧) ---|
总长 = 60 + 60 - 15 = 105 帧
```

**代码示例**：

```tsx
import { TransitionSeries, linearTiming } from '@remotion/transitions';
import { fade } from '@remotion/transitions/fade';

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={60}>
    <SceneA />
  </TransitionSeries.Sequence>
  
  <TransitionSeries.Transition 
    presentation={fade()} 
    timing={linearTiming({ durationInFrames: 15 })} 
  />
  
  <TransitionSeries.Sequence durationInFrames={60}>
    <SceneB />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

---

### 规则 4：TikTok 风格字幕

**两步实现**：

**Step 1：分页**

```tsx
import { createTikTokStyleCaptions } from '@remotion/captions';

const { pages } = createTikTokStyleCaptions({
  captions,
  combineTokensWithinMilliseconds: 1200, // 每 1.2 秒换一页
});
```

**Step 2：高亮当前词**

```tsx
const HIGHLIGHT_COLOR = '#39E508';

const CaptionPage = ({ page }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const currentTimeMs = (frame / fps) * 1000;
  const absoluteTimeMs = page.startMs + currentTimeMs;

  return (
    <div style={{ fontSize: 80, fontWeight: 'bold' }}>
      {page.tokens.map((token) => {
        const isActive = token.fromMs <= absoluteTimeMs && token.toMs > absoluteTimeMs;
        
        return (
          <span style={{ color: isActive ? HIGHLIGHT_COLOR : 'white' }}>
            {token.text}
          </span>
        );
      })}
    </div>
  );
};
```

---

### 规则 5：Zod 参数化

让视频变成「模板」，改参数就能批量生成：

```tsx
// 1. 定义 Schema
import { z } from 'zod';
import { zColor } from '@remotion/zod-types';

export const MyVideoSchema = z.object({
  title: z.string(),
  backgroundColor: zColor(),  // 颜色选择器
  duration: z.number().min(1).max(60),
});

// 2. 注册到 Composition
<Composition
  id="MyVideo"
  component={MyVideo}
  schema={MyVideoSchema}
  defaultProps={{
    title: 'Hello World',
    backgroundColor: '#000000',
    duration: 10,
  }}
  // ...
/>
```

**效果**：在 Remotion Studio 侧边栏出现可视化参数编辑器。

---

## 💡 Skill 设计洞见

### 1. 「红线规则」防止低级错误

| 问题 | Skill 的解法 |
|------|-------------|
| AI 用 CSS 动画 | 明确标注 **FORBIDDEN** |
| AI 不知道 Tailwind 动画类不行 | 明确标注 **FORBIDDEN** |
| AI 忘记安装依赖 | 每个规则都有安装命令 |

**核心思想**：**禁止比建议更有效**。告诉 AI「不能做什么」比「应该做什么」更重要。

---

### 2. 「代码即文档」

Skill 的每个规则都是：

```markdown
1. 简短说明（1-2 行）
2. 完整代码示例
3. 变体/高级用法
```

**效果**：AI 可以直接「抄」代码，而不是「理解后再写」。

---

### 3. 「包管理器适配」

每个需要安装依赖的规则都提供 4 种命令：

```bash
npx remotion add @remotion/transitions   # npm
bunx remotion add @remotion/transitions  # bun
yarn remotion add @remotion/transitions  # yarn
pnpm exec remotion add @remotion/transitions  # pnpm
```

**效果**：AI 能自动检测项目用的包管理器，运行正确的命令。

---

## 🎯 使用工作流

```
1. 安装 Skill
   npx skills add remotion-dev/skills
                    ↓
2. 在 AI 工具中描述需求
   "做一个产品介绍视频，3 秒开场动画，展示 3 个功能点"
                    ↓
3. AI 加载 Skill，生成 Remotion 代码
   - 自动用 useCurrentFrame() 做动画
   - 自动用 spring() 做弹性效果
   - 自动用 TransitionSeries 做转场
                    ↓
4. 本地预览（3000 端口）
   npm run dev → 打开 Remotion Studio
                    ↓
5. 不满意？继续调整
   "把弹性效果改成更柔和的"
                    ↓
6. 导出视频
   File → Export → MP4/GIF/WebM
```

---

## 📊 Skill vs 框架笔记 对比

| 维度 | 框架笔记（2026-01-06） | Remotion Skill |
|------|------------------------|----------------|
| **定位** | 人类学习入门 | AI 领域知识包 |
| **阅读者** | 开发者 | AI Agent |
| **内容形式** | 概念讲解 + 示例 | 规则 + 完整代码 |
| **使用方式** | 阅读 → 理解 → 手写代码 | AI 加载 → 直接生成代码 |
| **覆盖深度** | 核心 3 个 API | 30+ 细分场景 |
| **关系** | 「我学会」 | 「AI 学会」 |

---

## 🎬 适合的视频类型

| 类型 | 适合度 | 说明 |
|------|--------|------|
| **产品宣传视频** | ⭐⭐⭐⭐⭐ | 动态图形 + 文字动画 |
| **数据可视化** | ⭐⭐⭐⭐⭐ | 图表动画 + 数字滚动 |
| **技术演示** | ⭐⭐⭐⭐⭐ | 代码高亮 + 终端动画 |
| **短视频/TikTok** | ⭐⭐⭐⭐ | 字幕高亮 + 快节奏转场 |
| **教程视频** | ⭐⭐⭐⭐ | 分段讲解 + 动画演示 |
| **实拍剪辑** | ⭐⭐ | 不适合，用 Premiere |
| **AI 生成内容** | ⭐ | 不适合，用 Sora |

---

## ✅ 行动清单

### 立即可做

- [ ] 安装 Skill：`npx skills add remotion-dev/skills`
- [ ] 用 AI 生成一个简单视频，验证 Skill 效果
- [ ] 对比有/无 Skill 时 AI 生成代码的质量差异

### 短期实践

- [ ] 用 Skill 做一个真实的产品宣传视频
- [ ] 研究 Skill 的规则文件，学习「领域知识编码」方法
- [ ] 尝试给其他框架写类似的 Skill

### 长期提升

- [ ] 关注 Remotion Skill 的更新（30+ 规则会持续增加）
- [ ] 用 Skill 思维封装自己的领域知识

---

## 📋 核心要点（5 条）

1. **Skill = AI 的领域专家**：把 Remotion 专业知识编码成规则，让 AI 一秒变专家
2. **红线规则是关键**：禁止 CSS 动画、禁止 Tailwind 动画类，防止 AI 犯低级错误
3. **代码即文档**：每个规则都有完整可运行的代码，AI 直接抄作业
4. **30+ 规则覆盖全场景**：动画、转场、字幕、音频、3D、参数化，应有尽有
5. **框架笔记 + Skill 互补**：人学框架笔记，AI 用 Skill，各司其职

---

## 💬 金句摘录

> "make videos just with Claude Code"
> — Remotion 官方

> "Skill 就像给 AI 一本专业速查手册，让它从「懂一点」变成「专业级」。"

> "禁止比建议更有效——告诉 AI「不能做什么」比「应该做什么」更重要。"

---

## 🔗 延伸阅读

- [Remotion Skill GitHub](https://github.com/remotion-dev/skills)
- [Remotion 框架入门笔记](../dev-tools/2026-01-06-remotion-react-video-framework.md)
- [Anthropic Building Agents with Skills](./2026-01-25-anthropic-building-agents-with-skills.md)
- [Remotion 官方文档](https://www.remotion.dev/docs/)
