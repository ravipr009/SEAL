# TODO — Honest Gaps and How to Handle Them

Living list of known weaknesses in the current draft (`TRL.tex`) and concrete,
cheapest-first suggestions for closing each one. Companion to `REVISION_PLAN.md`
(section-by-section checklist) and `MEMORY.md` (project state).

Rule of the repo: never fabricate numbers — every gap below stays open (with its
red `\todo{}` marker where applicable) until real data exists.

---

## A. Missing experimental baselines
(claims made in Related Work / intro that no experiment currently backs)

### A1. Anomaly/OOD detector baseline — HIGH priority
- **Gap:** Related Work block 1 claims GMM–Mahalanobis / HMM detectors "do not
  transfer under spatial perturbation," but no experiment demonstrates this
  against SEAL.
- **How to handle (cheap):** train a GMM on the *same* safe demonstrations SEAL
  already uses (features + time-step, Eiband-style), threshold on Mahalanobis
  distance, run on the existing scooping and/or pick-and-place perturbation
  episodes. Report detection rate + false positives next to SEAL's.
  No new hardware, no new data collection — reuse recorded rollouts if logged.
- **Where it lands:** Scooping section (Q2), one added row/curve + 2–3 sentences.

### A2. Random-query baseline for active learning — HIGH priority, cheapest
- **Gap:** 2D nav shows post-query > pre-query, which conflates "more data helps"
  with "uncertainty-TARGETED selection helps." Standard active-learning reviewer
  objection.
- **How to handle:** rerun the 2D nav pipeline once with the 6 query points drawn
  uniformly at random from the demo bounding box (same budget, same retraining).
  Report mIoU: SEAL-active vs SEAL-random vs pre-query. Pure software, ~hours.
- **Where it lands:** Table `tab:miou_results` gets one extra row; one sentence
  in Sec. 2D nav.

### A3. Sequence-aware (HMM) localizer baseline — MEDIUM priority
- **Gap:** the memoryless-is-better argument (Method, Confidence-Gated
  Supervision) is purely structural; no experiment shows a history-based
  localizer masking anomalous transitions.
- **How to handle:** train an HMM (or simple windowed/filtered classifier) on the
  same segmented demos; at test time, inject the standard perturbation episodes
  and measure detection latency / miss rate of the perturbation-induced mode
  change vs the memoryless GPC. Even a 2D-nav-only version makes the point.
- **Where it lands:** small ablation table or paragraph; also retro-justifies the
  intro's one-sentence memoryless claim and Related Work's HMM critique.

---

## B. Numbers still owed to the draft (red \todo markers in the PDF)
- MI acceptance threshold ε actually used in the sufficiency check.
- GPC number of inducing points M.
- Level-1 empirical certificate: invariance-violation and reachability-success
  rates on scooping, pick-place, 2D nav (contribution 4 promises this).
- Calibration/reliability diagram for the variational GPC + the deployed τ.
- Product-kernel vs independent-GPs comparison on Stack task.

**Handle:** all from existing logs/models if saved; otherwise one evaluation
pass per task. None require new demonstrations.

## C. Analyses promised or implied
- **30 Hz claim** (abstract + Related Work): measure GPC inference latency vs
  dataset size / inducing points on the deployment machine. If it misses 30 Hz
  at realistic sizes, soften the claim instead.
- **Stage-2 pruning vs failure-relevant features:** verify whether MI minimization
  could prune an LLM-nominated failure-relevant feature that is near-constant in
  success-only demos (see Remark on feature/physical semantics in Sec III-B); if
  yes, implement an exemption and state it in Sec IV-B.
- **FPS normalization check:** verify in code that continuous features are
  standardized before furthest-point sampling (Eq. sampling). Possible real bug.
- **Bootstrap CIs on k-NN MI estimates** (N=2–7 demos is small).
- **Acquisition sensitivity:** top-decile cutoff and centring-weight W width.
- **User study stats:** add Wilcoxon signed-rank + effect size next to t-test
  (N=10).
- **Level-2 decision:** run the invariance/reachability-augmented acquisition
  experiment or explicitly demote it to future work. Contribution 2 wording is
  already hedged ("extends naturally"), but Sec. Method still promises "We
  report its effect in Sec. 2D nav" — that promise must be kept or cut.

## D. Figures
- Replace the 9 placeholder crops in `Figures/intro_frames/` with high-res
  originals (same filenames; then delete the "Frames are placeholders" caption
  sentence in Fig. 2).
- Fig. 1 TLI panel uses illustrative thresholds (θ>20°, F<0.5 N) matching the
  intro's illustrative example — decide: keep, or switch to generic c1/c2.
- 2D nav figure: restyle to Fig. 1's visual language (see TODO comment block at
  top of Sec. 2D nav in TRL.tex for the 4-step recipe + palette RGBs).
- New "guarantee figure" for Problem Formulation (regions = locations, guards =
  GPC boundaries, DS attractor arrow, rice-off crossing the wrong boundary).
- Calibration diagram (pairs with the τ number in B).

## E. Writing consistency passes
- "Automaton brittle to noise" (Related Work / scooping) vs "automaton is the
  near-ideal classifier" (Scooping) — re-read the softened version once end-to-end.
- Verify Fig. 2 test-panel text ("bottle knocked over → grasp from side
  engaged") matches what the real test episode showed.
- Rebuttal letter to the three mock reviews — never drafted.
- Sync `REVISION_PLAN.md` checkboxes with reality (still all unchecked).
