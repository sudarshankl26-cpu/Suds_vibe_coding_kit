# Sud's Vibe Coding Kit — Field Guide (v6.4)

*Concepts, the problems they solve, why they work, and how to run it.*
For solo, multi-tool development (Claude Code, Kilo/VSCodium, Zed, OpenCode, Windsurf,
Antigravity, Hermes, and others) on Windows 11 + WSL.

> **What this is.** A small set of files you drop into a project before you write code. Each
> file closes one specific way AI coding agents go wrong — drifting from requirements,
> inventing APIs, dropping tasks, forgetting structure, repeating a fixed bug. The kit is not
> a framework and adds no runtime dependency; it is plain Markdown plus a handful of
> Python/JS scripts wired to two commands: `just verify` and `just verify-all`.


---

## Start here — the soul of the kit, and how to actually use it

**If you read nothing else, read this.** An AI coding agent has no memory between sessions.
Left alone it drifts: it reinvents your types, forgets yesterday's decisions, quietly redefines
your domain words, and announces "done" without ever running the code. You cannot prompt this
away — the forgetting is structural. So the kit does the one thing that works: it writes the
ground truth to **plain files on disk** that the agent re-reads every session, and — wherever a
check has an objective answer — lets a single command fail the build. You never "run" the kit;
**your agent reads it.** Your job is to lay the files down once, then talk to your agent normally.

**It works in two phases, and beginners only stumble on the boundary between them.**

*Phase 1 — a one-time setup you do yourself, before opening any AI tool (~5 minutes, no terminal
wizardry).* Install `mise` once on your machine (`winget install jdx.mise`, or
`curl https://mise.run | sh` on WSL); make a folder named for your project; extract the kit into
it (**no sub-folder** — that folder IS your app); then double-click `scaffold.ps1` (WSL:
`bash scaffold.sh`). That's the whole setup. These steps only put the files in place and wire up
the safety gates — you don't need to understand them, nothing here can harm your computer, and
you still haven't needed the AI.

*Phase 2 — open your AI coding agent in the folder. The first message is the one people skip.*
Do **not** start with "build me a …". Your first message defines the **scope** and runs the
*grill* interview, which asks you one sharp question at a time and turns your answers into a spec
(`PRD.md`) so the agent builds the *right* thing. Paste this:

> Read CONTEXT.md and the files it lists. Then run the grill-to-prd skill
> (`skills/grill-to-prd.md`): interview me ONE question at a time, recommending an answer for
> each, until the scope is fully clear. Do NOT write code or scaffold yet. Then write PRD.md and
> we'll build phase by phase.

Skip the grill and the agent invents the requirements — the single most common way it builds the
wrong thing. Five minutes of questions here saves hours of rework.

**After that, day to day, you only need a handful of prompts** (the full library is `PROMPTS.md`):

- **Continue where you left off:** "Read RepoMapReadFirst.md and STATE.md, tell me where we are
  in two sentences, then continue."
- **Add a feature:** "I want to add <feature>. Update PRD.md / EXECUTION_PLAN.md for it first (as
  a separate commit), then build it phase by phase per AGENTS.md."
- **Switch to a different AI tool:** "Read CONTEXT.md and the files it lists, then continue from
  STATE.md, following AGENTS.md." (Any tool that reads AGENTS.md works.)
- **After a bug is fixed:** "Add it to SinsGotchasLearnings.md as a prose entry (an ast-grep rule
  only if it's a clean structural pattern)."
- **Upgrade the kit later:** in a terminal (not the agent), unpack the new kit elsewhere and run
  `migrate/migrate_kit.py --kit <new-kit> --plan` from your project, then `--apply`. See
  `migrate/MIGRATION.md`.

The rest of this guide explains *why* each piece exists. You don't need it to start — it's what
makes the kit make sense once you do.

---

## What's new in v6.4 — inline flags, reuse-first, a design standard, and an easier on-ramp

v6.4 is **prose-only**: no new gates, and no new machinery the agent has to satisfy. Two ideas are
borrowed from a heavyweight team harness (a SAFe-style multi-agent workflow) and kept solo-lean;
one is a new built-in standard; the rest is a friendlier on-ramp.

