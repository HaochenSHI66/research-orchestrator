# Reviewer Attack Checklist (Top-Venue Adversarial Stress-Test)

**Purpose.** This file is what a research-orchestrator subagent consults when adversarially stress-testing a paper's claims, design, and experiments against what reviewers at top venues (NeurIPS/ICML/ICLR, CVPR/ICCV/ECCV, ACL/EMNLP/NAACL/ARR, WSDM/KDD/SIGIR/WWW) actually attack. Each item is framed as: **the attack to preempt** + **what to VERIFY/PROVIDE before submission** + **source(s)**. Organized by attack family, not by venue. Use it as a checklist — find the gap, then make the claim defensible. The goal is *defensibility, not chasing SOTA* (see §11).

---

## Table of Contents
1. [Claims vs evidence / overclaiming](#1-claims-vs-evidence--overclaiming)
2. [Baselines: missing / weak / untuned](#2-baselines-missing--weak--untuned)
3. [Ablations & isolating the contribution](#3-ablations--isolating-the-contribution)
4. [Arbitrary / unjustified design choices (formulas, loss weights, thresholds, schedules)](#4-arbitrary--unjustified-design-choices)
5. [Statistical significance, seeds & variance, underpowered evals](#5-statistical-significance-seeds--variance-underpowered-evals)
6. [Gains from tuning/compute, not the idea ("bag of tricks")](#6-gains-from-tuningcompute-not-the-idea-bag-of-tricks)
7. [Data leakage / contamination / eval validity & splits](#7-data-leakage--contamination--eval-validity--splits)
8. [Novelty & related work / closest prior work](#8-novelty--related-work--closest-prior-work)
9. [Reproducibility (code, hyperparams, seeds, compute)](#9-reproducibility-code-hyperparams-seeds-compute)
10. [Limitations & ethics sections](#10-limitations--ethics-sections)
11. [Do NOT over-correct (official guidance)](#11-do-not-over-correct-official-guidance)
12. [Venue-specific notes (CV / NLP-ARR / DM-IR-WSDM)](#12-venue-specific-notes)
13. [Cheap pre-submission hygiene checks](#13-cheap-pre-submission-hygiene-checks)

---

## 1. Claims vs evidence / overclaiming

**The attack:** A narrow result (one benchmark family / toy task / single domain) is written as a broad method claim; abstract/intro claims exceed what experiments show; "best/SOTA/beats all others" stated without a test; qualitative figures used as load-bearing evidence.

**Verify before submission:**
- Every abstract/intro claim maps to a specific in-scope experiment; no extrapolation from toy/single-benchmark/single-domain to "general"/"real-world". Scope the wording down or test a second domain/distribution shift.
- Remove "best/SOTA/all others/superior/significant" unless backed by a statistical test or a number.
- Every improvement claim is backed by a quantitative metric on a defined dataset; qualitative figures are illustrative only.
- NLP: you only need evidence for *your stated claims*, not every conceivable experiment (ARR heuristic H13).

**Sources:**
- https://neurips.cc/Conferences/2025/ReviewerGuidelines (Quality: claims well supported)
- https://icml.cc/Conferences/2025/ReviewerInstructions (claims supported by clear and convincing evidence)
- https://iclr.cc/Conferences/2026/ReviewerGuide
- https://neurips.cc/public/guides/PaperChecklist (item 1: claims reflect contributions/scope)
- https://manusights.com/blog/pre-submission-review-machine-learning
- https://www.austintripp.ca/blog/2025-06-22-ml-conference-review-guide/
- CVPR deck "Common Mistakes"/"Writing" (unsupported claims; unnecessary adjectives): https://www.cs.ryerson.ca/~wangcs/resources/How-to-get-your-CVPR-paper-rejected.pdf
- http://aclrollingreview.org/reviewform ; http://aclrollingreview.org/reviewerguidelines (Soundness)

---

## 2. Baselines: missing / weak / untuned

**The attack:** Comparison omits a recent/stronger method, or baselines are not tuned fairly — a weak baseline manufactures a fake gain. (Dominant attack in DM/IR.)

**Verify before submission:**
- Include the strongest *current* and *peer-reviewed* baselines on the same task/benchmark; also include strong simple/traditional baselines (kNN, linear/logistic, well-tuned classical methods).
- Give baselines a tuning budget comparable to your method; document each baseline's tuning protocol (search space + final values).
- Position explicitly vs the closest 2–3 works and state the delta.
- DM/IR: comparable tuning budget is mandatory — reproducibility studies show reported gains "vanish in most cases" when baselines are tuned properly; with unsound comparisons *every* model can be reported to beat SOTA.

**Sources:**
- https://manusights.com/blog/pre-submission-review-machine-learning
- CVPR/ICCV/ECCV reviewer guidelines + deck "Compare With State of the Art": https://cvpr.thecvf.com/Conferences/2026/ReviewerGuidelines ; https://iccv.thecvf.com/Conferences/2025/ReviewerGuidelines ; https://eccv.ecva.net/Conferences/2026/ReviewerGuide ; https://www.cs.ryerson.ca/~wangcs/resources/How-to-get-your-CVPR-paper-rejected.pdf
- http://aclrollingreview.org/reviewerguidelines (baseline tuning adequacy)
- KDD Q7 / criteria: https://kdd2026.kdd.org/research-track-call-for-papers/ ; https://kdd2012.sigkdd.org/reviewing_criteria.shtml
- WSDM (public test collections + SOTA baselines): https://wsdm-conference.org/2026/index.php/call-for-papers/ ; https://www.wsdm-conference.org/2024/call-for-papers/
- "Everyone's a Winner!" RecSys 2023: https://web-ainf.aau.at/pub/jannach/slides/RecSys-2023-Everyone.pdf ; https://dl.acm.org/doi/10.1145/3604915.3609488
- SIGIR diffusion-recommender reproducibility: https://dl.acm.org/doi/full/10.1145/3795792 ; https://arxiv.org/pdf/2509.09414

---

## 3. Ablations & isolating the contribution

**The attack:** The paper does not prove which part of the method creates the gain ("ablation gap"); reads as "we tried a bunch of stuff and reported the highest number"; ablation baseline is a *different* model rather than the same model minus the component.

**Verify before submission:**
- One controlled ablation per claimed contribution: add/remove/replace that component only, everything else fixed (same model minus the component, not a different model).
- Provide a component-vs-gain table; show the headline gain disappears when the claimed-key component is removed, tracing the gain to the claimed mechanism.
- CV: judge each claim against its claimed contribution type (ECCV) — each claim needs matching evidence.

**Sources:**
- https://manusights.com/blog/pre-submission-review-machine-learning
- https://www.austintripp.ca/blog/2025-06-22-ml-conference-review-guide/
- CVPR deck "Novelty"/"Experimental Validation": https://www.cs.ryerson.ca/~wangcs/resources/How-to-get-your-CVPR-paper-rejected.pdf ; https://eccv.ecva.net/Conferences/2026/ReviewerContributionTypes ; https://eccv.ecva.net/Conferences/2026/ReviewerGuide
- http://aclrollingreview.org/reviewerguidelines (ablations justifying design choices)
- AblationBench (reviewers systematically demand component-removal ablations, ICLR 2023-2025): https://arxiv.org/abs/2507.08038 ; https://www.clrn.org/what-is-ablation-study/

---

## 4. Arbitrary / unjustified design choices

**The attack:** A loss like `L = L_task + lambda * L_aux` (or any weight / temperature / margin / threshold / schedule constant) set to a fixed value with no derivation, no sweep, no ablation — reads as cherry-picked to the reporting benchmark. A modification claimed to "help" may just shift where the good hyperparameters are (more per-task tuning, not less).

**Verify before submission:**
- For every hard-coded constant/weight/threshold/schedule: provide a derivation OR a sensitivity sweep over a range showing performance is stable (or showing where/why the chosen value is optimal); report neighboring values.
- Show the design choice works under a *single fixed* setting across tasks, not only after per-task retuning; consider reporting a hyperparameter-sensitivity metric (gap between per-env tuning and a single fixed setting; effective HP dimensionality) alongside the headline number.
- Pair with the one-at-a-time ablation (§3) so the gain traces to the claimed mechanism, not the tuned constant.

**Sources:**
- "The Role of Hyperparameters in Predictive Multiplicity" (arXiv 2025): https://arxiv.org/pdf/2503.13506
- "A Method for Evaluating Hyperparameter Sensitivity in RL" (NeurIPS 2024): https://arxiv.org/abs/2412.07165 ; https://arxiv.org/html/2412.07165v1
- AblationBench: https://arxiv.org/abs/2507.08038
- DM/IR parameter-sensitivity (KDD Q7): https://kdd2012.sigkdd.org/reviewing_criteria.shtml

---

## 5. Statistical significance, seeds & variance, underpowered evals

**The attack:** Single-run (or best-of-N) results presented as the method's score; no error bars; claimed gain lies within seed/init noise; small margin over a strong baseline with no variance estimate; test set too small to detect the claimed effect; variance from only one randomness source (init) while ignoring data sampling/order/augmentation/HP choice.

**Verify before submission:**
- Run multiple independent seeds (e.g. 3–5 for headline tables — *number is a community norm; sourced principle is "report variance/significance"*); report mean ± std (or full distribution), not the max; state N and the seeds used; state explicitly whether a number is single-run/max/average.
- Run an appropriate significance test on headline comparisons: paired bootstrap / permutation / t-test; Friedman + Nemenyi (critical-difference diagram) across multiple datasets. IR: prefer randomization/bootstrap/t-test on per-query differences; avoid sign/Wilcoxon as sole test; beware multiple-comparison inflation when reusing a test collection.
- Account for multiple variance sources (init, data sampling, data order, augmentation, HP choice); base "A beats B" on this fuller estimator.
- Do a power analysis tied to expected effect size; report test-set size vs minimum detectable effect; avoid over-claiming on small benchmarks.

**Sources:**
- https://neurips.cc/public/guides/PaperChecklist (item 7: error bars/CIs/significance)
- https://iclr.cc/Conferences/2026/ReviewerGuide ("most results not statistically significant" reject example)
- https://manusights.com/blog/pre-submission-review-machine-learning
- Henderson et al. "Deep RL that Matters" (AAAI 2018): https://arxiv.org/abs/1709.06560
- Reimers & Gurevych "Reporting Score Distributions" (EMNLP 2017): https://arxiv.org/abs/1707.09861
- Demšar "Statistical Comparisons of Classifiers" (JMLR 2006): https://www.jmlr.org/papers/volume7/demsar06a/demsar06a.pdf
- Card et al. "With Little Power Comes Great Responsibility" (EMNLP 2020): https://arxiv.org/abs/2010.06595
- Bouthillier et al. "Accounting for Variance in ML Benchmarks" (MLSys 2021): https://arxiv.org/abs/2103.03098
- Dodge et al. "Show Your Work" (EMNLP 2019): https://aclanthology.org/D19-1224/ ; https://arxiv.org/abs/1909.03004
- "How Many Random Seeds": https://arxiv.org/pdf/1806.08295 ; case for replication: https://www.mariushobbhahn.com/2020-03-22-case_for_rep_ML/ ; LeNet5 8.6–99.0% variance: https://arxiv.org/pdf/2312.06633
- ARR checklist C3 / Card et al.: http://aclrollingreview.org/responsibleNLPresearch/
- IR significance testing: https://dl.acm.org/doi/10.1145/3331184.3331259 ; https://dl.acm.org/doi/10.1145/2484028.2484163
- Counter-nuance (avoid significance theater on tiny eval sets): https://www.argmin.net/p/standard-error-of-what-now

---

## 6. Gains from tuning/compute, not the idea ("bag of tricks")

**The attack:** The method wins because it got more compute / bigger backbone / longer training / higher resolution / heavier augmentation / more HP tuning than the baseline — not because of the novel component. Reported field-wide gains often vanish under fair comparison.

**Verify before submission:**
- Equal HP-search budget and restarts for method and baselines; report the tuning protocol; show the gain survives at matched budget. Report expected-max validation performance vs number of HP trials / compute budget.
- Hold confounds fixed across method and baselines: backbone, input resolution, pretraining data, optimizer, schedule, augmentation, epochs, embedding dimensionality, param/FLOP budget. If any differ, report the matched-budget ("same-backbone") variant too, plus params/FLOPs/throughput.
- Isolate the single proposed change.

**Sources:**
- Lucic et al. "Are GANs Created Equal?" (NeurIPS 2018): https://arxiv.org/abs/1711.10337
- Melis et al. "On the State of the Art of Evaluation in Neural LMs" (ICLR 2018): https://arxiv.org/abs/1707.05589
- Musgrave et al. "A Metric Learning Reality Check" (ECCV 2020): https://arxiv.org/abs/2003.08505 ; https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123700681.pdf
- Ferrari Dacrema et al. "Are We Really Making Much Progress?" (RecSys 2019): https://arxiv.org/abs/1907.06902
- Dodge et al. (compute-budget confounding): https://aclanthology.org/D19-1224/ ; https://arxiv.org/abs/1909.03004
- CV fair-comparison / backbones: https://arxiv.org/abs/2310.19909 ; https://arxiv.org/html/2406.05612v1
- https://manusights.com/blog/pre-submission-review-machine-learning

---

## 7. Data leakage / contamination / eval validity & splits

**The attack:** Test info leaks into training/feature-selection/preprocessing; benchmark (or near-duplicates) appears in LLM pretraining; repeated tuning to the test set or use of a weak/saturated metric; erroneous splits (esp. random split on temporal data); sampled/improper ranking metrics; offline metrics treated as online. When leakage is fixed, complex ML often fails to beat decades-old logistic regression / simple heuristics.

**Verify before submission:**
- Strict train/val/test separation including preprocessing/feature selection done *inside* CV folds; lock the test set; tune only on validation; fill out a leakage checklist / model info sheet; confirm no test data touched during training or model selection.
- LLM claims: n-gram overlap / membership-inference contamination analysis (e.g. 13-gram); report train-test overlap; prefer dynamic / post-cutoff / freshly-constructed eval sets.
- DM/IR: split by time where appropriate (not random); use full (not sampled) ranking metrics or justify sampling; report uncertainty (nested CV + significance tests); check dataset realism; acknowledge the offline-vs-online gap; ensure artifacts match the paper's described method exactly.
- Document data splits/sizes/language/license (ARR B2/B5/B6); report multiple/stronger metrics; show robustness across benchmarks, not one saturated leaderboard.

**Sources:**
- Kapoor & Narayanan "Leakage and the Reproducibility Crisis in ML-based Science" (Patterns 2023): https://arxiv.org/abs/2207.07048 ; https://pmc.ncbi.nlm.nih.gov/articles/PMC10499856/
- LLM contamination surveys: https://arxiv.org/html/2406.04244v1 ; https://arxiv.org/html/2502.14425v2
- Musgrave et al. (test-set feedback, weak metrics): https://arxiv.org/abs/2003.08505
- Ferrari Dacrema et al. (test data in training): https://arxiv.org/abs/1907.06902
- DM/IR offline-eval guidelines: https://arxiv.org/abs/2211.01261 ; https://web-ainf.aau.at/pub/jannach/slides/RecSys-2023-Everyone.pdf
- ARR data documentation / checklist: http://aclrollingreview.org/reviewerguidelines ; http://aclrollingreview.org/responsibleNLPresearch/
- Unfair-comparison/leakage (NeurIPS/ICML/ICLR): https://manusights.com/blog/pre-submission-review-machine-learning

---

## 8. Novelty & related work / closest prior work

**The attack:** "Not novel / incremental — similar architecture and comparable performance to [prior work]"; "this is just X + Y"; missing related work essential to the contribution; not well-placed in the literature.

**Verify before submission:**
- State the precise technical advance and explicitly how it differs from the *single closest* prior method in one sentence; back it with an experiment prior work cannot produce; show what breaks without your specific addition.
- Cite recent/closely-related work — ICLR: even contemporaneous work you're aware of (no obligation for work published <2 months before deadline, but cite it); ACL/ARR: position against all close work published >3 months before the deadline.
- Cite arXiv preprints that inspired you (CV dual-submission policy), even though missing-arXiv-comparison alone cannot reject you.
- Do not equate "simple" with "not novel" — reviewers conflating novelty with complexity is itself a flaggable/invalid attack (see §11).

**Sources:**
- https://icml.cc/Conferences/2025/ReviewerInstructions (omitted essential related works)
- https://iclr.cc/Conferences/2026/ReviewerGuide (well-placed in literature; 2-month rule)
- https://neurips.cc/Conferences/2025/ReviewerGuidelines (originality ≠ new architecture)
- CVPR/ICCV deck + guidelines (novelty MUST be backed by specific references): https://cvpr.thecvf.com/Conferences/2025/ReviewerGuidelines ; https://iccv.thecvf.com/Conferences/2025/ReviewerGuidelines ; https://www.cs.ryerson.ca/~wangcs/resources/How-to-get-your-CVPR-paper-rejected.pdf
- ACL'23 (cite "highly similar work X/Y" to justify a novelty attack): https://2023.aclweb.org/blog/review-acl23/ ; http://aclrollingreview.org/reviewerguidelines
- Michael Black "Novelty in Science" (simple can be important): https://perceiving-systems.blog/en/news/novelty-in-science

---

## 9. Reproducibility (code, hyperparams, seeds, compute)

**The attack:** Code/environment/seeds/compute/data/hyperparameters too thin for another lab to rerun; undisclosed training details; artifacts inconsistent with the paper. (Only ~63.5% of a 255-paper ML sample were reproducible.)

**Verify before submission:**
- Ship exact hyperparameters *and how they were chosen* (search space + best-found values — "critically important for others to build on your work"), random seeds, data splits, compute budget (GPU hours/type/parallelism, memory, execution time), #params, environment/dependency files, and runnable scripts.
- Cite implementations/packages used; ensure the main-claim experiments are reproducible from the package; provide anonymized code/configs where possible.
- Artifacts: documented, complete, runnable from a clean OS, and consistent with the paper's described method (ACM badging for SIGIR/WSDM/KDD).
- If you claim a dataset contribution, commit to release.

**Sources:**
- NeurIPS checklist items 4–6, 8: https://neurips.cc/public/guides/PaperChecklist
- ICML supplementary materials: https://icml.cc/Conferences/2025/ReviewerInstructions
- https://manusights.com/blog/pre-submission-review-machine-learning
- CV reproducibility (code optional-check; dataset-contribution release expectation; ECCV no-release-promise caveat): https://iccv.thecvf.com/Conferences/2025/ReviewerGuidelines ; https://eccv.ecva.net/Conferences/2026/ReviewerGuide ; https://www.mariushobbhahn.com/2020-03-22-case_for_rep_ML/ (63.5%)
- ARR Responsible NLP checklist C1/C2/C4: http://aclrollingreview.org/responsibleNLPresearch/ ; reproducibility score: http://aclrollingreview.org/reviewform
- DM/IR replication (KDD Q5/Q6): https://kdd2012.sigkdd.org/reviewing_criteria.shtml ; WSDM self-contained main paper: https://wsdm-conference.org/2026/index.php/call-for-papers/
- ACM artifact badging: https://sigir.org/general-information/acm-sigir-artifact-badging/ ; https://www.acm.org/publications/policies/artifact-review-and-badging-current

---

## 10. Limitations & ethics sections

**The attack:** Authors hide/omit limitations or robustness-to-assumptions; missing/mis-titled Limitations section (desk-reject-class in NLP); unacknowledged limitations found by reviewers are worse than acknowledged ones; missing ethics/societal-impact discussion where required.

**Verify before submission:**
- Dedicated section literally titled "Limitations" naming strong assumptions and how robust results are to violating them. (NLP: mandatory, unnumbered, does not count toward page limit.)
- Honestly state weaknesses and negative societal impact where relevant; assess both strengths AND weaknesses of the approach.
- Complete the venue checklist truthfully with section references (checking "yes" without justification is a flagged problem); disclose AI-assistant usage.
- Theory papers: state the full set of assumptions of all theoretical results and include complete proofs (NeurIPS item 3).

**Sources:**
- https://neurips.cc/public/guides/PaperChecklist (items 2, 3) ; https://neurips.cc/Conferences/2025/ReviewerGuidelines
- ARR: http://aclrollingreview.org/authorchecklist ; http://aclrollingreview.org/reviewform ; http://aclrollingreview.org/responsibleNLPresearch/
- KDD Q8 (assess strengths and weaknesses): https://kdd2012.sigkdd.org/reviewing_criteria.shtml

---

## 11. Do NOT over-correct (official guidance)

Per official NeurIPS/ICLR/CVPR/ICCV/ECCV/ACL guidance, the following are **explicitly NOT valid sole grounds for rejection** — so the goal is *defensibility, not chasing SOTA*. Cite these to preempt unfair attacks and in rebuttals.

- **Not beating SOTA** on an existing benchmark is not by itself grounds for rejection. (ICLR; CVPR/ICCV/ECCV; ACL heuristic H5 — SOTA "neither necessary nor sufficient".)
- **"Not novel enough / too simple"** is not valid: novelty ≠ new architecture / complexity; "a simple idea can be important"; ACL bans unsupported novelty attacks (reviewer must cite specific highly-similar prior work). (NeurIPS originality note; Michael Black; ACL H1/H7.)
- **Missing comparison/citation to arXiv-only / non-peer-reviewed work** should not be the sole grounds for rejection (CV); no obligation to compare to work <2 months before deadline (ICLR).
- **No limitations / no negative-societal-impact section** is not grounds for reject (CV) — honest limitations should be weighted *positively*.
- **Minor easily-correctable flaws**; use of a withdrawn dataset (scrutiny, not auto-reject); **no promise to release code/data** (ECCV/CV).
- **Negative results / acknowledged limitations / methodological choices differing from reviewer taste** are not penalize-worthy (ACL H6/H15/H16).
- Significance reporting should fit the eval, not be performative — frequentist error bars can be near-meaningless on tiny eval sets ("illusion of rigor").

**Sources:**
- https://iclr.cc/Conferences/2026/ReviewerGuide ("lack of SOTA does not by itself constitute grounds for rejection")
- https://cvpr.thecvf.com/Conferences/2026/ReviewerGuidelines ; https://iccv.thecvf.com/Conferences/2025/ReviewerGuidelines ; https://eccv.ecva.net/Conferences/2026/ReviewerGuide
- https://neurips.cc/Conferences/2025/ReviewerGuidelines (originality note) ; https://neurips.cc/public/guides/PaperChecklist
- https://perceiving-systems.blog/en/news/novelty-in-science
- http://aclrollingreview.org/reviewerguidelines ; https://2023.aclweb.org/blog/review-acl23/ (heuristics H1/H5/H6/H7/H13/H15/H16)
- https://www.argmin.net/p/standard-error-of-what-now

---

## 12. Venue-specific notes

**CV (CVPR / ICCV / ECCV):**
- Rebuttal is **one page, no external links, NO new experiments/contributions** beyond small reviewer-requested clarifications (2018 PAMI-TC motion: no substantial additional experiments). Net effect: **all key experiments must already be IN the submission.** Pre-run the experiments a reviewer is most likely to demand: same-backbone/matched-budget baseline (§6), nearest-recent SOTA (§2), per-contribution ablation (§3), multi-seed variance (§5), one OOD/extra dataset (§1).
- Sources: https://github.com/cvpr-org/author-kit/blob/main/rebuttal.tex ; https://cvpr.thecvf.com/Conferences/2026/AuthorGuidelines ; https://iccv.thecvf.com/Conferences/2025/ReviewerGuidelines

**NLP / ARR (ACL/EMNLP/NAACL):**
- **Soundness and Excitement are scored separately** (plus Overall, since Feb 2025); Soundness is the primary axis (a paper can be sound but low-excitement, or vice versa). Soundness scores must be justified by review text.
- **"Limitations" section is mandatory** (unnumbered, does not count toward page limit). Complete the Responsible NLP checklist truthfully with section refs.
- Desk-reject risks: page limits (8 long/4 short), anonymity violations, leftover revision meta-text, salami-slicing, concurrent submission, incomplete profiles, undisclosed AI use.
- Sources: http://aclrollingreview.org/reviewform ; http://aclrollingreview.org/reviewerguidelines ; http://aclrollingreview.org/authorchecklist ; http://aclrollingreview.org/responsibleNLPresearch/

**DM / IR & WSDM (WSDM/KDD/SIGIR/WWW):**
- **Self-contained main paper**: all reproducibility-critical detail (hyperparameters, setup, parameter-sensitivity) must be in the **main body, not just supplementary** — supplementary is reviewed only at reviewer discretion.
- **Tuned baselines** with comparable budget (dominant DM/IR attack), public test collections + SOTA baselines, parameter-sensitivity analysis (KDD Q7).
- **Temporal splits** where appropriate (not random); no leakage; full (not sampled) ranking metrics; significance testing on per-query differences; **offline ≠ online** — acknowledge the gap.
- ACM artifact badging applies; artifacts must match the paper exactly.
- Sources: https://wsdm-conference.org/2026/index.php/call-for-papers/ ; https://www.wsdm-conference.org/2024/call-for-papers/ ; https://kdd2026.kdd.org/research-track-call-for-papers/ ; https://kdd2012.sigkdd.org/reviewing_criteria.shtml ; https://arxiv.org/abs/2211.01261 ; https://sigir.org/general-information/acm-sigir-artifact-badging/

---

## 13. Cheap pre-submission hygiene checks

(community norm / from CV "Common Mistakes" deck — low-cost, high-frequency dings)
- Typos; misuse of "a"/"the"; inanimate objects with verbs; inconsistent word usage; unnecessary adjectives ("superior!").
- "Laundry list" related work / copying sentences from abstracts; bad references; repeated boring statements.
- Minimal-but-sufficient equations ("no more, no less"); clear, sufficient figures; clear/terse writing.
- Format/process: page limits, anonymity, no leftover meta-text, no salami-slicing, no concurrent submission, informative (non-"TBD") title/abstract.
- Source: https://www.cs.ryerson.ca/~wangcs/resources/How-to-get-your-CVPR-paper-rejected.pdf ; http://aclrollingreview.org/authorchecklist

---

## Process meta-context (why mechanical attacks dominate)
Submission volumes exploded (NeurIPS 2,425→15,671, 2016–2024; ICML 6,538→9,653, 2023→2024), forcing many junior reviewers and noisy decisions — so reviewers lean on checklist-able, easy-to-defend objections (missing baseline, no error bars, no ablation, overclaim). Preempting the mechanical items above has outsized value.
- Source: https://imirzadeh.me/post/ml-conferences/ ; https://arxiv.org/pdf/2011.12919
