# research-orchestrator

**A Claude Code skill that runs a research project as a lean orchestrator — the main thread only plans, records, and decides; every unit of real work is delegated to a subagent that writes its detail to disk and returns a compact summary, so the conversation stays clear across a months-long project.**

> 把科研项目当成"指挥 + 记账"来跑:主线程只做决策,所有重活甩给 subagent,详细产物落盘、只回精简摘要,主对话几十步之后依然清爽。

---

## Why

Research projects are long. If you run literature reviews, write experiment code, parse training logs, and draft paper sections directly in one conversation, the context fills with detail that's irrelevant to the *next* decision — and you lose the thread of the whole project.

`research-orchestrator` flips this. The main thread holds the plan and the state. Subagents hold the mess. The result is a conversation you can still read 50 steps in.

## What it does

- **Lean main thread.** One rule: never do research work in the main thread. Every task goes to a subagent that writes its full output to a file under `.research/` and returns only a ≤200-word summary + artifact paths + the decision you now face. The detail is safe on disk; your context stays navigable.
- **Everything delegated.** Literature, experiment design, code, runs, analysis, writing, review — each is a dispatched subagent with a self-contained brief.
- **Auto-invokes your academic skills.** Each research stage maps to the matching skill from the [ARIS](#academic-skill-families) family (and falls back to `scholar-*` / `arl-* `/ `phd-*` / general skills). The orchestrator tells each subagent which skill to run.
- **Keeps everything.** All plans, designs (versioned, never overwritten), and decisions — *including the options that were rejected* — are persisted under the project's `.research/` directory.
- **Loops autonomously.** It keeps moving through ordinary work and only stops at genuine **human gates**: a research-direction choice, a missing credential/API key, an interactive login, or an irreversible/outward-facing action.
- **Second opinion from ChatGPT.** For high-stakes judgment calls (is this idea novel? is this design sound? does this claim follow?), it pulls an independent cross-analysis from ChatGPT via the [codex MCP server](#optional-integrations) — run inside a subagent, saved to `.research/chatgpt/`.

## How it works

```
1. Read .research/STATE.md (short — your living memory of the project).
2. Pick the next step from .research/plan.md.
3. Map the step → research stage → the right academic skill.
4. Dispatch a subagent (task + skill + .research/ output path + "return only a summary").
5. Record the summary to STATE.md (and decisions/ if a choice was made).
6. Update plan.md. Next step a human gate? Stop and ask. Otherwise loop.
```

The `.research/` workspace:

```
.research/
├── STATE.md      # short current-state snapshot — first thing every session reads
├── plan.md       # living master plan
├── designs/      # design docs, versioned, never overwritten
├── decisions/    # one entry per human-gate decision (options, choice, rationale)
├── artifacts/    # subagent working outputs
└── chatgpt/      # ChatGPT cross-analysis exchanges
```

## Install

Clone straight into your Claude Code skills directory:

```bash
git clone https://github.com/HaochenSHI66/research-orchestrator \
  ~/.claude/skills/research-orchestrator
```

That's it — Claude Code discovers it on next launch.

## Usage

Point Claude at a research project and ask it to push forward:

```
/research-orchestrator
```

or just, inside a research project:

> Keep this project moving — pick up where we left off.

The skill locates the project, opens (or bootstraps) `.research/`, reconciles the recorded state against reality, and starts the loop. It pauses only when it needs *you* — to choose a direction, hand over a key, log in, or approve something irreversible.

## Compatibility

- **Claude Code** with subagent support (the `Agent` tool). The orchestrator pattern depends on delegating to subagents.

### Academic skill families

The stage→skill map ([`references/stage-skill-map.md`](references/stage-skill-map.md)) targets the **ARIS** academic skill family (`aris-idea-discovery`, `aris-research-lit`, `aris-experiment-plan`, `aris-analyze-results`, `aris-paper-write`, `aris-citation-audit`, `aris-rebuttal`, …). If you don't have ARIS installed, subagents fall back to the closest available skill (`scholar-*`, `arl-*`, `phd-*`) or proceed without one — the orchestration still works, you just lose the stage-specific guidance.

### Optional integrations

- **codex MCP server** — for the ChatGPT second-opinion cross-analysis. If it isn't connected, the orchestrator treats that as a human gate rather than skipping the check. Everything else runs without it.

## Repo layout

```
research-orchestrator/
├── SKILL.md                       # the skill (orchestrator logic)
├── references/
│   ├── stage-skill-map.md         # research stage → academic skill mapping
│   └── workspace-layout.md        # .research/ structure + STATE/plan/decision templates
├── README.md
└── LICENSE
```

## License

MIT — see [LICENSE](LICENSE).
