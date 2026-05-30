---
name: focus
description: Tidy and right-size the files Claude loads on every session — CLAUDE.md and the memory index MEMORY.md. Analyzes them for obsolete content, disorganization, and size-limit problems, then proposes a plan (delete obsolete / reorder / split oversized files into indexed topic files) for the user to approve before applying anything. Use when the user runs /focus, asks to clean up, trim, or check their CLAUDE.md or memory, mentions context hygiene, or when these files have grown stale or oversized.
license: MIT
---

# focus — context-file hygiene

Keeps the files Claude loads on EVERY session lean, accurate, and well-indexed: **`CLAUDE.md`** (project and user-global `~/.claude/CLAUDE.md`) and the **memory index `MEMORY.md`**. Bloat, staleness, or exceeding size limits in these files silently degrade every session — through truncation, misleading instructions, and attention diluted by noise.

**This skill only analyzes and proposes. Never edit CLAUDE.md or MEMORY.md without explicit user confirmation.**

## Why it matters
- `MEMORY.md` is loaded every session as the routing **index** — but only the **first 200 lines or 25 KB, whichever comes first**; beyond that it's **truncated** (not loaded at startup, only readable on-demand), so part of the index silently disappears and relevant memories may never be recalled. [docs: code.claude.com/docs/en/memory]
- `CLAUDE.md` is loaded every session as **instructions**. Stale content actively misleads (e.g. pointing to deleted files); bloat costs tokens every turn and dilutes attention away from what matters.
- **Nested `CLAUDE.md` / `AGENTS.md`** (in subfolders) are loaded **on-demand** — only when you work in that subtree, not every session. So they carry no per-session tax, and the MEMORY ~200-line truncation rule does NOT apply to them. But a stale nested file still misleads while you work there → check them too, with a lighter pass, and **only the ones that are yours**.
- **`AGENTS.md`**: Claude Code does **not** auto-load it (it reads `CLAUDE.md`). It enters context only if **bridged** — via an `@AGENTS.md` import inside `CLAUDE.md` (imports are expanded at launch → it becomes effectively always-loaded), or a `CLAUDE.md`→`AGENTS.md` symlink (Windows needs admin → prefer the `@import`). So check its **bridge status** to pick the case: (a) **bridged** → full CLAUDE.md treatment (bridge + duplicated content = double-paid → consolidate); (b) **unbridged & for another tool** → not in your context → skip it, don't edit; (c) **unbridged BUT full of real project instructions** → Claude is silently IGNORING them → flag it ("bridge via `@import` or those instructions are lost") — the opposite of bloat: missing context. [docs: code.claude.com/docs/en/memory#agents-md]
- **`.claude/agents/` (subagent definitions)** are a DIFFERENT thing from `AGENTS.md` despite the name: one `.md` per subagent (frontmatter + system-prompt body). Per docs: descriptions are surfaced for the **delegation decision** (not a fixed per-session startup tax), and each **body loads only when that subagent is invoked**, in its own context. So treatment = **prune unused subagents + keep descriptions sharp** (clarity drives correct delegation), but **never trim a working system-prompt for tokens** (you'd break its behavior). [docs: code.claude.com/docs/en/sub-agents]

## Best-practice sizes
- **MEMORY.md** → a **lean index**: ideally one line per memory, with detail pushed to separate topic files. Keep well under the **200-line / 25 KB** load cap. It should read as a list of pointers, not a content dump.
- **CLAUDE.md** → **focused and accurate**: no hard limit, but treat a few hundred lines as a soft ceiling. Keep it as the entry index + active rules + current state; move reference material and long detail into linked docs.

## Procedure
1. **Locate & measure.** Find the always-loaded files: the project `CLAUDE.md`, the user-global `~/.claude/CLAUDE.md`, and the project `MEMORY.md` (plus its topic files, if any). Report each file's line count and approximate tokens (bytes ÷ 4). Also expand any **`@path` imports** inside the always-loaded `CLAUDE.md` — imported files are loaded at launch (up to 4 hops deep), so they count toward the always-loaded budget; measure and tidy them too. Then find **nested `CLAUDE.md`/`AGENTS.md`** in the repo — but **EXCLUDE vendored/third-party trees**: `node_modules/`, `vendor/`, `.git/`, and any cloned dependency or sub-repo (a subdir with its own `.git`). Those instruction files are upstream's, not yours. The remaining (yours) nested files get the same staleness/organization pass, but as **context-loaded** (no per-session-tax, no MEMORY line-limit framing).
2. **Flag size issues.** MEMORY.md over the line limit (→ truncated on load), or CLAUDE.md oversized.
3. **Detect content issues:**
   - **Obsolete** — references to deleted files/dirs, outdated dates/snapshots, completed or dead tasks, superseded sections.
   - **Disorganized** — detail inline that belongs in a linked or topic file; mixed concerns; a missing or stale index.
   - **Oversized** — a file past its best-practice size → candidate to **split**: move detail into topic files and leave a one-line pointer in the index.
   - **Duplication** — if `AGENTS.md` is bridged into `CLAUDE.md` (via `@AGENTS.md` import or symlink) AND the same instructions are also duplicated inline in `CLAUDE.md`, they load twice → propose keeping `AGENTS.md` as the single source and reducing `CLAUDE.md` to the `@AGENTS.md` import + only Claude-specific extras.
   - **Semantic rot (lint)** — beyond dead *file* references: **contradictions** between entries/topic-files, **stale-superseded** claims (an entry a newer note has obsoleted), and **orphan / dangling links** (index→topic-file links that don't resolve, or topic files with no inbound link from the index). The link checks are mechanically verifiable — run them every time. *(Inspired by the "lint" operation in Karpathy's LLM-Wiki gist.)*
   - **Irrelevant-in-always-loaded (evict by next-step relevance)** — content in the always-loaded layer that is reference-only / never conditions a decision → demote to a topic file **even if it's small**, because irrelevant tokens cost *attention*, not just budget. The eviction test is relevance-to-the-next-step, not age or size.
4. **Propose.** Group findings by file. For each issue say what you'd do — delete / reorder / split + index / **compact (merge redundancies)** / **evict to topic file** / fix link / keep — and why. Present clearly (a short list, or AskUserQuestion for the key decisions). Recommend, but let the user choose.
5. **Apply on confirmation.** Trim the index, move detail into topic files, update stale sections, **merge self-declared redundancies (compaction)**, **fix dangling/orphan links**, and keep indices in sync (if you move or rename something, update whatever points to it). Report before/after line and token counts.
6. **Install the discipline (optional, once per project).** So the project stays tidy *between* runs, offer to add a short context-hygiene rule to `CLAUDE.md` if it isn't already there. Confirm first, keep it to ~3 lines (don't bloat the file you just tidied), and skip if already present. Suggested text:
   > **Context hygiene** — keep `MEMORY.md` a lean index (one line per memory, detail in topic files, under the line limit) and `CLAUDE.md` accurate (no references to deleted files, current snapshot/date). Sync indices and snapshots in the SAME turn as the change, not later. Preserve info by moving it to topic files, not deleting.

This makes `focus` self-perpetuating: the skill is the periodic deep clean; the installed rule is the passive between-runs discipline that slows the drift.

## Principles
- **Analyze → propose → confirm.** Never silently rewrite CLAUDE.md or MEMORY.md.
- **Preserve by default.** Move content to topic files rather than deleting; only delete what the user confirms is obsolete.
- **Lean index, detail on demand.** The always-loaded files hold pointers + active rules; everything else lives in linked docs recalled when relevant.
- **Scope = what's loaded, not all `.md`.** Targets are only files that enter Claude's context — always-loaded (root/global `CLAUDE.md`, `MEMORY.md`), `@import`ed, contextual (nested `CLAUDE.md`), or bridged `AGENTS.md`. A plain doc/README that's never loaded is NOT focus's concern; being markdown doesn't make it in-scope.
- **Never touch vendored/upstream files.** Nested `CLAUDE.md`/`AGENTS.md` inside cloned repos, `node_modules`, or `vendor` belong to others — exclude them from the scan, never edit them.
- **Generic.** Works in any project. What counts as "obsolete" or "important" is judged live with the user, not hardcoded.
