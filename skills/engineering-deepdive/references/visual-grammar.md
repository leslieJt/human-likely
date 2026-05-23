# 图表语法：SVG 设计 + 渲染自查

技术 essay 的图不是装饰 —— 是<strong>信息压缩</strong>。一张好图要替代两百字 prose 才有放进去的价值。这份 reference 讲怎么设计、用什么类型、画完怎么自查。

## 一、SVG 还是 mermaid？

| 场景 | 选 SVG | 选 mermaid |
|------|--------|-----------|
| HTML 输出，要精确控制布局 | ✓ | |
| GitHub README，要原生渲染 | | ✓ |
| 复杂多色 / 跨象限的布局 | ✓ | |
| 简单流程图 / 类图 | | ✓ |
| 要离线渲染自查 | ✓ (用 cairosvg) | ✗ |
| 草稿快速验证想法 | | ✓ |

**默认选 SVG**，因为：
1. 控制力强 —— 颜色、字体、坐标都精确
2. 跨平台稳定 —— mermaid 在不同渲染器下表现不一致
3. 可以用 cairosvg / inkscape 离线渲染做布局自查（mermaid 的 mmdc 也行但慢）

## 二、五种 figure 类型 + 选型

### 类型 1：架构总览

**用途**：展示物理布局 —— 哪些模块、谁连谁、边界在哪。

**典型场景**：篇章开头那张"一图概括"。

**关键元素**：
- 用矩形分组（不同颜色区分逻辑层）
- 箭头用<strong>不同颜色</strong>区分数据流类型（同步 / 异步 / 失败兜底）
- 顶层一般是入口、底层是出口或外部依赖

**反模式**：每个箭头都同色 → 视觉上看不出哪条线是关键路径

### 类型 2：时序图 / Swimlane

**用途**：多个 actor 协作时，每个 actor 在做什么、什么时候做。

**典型场景**：篇章结尾那张"数据流整合图"。

**关键元素**：
- **X 轴严格是时间** —— 从左到右单调递增。一个事件不能放在它的前置事件左边。
- Y 轴每个 actor 一个泳道，labeled at far left
- 时间节点 T1 / T2 / ... 标在底部时间轴上
- 跨泳道的箭头表示"actor A 通知 actor B"

**反模式**：把"循环 N 次"画成时间轴上的 N 个矩形 → 视觉太长。改成 "tool loop (N×)" 一个矩形里说明就行。

### 类型 3：状态机

**用途**：有限状态 + 转移条件的逻辑。

**典型场景**：StreamingScrubber 这种跨 chunk 的状态机；缓存的 PENDING/READY/DELIVERED；连接的 CONNECTING/CONNECTED/CLOSED。

**关键元素**：
- 圆形表示状态（颜色区分语义：绿=正常、红=错误、黄=中间）
- 箭头标转移条件（用 rect 包住标签让它在曲线箭头上不被遮挡）
- 自环用 quadratic curve 画出来
- 区分初始 state 和终止 state

**反模式**：状态太多（>5 个）→ 拆成两张图，或者把无关 state 折叠。

### 类型 4：决策树

**用途**：多个触发条件 → 不同结果分支。

**典型场景**：on_session_switch 的 reset 二分；阈值 lookup 的优先级链。

**关键元素**：
- 上下分层：触发事件在顶部，决策在中间，结果在底部
- 用不同颜色区分分支（在 on_session_switch 例子里：reset=True 一色、reset=False 一色）
- 同一层的元素水平对齐

### 类型 5：对比图

**用途**：两个或多个方案的并排比较。

**典型场景**："1 个 provider vs 2 个 provider"、"frozen snapshot vs 动态 system prompt"。

**关键元素**：
- 两列（或三列）并排
- 同一行的元素是"对比维度"
- 用相同的视觉编码（同色 = 同语义层）让对比一目了然

## 三、设计一张能压缩 200 字 prose 的图

四个原则：

### 1. 每个视觉元素都对应一句 prose

