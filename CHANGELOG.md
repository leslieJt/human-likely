# Changelog

All notable changes to this plugin will be documented here.
This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-05-23

### Added
- `engineering-deepdive` skill — write technical deep-dives from scratch using question-driven organization, overview→detail→synthesis structure, naive-vs-answer comparison cards, SVG figures for information compression, and per-section "core highlight" callouts that abstract the engineering principle.
- `engineering-deepdive/references/structure.md` — detailed playbook for question-driven organization and the overview→detail→synthesis arc.
- `engineering-deepdive/references/visual-grammar.md` — five figure types (architecture / swimlane / state machine / decision tree / comparison), SVG-vs-mermaid selection, cairosvg self-check workflow.
- `engineering-deepdive/assets/template.html` — battle-tested HTML/CSS template (warm cream + burnt orange palette, callouts, KPI grid, naive/answer cards, diagram container).
- Crystallized from an iterative Hermes-Agent memory subsystem deep-dive session in May 2026.

### Changed
- README now lists both skills as complementary (humanize-doc = polish; engineering-deepdive = generate).
- Marketplace metadata bumped to 0.2.0 with both skills described.

## [0.1.0] - 2026-04-25

### Added
- Initial release as a Claude Code plugin.
- `humanize-doc` skill — rewrite technical docs in a senior-engineer voice with trade-offs, gotchas, and decision rationale; strips AI clichés.
