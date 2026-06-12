# Ascend SFT Launch (corrected run) — 2026-06-05

Skill: phd-launch
Server: target-cf (8×910B2, 64GB/card)
Goal: prepare data + launch corrected bf16 SFT, confirm healthy start. EXACTLY 2 cards.

## Gate log

### Gate 1: NPU availability — PASS
SSH OK (host ITS-SM-G-R113, user shihaochen).
8×910B2. Busy: cards 0,1,2,3 (running python procs, 26-62GB used).
IDLE: cards 4,5,6,7 — each ~3.4GB baseline used, ~62GB free, no processes.
Plan: claim cards 4,5 (exactly 2). ≥4 idle cards confirmed.

### Gate 2: Repo — PASS
Repo at ~/SHC/NoiseAware-DA, branch codex/training-stage-redesign, clean tree.
Was at 2f45761; fetched (PAT via ephemeral -c http.extraheader, NOT stored).
reset --hard origin → HEAD now `2276698 Add untracked src modules so the branch is a complete runnable project`.
Remote = clean https://github.com/HaochenSHI66/NoiseAware-DA.git; no extraheader in local config.

### Gate 3: Import check — PASS
`python -c "import src.rl.reward, src.rl.rollout, src.rl.rft_loop, src.evaluation.inference, src.training.train, src.training.dataset_v3"` → `imports ok`.
(env: source ascend set_env.sh + conda activate sft)

### Gate 4: Data upload — PASS
Mac build-inputs all present. Server data/training/ existed (baseline_*, train.json, val.json) but NO v3_* files; data/processed/ absent; no tokenized dir.
Decision: build_tokenized.py consumes v3_train.jsonl/v3_val.jsonl DIRECTLY (ShareGPT JSONL via --train-json/--val-json). The data/processed/*.regen.* files feed build_v3_dataset.py (which PRODUCES v3_*.jsonl), NOT build_tokenized. Since v3_*.jsonl already exist on Mac, only the 3 v3 files needed uploading. regen files NOT required.
Upload was flaky (server load avg ~10, intermittent conn drops); first scp truncated v3_train to 16.4MB. Re-uploaded all 3 with ServerAliveInterval + retry loop.
Verified server sizes EXACTLY match Mac:
  v3_train.jsonl  21298920  (21.3MB) OK
  v3_val.jsonl     1277436  (1.3MB)  OK
  v3_manifest.json    5082           OK

### Gate 7-prep: config + run script present — PASS
experiments_v3/sft_qwen7b_ascend_bf16/train_config.yaml exists: bf16 (quantization none), LR 2e-4, 3 epochs, LoRA r=64/alpha=128, per_device_bs=1 grad_accum=16 (eff batch 32), cutoff 8192, gradient_checkpointing on, HCCL.
scripts/run_sft_ascend_2card.sh exists: pins nproc_per_node=2, asserts exactly 2 cards, honors OUTPUT_ROOT + SEED env, torchrun → src/training/train.py --run noiseaware_da_v3_ascend_bf16.

### Gate 6: Base model availability — **BLOCKED (STOP)**
Config requires `Qwen/Qwen2.5-7B-Instruct` (text, bf16, ~15GB).
Exhaustive search — NOT present on the box:
  ~/.cache/huggingface/hub: has Qwen2.5-**VL**-7B-Instruct, Qwen3-32B, Qwen3-VL-8B — NONE is the text Qwen2.5-7B-Instruct.
  ~/SHC/models: covt-7b (model_type=qwen2_5_vl, a VL model, NOT text 7B), Qwen3-32B.
  find for "*Qwen2.5-7B-Instruct*" dir → no hits. No non-VL qwen2.5-7b safetensors. No HF_HOME override.
Per launch rules: model ABSENT (would require ~15GB download) = STOP + report as a DECISION; do NOT silently start a huge download.

### Gate 5: build tokenized — NOT RUN (deferred)
Blocked by Gate 6: the run cannot launch without the base weights, so building the
tokenized set would be partial work toward an unlaunchable run. Build needs only the
Qwen2.5-7B tokenizer (small, HF-token downloadable) and is CPU-only — ready to run the
moment the model decision is made. Exact command (verified from build_tokenized.py):
  python scripts/build_tokenized.py \
    --train-json data/training/v3_train.jsonl \
    --val-json data/training/v3_val.jsonl \
    --tokenizer Qwen/Qwen2.5-7B-Instruct \
    --output-dir data/training/tokenized \
    --name noiseaware_da_v3 --cutoff-len 8192
  → produces data/training/tokenized/noiseaware_da_v3_{train,val} (save_to_disk).

### Gates 7(deps)/8/9: NOT REACHED — no cards claimed, nothing launched.

## DECISION REQUIRED
Base model `Qwen/Qwen2.5-7B-Instruct` (~15GB bf16) is missing. Options:
  A. Authorize a server-side download via the HF token (slow on this box; flaky link
     during the session — load avg ~10). Then I resume: Gate 5 build → Gate 7 deps →
     launch on cards 4,5.
  B. Point the config/run to an already-present model — but none on the box matches
     (the only 7B-class items are VL models, architecturally wrong for this text SFT).
Recommend A. Until then: STOP. Cards NOT claimed; 4,5,6,7 left idle for others.

## Connectivity note
Server SSH dropped connections several times mid-command (load avg ~10, 6 users).
Used ServerAliveInterval=15 + retry loops to get clean transfers. A long download/run
should use tmux (already planned) to survive drops.