一个矩形 = 一个概念；一条箭头 = 一种关系；一种颜色 = 一种语义层。如果一个元素 reader 看不出对应什么概念，删掉。

### 2. 颜色用作语义编码，不是装饰

- 一篇文章里同一种颜色应对应同一种语义层（如：橙色 = built-in / 静态、绿色 = external / 动态、灰色 = 中间层）
- 不要因为"好看"加颜色

### 3. 文字与图形 1:1 对齐

- 矩形里的文字不能溢出
- 箭头上的标签不要遮挡箭头本身
- 跨象限的连线不要穿过矩形

### 4. 留足边距

- viewBox 边到内容至少 10px
- 同层元素之间至少 8px
- 文本基线对齐（同一行的所有文本 y 坐标一致）

## 四、cairosvg 自查流程

写完一张 SVG 后，<strong>在交付前用 Python 渲染成 PNG 自查布局</strong>。这能抓出：

- viewBox 溢出
- 文字超出矩形
- 箭头错位
- lane 标签被截断

```python
import re, cairosvg
from pathlib import Path

html = Path("output.html").read_text()
svgs = re.findall(r'(<svg[^>]*viewBox[^>]*>.*?</svg>)', html, re.DOTALL)

for i, svg in enumerate(svgs, 1):
    if 'xmlns=' not in svg:
        svg = svg.replace('<svg', '<svg xmlns="http://www.w3.org/2000/svg"', 1)
    cairosvg.svg2png(
        bytestring=svg.encode('utf-8'),
        write_to=f"/tmp/fig{i}.png",
        output_width=1200,
    )
```

然后用 image viewer / Read tool 看一遍 PNG，找布局问题。

**注意**：cairosvg 默认没有 CJK 字体，中文字会显示成方框。这<strong>只影响渲染检查</strong>，浏览器里有字体所以最终显示正常。检查时只看<strong>布局结构</strong>（矩形位置、箭头走向、是否溢出），不看文字是否清晰。

## 五、Caption 怎么写

每张图都带 caption：

```
Figure N · 标题（一句话讲这张图在做什么）
```

例：
- `Figure 1 · 双轨记忆架构总览`
- `Figure 4 · 流式围栏剥离状态机`
- `Figure 7 · 单轮 turn 内 memory 钩子的触发时序（X 轴 = 时间）`

注意：
- "Figure N" 用 mono 字体显得规整
- 标题简洁但点出关键 —— 不是 "图 7"，而是 "图 7 · 单轮 turn 内 memory 钩子的触发时序"
- 涉及容易误读的部分时标注（如 "X 轴 = 时间"）

## 六、常见 SVG 坐标坑

| 坑 | 表现 | 修复 |
|----|------|------|
| viewBox 太小 | 内容被裁掉 | 加大 viewBox 的 width/height，或缩小内容坐标 |
| 文字溢出矩形 | "MemoryManag" 而不是 "MemoryManager" | 加宽矩形、缩小字号，或用更短的标签 |
| 箭头穿过矩形 | 看起来像箭头钻进了某个 box | 用 quadratic curve 绕开，或重新排版 |
| 自环看不全 | 圆形状态机的自环没渲染完 | 检查 quadratic curve 的控制点是否在 viewBox 内 |
| 文字 baseline 不齐 | 同一行的文字高低参差 | 检查 y 坐标是否一致；text 的 y 是 baseline 不是 top |
| arrow marker 不显示 | 线画出来了但箭头没看到 | 确认 `markerUnits="strokeWidth"` 或 marker 的 refX/refY 配置正确 |

## 七、与 HTML 模板的关系

`assets/template.html` 里的 `.diagram` 类是图的容器：

```html
<div class="diagram">
  <div class="caption">Figure N · 标题</div>
  <svg viewBox="0 0 W H" xmlns="http://www.w3.org/2000/svg">
    ...
  </svg>
</div>
```

`.diagram` 会自动加白底 + 边框 + 居中 + caption 样式。直接用就好。
