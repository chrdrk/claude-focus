# focus

A context-hygiene skill for Claude Code. It right-sizes the files Claude loads into context every session — your `CLAUDE.md` and the auto-memory `MEMORY.md` index — by flagging bloat, stale references, redundancy, and oversize, then proposing a plan you approve before anything is edited.

`/focus`

## Why

The files loaded on every session quietly accumulate cruft, and it degrades every session in ways that are easy to miss:

- `MEMORY.md` is loaded as the routing index, but only the first **200 lines or 25 KB** (whichever comes first); past that it is truncated, so part of the index silently disappears and relevant memories may never be recalled.
- `CLAUDE.md` is loaded as instructions. References to deleted files actively mislead; redundancy and noise cost tokens every turn and dilute attention.

`focus` keeps both lean, accurate, and indexed, with detail pushed to on-demand topic files instead of sitting in the always-loaded layer.

## What it covers

- **Any project, not just code.** Research, ops, writing, or audit projects as much as codebases — it does not assume you are tidying a software repo.
- **Both always-loaded surfaces.** It curates the `CLAUDE.md` instructions and the auto-memory `MEMORY.md` index (with its 200-line / 25 KB cap), pushing detail into on-demand topic files.
- **It evicts, not only relocates.** Obsolete facts are removed on your confirmation, not just shuffled into sub-docs.
- **It proposes, never auto-edits.** Every change is shown and applied only after you approve.

## What it does

1. Locates and measures the always-loaded files — project `CLAUDE.md`, user-global `~/.claude/CLAUDE.md`, the auto-memory `MEMORY.md` and its topic files — plus any `@path` imports (expanded at launch, so they count) and *your* nested `CLAUDE.md` (vendored/cloned trees excluded).
2. Flags size problems (MEMORY.md over the cap, oversized CLAUDE.md).
3. Detects content problems: obsolete references, disorganization, oversize, cross-file duplication, and semantic rot — contradictions, stale-superseded claims, and orphan / dangling index links.
4. Proposes, grouped by file: delete / reorder / split into indexed topic files / compact / evict / fix link / keep — and why.
5. Applies only after you confirm, and reports before/after line and token counts.
6. Optionally installs a short context-hygiene rule into `CLAUDE.md` so the project drifts more slowly between runs.

## Principles

- Analyze, propose, confirm. Never silently rewrite `CLAUDE.md` or `MEMORY.md`.
- Preserve by default. Move content to topic files rather than deleting; delete only what you confirm is obsolete.
- Lean index, detail on demand. The always-loaded files hold pointers and active rules; everything else lives in linked topic files recalled when relevant.
- Scope is what gets loaded, not every `.md`. A doc that never enters context is not a target. Never touch vendored or upstream instruction files.

## Limits

- A single skill, not a plugin or suite — no hooks, no CLAUDE.md generation.
- It does not lint coding-harness files; other tools cover that.
- "Obsolete" and "irrelevant" are judgments made with you, not an algorithm. `focus` flags candidates; you decide.

## Install

Copy the `focus/` folder into your skills directory:

- User-global (all projects): `~/.claude/skills/focus/`
- Project-only: `<project>/.claude/skills/focus/`

Then invoke `/focus`, or just ask Claude to check or tidy your `CLAUDE.md` / memory.

## Credits

`focus` was built from hands-on use — keeping real projects' context files tidy — not derived from another tool's design. Two genuine influences:

- **Anthropic's documented memory pattern.** The lean-index plus on-demand topic-files approach, and the exact 200-line / 25 KB load cap, are from the Claude Code docs: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory).
- **Andrej Karpathy.** The semantic lint pass (contradictions, stale-superseded claims, orphan / dangling links), the evict-by-relevance test, and the compaction step are adapted from his writing on context engineering and his "LLM Wiki" gist (the `ingest / query / lint` model). The "context window is RAM" framing is his.

## License

MIT.