1. **Inline flags (`AGENTS.md` §1a).** Three text tags written where the thing lives, so the
   agent's reasoning survives between sessions and tools (§8): `#EXPORT_CRITICAL` (a
   non-negotiable constraint — never silently weaken; touching it is a §6 stop-and-ask),
   `#PLAN_UNCERTAINTY` (an assumption made to keep moving — resolved before a phase is "done"),
   and `#PATH_DECISION` (a one-line "why this over the obvious alternative", too small for an
   ADR). Threaded through the PRD / EXECUTION_PLAN / `plans/` templates, the grill, and the §H
   reconciliation + §I drift-audit prompts. Kept rare and real, the way ADRs are. Prompt:
   `PROMPTS.md` §P.
2. **Reuse before reinvent (`AGENTS.md` §2).** One tech-agnostic anti-pattern: before building a
   new route / component / module / query, check `RepoMapReadFirst.md` and grep for an existing
   equivalent, and match its shape — reuse first, create only when nothing fits. It's the
   portable core of "search first, reuse always" with no pattern library to ship.
3. **A built-in anti-"AI-slop" design standard (`DESIGN_GUIDELINES.md`).** When the project
   renders a UI, the agent follows one default visual standard instead of emitting the model's
   tells (emoji-as-icons, saturated colors, mismatched icon sets, gratuitous glassmorphism): one
   icon set at one stroke width, a neutral base + a single muted accent, one type family, and
   structure from whitespace + 1px borders. The principle is *intentional and consistent beats
   default-generated* — a design language defined in the PRD always wins (tag it `#EXPORT_CRITICAL`).
   Wired into §2, the grill, the plan/phase verification, and a copy-paste build prompt
   (`PROMPTS.md` §Q). Pure contract + suggestions; nothing gated.

Plus a much friendlier **on-ramp for beginners**: this guide now opens with a plain-language
"Start here" (the soul, the two-phase setup, and the can't-miss first prompt — define scope and
run the grill); the infographic gains a matching beginner section and an "everyday scenarios"
prompt grid; OpenAI **Codex** is an explicit supported tool (`.codex/`, reads AGENTS.md natively);
and there's a graphical **`GITHUB_README.md`** for publishing the kit.

What was **rejected** matters as much as what was kept: the source harness's 11 role-agents,
exit-state handoff gates, and stack-coupled CI/deploy ceremony — all the process-enforcement
weight v6.1 removed. A manifest-driven upgrade tool was prototyped too, then **cut to stay
prose-only**: the existing migrator already preserves your work, so a new config file wasn't worth
the surface area.

---

## What's new in v6.3 — tuned for 1M-context open-weight models

The kit is increasingly driven by 1M-context, reasoning-effort-tiered, agentic open-weight
models (GLM 5.2, DeepSeek V4 Pro, MiniMax M3, MiMo 2.5 Pro). v6.3 tunes the kit to how that
class of model actually performs best. As always: **suggestions, not gates** — nothing here adds
enforcement overhead.

1. **Phased planning for long/complex work (`AGENTS.md` §4b).** A big window is not a license to
   pour a whole large task into one runaway session — recall, cost, and prompt-cache hit rate all
   degrade as a trajectory bloats. So: for anything large (>3 files, a refactor, >~300k expected
   tokens), write a **dated working plan** `plans/<slug>-YYYY-MM-DD.md` (template in
   `skills/templates/plan.md.template`), phase it so **each phase's execution stays within ~300k
   tokens**, and execute phase by phase (commit + `just verify` each). Distinct from
   `EXECUTION_PLAN.md` (whole-project plan) and `STATE.md` (live cursor).
2. **Subagent delegation (Explore → Plan → Execute).** These models use subagents well. Where the
   tool supports them, an explore (read-only) subagent maps files, the plan step scopes phases,
   and independent slices fan out in parallel when a phase touches ~10+ files or has 3+
   independent pieces. Dependent work stays single-session. Ties into §6a — a stuck subagent
   escalates rather than thrashes.
