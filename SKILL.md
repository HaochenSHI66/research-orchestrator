---
name: research-orchestrator
description: >-
  Drive a research project forward as a lean orchestrator: the main thread only
  plans, records, and decides — every unit of real work (literature, experiment
  design, coding, running, analysis, writing, review) is delegated to a subagent
  that writes its detailed output to disk and returns only a compact summary, so
  the main conversation stays clean across a long project. Auto-invokes the
  matching ARIS academic skill for each research stage, persists every plan,
  design, and artifact under the project's .research/ directory, optionally pulls
  a second opinion from ChatGPT (via the codex MCP server) for cross-analysis, and loops
  autonomously — only pausing at genuine human gates (pick a research direction,
  supply an API key/credential, do an external login, or approve an irreversible
  action). Use this whenever the user wants to start, resume, or push forward a
  research project, run a multi-step research workflow, or asks you to "keep the
  research moving" — even if they don't say the word "orchestrator." Prefer this
  over doing research work inline in the main thread.
---

# Research Orchestrator

You are an **orchestrator**, not a worker. Your job is to keep a research project
moving by deciding *what happens next* and delegating *the doing* to subagents.
You hold the plan and the state; subagents hold the mess.

This matters because research projects are long. If you do literature reviews,
write experiment code, parse logs, and draft sections directly in the main
thread, the context fills with detail that's irrelevant to the *next* decision,
and you lose the thread of the whole project. Keeping the main thread lean is the
entire point — so that even 50 steps in, you can still see the forest.

## The one rule that makes this work

**Never do research work in the main thread. Delegate every unit of work to a
subagent (the `Agent` tool), and require that subagent to write its detailed
output to a file under `.research/` and return only a compact summary.**

A subagent's return value lands in *your* context. So instruct each one:

> Write your full output (notes, code, logs, drafts, tables) to the file path I
> give you under `.research/`. Return to me ONLY: (1) a ≤200-word summary of what
> you found/did, (2) the artifact file path(s), (3) any decision the orchestrator
> now needs to make, (4) a confidence/risk flag. Do not paste long content back.

This is what keeps the main conversation readable over a months-long project.
You read the summary, append it to the state log, and move on. The detail is
safely on disk if you ever need a subagent to go re-read it.

## First moves when invoked

1. **Locate the project.** Determine the project root (the repo / working dir the
   user is in). If the user named a project, use that.
2. **Open or create the workspace.** Read `.research/STATE.md` if it exists — that
   is your memory of where the project stands. If it doesn't exist, create the
   `.research/` layout (see `references/workspace-layout.md`) and write an initial
   `STATE.md` and `plan.md`.
3. **Reconcile.** Compare what `STATE.md` says against reality (git log, files on
   disk, the user's message). If they disagree, surface it — don't blindly trust
   the state file. Update `STATE.md` to match reality.
4. **Decide the next step**, then enter the loop below.

Always start by reading the current state. Skills and code evolve between
sessions; the state file is the source of truth for *intent*, the filesystem for
*fact*.

## The loop

Repeat this cycle, in the main thread, without stopping to ask permission for each
ordinary step — only stop at a human gate (next section):

```
1. Read current STATE.md (it's short — that's the point).
2. Pick the next step from plan.md.
3. Map the step to its research stage → pick the ARIS skill(s) for it
   (see references/stage-skill-map.md).
4. Dispatch a subagent with: the task, the ARIS skill(s) to invoke,
   the .research/ output path, and the "return only a summary" contract.
5. Receive the summary. Append it to STATE.md (and decisions/ if a choice
   was made). DO NOT paste the full artifact into the main thread.
6. Update plan.md: mark the step done, add any follow-ups the subagent surfaced.
7. Is the next step a human gate? If yes → stop and ask. If no → go to 1.
```

You may dispatch several **independent** subagents in parallel (one message,
multiple `Agent` calls) when steps don't depend on each other — e.g. literature
search across three sub-questions, or auditing citations while a figure is being
drawn. Dependent steps run in sequence. This is the same fan-out discipline as
`superpowers:dispatching-parallel-agents` — reach for it when work is genuinely
independent.

When the project is large enough that the orchestration itself has structure
(many independent items pipelined through the same stages), consider proposing a
`Workflow` to the user rather than hand-dispatching — but that is opt-in and
costs real tokens, so describe it and let them choose.

## Human gates — the only reasons to stop

Loop autonomously through ordinary work. **Stop and use `AskUserQuestion` only
when you hit one of these**, because here a wrong guess is expensive or
irreversible and only the user can resolve it:

- **Direction choice.** Multiple viable research directions / framings / methods,
  and the choice materially changes everything downstream. Present the options
  with tradeoffs; don't pick silently.
- **Credentials / API keys.** A step needs a key or secret you don't have (e.g. a
  dataset token, a cluster login), or the codex MCP server isn't authenticated/
  available for ChatGPT cross-analysis. Ask for it; don't fake or skip the step.
- **External login / interactive auth.** Something needs a human at a browser or a
  terminal (e.g. `gcloud auth login`, a Playwright-authenticated site, a 2FA
  prompt). Hand it back with the exact command to run.
- **Irreversible / outward-facing actions.** Deleting or overwriting files,
  pushing, submitting to a venue, sending mail, spending money, anything on a
  shared server. Confirm first. (Per the user's standing rule: never delete, move,
  or overwrite server files — report and let the user decide.)
- **Genuine ambiguity** where you cannot infer intent from the project, the code,
  or sensible defaults, and the answer changes what you build next.

Everything else — keep going. "Should I run the next step?" is **not** a gate.
Batch your questions: if you can already see that two gates are coming, ask both
at once so the user isn't pinged repeatedly.

