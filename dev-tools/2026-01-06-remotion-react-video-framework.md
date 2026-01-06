# Remotion：用 React 写视频的开源框架

> 学习日期：2026-01-06
> 来源：鲁工公众号文章
> 标签：#Remotion #React #视频制作 #前端 #VibeCoding

## 📖 快速概览

| 项目 | 内容 |
|------|------|
| **名称** | Remotion |
| **类型** | 开源框架（特殊许可证） |
| **核心理念** | 用写 React 网页的方式写视频 |
| **技术栈** | React + TypeScript（77.6%）+ Rust（渲染引擎） |
| **GitHub** | https://github.com/remotion-dev/remotion |
| **Star 数** | 25.2k ⭐ |
| **贡献者** | 296 人 |
| **最新版本** | v4.0.400（2026年1月5日） |

> 💡 **一句话总结**：Remotion 把视频变成了 React 组件——用 `useCurrentFrame()` 控制动画，用 `<Sequence>` 控制时间线，最后 `npx remotion render` 导出 MP4。**视频就是代码，代码就是视频。**

---

## 🗺️ 架构全景

```
┌─────────────────────────────────────────────────────────────┐
│                    Remotion 架构全景                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  开发者视角                                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ React 组件 + TypeScript                              │   │
│  │ - useCurrentFrame()  获取当前帧                      │   │
│  │ - <Sequence>         控制出场时机                    │   │
│  │ - interpolate()      动画插值                        │   │
│  │ - spring()           物理弹簧效果                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  Remotion Studio（实时预览）                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ npm run dev → 打开本地工作室                         │   │
│  │ - 时间线拖拽                                         │   │
│  │ - 实时预览                                           │   │
│  │ - 参数调整                                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  渲染引擎                                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 本地渲染：npx remotion render                        │   │
│  │ 云端渲染：Lambda / Cloud Run                         │   │
│  │ 输出格式：MP4 / GIF / WebM / PNG 序列                │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔑 三大核心 API

| API | 作用 | 类比理解 |
|-----|------|----------|
| **useCurrentFrame()** | 获取当前帧数（0, 1, 2, ...） | 动画的"时钟" |
| **Sequence** | 控制元素出场时机和持续时长 | 时间线上的"片段" |
| **interpolate / spring** | 动画插值和物理效果 | 缓动曲线生成器 |

---

## 💡 核心洞见

### 1. 视频 = 函数（帧数）

- 传统视频：时间线 + 关键帧 + 手动调整
- Remotion：`f(frame) → 画面`，纯函数式思维
- **类比**：从"画动画"变成"算动画"

### 2. React 组件 = 视频片段

- 组件可复用、可组合、可参数化
- 用 props 传递数据，用 state 管理状态
- **类比**：像搭乐高一样搭视频

### 3. 代码 = 可版本控制的视频

- 视频逻辑进 Git，可 diff、可 review
- 批量生成只需改参数
- **类比**：视频也能 CI/CD

### 4. Web 技术全栈可用

- CSS、Canvas、SVG、WebGL 全支持
- 任何 npm 包都能用
- **类比**：前端能做的，视频里都能做

---

## 📚 深度讲解

### 1. useCurrentFrame() 工作原理

视频的本质是一帧一帧的静态画面。`useCurrentFrame()` 告诉你"现在是第几帧"。

```javascript
import { useCurrentFrame } from "remotion";

export const MyAnimation = () => {
  const frame = useCurrentFrame();
  
  // frame = 0, 1, 2, 3, ... 随时间递增
  const x = frame * 2;           // 每帧向右移动 2px
  const opacity = frame / 60;    // 60帧内从 0 渐变到 1
  const rotation = frame * 3;    // 每帧旋转 3 度
  
  return (
    <div style={{
      transform: `translateX(${x}px) rotate(${rotation}deg)`,
      opacity: Math.min(1, opacity),
    }}>
      Hello Remotion!
    </div>
  );
};
```

**核心思想**：**动画 = f(frame)**

### 2. Sequence 控制时间线

```javascript
import { Sequence } from 'remotion';

export const MyVideo = () => {
  return (
    <>
      {/* 第 0 帧开始，持续 30 帧 */}
      <Sequence from={0} durationInFrames={30}>
        <div>第一段文字</div>
      </Sequence>
      
      {/* 第 30 帧开始，持续 30 帧 */}
      <Sequence from={30} durationInFrames={30}>
        <div>第二段文字</div>
      </Sequence>
      
      {/* 第 60 帧开始，持续 30 帧 */}
      <Sequence from={60} durationInFrames={30}>
        <div>第三段文字</div>
      </Sequence>
    </>
  );
};
```

**时间线可视化**：

```
帧:     0    30   60   90
        |-----|-----|-----|
文字1:  [=====]
文字2:        [=====]
文字3:              [=====]
```

### 3. interpolate vs spring

| 特性 | interpolate | spring |
|------|-------------|--------|
| **控制方式** | 手动定义输入/输出范围 | 物理参数（阻尼、刚度） |
| **效果** | 线性或自定义曲线 | 自然弹性效果 |
| **适用场景** | 精确控制 | 自然过渡 |
| **过冲** | 无 | 有（可配置） |

```javascript
// interpolate 示例
const opacity = interpolate(
  frame,           // 输入值
  [0, 30],         // 输入范围
  [0, 1],          // 输出范围
);