3. **`MODEL_NOTES.md` (new, on-demand).** Per-model operating tips — context budget, **reasoning-
   effort tiers** (use Max/high for planning and §4a hard units; lower for routine), sampling,
   stable-prefix caching. Kept separate from `AGENTS.md` so the always-read rules stay lean and
   cacheable.
4. **English / UTF-8 output rule (`AGENTS.md` §7).** All code, comments, identifiers, commits and
   docs in English (or the project's language) regardless of the model's strongest language.
5. **Refined §6a escalation ladder + web search encouraged.** The always-available rungs (read
   diagnostics; run a `verify_<lib>` probe / minimal repro) come first so *every* tool has a
   baseline — but **web search and Context7 are actively encouraged, not a last resort**. Because
   the model's training has a cutoff, §4 and §6a now push it to search *proactively* for current
   dependency versions, recent API/breaking changes, and fresh error messages.
6. **Prune-defaults nudge.** Stronger guidance to delete inapplicable shipped sins early, keeping
   the session-start index short and the prefix cacheable — most valuable on a tight budget.

Reusable prompts: `PROMPTS.md` §N (plan a long task) and §O (delegate a phase to subagents).

---

## What's new in v6.2 — eval-first for hard parts, and an anti-thrash ladder

v6.2 adds two **suggestions** (not gates) to `AGENTS.md`. Both follow v6.1's rule — *gate the
product, not the agent's process* — so they steer behavior without adding the bookkeeping v6.1
removed. There is no new script, no marker convention, and no new gate; the existing
`just verify` test gate does all the enforcing.

1. **Eval-first for hard parts (`AGENTS.md` §4a).** Some units are simply harder to build *and*
   to evaluate — complex/branchy logic, edge-case handling, input/data validation, parsers,
   money/units/date math — and that's where bugs ship quietly. For those (and only those —
   trivial code is exempt), the agent enumerates the edge cases, writes them as **failing tests
   first**, then implements until green. The grill marks such capabilities *risk-flagged* and
   `EXECUTION_PLAN.md`'s phase template carries the check. Quality rises exactly where it's
   hardest, and the existing `test` gate proves the evals pass — nothing self-referential added.
2. **Anti-thrash escalation ladder (`AGENTS.md` §6a).** When the same test/error survives ~3
   fix attempts, the agent stops guessing and **autonomously** consults whatever's available —
   LSP/type diagnostics, Context7 MCP for real library docs, web search for the exact error, or
   a `verify_<lib>` probe — using whichever the current tool has. It does *not* interrupt you;
   that's distinct from §6 "stop and ask," which is still reserved for ambiguous *requirements*.

Upgrading from v6.1 is trivial: it's purely a refresh of `AGENTS.md` (plus a few prompt/plan
notes). No infrastructure changed — no script, hook, or toolchain edits.

Reusable prompts for both live in `PROMPTS.md` §L (eval-first) and §M (escalate when stuck).

---

## What's new in v6.1 — the overhead is gone

v6 made two promises real (verbatim prompt capture via hooks; a closed learning loop) — but
its *enforcement* layer became the problem. Observed on a real build: the agent took ~4× longer
and spent roughly **80% of its reasoning proving it had followed the kit**, not doing the work
— timestamp re-stamping, regex rabbit holes, and chasing self-referential gates.

v6.1 keeps every safeguard's intent and removes the make-work:

1. **AST audits, not grep.** The defect taxonomy compiles to **ast-grep** rules
   (`rules/*.sins.yml`) instead of single-line grep. ast-grep matches the *syntax tree*, so
   rules are multiline, structural, language-aware (including **CSS**), and **cannot match
   their own explanatory comment** — the exact trap that caused 15-minute regex detours. One
   native cross-platform binary (pinned via `mise`); the v6 dual `audit_sins.{sh,py}` codegen
   is deleted. Rules are **optional** — most learnings stay as prose.
2. **`just verify` splits in two.** `just verify` runs only the code/correctness gates (lint,
   types, tests, library checks, schema, sins audit, dox) and **stays green while you build**,
   so you can run it as a sanity check and trust the result. `just verify-all` adds the
   bookkeeping gates and an app smoke test, and is what the pre-commit hook and CI run.
