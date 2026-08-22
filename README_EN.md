# deep-code-analyzer

[简体中文](README.md) | English

> A pipelined source-code expedition team with built-in QA — systematic, deep examination of any project (open source or private), producing structured architecture docs plus engineering-risk / code-defect / design-issue assessments.

**Version: v2.6.2** | **Language: English / Chinese** | **Form: META-SKILL**

---

## Introduction

`deep-code-analyzer` is a **portable protocol for deep source-code examination**. Using "pipeline + explicit state machine + contract-driven" design, it organizes an AI agent (or a group of agents) into an expedition team that **only analyzes, never modifies**, executing on any codebase:

1. Landscape scan → module mapping → dataflow tracing → pattern extraction
2. (after human confirmation) Targeted deep dives / feature reproduction / defect scanning
3. Report assembly → (optional) live verification

All assessment conclusions are **anchored to source-code evidence**: verifiable conclusions carry confidence labels; unverifiable ones are marked `unverified`. Personal preferences disconnected from evidence are forbidden.

---

## Boundary Card & Anti-pattern FAQ

> Required reading before first use — avoid the pitfalls.

| ✅ You may | ❌ You may not |
|-----------|-------------|
| Read project code, docs, config files | Directly modify, delete, or rewrite any project code |
| Analyze architecture, trace dataflows, extract design patterns | Compile, run, test, or deploy the project |
| Assess engineering risk, scan for defects, suggest improvements | Turn suggestions directly into code changes (output plans only; execution belongs to the user) |
| Analyze a specified module + related docs and produce **modification-plan references** | Decide on the user's behalf — all key nodes require confirmation |
| Produce architecture reports, feature-reproduction guides, migration plans | Skip human intervention points — after S4 you **must** stop and wait |

### Common Pitfalls

| Pitfall | Correct behavior |
|------|----------|
| "Optimize this code" → modifying it directly | This skill **never edits code directly**. The chain is: locate the problem (with source evidence) → propose a fix plan → **ask "shall we apply this plan?"**; execution belongs to the user or a coding mode |
| "Run the project to see how it works" → attempting execution | Answer: "I analyze but don't execute. Here's the entry file and how to start it… you can run it yourself." |
| Assuming S1–S4 auto-produce the report | After S4 there is a **mandatory stop awaiting confirmation**. Not clicking "skip" doesn't advance anything — that's by design, not a hang |
| Wanting changes made but unfamiliar with the source | Choose option **⑤ Modification reference** in the stage selector: specify the target module; it analyzes that module + related docs first and outputs a modification-plan reference |

---

## Stage Selection (when requirements are unclear)

- **Clear requirements** (a named algorithm/feature/defect target) → focused mode directly (quick S1 landscape → straight to S5/S6/S6B).
- **Unclear requirements** ("just analyze this project") → **after S1 completes**, show the project card plus a **stage selector** and let the user decide:

> [Stage selection · S1 done] Project landscape card produced. Choose next stages (multiple allowed):
> ① Continue full pipeline: S2 module mapping → S3 dataflow → S4 pattern extraction (stop again at the S4 intervention point)
> ② Deep-dive target: ____ (specifiable, e.g. "caching mechanism") → S5
> ③ Reproduce feature: ____ (specifiable) → S6
> ④ Defect scan → S6B
> ⑤ Modification reference: specify the module; analyze it + related docs, output a modification-plan reference → S5 focused + output
> ⑥ Skip deep dives, deliver report now → S7

---

## Design Philosophy: Six Iron Rules

| # | Rule | Meaning |
|---|------|------|
| 1 | **Orthogonal decoupling** | The Meta layer manages process only (S1→S8); the Atom layer executes only (one operation per step) |
| 2 | **Explicit state machine** | Every transition requires an enumerable quantitative gate (coverage ≥ 80%, links ≥ 5, patterns ≥ 3, subsections ≥ 7); no "then / next / roughly" |
| 3 | **Contract-driven** | Atomic skills exchange data only through the Context defined in `schemas/context.md`; reliance on implicit memory is forbidden |
| 4 | **Bounded execution** | ≤ 2 retries per stage, ≤ 24 total turns, deep-dive/reproduce loops ≤ 3 rounds, defect-scan loops ≤ 2 rounds |
| 5 | **Review before recording** | Any key conclusion must undergo at least one independent evidence re-check against original source before entering deliverables; unverifiable ones must be explicitly marked `unverified` |
| 6 | **Docs first** | When a docs directory exists, read documentation first to build a frame of understanding (three tiers, budgeted), then dissect the source against the documented structure as the expected baseline; doc/source mismatches are high-value anomaly signals |

