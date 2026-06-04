# Research stage → academic skill map

The orchestrator never runs these itself. It tells the dispatched subagent which
skill to invoke. The subagent runs the skill in its own context and returns a
compact summary. If the named skill doesn't fit, the subagent picks the closest
available skill and says which it used.

The ARIS family (`aris-*`) is the user's primary academic toolkit. Where ARIS has
no good match, fall back to `scholar-*`, `arl-*`, `phd-*`, or the `superpowers`
process skills.

## Stage map

| Stage | Primary ARIS skill(s) | Fallbacks |
|---|---|---|
| Idea discovery / framing | `aris-idea-discovery`, `aris-idea-creator` | `brainstorming-research-ideas`, `sci-scientific-brainstorming`, `arl-idea-generation` |
| Novelty check | `aris-novelty-check` | `arl-novelty-assessment` |
| Literature search | `aris-research-lit`, `aris-openalex`, `aris-semantic-scholar`, `aris-arxiv`, `aris-deepxiv` | `arl-literature-search`, `phd-literature-research`, `deep-research` |
| Literature synthesis / related work | `aris-research-lit` | `arl-related-work-writing`, `arl-survey-generation` |
| Research planning / pipeline | `aris-research-pipeline`, `aris-paper-plan` | `arl-research-planning`, `phd-experiment-design`, `write-plan` |
| Theory / derivation | `aris-formula-derivation`, `aris-proof-checker` | `arl-math-reasoning`, `sci-sympy` |
| Experiment design | `aris-experiment-plan`, `aris-ablation-planner`, `aris-experiment-bridge` | `phd-experiment-design`, `arl-experiment-design` |
| Experiment code | `aris-experiment-bridge` | `arl-experiment-code`, `phd-launch`, domain skills (`grpo-rl-training`, `peft`, `vllm`, etc.) |
| Running / monitoring | `aris-monitor-experiment`, `aris-experiment-audit` | `phd-launch`, `swanlab`, `weights-and-biases` |
| Results analysis | `aris-analyze-results`, `aris-result-to-claim` | `arl-data-analysis`, `scholar-results-analysis`, `sci-statistical-analysis` |
| Claims drafting | `aris-claims-drafting`, `aris-result-to-claim` | — |
| Figures / diagrams | `aris-paper-figure`, `aris-paper-illustration`, `aris-mermaid-diagram` | `paper-plot-from-data`, `paper-plot-from-image`, `arl-figure-generation` |
| Tables | — | `arl-table-generation` |
| Paper writing | `aris-paper-write`, `aris-paper-plan`, `aris-paper-compile` | `arl-paper-writing-section`, `scholar-ml-paper-writing`, `20-ml-paper-writing`, `paper-writer-imrad` |
| Citation / claim audit | `aris-citation-audit`, `aris-paper-claim-audit` | `scholar-citation-verification`, `aw-paper-audit` |
| Self-review before submit | `aris-research-review`, `aris-auto-review-loop` | `scholar-paper-self-review`, `arl-self-review`, `feedback-review-paper` |
| Rebuttal | `aris-rebuttal` | `arl-rebuttal-writing`, `scholar-review-response`, `phd-reviewer-defense` |
| Resubmission | `aris-resubmit-pipeline` | — |
| Slides / poster / talk | `aris-paper-slides`, `aris-paper-poster`, `aris-paper-talk` | `arl-slide-generation`, `presenton` |

## How to choose within a stage

- If several skills in a row apply (e.g. literature: openalex + semantic-scholar +
  arxiv), the subagent can use more than one — but keep each subagent's job
  bounded. Prefer one subagent per sub-question over one subagent doing everything.
- The user's projects are mostly ML/LLM/CV research. The large `cv-*`, `nlp-*`,
  `llm-*`, `tabular-*`, `timeseries-*` skill libraries are technique-level recipes
  — surface them to the subagent when a step is "implement technique X," not for
  high-level stage work.
- When unsure, the subagent should state which skill it used and why, so the
  orchestrator can correct the mapping on the next step.

## Cross-cutting verification references

These apply on top of the stage skills, per the top-venue standards in SKILL.md:

- **Citations** → `references/citation-verification.md` — download the real PDF and
  confirm the cited claim; wire `aris-arxiv` (download) + `aris-citation-audit`
  (claim audit) + `flonat-bib-validate --verify-doi` (cheap existence pre-gate).
- **Figures/tables** → `references/figure-discipline.md` — use `paper-plot-from-data`
  for ALL figures (fixed styles/palette); justified + diverse + venue-appropriate count.
- **Reviewer attacks** → `references/reviewer-attack-checklist.md` — what to stress-test
  every design/experiment/claim against before it's considered sound.
