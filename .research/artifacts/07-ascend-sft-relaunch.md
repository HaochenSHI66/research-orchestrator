# 07 — Ascend SFT Relaunch (bf16 LoRA, 2-card) — LAUNCH BLOCKED

Date: 2026-06-05
Server: `target-cf` (ITS-SM-G-R113), login `shihaochen`, 8×910B2 (64GB/card), HCCL, manual-card convention.
Repo: `HaochenSHI66/NoiseAware-DA` @ branch `codex/training-stage-redesign`, commit `2f45761`.
Skill used: `phd-launch` (pre-flight checklist).

## OUTCOME: STOPPED before claiming cards. Did NOT launch. Two blockers.

The launch was gated on a must-pass import-sanity + prerequisite check.
The clone is fine; the env (with the *correct* CANN script) is fine; but
(A) the full import set FAILS and (B) the required tokenized dataset is ABSENT.
Per the safety rails ("If a prerequisite fails ... STOP and report it as a
blocker — do not burn cards on a broken setup"), no cards were claimed.

---

## Step 1 — NPU inventory (`npu-smi info`)

| Card | HBM used / total | Process | State |
|------|------------------|---------|-------|
| 0 | 19894 / 65536 MB | python3 (pid 2631363, 16.5GB) | BUSY |
| 1 | 3428 / 65536 MB | none | IDLE (~62GB free) |
| 2 | 3430 / 65536 MB | none | IDLE (~62GB free) |
| 3 | 3430 / 65536 MB | none | IDLE (~62GB free) |
| 4 | 22466 / 65536 MB | python (pid 2908378, 19GB) | BUSY |
| 5 | 3430 / 65536 MB | none | IDLE (~62GB free) |
| 6 | 3431 / 65536 MB | none | IDLE (~62GB free) |
| 7 | 3430 / 65536 MB | none | IDLE (~62GB free) |

6 idle cards, each ~62GB free (>>32GB). Card availability PASSES. Intended
claim was 1,2 — but NOT claimed because downstream checks failed.

## Step 2 — Clone + checkout (PASS)

- Fresh clone to `~/SHC/NoiseAware-DA` (no prior copy existed).
- Token used in clone URL, then `git remote set-url origin` to the tokenless
  URL. Verified `git remote -v` shows NO token. Token never printed.
- `git checkout codex/training-stage-redesign` → tracking origin branch.
- `git log -1`: `2f457614e2efaf67fae0ebbee209d090fb838275`
  "feat(training): Ascend bf16 LoRA path (LR 2e-4) + fix D2 rollout reward column leak"
  (2026-06-04 21:54 +0800). Corrected commit CONFIRMED.
- Required files present: `experiments_v3/sft_qwen7b_ascend_bf16/train_config.yaml`,
  `scripts/run_sft_ascend_2card.sh`. PASS.

## Step 3 — Import sanity (GATED) — **FAILED**

### Environment note (resolved):
The stale `~/SHC/set_cann_env.sh` references libs that no longer exist
(`stubs/msprof_noop.so`, `cann_libs/.../liberror_manager.so` → both missing).
The CORRECT CANN env is the one the working URIS jobs use:
`source ~/SHC/Ascend/ascend-toolkit/set_env.sh` (+ `nnal/atb/set_env.sh`).

### Deps — PASS (with correct CANN env):
```
deps ok 2.10.0+cpu 4.57.6   # torch, torch_npu, transformers, peft, accelerate all import
```
(torch reports `+cpu` tag but torch_npu loads cleanly once CANN is sourced.)

### Modules — **FAIL** (genuine code defect in the pushed branch):
```
python -c "import src.training.train, src.evaluation.inference, src.rl.rollout, src.rl.rft_loop"
ImportError: cannot import name 'PersistentInterpreterBackend'
  from 'src.pipeline.l1_sandbox' (.../src/pipeline/l1_sandbox.py)
```
Root cause: `src/evaluation/inference.py:30` and `src/rl/rollout.py:46` both
`from src.pipeline.l1_sandbox import (... PersistentInterpreterBackend ...)`,
but `l1_sandbox.py` defines only `SandboxBackend`, `SubprocessBackend`,
`DockerBackend` (+ `create_sandbox`, `execute_code`). The
`PersistentInterpreterBackend` class was never added/committed. This is a
missing-sibling defect ON THE BRANCH, not an env issue.

### Scope of the defect (important nuance):
- `src.training.train` (the SFT entrypoint) imports CLEAN and does NOT
  transitively pull in `inference`/`rollout`/`rft_loop`. Verified:
  "broken modules pulled into train: NONE - train path clean".
- So the broken symbol affects the EVAL and RL stages (later), not this SFT
  launch's import path. SFT could import-start — BUT see blocker B.

## Step 3b — Dataset prerequisite — **FAIL (hard blocker for SFT itself)**

The launch script + config require a PRE-BUILT tokenized HF Dataset; tokenization
is a separate CPU step that is explicitly NOT done by the launch script.
- Config `dataset_name: noiseaware_da_v3` → `train.py` resolves to
  `data/training/tokenized/noiseaware_da_v3_{train,val}` and calls
  `load_from_disk`. `resolve_tokenized_dataset_paths(check_exists=True)` raises
  `FileNotFoundError` if a split is missing (train.py:637-638).
- `data/training/tokenized/` DOES NOT EXIST in the clone (only raw
  `train.json`/`val.json` are committed; the build inputs `v3_train.jsonl` /
  `v3_val.jsonl` referenced by `build_tokenized.py` are also absent).
- Server-wide read-only search found NO pre-built tokenized dataset, NO
  `SFT_backup`, NO `v3_*.jsonl` anywhere under `/home/shihaochen`.

=> Even ignoring the import defect, the SFT run would crash at startup with
`FileNotFoundError` on the tokenized splits. The data is simply not on this box.

## Steps 4–6 — NOT REACHED

No OUTPUT_ROOT created, no cards claimed (`ASCEND_RT_VISIBLE_DEVICES` NOT set),
no tmux session, no torchrun launched. No loss / grad_norm / ETA evidence —
training never started, by design (gated stop).

## Safety rails honored
- Read-only on all existing data; nothing touched/moved/deleted/built.
- 0 cards claimed (intended 2; not used).
- All work on server via ssh; no scp-down to the Mac.
- Token scrubbed from git remote; never printed.

## Recommended resolution (for the orchestrator / next launch)
1. CODE FIX: add `PersistentInterpreterBackend` to `src/pipeline/l1_sandbox.py`
   (or remove the unused import from inference.py/rollout.py if dead) and push
   to the branch. Blocks eval + RL stages; does NOT block SFT import.
2. DATA (blocks SFT itself): build the tokenized dataset on the server CPU —
   `python -m src.training.build_tokenized --train-json ... --val-json ...
   --out-dir data/training/tokenized --name noiseaware_da_v3` — which first
   needs the `v3_{train,val}.jsonl` build inputs (or repoint to wherever the
   user's authoritative tokenized v3 dataset lives; it was NOT found on this
   server). NOT done here: this is a data-build step the brief did not authorize
   and the rails forbid touching/creating data artifacts unprompted.
3. Use the correct CANN env: `source ~/SHC/Ascend/ascend-toolkit/set_env.sh`
   (NOT the stale `~/SHC/set_cann_env.sh`).

## Verification-ledger item
Import-sanity check: **FAILED** — the pushed branch (2f45761) is INCOMPLETE.
`PersistentInterpreterBackend` is imported by inference.py + rollout.py but
never defined in l1_sandbox.py.