---

## Workflow (8-stage pipeline)

```
S1 Landscape scan → S2 Module mapping → S3 Dataflow tracing → S4 Pattern extraction
    │
    ├── [Human intervention point] Scope & depth confirmation (mandatory stop)
    │
    ├─→ S5 Targeted deep dive (loops ≤ 3)
    ├─→ S6 Feature reproduction (loops ≤ 3)
    ├─→ S6B Defect scan (optional branch, off by default)
    │
    └─→ S7 Report assembly & delivery
           └─→ (optional) S8 Live verification (capability-driven, command whitelist)
```

| Stage | Name | Key transition gates |
|------|------|------------|
| S1 | Project landscape scan | Produces the project card: tech stack + project type + entry points + design principles (four required fields) + **docs_map (docs-first reconnaissance)** |
| S2 | Module mapping | Module list coverage ≥ 80% of top-level source directories; each module gets a one-line responsibility + entry path; **modules declared in docs_map serve as the expected baseline** |
| S3 | Dataflow tracing | Dataflow diagram ≥ 5 links; each link hypothesis-verified with confidence; hypothesis table ≥ 3 entries closed |
| S4 | Design pattern extraction | ≥ 3 patterns/highlights/conventions, each with unified evidence format + confidence |
| S5 | Targeted deep dive | Symbol trace table + call chains + state machine diagram + complexity analysis + alternatives comparison + configuration quick-reference |
| S6 | Feature reproduction | Feature location + dependency chain + step breakdown + data contracts + boundary conditions + code skeleton + configuration quick-reference |
| S6B | Defect scan | Each finding: file:line + type + evidence + severity (P0-P3) + confidence |
| S7 | Report assembly | All required sections covered + hard gate on diagram existence + key numbers carry `origin` annotations |
| S8 | Live verification | Command whitelist (read-only/benchmark classes); any write-command refused |

---

## Three Execution Modes

- **Full mode (default)**: execute S1→S7 in order.
- **Focused mode**: when the user's first message clearly names a deep-dive/reproduction/defect target, run a minimal S1 (tech stack + entry points only), then jump straight to the relevant stage (confirm once with the user before jumping).
- **Resume mode**: when `_context_checkpoint.json` exists, read the checkpoint, rebuild Context, and continue from the next unfinished stage — **completed stages are not re-run**.

---

## Key Mechanisms

### Human Intervention Point (mandatory stop after S4)

After S4 completes, the system **must stop** and present preliminary Stage 1–4 results, collecting instructions in four directions:

```
① Deep-dive targets (pre-filled [MUST-DIG] / [CANDIDATE] entries) → S5
② Reproduce feature: ____ → S6
③ Defect scan (off by default, saves tokens) → S6B
④ Skip, deliver report → S7
```

- **`[MUST-DIG]` forced consumption**: at least 1 `[MUST-DIG]` anomaly signal must enter S5, or the user must explicitly skip it with the reason recorded.
- **No-response handling**: if the user doesn't reply within 3 turns → skip S5/S6/S6B by default, write unconfirmed items into `human_decisions.pending_confirmations`, and proceed to S7 — **no errors, no infinite waiting**. At delivery, remind about completable items; replying "deep-dive <target>", "reproduce <feature>", or "scan defects" resumes from checkpoint.

### Context Persistence Contract (resumability)

- At each stage end, `stage_outputs + human_decisions + metadata` are written to `output_dir/_context_checkpoint.json` (overwriting, UTF-8).
- The checkpoint is the **single source of truth for cross-session recovery**; conversation context is just a cache.

### Docs-First Reconnaissance

When the project has a docs directory (`docs/`, `documentation/`, etc.), S1 reads documentation first to build the framework, then dissects the source:

| Tier | Content | Reading method |
|------|------|----------|
| L1 must-read | Docs directory tree (2 levels) + root indexes (`index.md`, `README.md`, `SUMMARY.md`, etc.) | Full read (small files) |
| L2 selective close-read | Architecture/design/specs + getting-started/guides | Close-read by relevance |
| L3 title scan | API/reference/changelogs | File names + first-line titles only |

- **Hard reading budget**: L2 + L3 combined ≤ 10 files; any single doc > 400 lines → first 400 lines + section headings only
- **Artifact `docs_map`**: docs structure tree + declared tech stack + declared modules (S2 expected baseline) + architecture concepts + glossary + drift signals
- **Drift detection**: modules declared in docs but missing from source → `[MUST-DIG]` anomaly signals; top-level directories present in source but absent from docs → `[OPTIONAL]` signals

Sampling tiers (token control):

