# Example: Resuming an existing project

This example shows what happens when you invoke `/research-orchestrator` on a project already in progress.

## Setup

```bash
cd ~/my-research-project   # .research/ already exists
/research-orchestrator
```

## What the skill does on resume

1. **Reads `.research/STATE.md`** — loads current project state
2. **Reads `.research/TODO.md`** — loads task board
3. **Crash recovery**:
   - Finds one stale `[~]` task from last session: "Run ablation A1"
   - Checks `.research/artifacts/ablation-a1.md` — file exists and looks complete
   - Marks task `[x]` with note "resumed: artifact found"
4. **Reconciles STATE.md** against `git log` — no discrepancy found
5. **Picks next `[ ]` task** from TODO.md: "Write Method §3.2"
6. **Dispatches subagent** with `aris-paper-write`, output path, and return contract
7. **Receives ≤200-word summary** — appends as `[assumption]` to STATE.md
8. **Promotes to `[verified]`** after checking artifact at `artifacts/method-3-2.md`
9. **Updates TODO.md** — moves task `[~]` → `[x]`
10. **Loops** to next task

## Stale task recovery in TODO.md

```markdown
## In progress
- [~] Run ablation A1 [bg] — training on H20, target: fmt_rate > 90%
  > RESUMED 06-12: artifact found at artifacts/ablation-a1.md → marked [x]
```

becomes:

```markdown
## Done
- [x] Run ablation A1 ✓ 06-12 `artifacts/ablation-a1.md`
```
