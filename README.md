<p align="center">
  <img src="assets/hero.svg" alt="Sud's Vibe Coding Kit" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-v6.5-1b2430?style=flat-square" alt="version">
  <img src="https://img.shields.io/badge/philosophy-suggestions%20%3E%20gates-3b5bdb?style=flat-square" alt="philosophy">
  <img src="https://img.shields.io/badge/runtime%20deps-none-2b8a6e?style=flat-square" alt="no runtime deps">
  <img src="https://img.shields.io/badge/platform-Windows%2011%20%2B%20WSL-444?style=flat-square" alt="platform">
  <img src="https://img.shields.io/badge/macOS%20%2F%20Linux-via%20one%20prompt-7b8494?style=flat-square" alt="mac/linux">
  <img src="https://img.shields.io/badge/tested%20with-GLM%20%C2%B7%20DeepSeek%20%C2%B7%20MiMo%20%C2%B7%20MiniMax-7950f2?style=flat-square" alt="tested models">
</p>

<p align="center"><b>A small set of files you drop into a project <i>before</i> you write code, so AI coding agents stop drifting — without spending their budget proving they followed a process.</b></p>

<p align="center"><sub>📰 <b>What's new in v6.5:</b> trustworthy gates that can't silently false-green · docs that can't rot silently · a learning loop with a motor · subagent economics · CRLF-immune gates on Windows.</sub></p>

---

## The problem it solves

An AI coding agent has **no memory between sessions**, so left alone it *drifts*: it reinvents your types, forgets yesterday's decisions, quietly redefines your domain words, and announces "done" without ever running the code. You can't prompt this away — the forgetting is structural.

So the kit does the one thing that works: it **writes the ground truth to plain files on disk** that the agent re-reads every session, and — *wherever a check has an objective answer* — lets a single command fail the build. You never "run" the kit; **your agent reads it.**

```mermaid
flowchart LR
  A["AI agent<br/>(no memory between sessions)"] -->|left alone| B["drifts:<br/>reinvents types · forgets decisions<br/>redefines terms · 'done' without running"]
  C["Ground truth on disk<br/>PRD · GLOSSARY · DATA_MODEL · AGENTS.md"] -->|re-read every session| D["builds the right thing"]
  E["just verify / verify-all<br/>(only checks with an objective answer)"] -->|fails the build on regressions| D
```

> **The design rule that keeps it lean:** *gate the product, not the process.* Check things with an objective truth (does it compile? does the API exist? does the app boot?). Never gate the agent's own bookkeeping — that was the v6 failure mode, where the agent spent ~80% of its reasoning proving it had followed the kit. v6.1 removed it; everything since stays prose-and-suggestions over enforcement.

---

## How to use it — two phases

The only thing beginners trip on is the seam between these two phases.

```mermaid
flowchart TD
  subgraph P1["Phase 1 — one-time setup · just you · ~5 min · no AI yet"]
    s1["Install mise (once, ever)"] --> s2["Folder = project, extract the kit into it"] --> s3["Double-click scaffold.ps1"]
  end
  subgraph P2["Phase 2 — open your AI agent"]
    f1["First message:<br/>define scope + run grill-to-prd"] --> f2["Agent writes PRD.md + contracts"] --> f3["Builds phase by phase, gated by just verify"]
  end
  s3 --> f1
```

### Phase 1 — one-time setup (Windows)

Don't overthink these; they only put the files in place and nothing here can harm your machine.

```bash
# 1. Install the one prerequisite — once, ever, on your computer:
winget install jdx.mise          # (it quietly installs just/uv/node/pnpm + ast-grep for you)

# 2. Make a folder, name it your project, and extract the kit INTO it (no sub-folder).
#    This folder IS your app.

# 3. Double-click scaffold.ps1   (WSL: bash scaffold.sh)
#    → sets up the toolchain, git, and the safety gates, makes the first commit,
#      and prints your first prompt. Safe to run twice.
```

### Phase 2 — open your AI agent (the first message matters)

Open your coding agent in the folder. **Don't start with "build me a …".** Your first message defines the **scope** and runs the *grill* — it interviews you one question at a time and turns your answers into a spec (`PRD.md`), so the agent builds the *right* thing instead of guessing. **This is the step people skip.**

```text
Read CONTEXT.md and the files it lists. Then run the grill-to-prd skill
(skills/grill-to-prd.md): interview me ONE question at a time, recommending an
answer for each, until the scope is fully clear. Do NOT write code or scaffold
yet. Then write PRD.md and we'll build phase by phase.
```

