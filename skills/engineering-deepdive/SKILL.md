---
name: engineering-deepdive
version: 0.1.0
description: "从零写一篇技术深度剖析（deep-dive）—— 用「问题驱动」组织、「总→分→合」分段、「朴素方案 ✗ vs 真实答案 ✓」对比卡片、SVG 图做信息压缩、每节末尾「核心亮点」callout 提炼工程主张。当用户要求「深度剖析 X」「写一篇 X 的 deep-dive」「engineering writeup」「架构分析」「design doc」「技术博客」「code walkthrough」「code 深度阅读」，或要求把一段代码 / 一个子系统 / 一项架构决策写成中长篇可读文章时使用。与 humanize-doc 的区别：humanize-doc 是「改写已有文档去 AI 味」，本 skill 是「从零构思和写作」。两个 skill 可以串联：先用本 skill 写初稿，再用 humanize-doc 抛光语气。"
---

# Engineering Deepdive — 写一篇能让人读完的技术深度剖析

把一段代码 / 一个子系统 / 一项架构决策，写成一篇<strong>既讲清机制、又传递工程哲学</strong>的中长篇文章。读完的人不仅"知道它怎么实现"，还能拿走"为什么这么做"和"我以后该怎么想这类问题"。

## 何时触发

- 用户说「深度剖析」「deep-dive」「架构分析」「engineering writeup」「design doc」「code walkthrough」「技术博客」「代码深度阅读」。
- 用户要解释某段代码 / 某个机制 / 某个 trade-off 的来龙去脉。
- 用户要「配图」「做架构图」「流程图」「图文并茂」的技术文档。
- 篇幅中长（1000+ 字）—— 不是 README、不是 API doc、不是简短问答。

**不要用于**：
- 改写既有文档的语气 —— 那是 humanize-doc 的活
- 简短问答、代码注释、API 参考文档
- 入门教程（这套风格预设读者有工程背景）

## 角色设定（写在产出前）

> 你是写过 Brendan Gregg / Julia Evans / Russ Cox / Dan Luu 风格博客的人。你习惯：把"为什么"放在"是什么"之前；用具体的代码片段、文件路径、行号、commit hash 锚定每个断言；不堆功能、不列特性，讲取舍；一张图能替代两百字 prose 就坚决画图。读者是聪明的工程师，不需要科普 Python 是什么，但也未必读过这段代码 —— 你的工作是<strong>把你脑子里的理解 transmit 给他</strong>。

## 五个核心写作模式

### 1. 问题驱动的组织（替代功能列表）

**不要这么写**："这个模块有 5 个功能：A / B / C / D / E。下面分别介绍……"

**这么写**："一个长期跑的 agent 该怎么记事？这是这个子系统在回答的问题。下面 10 个子问题逐层把答案拆开。"

差别：功能列表把负担推给读者（你得自己整理出主线），问题驱动把主线焊在文章骨架上。

操作要点：
- 章节标题写成问句（"Q1: ...?" 或 "怎么 X？"）
- 每个问题先列<strong>朴素方案为什么不够</strong>，再给<strong>真实采用的方案</strong>
- 问题之间有 narrative flow（前一问的答案自然引出后一问）

详见 [`references/structure.md`](references/structure.md)。

### 2. 总→分→合的三段结构

骨架：

| 段 | 长度 | 目标 |
|----|------|------|
| 总 | 1-2 段 + 1 张总览图 | 读者读完 2 分钟脑子里就有全图 |
| 分 | 主体（70-80% 篇幅） | 每个核心问题一节，深入挖 |
| 合 | 1 张数据流整合图 + 一句话总结 + 3-5 条隐性契约 | 把所有细节重新串成一条线，提炼出能推广的工程主张 |

**合的部分不是"再总结一遍"**，而是把读者已经吸收的细节<strong>抽象到模式层</strong>。同样一份代码，读者拿走"这套设计在解决 prefix cache 经济学"和"这套设计支持 frozen snapshot"是两种完全不同的内化深度。

### 3. 朴素方案 ✗ vs 真实答案 ✓ 的对比卡片

讲一个设计决策时，先列 2-3 个"看起来合理的朴素方案"和它们的失败模式，再给真实采用的方案。这做两件事：

1. **让读者明白这个问题非平凡** —— 不展示朴素方案的失败，读者会觉得设计过度工程化
2. **让"答案"显得有说服力** —— reader 已经知道朴素方案不行，对真实方案的接受度大幅提高

样式：见 [`assets/template.html`](assets/template.html) 里的 `.naive`（红 ✗）和 `.answer`（绿 ✓）类。markdown 形式：

```markdown
<div class="naive">
  <div class="title">方案 A · 把所有对话喂回 context</div>
  <div class="why">几轮就爆 context window；compression 之后旧事实丢失。</div>
</div>

<div class="answer">
  <div class="title">真实方案：两条物理通道</div>
  <p>核心事实走静态层、语义召回走动态层，物理上分开。</p>
</div>
```

### 4. 图表做信息压缩

一张图能替代 200 字 prose 才有价值。常用五种 figure 类型：

| 类型 | 用途 | 例子 |
|------|------|------|
| 架构总览 | 模块 / 通道 / 边界 | 双轨记忆架构（哪些模块、谁注入到哪） |
| 时序图 / swimlane | 多 actor 的协作时序 | 一轮 turn 内各钩子何时触发 |
| 状态机 | 有 N 个状态 + 转换的逻辑 | StreamingScrubber 的 OUTSIDE/IN_SPAN |
| 决策树 | 多个触发条件 → 不同分支行为 | on_session_switch 的 reset 二分 |
| 对比图 | 两个方案的并排比较 | 1 个 provider vs 2 个 provider 的 schema bloat |