3. **Logs are passive evidence.** Prompt capture is timezone-aware and only records *genuine
   prompts* — tool-call echoes and pasted file dumps are filtered out (v6 logged 200-line file
   reads as "prompts"). The agent never edits the log or re-stamps a timestamp; a cross-tool
   out-of-order timestamp is a harmless note, not a build failure.
4. **No STALE-AUDIT loop.** `just audit-sins` self-heals: it regenerates the rules whenever the
   taxonomy changes, so capturing a learning never triggers a sync-then-re-verify dance.
5. **Learnings steer from the start.** `AGENTS.md` §0b holds an always-on digest of the
   highest-leverage failure modes, so the agent avoids the common patterns from the first line;
   the full 77-entry taxonomy stays on demand via the index, with the structural ones enforced.
6. **Run & observe the app.** A portable Playwright `smoke` gate + template replaces v6's habit
   of deferring every GUI check as "[~] can't verify". "Done" means run-and-observed.

Full upgrade steps: `MIGRATION_v6.1.md`.

---

## 1. The core idea

Agents drift; every drift category has a matching artifact that closes it. An agent left alone
reinvents types, hallucinates library calls, redefines your domain terms, declares features
finished without testing, and re-explores your file tree from scratch each session. You can't
prompt these away reliably — the agent has no persistent memory between sessions. So the kit
writes the ground truth to disk and, *wherever a check can be made deterministic*, enforces it
with a gate that fails the build.

**The mechanism in one sentence:** move every lesson to the most automatic layer it can reach —
a hook that fires without the model beats a gate the build runs, which beats a type the
compiler checks, which beats a sentence in a doc the agent might skip.

**The v6.1 corollary:** only make deterministic what has an *objective truth*. Gate the
**product** (does it compile? does the API exist? is this known-bad pattern present? does the
app boot?). Do **not** gate the agent's own **prose** (was the log well-formed? are the
timestamps ordered?) — that creates self-referential loops the agent can only escape by editing
the bookkeeping the gate also polices. v6's worst overhead was exactly this; v6.1 removes it.

---

## 2. The problems it solves

| Failure mode | What closes it | Enforced? |
|---|---|---|
| Builds the wrong thing (misalignment) | grill-to-prd interview → PRD.md | Process |
| Redefines domain terms inconsistently | GLOSSARY.md | Process |
| Invents fields / wrong units (0–1 vs 0–100) | DATA_MODEL.md + audit_schema | Gated |
| Hallucinates a library's API | verification/verify_* | Gated |
| Repeats a bug fixed once before | SinsGotchasLearnings → **ast-grep** rules | Gated |
| Given 3 tasks, completes 2 | OpenTasksMustCompleteAll + check-tasks | Gated (verify-all) |
| Prompt log paraphrased / non-prompts logged | hooks + filters + check-prompt-log | Hook + Gated |
| Ships without ever running the app | `just smoke` (Playwright) | Gated (verify-all) |
| Botches a hard unit (complex logic / edge cases / validation) | eval-first suggestion (AGENTS §4a) + existing `test` gate | Process |
| Thrashes / loops on the same failure | escalation ladder (AGENTS §6a) → probe / Context7 / web | Process |
| Long/complex task blows out context & budget | phased plan file (AGENTS §4b) ~300k/phase + subagents | Process |
| Mid-build mega-task, no resumable structure | `plans/<slug>-DATE.md` + STATE.md per phase | Process |
| Mixed-language code/comments from the model | English/UTF-8 output rule (AGENTS §7) | Process |
| Buries a hard constraint / silently assumes | inline flags #EXPORT_CRITICAL / #PLAN_UNCERTAINTY (AGENTS §1a) | Process |
| Reinvents an existing in-repo solution | reuse-first anti-pattern (AGENTS §2) + RepoMapReadFirst | Process |
| Ships AI-slop UI (emoji icons, harsh colors, glassmorphism) | DESIGN_GUIDELINES.md + AGENTS §2 | Process |
| Re-walks hundreds of files each session | RepoMapReadFirst.md | Process |
| No trace of what was asked vs delivered | UserPromptLog + WorkLog | Hook + git |
| Loses place between sessions/tools | STATE.md | Process |
| Rules drift across tool configs | AGENTS.md + pointer stubs | By design |

