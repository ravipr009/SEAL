# SEAL → TRL Revision Plan

Working copy: `TRL.tex`. Build: `latexmk -pdf TRL.tex`.

## Core positioning (the spine)

> All prior mode-based methods and SEAL solve the **same** problem: localize the
> continuous state onto a **known task automaton** (modes + transitions given by
> design). They differ only in **how the localization function `L: state -> mode`
> is obtained.** SEAL learns `L` from a few *safe, segmented demonstrations* with
> uncertainty-targeted active queries — no resets, no perturbations, no failure
> data, no oracle.

| Method | Automaton (modes+transitions) | How `L` is obtained | Cost / safety |
|---|---|---|---|
| TLI   | known (LTL)                   | **hand-designed** sensor model | expert design effort |
| GLiDE | known (LLM feasibility matrix)| **learned** via perturb + oracle + reset | expensive, unsafe, needs resets |
| SEAL  | known (nominal sequence)      | **learned** from few segmented demos + active queries | cheap, safe, no resets/failure data |

**Retract:** "SEAL bypasses the automaton / any-to-any / no transition structure."
We KEEP the designed automaton; we learn its grounding.

**"Memoryless" reinterpreted:** property of the *localizer* (estimates current mode
from instantaneous state, not history) — NOT a claim that there is no automaton.

## Guarantee story (invariance & reachability)

- **Reachability -> by construction.** Primitives are dynamical systems with
  attractors placed in the next mode's region (same as TLI). Make explicit.
- **Invariance, geometric part -> ensured** by the DS flow.
- **Invariance, physical/contact part (e.g. rice on spoon) -> NOT ensured**; its
  breakage IS the failure signal, and the **learned GPC is the invariance monitor**
  that detects it (replaces TLI's hand-built sensor model).
- **Level 1 (verify):** measure invariance-violation & reachability-success rates
  from ordinary rollouts in feature space. GLiDE-style certificate WITHOUT
  perturb/reset/oracle.
- **Level 2 (enforce):** add invariance/reachability violation as a second
  acquisition signal so active queries refine guards toward true closed-loop dynamics.
- **Level 3 (future work):** co-train policies -> *guaranteed* invariance.

## Revised contributions

1. Localization-function learning on a known automaton (LLM+MI feature pipeline;
   MI-sufficiency = observability certificate).
2. Uncertainty-targeted guard refinement (safe, reset-free, no failure data);
   Level 2 adds invariance/reachability-driven queries.
3. Confidence-gated switching: decouples latency from false-positive rate
   (unavailable to automata / TLI / GLiDE). **Foreground this.**
4. Empirical invariance/reachability certification (Level 1).

## Section-by-section checklist

- [ ] **Intro + Fig 2**: reframe to known-automaton + learned `L`; retract any-to-any. Fix stray `?` in abstract.
- [ ] **Related Work**: recharacterize TLI (hand-designed L, learns policies, invariance+reachability theorem) & GLiDE (LLM feasibility matrix, learns L via perturb+oracle+reset, ALSO uses LLM prior). Cite TLI at invariance/reachability.
- [ ] **Preliminaries**: state known automaton + Assumption 1 (instantaneous observability, certified by MI ~ H(Y)).
- [ ] **Feature engineering**: frame MI check as observability certificate; define accept threshold epsilon; add bootstrap CIs.
- [ ] **Localization/GPC**: reframe as mode estimation; guards = decision boundaries.
- [ ] **NEW subsection — Reachability & Invariance**: reachability by DS construction; invariance split; GPC as learned monitor; resolve "memoryless" precisely; name N_thresh as dwell-time.
- [ ] **Active learning**: specify W, 10% cutoff, feature normalization before FPS; add Level 2 violation term.
- [ ] **Experiments**: report Level 1 violation/reachability rates; fix efficiency currency (human-time vs sim-time); state why GLiDE can't run on scooping (no reset); reconcile automaton "near-ideal" vs "noise-brittle"; benchmark 30 Hz.
- [ ] **User study**: own scene-setup confound; promote MMD + mIoU; state ground-truth mIoU definition; effect sizes + Wilcoxon.
- [ ] **Conclusion/Limits**: Level 3 future work; runtime-uncertainty re-query.
- [ ] **Appendix**: reframe h_carton as intentional demo; add calibration diagram for variational GPC.
- [ ] **Minors**: "he minimal"->"the minimal"; M notation collision (rename GPC to g); cite TLI.

## Figure audit

### Keep as-is (support the new story)
- `simple.png` (Fig 1, intro): active querying + robust execution. Good.
- `framework.png` (Fig 3, method): active-learning loop + LLM+MI. Good.
- `2d_nav_figure.png`, `comparison_GLiDE.png`, `compilation.png`,
  `robosuite_wiping.png`, `stacking_pnp.png`, `restocking_banner.png`,
  `pref.png`, `miou.png`, `ablation.png`: results/setup, unaffected.

### REVISE (currently contradict the new positioning)
- **`seal_ltl_comaprison.png` (Fig 2) — MUST revise.** Currently frames LTL="has
  automaton" vs SEAL="any-to-any classifier that directly predicts transitions"
  (gray dotted arrows + "directly predicts the appropriate mode to transition to").
  This is exactly the claim we are retracting. Replace with a figure where the
  **automaton is identical/shared** and only the `L` box differs
  (hand-designed vs learned).

### NEW figures needed
1. **Positioning figure (replaces Fig 2).** Three columns TLI | GLiDE | SEAL, all
   sharing the SAME automaton a->b->c->d; highlight only the `L` box:
   TLI = hand-designed sensor model (human icon); GLiDE = learned via
   perturb+reset+oracle (warning/reset icon); SEAL = learned from safe segmented
   demos (checkmark). Candidate base: adapt `automata_comparison.png` (unused),
   which already shows automaton + L for two methods.
2. **Guarantee figure (for new Method subsection).** Feature-space regions =
   automaton locations; guards = GPC decision boundaries; DS attractor arrow
   flowing into next region (reachability); a perturbation (rice-off) crossing the
   wrong boundary triggering the monitor (invariance break -> recovery). Panels for
   Level 1 (measure violations) and Level 2 (query at violations).
3. **Calibration diagram** (appendix, answers reviewer on variational GPC
   underconfidence): reliability curve of gated confidences.

### Unused files on disk (candidates / ignore)
- `automata_comparison.png` — reuse as base for new positioning figure.
- `overview.png`, `1.png`, `mi_ablation_new.png` — inspect before reuse.