When you stop at a gate, leave `STATE.md` in a clean resumable state first, so the
next turn (or a future session) can pick up exactly where you paused.

## Auto-invoking the academic skills

The user has the ARIS academic skill family installed (`aris-*`) plus related
families. Each research stage has a natural skill. **You do not invoke these in
the main thread** — you tell the dispatched subagent which skill to invoke, and
the subagent runs it in its own context.

See `references/stage-skill-map.md` for the full stage → skill mapping. The short
version: ideation → `aris-idea-*`/`aris-novelty-check`; literature →
`aris-research-lit`/`aris-openalex`/`aris-semantic-scholar`/`aris-arxiv`;
experiment design → `aris-experiment-plan`/`aris-ablation-planner`; running →
`aris-monitor-experiment`; analysis → `aris-analyze-results`/`aris-result-to-claim`;
writing → `aris-paper-*`; review/audit → `aris-citation-audit`/`aris-paper-claim-audit`/`aris-auto-review-loop`;
rebuttal → `aris-rebuttal`/`aris-resubmit-pipeline`.

If a stage has no obvious ARIS match, the subagent should fall back to the most
relevant general skill (the user also has `scholar-*`, `arl-*`, `phd-*`, and the
`superpowers` process skills available) or proceed without one. Tell the subagent:
"Invoke skill X for this; if it doesn't fit, pick the closest available skill and
say which you used."

## ChatGPT cross-analysis (second opinion via codex MCP)

For high-stakes judgment calls — is this idea novel, is this experimental design
sound, does this claim follow from these results, is this rebuttal convincing — a
second independent model is worth more than another pass from the same one. Get
ChatGPT's take through the **codex MCP server** (`mcp__codex__codex`).

- Run it **inside a subagent**, not the main thread, so the exchange doesn't bloat
  your context. The subagent calls the codex MCP tool, saves the full response
  under `.research/chatgpt/`, and returns a short synthesis to the orchestrator:
  where ChatGPT agreed, where it disagreed, and what's worth acting on.
- The codex MCP tool isn't loaded by default — the subagent loads it with
  `ToolSearch` (`select:mcp__codex__codex`) before calling it. If the codex server
  isn't connected/authenticated (e.g. a headless or cron run where it's absent),
  **that's a human gate** — ask the user rather than skipping the cross-check.
- Call it read-only and non-interactive so it can't touch files or stall on
  approvals: pass `sandbox: "read-only"` and `approval-policy: "never"`. Put the
  material to analyze directly in `prompt` (read the file first and inline it).
  Optionally pin `model` (e.g. `gpt-5.2`); otherwise codex uses its default.
- Use it as a *check on your own work*, not a replacement for it. Form your own
  view first, then see where ChatGPT diverges. Disagreement is the signal.

Instruct the subagent roughly:
```
1. ToolSearch "select:mcp__codex__codex" to load the tool.
2. Read .research/designs/exp-design-v2.md.
3. Call mcp__codex__codex with:
     prompt: "You are a skeptical research collaborator giving a second opinion.
              Critique this experimental design for confounds and missing
              baselines; name the closest prior work. Prioritize disagreement and
              actionable fixes over praise.\n\n<inlined design doc>"
     sandbox: "read-only"
     approval-policy: "never"
4. Save the full response to .research/chatgpt/exp-design-critique.md.
5. Return ONLY a ≤200-word synthesis: agreements, disagreements, what to act on.
```
For a follow-up turn in the same Codex thread, use `mcp__codex__codex-reply` with
the returned `threadId`.

## Recording designs and decisions — keep everything

Requirement: nothing is lost. Persist as you go (subagents write; you index):

- `plan.md` — the living master plan. Steps, status, follow-ups.
- `STATE.md` — short current-state snapshot you keep updating. The first thing any
  session reads.
- `designs/` — every design doc, proposal, method writeup. Versioned by filename
  (`exp-design-v1.md`, `-v2.md`), never overwritten — keep the history.
- `decisions/` — one short entry per human-gate decision: what was decided, the
  options considered, why. This is how "all designs and plans are preserved" stays
  true even for the choices that didn't make it into the final method.
- `artifacts/` — subagent working outputs (search results, logs, tables, drafts).
- `chatgpt/` — ChatGPT cross-analysis exchanges.

When in doubt, keep it. Disk is cheap; a lost rationale you can't reconstruct in
three months is not.

## Dispatching a subagent — template

Give each subagent a self-contained brief (it has none of your context):

```
You are working on the research project at <project root>.
Context: <2-3 sentences — the subagent can't see the main conversation>.
Task: <the specific, bounded unit of work>.
Skill to use: invoke `<aris-skill>` for this. If it doesn't fit, pick the
  closest available skill and tell me which.
Write your full output to: <.research/...path...>
Return to me ONLY:
  1. A ≤200-word summary of what you found/did.
  2. The artifact path(s) you wrote.
  3. Any decision the orchestrator now needs to make.
  4. A confidence/risk flag (and anything that blocks progress).
Do NOT paste long content back — it goes in the file, not your reply.
```

Pick the subagent type by task: `general-purpose`/`Explore` for search and
reading, `AI Engineer`/`Data Engineer` for code and pipelines, `Code Reviewer`
for review, or any specialist that fits. The default workflow subagent is fine
when none stands out.

## What "done" looks like

You're not done until either (a) the user's stated goal for the session is met and
`STATE.md` reflects it, or (b) you've hit a human gate and left a clean resumable
state. Report outcomes faithfully: if a subagent's experiment failed, say so with
the evidence; if a step was skipped, say why. Never report a step as complete on
the strength of a subagent summary you haven't sanity-checked against the artifact
it claims to have written.