> Skip the grill and the agent invents the requirements — the #1 way it builds the wrong thing. Five minutes of questions here saves hours of rework.

---

## macOS / Linux — one prompt to adapt the kit

The kit is authored Windows-first (it ships `scaffold.sh` for WSL/Unix already). To adapt the rest for macOS or Linux, paste this into your agent **after extracting the kit**:

```text
I'm on macOS/Linux. Adapt this kit for my platform WITHOUT changing its rules or
philosophy:
1. Use scaffold.sh (not the .ps1). Install the one prerequisite with
   `curl https://mise.run | sh`  (macOS alternative: `brew install mise`).
2. When the final phase generates server-control scripts, generate POSIX
   start-server.sh / stop-server.sh (kill-by-port via lsof or fuser) instead of the
   PowerShell .ps1 templates — still reading PORT from .env, never hardcoded.
3. Keep just, uv, pnpm, ast-grep, the justfile, and ALL gates exactly as they are —
   they're already cross-platform via mise.
4. Leave AGENTS.md and every contract file unchanged.
Show me a summary of what you changed.
```

Everything else (`just`, `uv`, `pnpm`, `ast-grep`, the gates) is already cross-platform through `mise`.

---

## Everyday scenarios (copy, paste, go)

| When you're… | Paste this |
|---|---|
| **Continuing where you left off** | `Read RepoMapReadFirst.md and STATE.md, tell me where we are in two sentences, then continue.` |
| **Adding a feature** | `I want to add <feature>. Update PRD.md / EXECUTION_PLAN.md for it first (separate commit), then build it phase by phase per AGENTS.md.` |
| **Planning a big/complex change** | `This is a large task. Follow AGENTS.md §4b: write a dated plan in plans/, keep phases small, show me the plan, then execute phase by phase.` |
| **Switching to a different AI tool** | `Read CONTEXT.md and the files it lists, then continue from STATE.md, following AGENTS.md.` |
| **Building any UI (avoid "AI slop")** | See `PROMPTS.md` §Q — encodes `DESIGN_GUIDELINES.md` into the build prompt. |
| **After fixing a bug** | `Add it to SinsGotchasLearnings.md as a prose entry (an ast-grep rule only if it's a clean structural pattern).` |

---

## What's in the kit

| Group | Files | Job |
|---|---|---|
| **Contracts** (ground truth) | `PRD.md` · `GLOSSARY.md` · `DATA_MODEL.md` · `ARCHITECTURE.md` · `DESIGN_GUIDELINES.md` · `types/` | What the app does, every term, the data grain & units, the architecture, the visual standard. |
| **Governance** (how the agent works) | `AGENTS.md` (single source of truth) · `CONTEXT.md` · `EXECUTION_PLAN.md` · `STATE.md` · `MODEL_NOTES.md` · `PROMPTS.md` | Rules, routing, the phased plan, the live cursor, per-model tips, the reusable prompt library. |
| **Memory & gates** (enforced only where objective) | `RepoMapReadFirst.md` · `SinsGotchasLearnings.md` + `rules/*.sins.yml` · `SinsArchive.md` · `OpenTasksMustCompleteAll.md` · `UserPromptLog.md` · `WorkLogAfterEachRun.md` · `justfile` | The repo map, the defect taxonomy (some compiled to ast-grep rules), the sins archive (retired entries), the anti-drop checklist, passive prompt log, work log, and the gates: `just verify` / `just verify-all` / `just doctor` / `just sins-triage`. |

Every other tool-config file (`CLAUDE.md`, `GEMINI.md`, `.cursor/`, `.kilocode/`, `.windsurfrules`, `.zed/`, `.rules`, `.codex/`) is a **three-line pointer** back to `AGENTS.md`, so rules can't drift.

### Supported tools

Claude Code · OpenAI Codex CLI · Gemini / Antigravity · Cursor · Windsurf · Kilocode · Zed · OpenCode · Stagewise · Qoder · Hermes — **any tool that reads `AGENTS.md` works.** Tested with GLM 5.2, DeepSeek V4 Pro, Xiaomi MiMo 2.5 Pro, MiniMax M3, Claude, and Gemini.

---

## Why it works

- **It externalizes the memory the agent doesn't have** — files named so the filename itself signals the purpose (`OpenTasksMustCompleteAll`, `RepoMapReadFirst`).
- **It converts the few objectively-checkable rules into gates** — a rule in a doc is a suggestion; the same rule as a `just verify` check is a wall.
- **It makes the right thing the cheap thing** — re-orienting from `RepoMapReadFirst.md` costs ~2k tokens instead of a 50k re-walk; trusting a green gate beats re-checking by hand.
- **It refuses to gate process** — anything that would police the agent's own prose is left as a suggestion, so the agent spends its budget building, not proving.

Read `VibeCodingKit_FieldGuide_v6.5.md` for the full rationale, or open `VibeCodingKit_Infographic_v6.5.html` for the visual one-pager.

---

## What's new in v6.5

v6.5 attacks the three failure modes observed across real builds — **gates that silently no-op'd, descriptive files that decayed into fiction, and a sins file that documented mistakes without preventing repeats** — and stays true to v6.1's "gate the product, not the process." No new per-turn ceremony.

1. **No silent false-greens (`run_gate.py`, `just doctor`).** A gate whose stack is configured but whose tool is missing now **fails loudly** (exit 3, with the exact setup command) instead of printing "skipping" and passing. The JS package manager is **detected from the lockfile** (pnpm/npm/yarn/bun — never assumed). New **`just doctor`** self-tests the harness so a green `verify` is *earned*. The root cause this closes: a real harness once silently no-op'd for an entire phase because "skipping" warnings were scrolled past.
2. **Docs that can't rot silently (`dox-check`).** Contract docs are scanned for backticked repo paths: wholesale rot (≥3 missing and >30%) is a hard failure; `RepoMapReadFirst.md` additionally fails on leftover `[fill in]` placeholders once the repo has real source. An agent that catches one stale doc rationally stops trusting all of them.
3. **The learning loop gets a motor.** Entries now carry retrieval metadata (`Scope:` / `Trigger:` / `Enforced:` / `Digest:` / `Recurrences:`), and the generated index is organized for retrieval at the point of action: a **★ALWAYS block**, a **trigger map** ("about to touch `fetch` / `useEffect` / `src/db/**`? → read exactly these §s"), and enforced entries collapsed out of the read path. The **recurrence ratchet**: a repeat is never re-documented — increment `Recurrences:`, and at 2+ promotion up the enforcement ladder (type → ast-grep rule → test → digest) is mandatory. `just sins-triage` reports loop health.
4. **Subagent economics (§4b).** A **context firewall** (subagents write to gitignored `.scratch/<task>/`, return path + 3-line summary + confidence — never raw dumps), **handoff packets** (delegated prompts written for zero chat context), and a **stakes / reversibility / ambiguity floor** for routing slices across model tiers.
5. **CRLF-immune gates on Windows.** Every gate normalizes `\r\n` → `\n` on read, so a Windows editor that saves CRLF can no longer silently break the regexes that anchor on `\n`.

> **Upgrading from v6.4?** It's a surgical swap of ~10 infra files (your contracts, learnings, logs, STATE.md, and src/ are untouched). See `migrate/MIGRATION.md` §"v6.4 → v6.5 manual swap" inside the kit.

---

## Inspirations & credits

This kit stands on existing ideas and tools:

- **`grill-to-prd`** is adapted from **Matt Pocock's** `grill-with-docs` skill — a relentless one-question-at-a-time requirements interview.
- **`AGENTS.md`** follows the emerging cross-tool agent-instructions convention, so one rules file serves every agent.
- **[ast-grep](https://ast-grep.github.io/)** powers the structural defect audits (a real parser, never regex on code).
- **[mise](https://mise.jdx.dev/)** is the one-prerequisite toolchain manager; **[just](https://github.com/casey/just)** is the command runner behind the two gates.
- **[Playwright](https://playwright.dev/)** backs the `smoke` gate so "done" means run-and-observed.
- The optional hierarchical `AGENTS.md` layer (§10) is inspired by the **DOX** documentation-as-context framework.
- v6.4's *inline flags* and *reuse-first* ideas were distilled from studying a production SAFe multi-agent harness — taking the ideas, not the team-process overhead.
- v6.5's *subagent economics* (context firewall, handoff packets, routing floor) were borrowed from a premium-model orchestration harness and made tool-agnostic.

---

<p align="center"><sub>Plain files + two commands. No GitHub, no service, no runtime dependency required.</sub></p>
