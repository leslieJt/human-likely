# human-likely

> Make Claude write technical docs like a **senior engineer writing to teammates** — with trade-offs, war stories, and accountability. **No AI fluff.**

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/version-0.2.0-orange.svg" alt="Version 0.2.0">
  <img src="https://img.shields.io/badge/platform-Claude%20Code%20Plugin-blueviolet.svg" alt="Claude Code Plugin">
  <img src="https://img.shields.io/badge/language-中文%20%2F%20English-green.svg" alt="中文 / English">
</p>

---

## What is this?

A [Claude Code Plugin](https://code.claude.com/docs/en/plugins.md) that ships two complementary Skills for technical writing:

| Skill | When to use | What it does |
|-------|-------------|--------------|
| [`humanize-doc`](skills/humanize-doc/SKILL.md) | Rewrite / polish / improve existing docs; complaints like "too AI-sounding", "talk like a human", "write like a senior engineer" | **Polish** existing text |
| [`engineering-deepdive`](skills/engineering-deepdive/SKILL.md) | Write a technical deep-dive, architecture analysis, design doc, engineering writeup, or code walkthrough from scratch | **Generate** new long-form technical content |

**Recommended workflow:** use `engineering-deepdive` to draft a question-driven, well-structured article with diagrams, then run `humanize-doc` to strip any remaining AI clichés and polish the tone.

---

## Why does this exist?

LLMs default to a recognizable writing pattern: passive voice, feature lists, filler phrases ("综上所述", "旨在", "赋能", "全方位"), and zero decision rationale. The result *reads* correct but *feels* hollow — no one believes the author actually built or debugged the system.

`human-likely` fixes this by injecting two things into Claude's writing process:

1. **Decision traces** — every non-trivial design choice shows the naive approach that failed and *why* the real approach was picked.
2. **Structural discipline** — question-driven organization replaces feature lists; overview-detail-synthesis replaces "first... second... in conclusion...".

The difference isn't vocabulary. It's whether the reader finishes thinking "this person has actually shipped this" vs "this was generated".

---

## Quick Start

### Option 1: Install from this repo (recommended)

Register this repo as a single-plugin marketplace, then install:

```bash
# 1. Add marketplace (reads .claude-plugin/marketplace.json)
claude plugin marketplace add https://github.com/leslieJt/human-likely

# 2. Install the plugin
claude plugin install human-likely@human-likely
```

Launch `claude` in any project and start writing. Claude auto-activates the right Skill based on context, or trigger explicitly:

```
/human-likely:humanize-doc
/human-likely:engineering-deepdive
```

### Option 2: Local development

Clone and mount directly with `--plugin-dir` (changes take effect immediately, no marketplace needed):

```bash
git clone https://github.com/leslieJt/human-likely.git
cd /your/project
claude --plugin-dir /path/to/human-likely
```

### Option 3: Manual install as user-level Skill

Copy the skill directories into `~/.claude/skills/` — the Skills themselves don't depend on plugin context:

```bash
cp -r human-likely/skills/humanize-doc ~/.claude/skills/
cp -r human-likely/skills/engineering-deepdive ~/.claude/skills/
```

---

## Skills in Detail

### `humanize-doc` — Polish existing docs

**Trigger phrases:** "rewrite this", "polish", "too AI-sounding", "说人话", "像资深工程师那样写"

**What it does:**

1. **Diagnose** — identify the audience, the real pain point, and where AI clichés hide.
2. **Extract technical skeleton** — lock down facts that must not be lost (APIs, fields, numbers, constraints).
3. **Rewrite** following four principles:
   - **Rigorous** — exact numbers, specific function names, concrete paths. Never blur "2,847 QPS" into "fast".
   - **Human** — inject decision rationale, trade-offs, and war stories. At least one "why we didn't pick the other approach" and one concrete gotcha.
   - **De-mechanized** — delete all AI clichés ("综上所述", "旨在", "赋能", "全方位", "不仅...而且...").
   - **Balanced** — professional but not stiff, conversational but not sloppy. 0-3 emoji max in a technical essay.
4. **Self-check** against a strict checklist before delivery.

**Example transformation:**

| Before (AI-flavored) | After (human-likely) |
|---|---|
| 在当今快速发展的微服务架构领域，服务间的可靠通信至关重要。本方案旨在通过引入消息队列，全方位提升系统的稳定性与扩展性。 | 上个季度我们因为下游 HTTP 直连超时，连环挂了三次。这次把订单事件改走 Kafka，主要是为了把"下游慢"和"上游能不能下单"解耦——副作用是引入了消息重复的问题，所以下游必须做幂等。 |

### `engineering-deepdive` — Generate from scratch

**Trigger phrases:** "deep-dive", "深度剖析", "architecture analysis", "design doc", "engineering writeup", "code walkthrough", "技术博客"

**Five core writing patterns:**

| # | Pattern | What it replaces |
|---|---------|-----------------|
| 1 | **Question-driven organization** | Feature lists ("this module has 5 features...") |
| 2 | **Overview → Detail → Synthesis** | Flat enumeration with no narrative arc |
| 3 | **Naive approach vs. Real answer** cards | Unexplained design decisions |
| 4 | **SVG diagrams for information compression** | Walls of prose that a single figure could replace |
| 5 | **Core insight callouts** (end of each section) | "In this section we discussed..." summaries |

**Workflow:**

1. **Material collection** (30%) — `grep`, `git log`, `wc -l` — anchor every claim with real data.
2. **Outline** (10%) — 5-10 core questions, ordered so each answer naturally leads to the next.
3. **Overview** (10%) — one sentence + one architecture diagram. Reader gets the full picture in 2 minutes.
4. **Deep-dive** (30%) — per question: problem statement → naive approach card → real approach card → optional diagram → core insight callout.
5. **Synthesis** (10%) — one integrated data-flow diagram + 3-5 generalizable engineering principles.
6. **HTML rendering** (optional, 10%) — uses the bundled `template.html` with warm cream + burnt orange palette.

**Bundled references:**

| File | Purpose |
|------|---------|
| [`references/structure.md`](skills/engineering-deepdive/references/structure.md) | Detailed playbook for question-driven organization and the overview→detail→synthesis arc |
| [`references/visual-grammar.md`](skills/engineering-deepdive/references/visual-grammar.md) | Five figure types (architecture / swimlane / state machine / decision tree / comparison), SVG design principles, `cairosvg` self-check workflow |
| [`assets/template.html`](skills/engineering-deepdive/assets/template.html) | Battle-tested HTML/CSS template with callouts, KPI grid, naive/answer cards, diagram containers |

---

## Project Structure

```
human-likely/
├── .claude-plugin/
│   ├── plugin.json            # Plugin manifest
│   └── marketplace.json       # Single-plugin marketplace entry
├── skills/
│   ├── humanize-doc/
│   │   └── SKILL.md           # Rewriting rules, checklist, examples
│   └── engineering-deepdive/
│       ├── SKILL.md           # Writing patterns, workflow, examples
│       ├── references/
│       │   ├── structure.md       # Question-driven + overview→detail→synthesis playbook
│       │   └── visual-grammar.md  # SVG figure types + cairosvg self-check
│       └── assets/
│           └── template.html      # HTML/CSS template (warm cream + burnt orange)
├── README.md
├── CHANGELOG.md
└── LICENSE (MIT)
```

Follows the [Claude Code Plugin Reference](https://code.claude.com/docs/en/plugins-reference.md) standard layout — skill directories at repo root, only `plugin.json` and `marketplace.json` under `.claude-plugin/`.

---

## Design Philosophy

Both skills share a unified writing philosophy:

| Principle | What it means in practice |
|-----------|--------------------------|
| **Rigorous** | APIs, fields, numbers, complexity bounds — nothing gets blurred. `os.replace()` not "atomic file operation". `609 lines` not "about 500 lines". |
| **Human** | Every non-trivial design shows the decision: what was considered, what was rejected, and why. At least one trade-off and one concrete failure scenario per document. |
| **De-mechanized** | Zero tolerance for AI clichés. No "综上所述", no "旨在", no "赋能", no "全方位". Short sentences. Concrete nouns. No filler connectors. |
| **Balanced** | Professional but not pompous. Conversational but not flippant. "这块挺烦的" is fine; "这块真的太他妈烦了" is not. 0-3 emoji in a technical essay; more looks unserious. |

`humanize-doc` uses these as a rewriting checklist. `engineering-deepdive` bakes them into the article's skeleton — question-driven organization, naive-vs-answer comparison cards, per-section core insight callouts, and decision traces woven into the narrative structure.

---

## When to Use Which Skill

```
                ┌─────────────────────────────┐
                │  Do you have existing text?  │
                └──────────┬──────────────────┘
                           │
               ┌───────────┴───────────┐
               │ Yes                   │ No
               ▼                       ▼
        ┌──────────────┐      ┌────────────────────┐
        │ humanize-doc │      │ engineering-deepdive│
        │  (polish)    │      │  (generate)         │
        └──────┬───────┘      └────────┬────────────┘
               │                       │
               └───────────┬───────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │ Optional: run         │
               │ humanize-doc again    │
               │ for final polish      │
               └───────────────────────┘
```

| Scenario | Use |
|----------|-----|
| "Rewrite this RFC, it reads like ChatGPT wrote it" | `humanize-doc` |
| "Write a deep-dive on our caching layer" | `engineering-deepdive` |
| "Polish this design doc and add trade-offs" | `humanize-doc` |
| "Create an architecture analysis with diagrams" | `engineering-deepdive` |
| "Write a technical blog post about our memory subsystem" | `engineering-deepdive` → then `humanize-doc` |

---

## Advanced Options

### `humanize-doc` options

Users can append these requirements:

- **Pain-point opening** — first paragraph must explain "what happens if we don't do this"
- **Gotcha section** — add a "lessons learned / watch out for" section at the end
- **Narrative flow** — no numbered lists; force paragraph-based prose (good for RFCs, Design Docs)
- **Preserve structure** — keep existing section headings, only rewrite content (good for READMEs with fixed templates)
- **Bilingual / mixed** — keep English technical terms (API, SLA, idempotent) untranslated

### `engineering-deepdive` options

- **HTML + SVG rendering** — use the bundled `assets/template.html` as a starting point
- **PDF output** — convert HTML to PDF via `weasyprint`
- **Mermaid instead of SVG** — for GitHub READMEs where mermaid renders natively
- **Emoji budget** — 0-3 semantic symbols (warning, lightning, pin) are fine; decorative emoji (rocket, sparkles, target) are not
- **Chain with humanize-doc** — run `humanize-doc` after the initial draft to catch any residual AI patterns

---

## Requirements

- [Claude Code](https://code.claude.com/) (the plugin host)
- No additional dependencies — the skills are pure prompt-based (SKILL.md files)
- For HTML rendering: a modern browser
- For SVG self-check: Python 3 + `cairosvg` (`pip install cairosvg`)
- For PDF export: `weasyprint` (`pip install weasyprint`)

---

## Contributing

PRs welcome:

- **Add a new Skill:** create `skills/<your-skill>/SKILL.md`. The plugin manifest (`"skills": "./skills/"`) auto-discovers it.
- **Adjust rules:** edit the relevant `SKILL.md` directly, but **log your change in [`CHANGELOG.md`](CHANGELOG.md)** — every rule is itself a trade-off, and should explain why it was added.
- **Improve the HTML template:** edit `skills/engineering-deepdive/assets/template.html`.
- **Add references:** place new reference docs in the relevant `references/` directory.

### Commit message convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(humanize-doc): add bilingual rewriting mode
fix(engineering-deepdive): correct swimlane SVG alignment
docs: update README with usage examples
chore: bump marketplace version
```

---

## Changelog

See [`CHANGELOG.md`](CHANGELOG.md) for the full version history.

**Latest: v0.2.0** (2026-05-23)
- Added `engineering-deepdive` skill for writing question-driven technical deep-dives from scratch
- Added reference docs (`structure.md`, `visual-grammar.md`) and HTML template
- README updated to cover both skills

**v0.1.0** (2026-04-25)
- Initial release with `humanize-doc` skill

---

## FAQ

**Q: Does this work with English content?**
A: Yes. The skills are written in Chinese but the principles are language-agnostic. Claude will apply the same writing philosophy regardless of the output language. Technical terms are always kept in English.

**Q: Can I use just one skill without the other?**
A: Absolutely. Each skill is independent. Install only the one you need, or use both in sequence.

**Q: Will this change my code?**
A: No. These are writing skills — they only affect document output. No code modifications, no dependencies injected into your project.

**Q: How is this different from a custom system prompt?**
A: Skills are more structured than system prompts. They include step-by-step workflows, self-check checklists, reference materials, and HTML templates. A system prompt says "write like a senior engineer"; a skill defines *exactly what that means* with examples, anti-patterns, and verification steps.

**Q: Can I customize the HTML template colors?**
A: Yes. Edit the CSS variables in `:root` at the top of `assets/template.html`. The default palette is warm cream (`#fafaf7`) + burnt orange (`#b8470c`).

---

## License

MIT — see [`LICENSE`](LICENSE).

---

<p align="center">
  <sub>Built by <a href="https://github.com/leslieJt">@leslieJt</a></sub>
</p>
