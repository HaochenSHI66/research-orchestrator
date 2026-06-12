# Artifact 17 — NoiseAware-DA First SFT Run (1-card Ascend) — Launch Log

**Date:** 2026-06-09
**Operator:** Claude (phd-launch skill invoked)
**Task:** Bring up first real SFT run of reframed NoiseAware-DA pipeline on shared Ascend 910B2 box, ONE card, confirm healthy start.
**Outcome:** BLOCKED at Gate 1 (insufficient disk). No card claimed. No download started. No training launched.

---

## Gate 1 — Environment + Resources

### SSH / login — PASS
- `sshpass -p '***' ssh target-cf` → connected.
- `whoami` = `shihaochen`, `HOME=/home/shihaochen`, host `ITS-SM-G-R113`.

### CANN + conda — PASS (with note)
- `source ~/SHC/Ascend/ascend-toolkit/set_env.sh` → OK (npu-smi on PATH after sourcing).
- `conda` not on PATH in non-interactive ssh. Conda located at `~/SHC/miniconda3`.
  Activate via: `source ~/SHC/miniconda3/etc/profile.d/conda.sh && conda activate sft`.
- Envs present: `base`, `cann_py311`, `sft`. Target env `sft` exists.

### NPU cards — PASS (≥1 idle)
8× 910B2, 64GB/card (65536 MB HBM). Snapshot:
- Card 0: IN USE — procs 3346248 (python, 3126MB) + 2631363 (python3, 16526MB); HBM 23046MB.
- Card 7: IN USE — proc 3473765 (python, 18892MB); HBM 22267MB.
- Cards 1,2,3,4,5,6: **IDLE** — no processes, HBM ~3.4GB baseline each.
- => 6 idle cards available. Candidate for the run: **card 1** (or any of 2–6).

### Disk — **FAIL (BLOCKER)**
```
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/openeuler-home  364G  320G   26G  93% /home
```
- Only **26GB free**, filesystem at **93%**. Gate requires ~40GB (model ~15GB + tokenized datasets + outputs).
- `Qwen2.5-7B-Instruct` is **NOT** present (`~/SHC/models/` has only `covt-7b` 16G and `Qwen3-32B` 62G).
- Top consumers under ~/SHC:
  ```
  77G  ~/SHC/models           (Qwen3-32B 62G, covt-7b 16G)
  43G  ~/SHC/birdclef2026_clef
  9.0G ~/SHC/Ascend
  8.3G ~/SHC/miniconda3
  4.6G ~/SHC/URIS
  4.2G ~/SHC/checkpoints
  3.7G ~/SHC/thinkprobe
  121M ~/SHC/NoiseAware-DA
  ```
- All large dirs belong to OTHER projects. Per safety rules (read-only on existing data/results/SFT_backup; NEVER delete/move server files without user decision), I did not remove anything.

**Decision: STOP. Downloading the ~15GB model would leave ~11GB; building 3 tokenized SFT datasets + writing training checkpoints/logs would almost certainly exhaust /home and crash the run mid-training. This is exactly the expensive mid-run failure the pre-flight checklist exists to prevent. Halting before claiming the card or starting any download.**

---

## Gates 2–7 — NOT ATTEMPTED (blocked by Gate 1)
- Gate 2 (repo update / import check): not run.
- Gate 3 (model download): not started — would consume the disk.
- Gate 4 (build 3 ablation datasets): not run.
- Gate 5 (1-card launch script): not built.
- Gate 6 (launch l1_unified_soft seed 42): not launched.
- Gate 7 (watch first steps): N/A.

---

## What the user needs to decide (to unblock)
Free up disk so there is ~40GB headroom, OR redirect model/output to a different filesystem. Options (USER decides — I will not act without explicit approval):
1. Confirm whether `~/SHC/models/Qwen3-32B` (62G) or `~/SHC/birdclef2026_clef` (43G) are still needed; if not, removing one frees ample space.
2. Point model dir + `OUTPUT_ROOT` at another mount with free space (none confirmed yet — `/home` is the only mount inspected; would need user to name an alternative).
3. Use a smaller/quantized model if the pipeline allows (not in current spec — spec mandates Qwen2.5-7B-Instruct bf16).

## Risk flag: HIGH — do not proceed until disk headroom resolved.