| Tier | Module size (files) | Sampling strategy | Notes |
| :--- | :--- | :--- | :--- |
| **Low** | ≤ 20 | Full read, no sampling | Small context pressure; completeness preserved |
| **Medium** | 21 – 100 | Read `min(15, max(5, ceil(n/10)))` core files | Dynamic sampling balancing coverage vs tokens |
| **High** | > 100 | No auto-sampling; mark `[CANDIDATE]` into the deep-dive pool | Human decision; prevents token waste |

### Source-Credibility State Machine

- Read in repo → `repo_verified`
- Inferred within repo → `repo_inferred`
- External dependency located & read (venv/node_modules found) → `external_read`
- External dependency not read → `external_unread`, **must attach a locate command** (e.g. `pip show <pkg>`), then skip that module

### Capability Probing

After S1, probe host capabilities into `metadata.capabilities`: can commands be executed? Can out-of-repo dependencies be read? Can tokens be counted? Are structured interaction components available? Results determine later options (e.g. S8 runs only when `can_execute_commands` is true).

---

## Directory Structure

```
deep-code-analyzer/
├── SKILL.md                     # Meta-layer main file (orchestration + global rules)
├── README.md                    # This document
├── atoms/                       # Atom layer: independently executable atomic steps
│   ├── 01-scan-landscape.md     # S1 project landscape scan
│   ├── 02-map-modules.md        # S2 module mapping
│   ├── 03-trace-dataflow.md     # S3 dataflow tracing
│   ├── 04-extract-patterns.md   # S4 design pattern & principle extraction
│   ├── 05-deep-dive-target.md   # S5 targeted deep dive (loopable)
│   ├── 06-reproduce-feature.md  # S6 feature reproduction research (loopable)
│   ├── 06b-scan-defects.md      # S6B defect & engineering-risk scan (optional)
│   ├── 07-assemble-report.md    # S7 report assembly & delivery
│   └── 08-measure.md            # S8 live verification (optional)
├── references/
│   ├── capabilities.md          # 12+1 dimension capability checklist
│   └── report-template.md       # Unified report skeleton template
└── schemas/
    └── context.md               # Context structure contract (sole cross-atom data channel)
```

---

## Quick Start

### Install as a Skill

Place this directory into your host environment's skills directory (e.g. `skills/deep-code-analyzer/`) for registration via the host's skill-loading tool. If unavailable, read `SKILL.md` plus everything under `atoms/`, `schemas/`, `references/` directly from the install location and execute per the Meta layer — the pipeline is unchanged.

### Trigger Phrases

Say any of these to any AI agent:

- "Analyze project XX's structure"
- "Deep-dive XX algorithm/mechanism"
- "Explain XX's architecture"
- "Help me understand XX's codebase"
- "Reproduce XX feature"
- "Assess XX's engineering risks"
- "Scan XX for code defects"

### Environment Adaptation

Tool names inside atoms are **functional descriptions** (list directory, read file, full-text search, cross-file jump, etc.), mapped at runtime to equivalent tools provided by the host. Never abort the flow because a specific tool name doesn't exist.

---

## Report Deliverables

- Default form is a **file set**: `README.md` (architecture overview diagram), `architecture-analysis.md` (module dependency graph), `tech-stack.md` (dependency topology diagram) + topical files, written under `<project-root>/<project-name>-analysis/`.
- Each of the three base files contains ≥ 1 non-empty diagram (Mermaid or ASCII) — diagram existence is a hard gate.
- Key numbers carry `origin` annotations; the report section tree strictly matches `references/report-template.md`.

---

## Maintenance & Regression Verification

After **any** structural change to this skill (sampling rules, contract fields, atom structure), a **regression verification run is mandatory**: execute the full pipeline (or at least S1–S4 + one S5) on a small open-source project of 300–500 files, checking every item:

- [ ] Checkpoint bookkeeping self-check gate passes
- [ ] All report key numbers carry `origin` annotations
- [ ] Giant modules appear in the human-intervention-point candidate pool
- [ ] Report section tree matches `references/report-template.md`
- [ ] Dead-end external dependencies have locate commands or explicit `external_unread`
- [ ] Un-measured performance/complexity conclusions carry `[inference]` annotations
- [ ] **Docs-first**: projects with docs produce a `docs_map` with drift signals in the anomaly pool; docs-less projects get `docs_map=null` without blocking

The change isn't effective until every checkbox is green.

---

## License

Licensed under the [MIT License](LICENSE).

You are free to use, modify, and distribute this project's code, provided the original copyright notice is retained.
Provided "as is", without warranty of any kind.

Copyright © 2026 QinLuza 魔法少女独断万古
