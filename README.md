<div align="center">

# research-orchestrator

**A Claude Code skill that runs research projects as a lean orchestrator.**  
The main thread only plans and decides — all work is delegated to subagents  
that write to disk and return compact summaries, so your conversation stays  
navigable months into a project.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-blueviolet)](https://claude.ai/code)
[![ARIS](https://img.shields.io/badge/skills-ARIS%20family-green)](#academic-skills)

</div>

---

## The problem

Research projects are long. If you run literature reviews, write experiment code, parse training logs, and draft paper sections in one conversation, the context fills with detail that's irrelevant to the *next* decision — and 30 steps in, you've lost the thread of the whole project.

`research-orchestrator` flips this. **The main thread holds the plan. Subagents hold the mess.** Every task — a literature search, an experiment run, a paper section, a citation audit — is dispatched to a subagent that writes its full output to disk and returns only a ≤200-word summary. The conversation stays clean. The detail is safe on disk. You can still see the forest 50 steps in.

---

## Install

```bash
git clone https://github.com/HaochenSHI66/research-orchestrator \
  ~/.claude/skills/research-orchestrator
```

Then, inside any research project:

```
/research-orchestrator
```

The skill reads `.research/STATE.md`, picks up where you left off, and keeps moving.

---

## How it works

```
┌──────────────────────────────────────────────────────────────────────┐
│                           Main Thread                                │
│                                                                      │
│   Read STATE.md → Read TODO.md → Pick next step → Map to ARIS skill │
│          ↑                                                      │    │
│          └──── Update STATE + TODO ◄──── ≤200-word summary ◄───┘    │
│                                              │                       │
│   Human gate? → Stop and ask.         Subagent writes full           │
│   Otherwise → loop.                   output to .research/           │
└──────────────────────────────────────────────────────────────────────┘
```

The orchestrator never does research work itself. It decides what happens next, briefs a subagent, and records the outcome. That's the entire job.

### The dispatch contract

Every subagent gets a self-contained brief:

```
Context: <2–3 sentences — the subagent can't see the main conversation>
Task: <specific, bounded unit of work>
Skill: invoke `aris-xxx` for this
Output path: .research/artifacts/xxx.md
Return ONLY:
  1. ≤200-word summary
  2. Artifact path(s)
  3. Evidence for each load-bearing claim [verified] / [sourced] / [assumption]
  4. Skills/tools actually used
  5. Any decision the orchestrator now needs
  6. Confidence / risk flag
```

This contract is what keeps the main thread clean. A subagent's full notes, code, logs, and drafts stay on disk. Only the summary comes back.

---

## What it does

| | |
|---|---|
| **Clean main thread** | All literature search, experiment code, analysis, and writing goes to subagents. You see summaries; detail lives on disk. |
| **Persistent workspace** | Plans, designs, and decisions — including options that lost — are all kept under `.research/`. Every session reads `STATE.md` first. |
| **Running TODO** | `TODO.md` is updated every loop iteration with status, locked decisions, and open questions — a single source of truth for the whole project. |
| **Stage → skill mapping** | Each research stage auto-maps to the matching ARIS academic skill. Ideation, literature, experiments, writing, review, rebuttal — all covered. |
| **Evidence-gated** | Nothing load-bearing enters STATE as fact without a file quote, command output, or fetched URL. Claims are tagged `[verified]`, `[sourced]`, or `[assumption]`. |
| **Competitive sweep** | Before any direction choice, the orchestrator fans out parallel subagents to map the existing literature and surface direct overlap, competitive risk, and open gaps. |
| **Venue-standard verification** | Every design choice, formula, and result is stress-tested against what NeurIPS/ICML/ICLR/CVPR/ACL reviewers actually attack: unjustified constants, weak baselines, single-run results, missing ablations, fabricated numbers. |
| **ChatGPT cross-analysis** | High-stakes judgment calls (novel? sound? claim follows?) get an independent second opinion from GPT-5.5 via the codex MCP server, saved to `.research/chatgpt/`. |
| **Autonomous loop** | Runs through ordinary work without stopping. Pauses only at genuine human gates: direction choices, missing credentials, interactive logins, irreversible actions. |

---

## Evidence system

One of the most common ways AI-assisted research goes wrong: a subagent asserts something, the orchestrator records it as fact, and the error propagates invisibly through every downstream step.

`research-orchestrator` treats every subagent summary as a **claim**, not proof. Before anything load-bearing enters `STATE.md`, a design doc, or a decision, it needs evidence:

| Tag | Meaning |
|-----|---------|
| `[verified: file:line]` | Checked against a real artifact — a file quote, test output, run log |
| `[sourced: url]` | Claim about the outside world — fetched this session, not from model memory |
| `[assumption: model-prior]` | Not yet verified — must be resolved before it can drive an expensive decision |

An `[assumption]` is never silently promoted to fact. If a step depends on an unverified claim, that's a human gate — not something to paper over.

All load-bearing claims are tracked in `.research/verification-ledger.md`. No expensive or irreversible action (compute spend, submission, push) proceeds while the ledger has open `unverified` items.

---

## Competitive intelligence sweep

Before the first direction choice on any new project (or after a topic pivot), the orchestrator runs a mandatory sweep:

1. **Identify venues** — top 2–3 conferences/journals for the area
2. **Parallel paper collection** — 3–5 subagents fan out across keyword angles using `aris-semantic-scholar`, `aris-openalex`, `aris-arxiv`; each downloads the top matching PDFs
3. **Per-paper analysis** — one subagent per paper extracts contribution, method, benchmarks, results, and limitations
4. **Positioning map** — a synthesis subagent builds a contribution-space table and delivers a verdict: direct overlap / competitive risk / gap filler / novel axis
5. **Human gate** — the map is presented to the user; direction is never chosen silently

The goal is that no direction is picked with unknown competitive risk. The positioning map also drives the benchmark decision: reuse existing benchmarks unless they provably can't measure the claimed contribution.

---

## Human gates

The orchestrator loops autonomously through ordinary work. It stops **only** when a wrong guess would be expensive or irreversible:

| Gate | When it fires |
|------|--------------|
| **Direction choice** | Multiple viable framings/methods; choice changes everything downstream |
| **Benchmark protocol** | No existing benchmark can measure the claimed contribution |
| **Missing credentials** | A step needs an API key, dataset token, or cluster login |
| **Interactive auth** | Something needs a human at a browser (OAuth, 2FA, `gcloud auth login`) |
| **Irreversible action** | Deleting files, pushing, submitting, spending money |
| **Unverified load-bearing claim** | The ledger has an open item that a planned step depends on |

Everything else — keep going. "Should I run the next step?" is not a gate. When multiple gates are visible ahead, they're batched into one question so the user isn't pinged repeatedly.

---

## Workspace

Bootstrapped on first run:

```
.research/
├── STATE.md               ← current-state snapshot; first thing every session reads
├── TODO.md                ← running task board + locked decisions + open questions
├── plan.md                ← living master plan (high-level milestones)
├── verification-ledger.md ← status of every load-bearing claim
├── designs/               ← design docs, versioned, never overwritten
├── decisions/             ← one entry per human-gate decision (options, choice, rationale)
├── competitive-sweep/     ← positioning map + per-paper analyses
├── artifacts/             ← subagent working outputs
└── chatgpt/               ← ChatGPT cross-analysis exchanges
```

`TODO.md` is the project's single source of truth — it tracks tasks, records locked decisions with rationale, and lists open questions for the user. It's updated every loop iteration so it always reflects reality.

`designs/` is versioned by filename (`exp-design-v1.md`, `v2.md`) and never overwritten. Old versions stay — the history of what was tried and rejected is part of the project record.

---

## Academic skills

The stage → skill map targets the **ARIS** family (`aris-*`). Falls back to `scholar-*` / `arl-*` / `phd-*` if ARIS isn't installed — orchestration still works either way.

| Stage | Skills |
|---|---|
| Ideation | `aris-idea-discovery`, `aris-idea-creator`, `aris-novelty-check` |
| Literature | `aris-research-lit`, `aris-openalex`, `aris-semantic-scholar`, `aris-arxiv` |
| Experiment design | `aris-experiment-plan`, `aris-ablation-planner` |
| Running & monitoring | `aris-monitor-experiment` |
| Analysis | `aris-analyze-results`, `aris-result-to-claim` |
| Writing | `aris-paper-write`, `aris-paper-plan`, `aris-paper-figure` |
| Review & audit | `aris-citation-audit`, `aris-paper-claim-audit`, `aris-auto-review-loop` |
| Rebuttal | `aris-rebuttal`, `aris-resubmit-pipeline` |

The orchestrator never invokes these itself — it tells each dispatched subagent which skill to run. The subagent runs it in its own context. See [`references/stage-skill-map.md`](references/stage-skill-map.md) for the full mapping.

---

## Requirements

- **Claude Code** with the `Agent` tool (subagent support required)
- **ARIS skills** *(recommended)* — see [academic skills](#academic-skills) for fallback behavior
- **codex MCP server** *(optional)* — for ChatGPT cross-analysis; absence triggers a human gate rather than a silent skip

---

## Repo layout

```
research-orchestrator/
├── SKILL.md                         ← the skill (orchestrator logic)
├── references/
│   ├── stage-skill-map.md           ← research stage → ARIS skill mapping
│   ├── workspace-layout.md          ← .research/ structure + file templates
│   ├── reviewer-attack-checklist.md ← what top-venue reviewers attack
│   ├── figure-discipline.md         ← figure/table standards
│   └── citation-verification.md     ← citation verification protocol
└── README.md
```

---

## License

MIT — see [LICENSE](LICENSE).
