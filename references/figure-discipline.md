# Figure & Table Discipline for Top-CS-Venue Papers

Research compiled for the research-orchestrator workflow. External claims are grounded in fetched sources (URL after each). Items marked **[UNSOURCED]** rely on synthesis/convention, not a fetched page.

---

## PART A1 — Page Limits (primary sources) and what counts as a "float"

A "float" = a figure OR a table. At all these venues figures/tables count toward the **main-text page limit** (except ACL/ICLR where it's framed as content pages), so floats compete directly with prose for space — this is the core constraint that bounds reasonable counts.

| Venue | Main-text limit | Figures/tables count? | References/appendix | Source |
|-------|-----------------|------------------------|----------------------|--------|
| NeurIPS 2025 | 9 content pages (10 camera-ready) | Yes — "including all figures and tables" | Refs, checklist, technical appendices do NOT count | https://neurips.cc/Conferences/2025/CallForPapers |
| ICML 2025 | 8 pages main | Yes (in main) | Refs, impact statement, appendices unlimited | https://icml.cc/Conferences/2025/CallForPapers |
| ICLR 2025 | 6–10 pages (11th page = desk reject) | Yes (content) | Refs/appendix separate | https://iclr.cc/Conferences/2025/CallForPapers |
| CVPR 2025 | 8 pages | Yes — "including figures and tables" | Refs-only pages allowed; supplementary separate (no new results) | https://cvpr.thecvf.com/Conferences/2025/AuthorGuidelines |
| ACL 2025 (long) | 8 pages content | Yes (in content) | Refs unlimited; mandatory "Limitations" section doesn't count | https://2025.aclweb.org/calls/main_conference_papers/ |
| KDD / WSDM | ~8–9 pages | Yes | **WSDM 2026 counts appendices inside the page limit** (per codex; verify) | (codex-cited) https://wsdm-conference.org/2026/index.php/call-for-papers/ |

Caveat: codex notes 2026 budgets are similar but appendix-inclusion varies — WSDM counts appendices in-page, and several venues state reviewers are not obligated to read supplementary material. Do not assume "unlimited appendix" universally.

---

## PART A2 — Reasonable number of figures + tables

There is no published per-venue "average float count" dataset that I could source this session (searches for figure/table counts at NeurIPS/CVPR returned no quantitative study) — **[UNSOURCED, convention + area-chair judgment]**.

Working guidance for an 8–9 page main paper (refined with codex):
- **Default target: 5–8 high-value floats total** for empirical ML; full reasonable band **4–8**.
- Better primitive than count (codex): **visual real estate should be ~25–40% of the main paper, rarely >50% unless vision-heavy.** One large multi-panel figure can cost as much as four small floats.

Typical breakdown (the recognized "slots"):
- **1 teaser / Figure-1 conceptual figure** — standard opener, especially in CV (often top of page 1–2). https://www.researchgate.net/figure/The-stages-of-the-computer-vision-pipeline_fig1_305906658
- **1 method / architecture / pipeline figure** — shows the approach.
- **1 main-results table** — headline quantitative comparison vs baselines.
- **1–2 ablation tables** — component/design-choice contributions.
- **1–2 analysis / qualitative figures** — convergence curves, error/failure analysis, qualitative examples.

Per-venue skew (codex):
- CVPR/ICCV vision: 6–9 floats normal; qualitative + method figures expected.
- NeurIPS/ICLR empirical ML: 5–8.
- ICML empirical/theory mix: 4–7 (theory fewer).
- ACL/NLP: 3–6 — tables often matter more than figures.
- KDD/WSDM/recsys/search: 4–7, often table-heavy + one efficiency/diagnostic plot.
- Theory-heavy: 1–4 (theorem map, assumption table, synthetic sanity plot).

Failure modes: too many floats squeeze out prose; too few leave claims visually unsupported.

---

## PART A3 — Figures that "earn their place"

Principle (refined): **each float should have ONE primary takeaway that the prose explicitly uses, and map to a specific claim.** No decorative or redundant plots. If a figure doesn't change a reader's belief about a claim, cut it or move to the appendix.

Codex correction: "every figure must make one *indisputable* point" is **too brittle** — good figures often legitimately show tradeoffs, uncertainty, boundary conditions, or failure modes. Reframe as honest + interpretable + claim-relevant, not indisputable.
- Bad caption: "Our method is best."
- Good caption: "Accuracy improves most in the low-label regime; gains shrink above 10k labels."
- Multi-panel figure OK if panels ladder up to one message; six unrelated observations in one figure is not.

Source principles:
- **Rougier, Droettboom & Bourne, "Ten Simple Rules for Better Figures," PLoS Comput Biol 10(9):e1003833 (2014)**, https://journals.plos.org/ploscompbiol/article?id=10.1371%2Fjournal.pcbi.1003833 — the ten rules: (1) Know Your Audience, (2) Identify Your Message, (3) Adapt the Figure to the Support Medium, (4) Captions Are Not Optional, (5) Do Not Trust the Defaults, (6) Use Color Effectively, (7) Do Not Mislead the Reader, (8) Avoid "Chartjunk", (9) Message Trumps Beauty, (10) Get the Right Tool. Rule 2: a figure expresses an idea/result hard to convey in words; Rule 8: drop visual elements that don't aid understanding.
- **Claus Wilke, "Fundamentals of Data Visualization"**, https://clauswilke.com/dataviz/introduction.html — a viz must accurately convey data (not mislead/distort) and be aesthetically clean; jarring colors/imbalance distract. "Visualizations need to earn their keep — if it doesn't have a job, it doesn't belong." Okabe-Ito is the book's default categorical palette.

Reviewer expectations codex added (treat as a checklist):
1. **Self-contained captions** — dataset/split, metric direction, #seeds/samples, what error bars mean, the takeaway. Reviewers skim figures first.
2. **Show uncertainty when variance matters** — CIs, std/sem across seeds, bootstrap, paired tests, tie markers; don't bold meaningless third-decimal wins.
3. **Axis honesty** — label units, state higher/lower-is-better, avoid truncated y-axes unless justified, consistent scales across panels, mark log scales, avoid dual axes.
4. **Efficiency/compute plots** increasingly expected if practicality is claimed (latency, memory, params, FLOPs, train/inference/token cost, quality-vs-cost Pareto).
5. **Qualitative selection discipline** — state random/representative/cherry-picked/failure; include failures; compare vs strongest baseline.
6. **Table discipline** — group columns by task/dataset, few decimals, bold only meaningful wins, mark ties, define abbreviations, avoid giant tables.
7. **Accessibility** — colorblind-safe palettes, redundant markers/linestyles, readable fonts at 100% zoom, high contrast, vector graphics. ICML/CVPR guidance explicitly mention color-vision/print readability.
8. **Main claims need main-paper evidence** — appendix holds full sweeps/proofs/extra qualitative, not the evidence required to believe the central contribution.
9. **Don't cargo-cult diversity** — repeating one plot type is fine if it's the cleanest comparison across datasets/settings; forced diversity is worse than a coherent figure program.
10. **Reviewer-facing traceability** — each float answers "Which claim does this support?" and "Where is the fuller evidence?" (e.g., "Full per-dataset results in Appendix C").

---

## PART A4 — Figure DIVERSITY (mix of types)

A strong paper mixes figure TYPES rather than repeating one; monotonous repetition (e.g., 6 near-identical bar charts) is a weakness **[UNSOURCED convention, endorsed by codex with the caveat above]**. Expected type mix:
- Schematic / architecture / pipeline diagram
- Quantitative comparison (bar / line)
- Ablation (table or plot)
- Qualitative examples (with failures)
- Training / convergence curves
- Error / failure analysis

These map to the standard CV/ML conventions found in practice (teaser, pipeline, ablation study, qualitative results): https://www.researchgate.net/figure/Visual-results-of-the-ablation-study... and https://www.researchgate.net/figure/The-stages-of-the-computer-vision-pipeline_fig1_305906658

---

## PART A5 — Colorblind-safe palette guidance (sources)

- **Okabe-Ito (a.k.a. Wong palette, Nature Methods 2011)** — gold-standard 8-color categorical palette, distinguishable under all common color-vision deficiencies; default categorical palette in Wilke's book; recommended by Nature. https://conceptviz.app/blog/okabe-ito-palette-hex-codes-complete-reference
- **Viridis** — recommended for continuous/sequential data; perceptually uniform and CVD-safe. https://conceptviz.app/blog/scientific-color-palette-for-research-papers-and-posters
- **ColorBrewer** — "colorblind safe" sets for sequential/diverging (heatmaps, maps). Same source above.
- **Paul Tol palettes** — publication-ready qualitative/diverging. Same source above.
- Rule of thumb: Okabe-Ito for categories, Viridis for continuous, ColorBrewer/Tol for diverging.

---

## PART B — The user's pre-configured color / plotting skill (LOCATED, read-only)

**Exact skill NAME: `paper-plot-from-data`** (the `name:` field in its SKILL.md frontmatter is `plot-from-data`; the on-disk skill directory and the name the orchestrator should reference is **`paper-plot-from-data`**).

- Location: `/Users/shihaochen/.claude/skills/paper-plot-from-data/`
- Source repo: `Trae1ounG/paper-plot-skills` — styles distilled from MemEvolve, SPICE, SDPO, DAPO, DoRA, etc.
- Purpose: generate publication-quality matplotlib figures by selecting a **fixed pre-built style** and substituting user data. All outputs are `dpi=300` PNG.

**Fixed styles it provides (8):**

| Style | Type | Script | Use case |
|-------|------|--------|----------|
| `bar_paired_delta` | bar | `scripts/bar_memevolve.py` | Baseline vs method paired comparison + gain arrows |
| `bar_grouped_hatch` | bar | `scripts/bar_spice.py` | Multi-method ablation, hatched main method, value labels |
| `line_confidence_band` | line | `scripts/line_selfdistill.py` | Training curve with confidence band |
| `line_training_curve` | line | `scripts/line_aime.py` | Vertical breakpoint line + horizontal reference line |
| `line_loss_with_inset` | line | `scripts/line_loss_inset.py` | L-shaped spine + zoom inset |
| `scatter_tsne_cluster` | scatter | `scripts/scatter_tsne.py` | t-SNE clusters + annotation boxes |
| `scatter_broken_axis` | scatter | `scripts/scatter_break.py` | Broken X-axis, multi-marker series |
| `radar_dual_series` | radar | `scripts/radar_dora.py` | Dual-method multi-dimension, regular-octagon grid |

**Where style specs / scripts / examples live:**
- Style specs (exact rcParams, colors, font sizes, spines, tick directions): `/Users/shihaochen/.claude/skills/paper-plot-from-data/references/<style_name>.md`
- Scripts (data region clearly marked at top of each file): `/Users/shihaochen/.claude/skills/paper-plot-from-data/scripts/`
- Gallery: `originals/` (paper originals) + `repro/` (reproductions); `GALLERY.md`

**HOW a subagent must invoke it (to keep style/color FIXED and consistent across all figures):**
1. Confirm chart type + data.
2. Pick the matching style (infer from data shape if unspecified).
3. Read `references/<style_name>.md` for exact parameters BEFORE generating.
4. Copy the matching `scripts/<script>.py`, replace ONLY the marked data region (keep array dims/types; if category count changes, sync color list + width calc; edit axis/legend label strings).
5. Run `python3 scripts/<script>.py`.
6. Inspect output; minor tweak of colors/labels/font sizes only if needed.

This is the skill the research workflow should use for ALL figures so palette/style stay consistent. (Companion skill `paper-plot-from-image` reproduces a figure from a screenshot — not the data-driven generator.)

---

## PART C — Codex confirmation (gpt-5.5, xhigh reasoning)

Requested model `gpt-5.5-codex` was REJECTED ("not supported when using Codex with a ChatGPT account"); used **gpt-5.5** with `model_reasoning_effort=xhigh`, sandbox=read-only, approval-policy=never. Full reply saved at `/tmp/ro-research/07b-codex-figures.md`.

Codex's main agreements: directionally correct; 5–8 floats is a sound empirical-ML default; figure-to-claim mapping and cutting decorative plots are right.

Codex's main DISAGREEMENTS / corrections:
1. "One **indisputable** point per figure" is too strong → use "one **primary takeaway** the prose explicitly uses, with evidence and uncertainty visible."
2. Float count is the wrong primitive → measure **visual real estate (~25–40% of paper)**.
3. "Unlimited appendix" is not universal (WSDM 2026 counts appendices in-page; reviewers not obliged to read supplementary).
4. Per-venue ranges should be wider/segmented (see table in A2).
5. Don't cargo-cult diversity — repetition is fine when it's the cleanest comparison.

Codex's top ADDITIONS (already folded into A3 checklist): self-contained captions; show uncertainty/significance; axis honesty; efficiency/compute Pareto plots; qualitative selection discipline incl. failures; table discipline; accessibility/colorblind-safe; main claims need main-paper evidence; reviewer-facing traceability.

---

## PART D — Table LaTeX Design Standards

Sourced from: ICLR 2026 paper "Ensembling Pruned Attention Heads For Uncertainty-Aware Efficient Transformers" (U2IS / ENSTA / Institut Polytechnique de Paris), arXiv:2510.18358 [verified: LaTeX source downloaded directly from arXiv e-print]. Tables in this paper passed top-venue review and represent the canonical NeurIPS/ICLR table template.

### Required packages

```latex
\usepackage{booktabs}        % \toprule \midrule \bottomrule \cmidrule(lr){}
\usepackage{array}           % column spec extensions
\usepackage{multirow}        % \multirow[c]{n}{*}{...}
\usepackage[normalem]{ulem}  % \underline (second-best; normalem = don't break \emph)
\usepackage{colortbl}        % \rowcolor
\definecolor{lightgreen}{rgb}{0.76, 0.88, 0.76}   % proposed-method row highlight
```

### Structural template (main-results table)

```latex
\begin{table}[!t]
\centering
\caption{\textbf{Short imperative title.} One sentence: dataset/split, metric direction.
Best in \textbf{bold}, second-best \underline{underlined}.}
\setlength{\tabcolsep}{3pt}
\renewcommand{\arraystretch}{1.15}
\resizebox{\linewidth}{!}{%
\begin{tabular}{l c c c @{\hspace{4pt}\vrule\hspace{2pt}\vrule\hspace{4pt}} c c c}
\toprule
% Three-tier header: dataset (large scshape) → metric group → individual metric ↑/↓
\multirow[c]{3}{*}{\large\textbf{Method}} &
\multicolumn{3}{c}{\textbf{\large\scshape Dataset A}} &
\multicolumn{3}{c}{\textbf{\large\scshape Dataset B}} \\
\cmidrule(lr){2-4}\cmidrule(lr){5-7}
\multicolumn{1}{c}{Acc} & \multicolumn{2}{c}{OOD Avg} &  ...  \\
\cmidrule(lr){3-4}\cmidrule(lr){...}
 & $\uparrow$ & $\uparrow$ & $\downarrow$ & $\uparrow$ & $\uparrow$ & $\downarrow$ \\
\midrule
Baseline          & 84.1 & 45.2 & 12.3 & 82.0 & 44.1 & 13.5 \\
\rowcolor{lightgreen} \textsc{Our Method} & \textbf{86.3} & \underline{47.6} & \textbf{10.1} & \textbf{84.5} & \textbf{46.8} & \underline{11.2} \\
\bottomrule
\end{tabular}}
\end{table}
```

### Convention table

| Convention | Rule |
|---|---|
| Caption position | **Above** the table — always |
| Caption structure | `\textbf{Short title.}` then dataset/split/metric-direction. End with "Best in bold, second-best underlined." |
| Proposed method row | `\rowcolor{lightgreen}` (rgb 0.76/0.88/0.76) |
| Method names | `\textsc{Small Caps}` throughout |
| Best value | `\textbf{bold}` |
| Second-best value | `\underline{underlined}` |
| Horizontal rules | `\toprule` / `\midrule` / `\bottomrule` (booktabs); `\cmidrule(lr){i-j}` for sub-headers |
| Dataset column separator | `@{\hspace{4pt}\vrule\hspace{2pt}\vrule\hspace{4pt}}` (double vertical rule) |
| Auto-scaling | `\resizebox{\linewidth}{!}{}` wrapper always |
| Spacing | `\tabcolsep=3pt`, `\arraystretch=1.15` |
| Metric direction | `$\uparrow$` / `$\downarrow$` in each column header |
| Significance / p-values | Not in main tables — move to appendix or omit |
| ±std (appendix) | `\newcommand{\res}[2]{#1{\scriptsize\textcolor{blue!60}{$\pm$#2}}}` |
| Inline delta (appendix) | `(\textcolor{green}{+0.34})` for improvement over baseline |

### Why these patterns satisfy reviewers

- `\cmidrule(lr){}` trimmed rules → prevents "table looks cluttered" critique
- `\rowcolor` → proposed method scannable at a glance without relying on text weight alone
- Three-tier headers with `\scshape` dataset names → multi-dataset comparison legible in one look
- Captions **above** (not below) → matches ICLR/NeurIPS reviewer skim direction
- `\resizebox` → prevents overfull hbox across varying column counts
- `\arraystretch=1.15` → enough breathing room without wasting vertical space

---

## PART E — Plotting frameworks toolkit (`~/paper-figure-code/`)

The user maintains a curated plotting toolkit at `~/paper-figure-code/`. These
supplement (not replace) the `paper-plot-from-data` style scripts.

| Need | Tool | Path |
|---|---|---|
| Venue-exact figure size (pt-level): NeurIPS/ICML/CVPR/ICLR/ECCV/AAAI | `tueplots` | `~/paper-figure-code/frameworks/tueplots/` |
| One-line journal style: Nature / IEEE / Science | `SciencePlots` | `~/paper-figure-code/frameworks/SciencePlots/` |
| SVG/PDF output for Illustrator/Inkscape editing | `LovelyPlots` | `~/paper-figure-code/frameworks/LovelyPlots/` |
| Nature Reviews style + `make_nature_figure()` | `sciplotlib` | `~/paper-figure-code/frameworks/sciplotlib/` |
| Size constants only (NeurIPS/ICML/ICLR/Informatics) | `mpl_sizes` | `~/paper-figure-code/frameworks/mpl_sizes/` |
| Real ML training curve with line style / alpha tutorial | tutorial | `~/paper-figure-code/tutorials/deep_learning_plotting_example/` |
| Annotated technique tutorial (boxplot, fill_between, spines) | tutorial | `~/paper-figure-code/tutorials/matplotlib_for_papers/` |
| Real multi-method bar chart (GPT-4-LLM instruction tuning) | paper script | `~/paper-figure-code/papers/GPT-4-LLM/plots/` |
| Scaling-law log-log iso-FLOP curves | paper script | `~/paper-figure-code/papers/Minchilla/plot_isoflops.py` |

**Recommended layering order:**
1. Set figure SIZE with `tueplots`: `plt.rcParams.update(bundles.neurips2023())` (or iclr/icml/cvpr/etc.)
2. Apply color/style with `paper-plot-from-data` script (keep palette consistent)
3. Export at `dpi=300` PNG (or SVG via `LovelyPlots` if post-processing needed)

**Never** set raw `figsize` manually — always derive from `tueplots` so the figure
renders at the correct column width in the LaTeX template.

**Journal style shortcut:**
```python
import scienceplots
plt.style.use(['science', 'nature'])          # Nature/Science journal
plt.style.use(['science', 'ieee'])            # IEEE single-column
plt.style.use(['science', 'bright'])          # colorblind-safe, any venue
plt.style.use(['science', 'no-latex'])        # when LaTeX not installed
```
