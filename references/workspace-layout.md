# The .research/ workspace

Every research project gets a `.research/` directory at its root. This is where
"keep everything" lives. The orchestrator indexes; subagents write the detail.

```
.research/
├── STATE.md                 # short, always-current. The FIRST thing every session reads.
├── plan.md                  # living master plan: steps, status, follow-ups.
├── verification-ledger.md   # every load-bearing claim + status + evidence pointer.
├── designs/                 # design docs, proposals, method writeups. Versioned, never overwritten.
├── decisions/               # one entry per human-gate decision (options, choice, rationale).
├── artifacts/               # subagent working outputs: search results, logs, tables, drafts.
└── chatgpt/                 # codex (ChatGPT) cross-analysis exchanges.
```

`.research/` is a working/process directory. **Add it to `.gitignore` by default** —
its artifacts and chatgpt exchanges can capture secrets (server passwords, tokens) and
large blobs, and must never be pushed to a shared repo without scrubbing. If you want
the record tracked, track only `STATE.md`, `plan.md`, `verification-ledger.md`,
`designs/`, and `decisions/`, and scrub secrets first. Confirm with the user before
changing `.gitignore`.

## verification-ledger.md template

The anti-fabrication backbone. Every load-bearing claim — a reported number, a design
choice, a cited reference, a "this is implemented" claim, an external fact — gets a
row. Nothing expensive or irreversible happens while a row it depends on is
`unverified` or `refuted`.

```markdown
# <Project> — Verification Ledger
_Last updated: <date>_

| Claim | Type | Status | Evidence | Notes |
|---|---|---|---|---|
| SFT CER = 0.61 | result-number | verified | results/seed42/eval.json:12 + rerun cmd | 3-seed mean |
| LR=2e-4 right for r64 QLoRA | design-choice | refuted | codex+subagent: needs sweep | run sensitivity sweep |
| "[12] is closest prior work" | citation | sourced | arXiv:2401.xxxx PDF p.4, claim-matched | VERIFIED |
| Verifier L3 is implemented | design-impl | verified | src/pipeline/l3_rules.py + passing test | |
| WSDM 2027 deadline = <date> | external-fact | unverified | nothing fetched yet | GATE: fetch+cite |

Status: verified (re-checkable outcome) · sourced (fetched URL) · unverified
(assumption, not yet checked) · refuted (failed a check — must fix). "Load-bearing"
= the paper or a decision depends on it.
```

## STATE.md template

Keep this short. It is your working memory — if it grows long, it's failing its
job. Link out to detail rather than inlining it.

```markdown
# <Project> — Research State
_Last updated: <date> by orchestrator_

## Goal
<1-3 sentences: what success looks like for this project / this phase>

## Current phase
<e.g. "experiment design v2 — addressing confound found in v1">

## Done (recent)
- <step> → <artifact path> — <one-line outcome>
- <step> → <artifact path> — <one-line outcome>

## Next
1. <next step> → stage: <stage> → skill: <aris-skill>
2. <step>

## Open decisions / blocked
- [GATE] <what needs the user> — waiting on <key / direction / login>

## Key facts the orchestrator must not forget
- <constraint, deadline, hard requirement, e.g. "must submit to CVPR 2027">
```

## plan.md template

```markdown
# <Project> — Research Plan

## Research question
<the question / hypothesis>

## Phases
### Phase 1 — <name>  [done | in-progress | todo]
- [x] <step> — <artifact>
- [ ] <step>
### Phase 2 — <name>
- [ ] <step>

## Follow-ups surfaced during work
- <thing a subagent flagged that we should do later>
```

## decisions/ entry template

One file per decision, e.g. `decisions/2026-06-04-method-choice.md`:

```markdown
# Decision: <title>
_Date: <date>_

## Context
<why a choice was needed>

## Options considered
1. <option> — pros / cons
2. <option> — pros / cons

## Decision
<what was chosen>

## Rationale
<why — including the user's input at the gate, and any ChatGPT cross-analysis>
```

Recording the *rejected* options matters as much as the chosen one — that's how the
project's full design space stays preserved, which is the user's explicit goal.