---

## 3. The files, and why each exists

**Contracts — the ground truth (author in order)**
- **PRD.md** — what the app does, who for, and crucially what it deliberately does *not* do.
- **GLOSSARY.md** — one exact definition per domain term; definition drift is a top bug source.
- **DATA_MODEL.md** — entities, grain (what makes a row unique), invariants, explicit units.
- **ARCHITECTURE.md** — components, data flow, tech choices with reasoning, explicit anti-goals.
- **DESIGN_GUIDELINES.md** — the default visual standard (anti-AI-slop); used only when the project renders a UI. A design language pinned in the PRD overrides it.

**Governance — how the agent behaves**
- **AGENTS.md** — the single source of truth for agent rules. §0 session-start protocol, §0a
  lightweight per-turn discipline, **§0b generalized failure-mode digest (always on)**, **§1a
  inline flags**, §2 anti-patterns, §3 the two-command gate, **§4a eval-first for hard parts**, **§4b phased
  planning + subagents** (v6.3), and **§6a the anti-thrash escalation ladder** — all suggestions.
  Every other tool-config file (`CLAUDE.md`, `.windsurfrules`, `.cursor/`, `.kilocode/`,
  `GEMINI.md`, `.rules`) is a three-line pointer back to it, so rules can't drift.
- **MODEL_NOTES.md** (v6.3) — on-demand per-model operating tips (context budget, reasoning-
  effort tiers, sampling, caching) for non-frontier models. Separate from AGENTS.md so the
  always-read surface stays lean.
- **EXECUTION_PLAN.md** — the build in phases, each shippable and verified before the next.
  The plan lives here; the live cursor lives in `STATE.md`. For one large mid-build task, use a
  dated **`plans/<slug>-DATE.md`** working plan (v6.3, §4b) instead of bloating this file.

**Memory & enforcement**
- **UserPromptLog.md** — append-only verbatim record of every instruction. Hooks
  (`.claude/settings.json`, `.opencode/plugins/prompt-logger.js`) capture it outside the
  model's control; in v6.1 they filter out tool-echoes/file-dumps and stamp offset-aware
  timestamps. **Passive evidence — the agent never edits it.**
- **SinsGotchasLearnings.md** — the incident-indexed defect taxonomy (77 shipped entries). Each
  entry is prose; a few carry an optional `### Audit Rule` (ast-grep) that `just audit-sins`
  compiles to `rules/*.sins.yml` and runs. The highest-leverage entries are mirrored into
  `AGENTS.md §0b`.
- **SinsGotchasLearnings_INDEX.md** — generated one-line-per-incident index, read at session
  start; open a full entry on demand.
- **OpenTasksMustCompleteAll.md** — the checklist of **real deliverables** for the current unit
  of work; `check-tasks` (in `verify-all`) fails while any is open or deferred without a reason.
  v6.1: process steps are **not** tasks.
- **RepoMapReadFirst.md / STATE.md / WorkLogAfterEachRun.md** — the living codebase map (~2k
  tokens instead of a 50k re-walk), the machine-readable build cursor, and a short
  one-entry-per-session record of what was delivered.
