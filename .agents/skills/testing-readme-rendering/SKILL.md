---
name: testing-readme-rendering
description: Test README.md rendering on GitHub for the human-likely project. Use when verifying README changes render correctly (badges, tables, links, code blocks).
---

# Testing README Rendering

## When to Use
After modifying README.md, verify it renders correctly on GitHub before merging.

## Test Procedure

1. **Navigate to the rendered README** on the PR branch:
   `https://github.com/leslieJt/human-likely/blob/<branch>/README.md`

2. **Verify badges** — should render as colored pill-shaped images, not raw markdown `![...]` text. The repo uses shields.io-style badges via `img.shields.io`.

3. **Verify tables** — GitHub renders markdown tables with borders. Check that:
   - Column headers are visible and bold
   - Rows have proper borders
   - Any links inside table cells are clickable (blue, underlined)

4. **Verify internal links** — click each relative link in the README to confirm it resolves to the correct file on the branch. Common links to check:
   - `skills/humanize-doc/SKILL.md`
   - `skills/engineering-deepdive/SKILL.md`
   - `skills/engineering-deepdive/references/structure.md`
   - `skills/engineering-deepdive/references/visual-grammar.md`
   - `skills/engineering-deepdive/assets/template.html`
   - `CHANGELOG.md`
   - `LICENSE`

5. **Verify ASCII art / flowcharts** — should appear inside a monospace `<pre><code>` block with box-drawing characters properly aligned. GitHub renders fenced code blocks in monospace font.

6. **Verify content accuracy** — cross-check version numbers against `.claude-plugin/marketplace.json` and project structure tree against actual `find` output.

## Tips

- Use the DOM/HTML output from the browser tool to verify link `href` attributes without clicking each one individually.
- The repo has no CI configured, so there are no automated checks to wait for.
- Badge images are proxied through `camo.githubusercontent.com` — if they appear broken, it might be a caching issue. Wait and refresh.
- The `marketplace.json` has the canonical version number (currently 0.2.0). `plugin.json` may have a different version (0.1.0) — use `marketplace.json` as the source of truth for the badge.

## Devin Secrets Needed
None — this is a public repo and testing only requires read access to GitHub.
