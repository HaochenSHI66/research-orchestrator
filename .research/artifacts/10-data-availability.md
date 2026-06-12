# 10 — Data Availability for noiseaware_da_v3 SFT

Date: 2026-06-05
Scope: READ-ONLY investigation. Is the v3 training data on GitHub? What's local? Cheapest path to a runnable tokenized dataset on Ascend.

## Bottom line

- The **tokenized** dataset (`data/training/tokenized/noiseaware_da_v3_{train,val}`, HF `save_to_disk`) is **NOT in GitHub** and is explicitly **.gitignored** (`data/training/tokenized/`).
- The **canonical v3 build-inputs** (`v3_train.jsonl`, `v3_val.jsonl`, `v3_test.jsonl`, `v3_manifest.json`, `v3_train.json`) are **NOT in GitHub either** (absent on both `main` and `codex/training-stage-redesign`). They exist **only in the local working tree as UNTRACKED files**.
- The upstream **processed-trajectory JSONL** that `dataset_v3.py` consumes (`data/processed/*.regen.jsonl|json`) are **also not in GitHub** (`data/processed/` is .gitignored) but **DO exist locally and intact**.
- The tokenizer scripts are present and tracked: `src/training/dataset_v3.py` (record builder → `v3_*.json`) and `src/training/build_tokenized.py` (the `save_to_disk` tokenizer).

So: GitHub alone canNOT reconstitute the tokenized dataset. The recoverable assets all live in the **local Mac working tree**.

## What GitHub actually has under data/ (training-relevant)

Both `main` and `codex/training-stage-redesign` are identical for `data/training/` and contain ONLY legacy/baseline JSON (the v2-era files), NO v3:

```
12443339  data/training/baseline_b1_train.json
10532225  data/training/baseline_b2_train.json
10530982  data/training/baseline_b2b_train.json
 5649236  data/training/baseline_b3_train.json
 5189875  data/training/baseline_b4_train.json
10520738  data/training/baseline_b5_train.json
10532225  data/training/baseline_b9_train.json
     586  data/training/build_stats.json
     308  data/training/dataset_info.json
10585712  data/training/train.json
 1176309  data/training/val.json
```

No `v3_*`, no `tokenized/`, no `data/processed/*` in the repo (tree `truncated: false`, so this is complete).

## .gitignore (relevant lines)

```
# Data (too large for git)
data/raw/
data/processed/
data/annotated/batches/
data/batch/
data/training/tokenized/
...
# Keep small + critical artifacts
!data/training/*.json        # <- whitelists *.json only, NOT *.jsonl
```

Note: `data/training/tokenized/` is hard-ignored. `data/processed/` is ignored (so the v3 build-inputs never reach GitHub). The whitelist re-includes `data/training/*.json` but NOT `*.jsonl`, which is why `v3_train.json` (legacy) was once tracked while `v3_*.jsonl` are not — and currently none of the v3 files are committed at all.

## Local working tree (what physically exists despite "deleted" status)

`git status` shows tracked v2/baseline JSON as **deleted** (`D`), but the v3 files are present as **untracked** (`??`):

```
 D data/training/train.json            # legacy deleted from disk
 D data/training/baseline_b*_train.json
 M data/training/dataset_info.json
?? data/training/v3_manifest.json
?? data/training/v3_test.jsonl   (2.5 MB)
?? data/training/v3_train.json   (36 MB, legacy json-list form)
?? data/training/v3_train.jsonl  (21 MB)   <- canonical SFT train input
?? data/training/v3_val.jsonl    (1.3 MB)  <- canonical SFT val input
```

`data/processed/` (upstream inputs named in v3_manifest.json) exists and is intact locally, e.g.:

```
phase1_all_verified.regen.json        3.9 MB
phase2_verified_chunk0.regen.jsonl   13.8 MB
phase2_verified_chunk1.regen.jsonl   12.6 MB
phase2_verified_chunk2.regen.jsonl   14.2 MB
phase2_verified_chunk3.regen.jsonl   13.7 MB
phase2_supplement_4omini_verified.regen.jsonl  9.5 MB
phase_deepseek_verified.regen.json   11.3 MB
```

These 7 files (with sha256s) are exactly the `input_paths` recorded in `data/training/v3_manifest.json`.

## Pipeline (confirmed from scripts)

1. `src/training/dataset_v3.py` `--input <processed *.jsonl/json> ... --output data/training/v3_train.json` → builds weighted record set (the v3_*.jsonl/json).
2. `src/training/build_tokenized.py` `--train-json v3_train.jsonl --val-json v3_val.jsonl --tokenizer <Qwen2.5-7B> --out-dir data/training/tokenized --name noiseaware_da_v3` → `save_to_disk` → `noiseaware_da_v3_{train,val}`. CPU-only, no API.

## Recommended cheapest path

The tokenized set is NOT recoverable from GitHub. It must be rebuilt, and the only place the inputs survive is the **local Mac**. Cheapest route:

1. **scp the already-built canonical inputs** from local Mac to Ascend:
   - `data/training/v3_train.jsonl` (21 MB) and `data/training/v3_val.jsonl` (1.3 MB) — these are the direct `build_tokenized.py` inputs.
2. On Ascend run `build_tokenized.py --train-json v3_train.jsonl --val-json v3_val.jsonl --tokenizer <Qwen2.5-7B-Instruct> --out-dir data/training/tokenized --name noiseaware_da_v3`. CPU, cheap, no network/API.

If the orchestrator distrusts the prebuilt jsonl, the fully reproducible (still local-only) fallback is to also copy the 7 `data/processed/*.regen.*` files (~79 MB) and re-run `dataset_v3.py` first, then `build_tokenized.py`. Both source sets are present locally; `v3_manifest.json` carries sha256 for integrity checks.

There is NO need to involve the original AutoDL box or regenerate trajectories via LLM — all source artifacts survive locally.

## Risk flags

- The v3 inputs are **untracked and uncommitted** — they exist on exactly ONE disk (the Mac). No GitHub backup. If that disk is lost, the data is gone (would require full regeneration). Recommend the user commit/back up `v3_*.jsonl` + manifest (note: `*.jsonl` is currently not git-whitelisted, and processed/ is hard-ignored).
- Could not byte-verify the local files against `v3_manifest.json` sha256 in this pass (sizes match the manifest, which is strong but not conclusive).
- `build_tokenized.py` needs the Qwen2.5-7B tokenizer with a chat template supporting the `tool` role; confirm that tokenizer is present on Ascend.