**SVG 而非 mermaid** —— 控制力强、跨平台稳定、可以在 Python 里用 cairosvg 离线渲染自查布局。

详见 [`references/visual-grammar.md`](references/visual-grammar.md)。

### 5. 核心亮点反射（每节末尾的 callout）

每节结尾加一个 callout，不停留在"这段代码做了什么"，提炼出<strong>"这个决策能教别人什么"</strong>。

模板：

> **\[一句话工程主张\]**：\[这个决策具体做了什么的 1 句重述\]。\[这个主张能推广到哪些其他场景\]。

例：

> **Reader-free 的并发安全**：大部分文件并发方案让 reader 也加锁。Hermes 让原子 rename 保证可见性的离散性，reader 不需要协议参与。这把"并发协议复杂度"封闭在 writer 路径上 —— 外部任何 process 都能 `cat` 文件而不需要懂锁约定。这种"把协议复杂度封闭在最少代码路径"的模式可以用在所有 "多读者-单写者" 文件设计里。

## 操作流程

1. **材料收集**（30% 时间）
   - `grep` / `cat` / `git log` 把要写的代码段全部拉过来读一遍
   - 找 commit history / issue 引用 / 注释 —— 理出"为什么这么写"的痕迹
   - 实测数字（`wc -l`、tests count、SVG 渲染验证）—— 不要凭印象写。"约 500 行" 永远不如 "609 行"。

2. **拟提纲**（10%）
   - 列 5-10 个核心问题。最好的核心问题是<strong>读者真的会问、但官方文档不答</strong>的那种。
   - 排顺序：每个问题的答案能引出下一个问题。
   - 第一问通常是"朴素方案为什么不够"，最后一问通常是"什么时候这套设计自己也会崩"。

3. **写总**（10%）
   - 一句话讲完整套设计 + 一张总览图。读者看完这两分钟就能在脑子里建立全图。
   - 总览图三选一：物理布局 / 数据流 / 模块关系。

4. **逐题展开**（30%）
   - 每节：问题描述 → 朴素方案卡片 → 真实方案卡片 → 可选配图 → 核心亮点 callout
   - 中间不要插 "首先 / 其次 / 综上 / 不难看出"

5. **写合**（10%）
   - 数据流单图把所有节点重新串一遍（推荐 swimlane 时序图）
   - 一句话总结
   - 3-5 个隐性契约 / 设计哲学反射

6. **HTML 渲染**（如需，10%）
   - 复制 [`assets/template.html`](assets/template.html) 的 CSS
   - 用 cairosvg 渲染每张 SVG 检查布局（见 visual-grammar.md）
   - 一图错位时立即修，不要等读者看到

## 进阶选项

- **HTML + SVG 渲染**：用 `assets/template.html` 作为起点
- **PDF 输出**：可以用 weasyprint 把 HTML 转 PDF
- **mermaid 替代 SVG**：如果目标是 GitHub README（mermaid 原生渲染），用 mermaid；否则 SVG 控制力强
- **emoji 用量**：技术 essay 一般 0-3 个，多了显得不严肃。⚡ ⚠ 📌 这种<strong>语义化的符号</strong>比 🚀 🎯 ✨ 这种<strong>装饰性 emoji</strong>更适合
- **中英混排**：术语和代码用英文（`PluginManager`, `flock`, `os.replace`），prose 用中文
- **与 humanize-doc 串联**：写完初稿后跑一遍 humanize-doc 抛光语气，特别是去掉残留的"旨在 / 助力 / 综上"

## 反例与正例（片段对照）

### 反例（功能列表式 + AI 套话）

> Hermes 的 memory 子系统是一个**强大且灵活**的解决方案，旨在为 agent 提供全方位的记忆能力。它具有以下五大优势：
> 1. 持久化存储
> 2. 跨会话召回
> 3. 多 provider 支持
> 4. 流式输出过滤
> 5. ...
>
> 综上所述，这是一个**面面俱到**的系统。

读完反应："好的，记下了。" 但读者其实没真正理解这个系统在干什么、为什么这么设计。

### 正例（问题驱动 + 具体锚定）

> **Q1: 一个跑一年的 personal agent，记忆应该长什么样？**
>
> <div class="naive">方案 A · 把所有对话喂回 context —— 几轮就爆 context window，compression 之后旧事实丢失，用户问"我上周说过我喜欢..."模型回答不出来。</div>
>
> <div class="naive">方案 B · 全部走 RAG —— 关键事实（用户名字、当前项目根目录）每次都要召回，但 recall 召回不稳定，偶尔命中偶尔漏。</div>
>
> <div class="answer">Hermes 的观点：两类记忆走两条通道。"总是有效"（用户偏好、项目根目录）走<strong>静态层</strong>，进 system prompt；"按需召回"（具体对话、过去某次调试经过）走<strong>动态层</strong>，prefetch 时进 user message。物理上分开，逻辑上不互相污染。</div>
>
> **核心亮点**：Hermes 不试图发明一个"统一的记忆系统"。它承认<strong>不同类型的记忆有不同的访问模式</strong>，物理层面就把它们分开 —— 后续所有工程取舍都站在这个分类基础上。

读完反应："噢，原来是这样选的。下次我也可以这么想问题。"

## 一句话记忆

**别罗列功能，提出问题；别只讲机制，提炼主张。每张图替代两百字 prose，每节结尾问一句"这能教别人什么"。**
