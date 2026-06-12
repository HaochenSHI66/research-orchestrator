# Example: Starting a new research project

This example shows what happens when you invoke `/research-orchestrator` on a fresh project.

## Setup

```bash
cd ~/my-research-project
/research-orchestrator
```

## What the skill does on first run

1. **Locates the project root** — the current directory
2. **Bootstraps `.research/`** with `STATE.md`, `plan.md`, `TODO.md`, and `verification-ledger.md`
3. **Runs crash recovery** — no stale tasks on first run, skips
4. **Runs competitive sweep** (mandatory for new project):
   - Dispatches subagents to identify top venues
   - Fans out 3–5 parallel paper-collection subagents
   - Per-paper analysis fan-out (up to 20 papers)
   - Synthesis subagent produces `.research/competitive-sweep/positioning-map.md`
5. **Presents positioning map** → waits for user direction choice (human gate)
6. **Records decision** to `.research/decisions/direction-choice.md`
7. **Enters the loop** — dispatches the first research task

## Initial `.research/TODO.md`

```markdown
# TODO — my-research-project
_Updated: 2026-06-12 | Session 1_

## Goal & constraints
- **Target:** [to be determined after direction choice]
- **Non-negotiables:** evidence-gated, top-venue bar, no fabricated numbers

## Locked decisions
(none yet — direction choice pending)

## Done
- [x] Bootstrap .research/ workspace ✓ 06-12

## In progress
- [~] Competitive intelligence sweep — collecting papers across 3 keyword angles

## Remaining
### Direction choice
- 🚪 Review positioning map and pick research direction

## Open questions for user
- Which research direction to pursue? (positioning map at .research/competitive-sweep/positioning-map.md)
```