// spring 示例
const scale = spring({
  frame,
  fps,
  config: {
    damping: 200,    // 阻尼
    stiffness: 100,  // 刚度
  },
});
```

---

## 🎯 Vibe Video 工作流

```
1. 自然语言描述
   "我想做一个数据可视化视频，展示用户增长曲线"
                    ↓
2. Claude Code 生成代码
   // Claude 生成的 Remotion 组件
   export const UserGrowthChart = () => { ... };
                    ↓
3. Remotion Studio 预览
   npx remotion studio
   实时预览效果，不满意继续调整
                    ↓
4. 渲染导出
   npx remotion render
   输出 MP4 / GIF
```

**示例 Prompt**：

```
Remotion 是一个基于 React 和 TypeScript 构建视频的开源框架。
请使用 Context7 MCP 仔细阅读 Remotion 官方文档，然后：

1. 创建一个 60 秒的技术演示视频
2. 主题：Vibe Coding
3. 内容包括：
   - 开场标题动画（3秒）
   - 3 个核心概念介绍（每个 15 秒）
   - 结尾 CTA（5 秒）
4. 风格：深色背景、代码高亮、弹性动画
```

---

## 🎬 适用场景对比

### 适合的场景

| 场景 | 原因 | 示例 |
|------|------|------|
| **数据可视化** | 数据驱动，代码生成 | 图表动画、仪表盘 |
| **批量生成** | 参数化，一套代码多个视频 | 电商产品介绍 |
| **技术演示** | 代码高亮、终端动画 | 教程、Demo |
| **个性化视频** | 传入不同数据 | 年度总结、证书 |
| **动态图形** | CSS/Canvas/WebGL | Logo 动画、片头 |

### 不适合的场景

| 场景 | 原因 | 推荐方案 |
|------|------|----------|
| **实拍剪辑** | 需要大量素材处理 | Premiere / DaVinci |
| **复杂特效** | AE 插件生态更丰富 | After Effects |
| **AI 生成内容** | 需要 DiT 模型 | Sora / 可灵 |
| **实时直播** | 不是为实时设计 | OBS / vMix |

### 工具对比

| 工具 | 优势 | 劣势 |
|------|------|------|
| **Remotion** | 代码控制、批量生成、版本管理 | 学习曲线、设计依赖 |
| **Premiere** | 素材剪辑、专业调色 | 手动操作、无法批量 |
| **After Effects** | 特效丰富、插件生态 | 复杂、渲染慢 |
| **Sora/可灵** | AI 生成、创意无限 | 不可控、成本高 |

---

## 🚀 渲染方式

### 本地渲染

```bash
npx remotion render
```

| 优点 | 缺点 |
|------|------|
| 简单直接 | 受本地硬件限制 |
| 无需配置 | 长视频渲染慢 |
| 免费 | 无法并行 |

### 云端渲染（Lambda / Cloud Run）

```javascript
import { renderMediaOnLambda } from "@remotion/lambda";

const result = await renderMediaOnLambda({
  region: "us-east-1",
  functionName: "remotion-render",
  composition: "MyVideo",
  inputProps: { data: myData },
});
```

| 优点 | 缺点 |
|------|------|
| 并行处理 | 需要配置云服务 |
| 无限扩展 | 有成本 |
| 批量高效 | 冷启动延迟 |

### 选择建议

| 场景 | 推荐方案 |
|------|----------|
| 开发调试 | 本地渲染 |
| 单个视频 | 本地渲染 |
| 批量生成（10+） | 云端渲染 |
| 用户触发生成 | 云端渲染 |

---

## ✅ 行动清单

### 立即可做（今天）

- [ ] 创建第一个 Remotion 项目：`npx create-video@latest`
- [ ] 运行 `npm run dev`，打开 Remotion Studio
- [ ] 修改 Hello World 模板，理解 `useCurrentFrame()`

### 短期实践（本周）

- [ ] 用 `Sequence` 做一个多段落的视频
- [ ] 尝试 `spring` 效果，做一个弹性动画
- [ ] 用 Claude Code 生成一个完整的视频组件

### 长期提升（持续）

- [ ] 阅读官方文档的高级主题（音频、参数化视频）
- [ ] 尝试云端渲染（Lambda）
- [ ] 做一个真实项目：技术演示视频 / 数据可视化

---

## 📋 核心要点（5 条）

1. **Remotion = React + 视频**：用写网页的方式写视频，前端开发者零学习成本
2. **三大核心 API**：`useCurrentFrame()`（获取帧）、`Sequence`（控制时间线）、`interpolate/spring`（动画效果）
3. **视频 = 函数（帧数）**：每一帧的画面都是计算出来的，纯函数式思维
4. **适合场景**：数据可视化、批量生成、技术演示、个性化视频
5. **Vibe Video**：自然语言描述 → Claude Code 生成代码 → Remotion 渲染视频

---

## 💬 金句摘录

> "视频就是代码，代码就是视频。"

> "Remotion 把视频制作变成了编程问题，Claude Code 把编程问题变成了 Vibe 问题。"

> "对于前端开发者来说，这简直是降维打击。"

---

## 🔗 延伸阅读

- [Remotion 官方文档](https://www.remotion.dev/docs/)
- [Remotion GitHub](https://github.com/remotion-dev/remotion)
- [Remotion 模板库](https://www.remotion.dev/templates)
- [Motion Canvas](https://motioncanvas.io/)（另一个代码驱动的视频框架）
