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
┌─────────────────────────────────────────────────────────────────┐
│                        Main Thread                              │
│                                                                 │
│  Read STATE.md → Pick next step → Map to ARIS skill → Dispatch │
│       ↑                                                    │    │
│       └──── Append summary to STATE.md ◄────── Subagent ───┘   │
│                                                                 │
│  Human gate? → Stop and ask.   Otherwise → loop.               │
└─────────────────────────────────────────────────────────────────┘
```

Every step goes to a subagent with a self-contained brief. The subagent writes its full output to `.research/` and returns a ≤200-word summary — the only thing that lands in your context.

---

## What it does

| | |
|---|---|
| **Clean main thread** | All literature search, experiment code, analysis, and writing goes to subagents. You see summaries; detail lives on disk. |
| **Persistent workspace** | Plans, designs, and decisions — including options that lost — are all kept under `.research/`. Every session reads `STATE.md` first. |
| **Stage → skill mapping** | Each research stage auto-maps to the matching ARIS academic skill. Ideation, literature, experiments, writing, review, rebuttal — all covered. |
| **Evidence-gated** | Nothing load-bearing enters STATE as fact without a file quote, command output, or fetched URL. Claims are tagged `[verified]`, `[sourced]`, or `[assumption]`. |
| **ChatGPT cross-analysis** | High-stakes design and novelty checks get an independent second opinion via the codex MCP server, saved to `.research/chatgpt/`. |
| **Autonomous loop** | Runs through ordinary work without stopping. Pauses only at genuine human gates: direction choices, missing credentials, interactive logins, irreversible actions. |

---

## Workspace

Bootstrapped on first run:

```
.research/
├── STATE.md               ← current-state snapshot; first thing every session reads
├── TODO.md                ← running task board; updated every loop iteration
├── plan.md                ← living master plan (high-level milestones)
├── verification-ledger.md ← status of every load-bearing claim
├── designs/               ← design docs, versioned, never overwritten
├── decisions/             ← one entry per human-gate decision
├── artifacts/             ← subagent working outputs
└── chatgpt/               ← ChatGPT cross-analysis exchanges
```

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

---

## Requirements

- **Claude Code** with the `Agent` tool (subagent support required)
- **ARIS skills** *(recommended)* — see [academic skills](#academic-skills) for fallback behavior
- **codex MCP server** *(optional)* — for ChatGPT cross-analysis; absence triggers a human gate rather than a silent skip

---

## Repo layout

```
research-orchestrator/
├── SKILL.md                        ← the skill (orchestrator logic)
├── references/
│   ├── stage-skill-map.md          ← research stage → ARIS skill mapping
│   ├── workspace-layout.md         ← .research/ structure + templates
│   ├── reviewer-attack-checklist.md
│   ├── figure-discipline.md
│   └── citation-verification.md
└── README.md
```

---

## License

MIT — see [LICENSE](LICENSE).
