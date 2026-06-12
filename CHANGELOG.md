# Changelog

## [Unreleased]

### Added
- `TODO.md` system: persistent task board updated every loop iteration, with locked decisions and open questions
- Startup crash/resume recovery: detects stale `[~]` tasks, validates artifact existence on session start
- Failure states `[!]` (failed) and `[?]` (partial/needs-retry) to the TODO status vocabulary
- Experiment provenance schema: `git_commit`, `command`, `seed`, `hardware`, `dataset_version`, `metrics_path`, `log_path` required fields
- Write ownership rule: orchestrator is sole writer to `STATE.md`, `TODO.md`, `plan.md`, `verification-ledger.md`
- Competitive sweep trigger conditions: mandatory for new direction/pivot, optional/skippable for implementation and scoped tasks
- Clarified `sandbox: read-only` scope to codex MCP calls only

### Changed
- Loop now appends summaries as `[assumption]` (PENDING) before evidence promotion — fixes evidence-gate/loop contradiction
- Loop uses `[ ]`/`[~]`/`[x]` markers throughout, consistent with Todolist section
- "Every design must be implemented" softened to "accepted designs must be implemented"
- "Genuine ambiguity" human gate tightened to four specific triggers
- README: reduced redundancy across The problem / What it does / Design principles
- README: hero tagline updated to "plans, dispatches, verifies, and records"
- README: ChatGPT cross-analysis surfaced as required for soundness claims
- README: Requirements section now shows degraded behavior per missing dependency

### Added to README
- Table of contents with jump links
- Limitations section (token cost, hard dependencies, no-rollback, state accumulation)