- **types/, verification/, tests/fixtures/, skills/** — code-level contracts, library-API check
  scripts, canonical inputs with planted bugs, the grill-to-prd interview, and the
  `smoke.spec.ts` template for running the app.

---

## 4. Why it works

- **It externalizes memory the agent doesn't have.** The files are the missing memory, named so
  the filename itself signals the purpose.
- **It converts prose into gates — and hooks.** A rule in a doc is a suggestion; the same rule
  as a `just verify` check is a wall; a hook that runs whether or not the model cooperates is
  the strongest layer.
- **It uses a real parser for code checks.** v6.1's ast-grep audits honor the kit's own rule
  ("parse structured text with a real parser") — the v6 grep audits violated it.
- **It makes the right thing the cheap thing.** Re-orienting from `RepoMapReadFirst.md` is
  cheaper than re-walking the tree; trusting a green `just verify` is cheaper than re-checking
  by hand — and in v6.1 the green gate is actually reachable mid-work, so the agent trusts it
  instead of routing around it.

**Naming is part of the prompt.** Evocative, purpose-revealing filenames (SinsGotchasLearnings,
OpenTasksMustCompleteAll, RepoMapReadFirst) steer behavior better than bland ones.
Ecosystem-standard names (AGENTS.md, GEMINI.md, PRD) are left alone — tools match them literally.

---

## 5. How to run it

**First-time setup (new project).** One prerequisite, installed once ever: `mise`
(`winget install jdx.mise`, or `curl https://mise.run | sh`). Then:

1. Make a folder, name it your project, extract the kit files into it (no sub-folder).
2. Double-click `scaffold.ps1` (WSL: `bash scaffold.sh`) — installs the toolchain (incl.
   `ast-grep`), inits git + the gate hooks, makes the first commit, prints your grill prompt.
3. Open your agent in the folder and paste the grill prompt; answer the interview; it writes the
   contracts and then builds phase by phase.

**The daily loop**
- **Start:** "Read RepoMapReadFirst.md and STATE.md, tell me where we are, then continue."
- **While building:** run `just verify` freely — fast code gates that stay green; trust it.
- **Give work:** the hook logs your prompt; the agent pulls the real deliverables into
  OpenTasks. (No placeholders to fill, no process steps to log.)
- **Hit a bug:** fix it, add a prose `SinsGotchasLearnings.md` entry (an ast-grep `### Audit
  Rule` only if it's a clean structural pattern). No sync dance — `audit-sins` self-heals.
- **Before a commit / end of phase:** `just verify-all` must be green (the pre-commit hook runs
  it). It includes the task gate, prompt-log check, and the app `smoke` test.

**Commands you'll actually type**

| Command | What it does |
|---|---|
| `just verify` | Fast **code** gates (lint, types, tests, library checks, schema, sins audit, dox). Run freely. |
| `just verify-all` | Everything in `verify` + tasks + prompt-log + app smoke. Pre-commit / CI. |
| `just audit-sins` | ast-grep scan of the taxonomy's rules (self-heals if stale). |
| `just sins-sync` | Explicitly regenerate `rules/` + the index from the taxonomy. |
| `just smoke` | Run & observe the app (Playwright), if a `smoke` script is configured. |
| `just map-sync` | Regenerate RepoMapReadFirst.md (keeps your curated notes). |
| `just state` | Print the current build cursor from STATE.md. |

---

## 6. Which files to reference, and when

| When you're… | Point the agent at… |
|---|---|
| Starting a brand-new project | `skills/grill-to-prd.md` |
| Resuming any session | `RepoMapReadFirst.md` + `STATE.md` |
| Giving multiple tasks | `OpenTasksMustCompleteAll.md` |
| Defining or arguing a term | `GLOSSARY.md` |
| Touching the database / units | `DATA_MODEL.md` + `schema_rules.json` |
| Adding a new library | `verification/` (write `verify_<lib>` first) |
| Right after fixing a bug | `SinsGotchasLearnings.md` (prose; rule if structural) |
| About to start any task | `AGENTS.md` §0b digest, then the INDEX for the area |
| Unsure of the rules | `AGENTS.md` |

---

## 7. Keep in mind

- **Match ceremony to the job.** The logs and task-tracking earn their keep on multi-phase
  builds where tasks get dropped. For a throwaway script, keep `PRD.md` + `AGENTS.md` and delete
  the rest — the gates no-op cleanly when their files are absent.
- **Trust the green gate; don't re-check by hand.** If `just verify` passes, the known-bug
  patterns, library APIs, data contracts, and types are clean. Re-auditing them manually just
  burns tokens.
- **Don't gate your own prose.** If you find yourself editing a log or a timestamp to make a
  gate pass, stop — that's the v6 failure mode. The gate should describe an objective truth
  about the product, or it shouldn't be a hard gate.

The whole kit is `suds-vibe-coding-kit-v6.4.tar`. Nothing here requires GitHub or any service;
it is plain files plus two commands.
