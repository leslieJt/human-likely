# human-likely

> 让 Claude 把技术文档写得像「资深工程师写给同事看」——有取舍、有踩坑、有担当，**没有 AI 套话**。

一个 [Claude Code Plugin](https://code.claude.com/docs/en/plugins.md)，目前包含两个互补的 Skill：

| Skill | 触发场景 | 形态 |
|-------|---------|------|
| [`humanize-doc`](skills/humanize-doc/SKILL.md) | 改写 / 润色 / 重写既有技术文档；抱怨"太像 AI 写的""说人话""像资深工程师那样写" | **抛光** |
| [`engineering-deepdive`](skills/engineering-deepdive/SKILL.md) | 从零写技术深度剖析、deep-dive、架构分析、design doc、engineering writeup、code walkthrough | **生成** |

两个 skill 互补：先用 `engineering-deepdive` 写问题驱动 + 总分合 + 配图的初稿，再用 `humanize-doc` 抛光语气、去残留套话。

---

## 安装

### 方式一：从这个仓库（推荐）

把仓库注册为一个**单插件 marketplace**，再装插件：

```bash
# 1. 添加 marketplace（会读取 .claude-plugin/marketplace.json）
claude plugin marketplace add https://github.com/leslieJt/human-likely

# 2. 安装插件
claude plugin install human-likely@human-likely
```

之后在任意项目里启动 `claude`，输入要改写的文档即可。Claude 会根据 Skill 的 `description` 自动启用，也可以显式触发：

```
/human-likely:humanize-doc
```

### 方式二：本地开发

clone 之后用 `--plugin-dir` 直接挂载（不走 marketplace，便于改完即时生效）：

```bash
git clone https://github.com/leslieJt/human-likely.git
cd /your/project
claude --plugin-dir /path/to/human-likely
```

### 方式三：手动安装为用户级 Skill

如果暂时不想走 plugin 流程，把 `skills/humanize-doc/` 复制到 `~/.claude/skills/` 即可——Skill 本身不依赖 plugin 上下文。

---

## 目录结构

```
human-likely/
├── .claude-plugin/
│   ├── plugin.json          # 插件清单
│   └── marketplace.json     # 单插件 marketplace 入口
├── skills/
│   ├── humanize-doc/
│   │   └── SKILL.md
│   └── engineering-deepdive/
│       ├── SKILL.md
│       ├── references/
│       │   ├── structure.md      # 问题驱动 + 总分合 详细展开
│       │   └── visual-grammar.md # 5 种 figure 类型 + cairosvg 自查
│       └── assets/
│           └── template.html     # 实战 HTML 模板（CSS + 占位结构）
├── README.md
├── CHANGELOG.md
└── LICENSE
```

遵循 [Claude Code Plugin Reference](https://code.claude.com/docs/en/plugins-reference.md) 的标准布局——组件目录在仓库根，**只有** `plugin.json` 和 `marketplace.json` 在 `.claude-plugin/` 下。

---

## 设计原则

两个 skill 共享一套写作哲学：

1. **严谨** — 技术内核（接口、字段、量级、复杂度）不能丢。具体路径、行号、commit hash 比泛指有价值得多。
2. **人味** — 把"决策"和"权衡"写出来。每个非平凡设计背后都要展示朴素方案的失败模式。
3. **去机械化** — 删掉"综上所述""赋能""全方位"等 AI 套话。短句、具体、不连接词。
4. **平衡** — 专业但不端着，口语但不轻浮。技术 essay 里 0-3 个 emoji，多了显得不严肃。

`humanize-doc` 把这些用作改写既有文档的 checklist；`engineering-deepdive` 把这些编进新文章的<strong>骨架</strong>—— 用问题驱动 + 总分合的结构、朴素方案 vs 答案的对比卡片、每节末尾的核心亮点 callout、把"决策痕迹"焊在篇章组织里。

---

## 贡献

欢迎 PR：
- 新增 Skill：在 `skills/<your-skill>/SKILL.md` 下添加，并在 `plugin.json` 自动覆盖（`"skills": "./skills/"` 已声明）。
- 调整规则：直接改 `humanize-doc/SKILL.md`，但**别忘了在 `CHANGELOG.md` 记一笔**——这份文档自己就是 trade-off 的产物，每条规则都该说得清楚为什么加进来。

## License

MIT — 见 [`LICENSE`](LICENSE)。
