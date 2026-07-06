# SEAL Project Memory

This file is the single place to catch up on the paper revision — what we decided,
why, what's done, and what's left. Companion file: `REVISION_PLAN.md` (the detailed
section-by-section checklist and figure audit).

---

## 1. The core positioning (the spine of the whole revision)

The paper was revised after three mock expert reviews (a formal-control/MIT-style
reviewer, the GLiDE+TLI author, and a hybrid-control theorist). All three converged
on one structural issue, which became the fix:

> All prior mode-based methods and SEAL solve the **same** problem: localize the
> continuous state onto a **known task automaton** (modes + transitions given by
> design). They differ only in **how the localization function `L: state -> mode`
> is obtained.**

| Method | Automaton (modes+transitions) | How `L` is obtained | Cost / safety |
|---|---|---|---|
| **TLI**   | known (LTL)                    | **hand-designed** sensor model | expert design effort |
| **GLiDE** | known (LLM feasibility matrix) | **learned** via perturb + reset + outcome oracle | expensive, unsafe, needs resets |
| **SEAL**  | known (nominal sequence)       | **learned** from a few safe segmented demos + active queries | cheap, safe, no resets/failure data |

**What we retracted:** the original paper claimed "SEAL completely bypasses the
automaton / any-to-any transitions / no transition structure." This directly
contradicted Algorithm 1, which uses a hand-given nominal sequence `S`. We now
**keep** the designed automaton; SEAL's contribution is learning its *grounding*
(the localization function `L`), not discarding the automaton.

**"Memoryless" reinterpreted:** it's a property of the *localizer* — it estimates
the current mode from the instantaneous state rather than history — **not** a claim
that there is no automaton or no transition structure.

### The guarantee story (invariance & reachability)

- **Reachability → by construction.** The primitives are goal-directed dynamical
  systems with attractors placed in the next mode's region — the same free lunch
  TLI gets from its DS policies.
- **Invariance splits into two halves:**
  - *Geometric* part (e.g., staying between bowl and plate) → ensured by the DS flow.
  - *Physical/contact* part (e.g., "rice stays on the spoon") → **not** ensured by
    the primitive — its breakage **is** the failure signal, and the **learned GPC
    classifier is the invariance monitor** that detects it, replacing TLI's
    hand-built sensor model.
- **Level 1 (verify):** measure invariance-violation and reachability-success rates
  from ordinary primitive rollouts, in feature space. A GLiDE-style empirical
  certificate, but obtained *without* perturb/reset/oracle.
- **Level 2 (enforce):** add invariance/reachability violations as a second
  acquisition-function term, so active queries pull learned guards toward the true
  closed-loop dynamics.
- **Level 3 (future work, not attempted this cycle):** co-train the primitives with
  the classifier to *guarantee* (not just verify) invariance.

### Title

Chosen title: **"SEAL: Safe, Efficient Active Learning for Task-Level Recovery in
Long-Horizon Manipulation."** SEAL is kept as the name (re-expanded acronym) rather
than renamed, to avoid repo/paper-wide churn. "Mode" was removed from the title
(too much insider jargon for a title) but is still defined plainly at first use in
the abstract.

---

## 2. What's done (committed + pushed to `origin/main`)

- Reframed intro + Fig. 2 to "shared automaton, only `L` differs."
- **Full rewrite of `TRL.tex`** in TLI/GLiDE-caliber structure:
  - New abstract.
  - New **Problem Formulation** section: hybrid automaton, guard definitions, formal
    Definitions of mode invariance & goal reachability, Assumption 1 (instantaneous
    observability, certified by MI ≈ H(Y)), Assumption 2 (primitive competence), and
    a Proposition tying localization to closed-loop sequencing.
  - **Related Work** rewritten with honest TLI/GLiDE positioning (see table above).
  - **Method** reorganized: feature engineering framed as an observability
    certificate; GPC localizer + product kernel explained; active learning with the
    optional invariance/reachability-driven guard refinement (Level 2); confidence-
    gating with `N_thresh` named explicitly as a dwell-time/debounce condition.
  - **Experiments** reframed around four research questions (Q1–Q4); added the
    efficiency-currency caveat (human-time vs. sim-time) against GLiDE; states
    explicitly "GLiDE cannot run on scooping because resets are unsafe"; reconciles
    the automaton being both "accurate" and "noise-brittle."
  - **User study**: explicitly owns the scene-setup confound (authors configured
    SEAL query scenes, participants configured their own for Unguided); flags MMD
    (diversity) and mIoU as the confound-robust results; states the ground-truth
    mIoU definition (independent hand-designed automaton — not circular).
  - **Fig. 2** is now an inline TikZ positioning figure (shared automaton, three
    columns TLI | GLiDE | SEAL), replacing the old contradictory image.
- **Fixed pre-existing build bugs** (found while compiling, unrelated to the
  content rewrite): Algorithm 1 was missing its `algorithmic` environment wrapper;
  an unescaped `&` in `bib/example.bib` broke typesetting; renamed the GPC model
  symbol `M` → `g` in Algorithm 1 (it collided with `M` = mode set).
- All existing experimental numbers, citations, and figure files were preserved —
  nothing fabricated.
- Retitled per above.
- Builds clean: `latexmk -pdf TRL.tex` → exit 0, 0 errors, 0 undefined
  citations/references.

---

## 3. What's remaining

### (a) Data you must supply — visible as red `\todo{...}` markers in the PDF
Cannot be filled without real experiments/numbers:
- MI acceptance threshold `ε` actually used in the sufficiency check.
- GPC number of inducing points `M`.
- **Level-1 rates**: invariance-violation & reachability-success measurements on
  scooping, pick-place, and 2D navigation (the empirical certificate).
- Calibration/reliability diagram for the variational GPC posterior + the chosen `τ`.
- Product-kernel vs. independent-GPs comparison on the Stack task.

### (b) Analyses to run (from the reviewer-response plan)
- **30 Hz benchmark** — substantiate or correct the real-time claim in the abstract
  (measure inference latency vs. dataset size / inducing points).
- **FPS normalization check** — confirm continuous features are standardized before
  furthest-point sampling (Eq. for active-learning batch selection) — a possible
  real bug, not yet verified against the actual code.
- **Bootstrap confidence intervals** on the k-NN MI estimates (answers "estimator is
  noisy at N=2–7 demonstrations").
- **Sensitivity ablation** on the acquisition function's top-decile cutoff and the
  centring-weight `W`'s width.
- **Statistical robustness** for the user study — add Wilcoxon signed-rank +
  effect sizes alongside the existing t-test (N=10 is small).

### (c) Figures
- **Polish** the positioning TikZ (Fig. 2): title text slightly overlaps the box
  edge; the red recovery arrow visually passes under mode `b`.
- **New guarantee figure** for the Problem Formulation section: feature-space
  regions = automaton locations, guards = GPC decision boundaries, a DS attractor
  arrow into the next region (reachability), and a rice-off perturbation crossing
  the *wrong* boundary to show the invariance monitor firing. No figure currently
  exists for this, and it's the core formal-story visual.
  - Draft TikZ location once made: alongside `Figures/positioning_figure.tex`.
- Calibration diagram (ties to 3a).
- Photo-based figures need the user's actual robot images — Claude can only
  produce the schematic/vector (TikZ) parts.

### (d) Writing/decisions still open
- Decide whether to actually run the **Level 2** experiment (invariance/
  reachability-driven guard refinement) this cycle, or mark it as preliminary/
  future work only — this was flagged as a decision point, not yet resolved.
- Final read-through to make sure the "automaton is not robust to noise" (Related
  Work) vs. "automaton is the near-ideal classifier" (Scooping section) framing
  reads as consistent (a softened version was written but not independently re-read).
- **Rebuttal letter** — the point-by-point response to the three mock reviews was
  planned in conversation but never drafted as a standalone document.

### (e) Housekeeping
- Tick the completed checkboxes in `REVISION_PLAN.md` (it currently still shows
  everything as unchecked `[ ]` even though most items are done — needs a sync pass).

---

## 4. Repo / build reference

- **Location:** `/Users/ravi-imac/Library/CloudStorage/OneDrive-IndianInstituteofScience/Overleaf/SEAL`
- **Git remote:** `git@github.com:ravipr009/SEAL.git` (private repo), branch `main`.
  SSH auth already works from this machine.
- **Files:** `paper.tex` = original (untouched); `TRL.tex` = the revision working
  copy (all edits happen here). `bib/example.bib` is the bibliography.
- **Bibliography gotcha (already fixed, keep in mind for any new `.tex` copies):**
  `\bibliographystyle{...}` and `\bibliography{...}` must use the path **without**
  the file extension (no `.bst`/`.bib` suffix) — adding the extension breaks bibtex
  silently (this was the original "citations show as `?`" bug).
- **Build command — always use latexmk, never bare pdflatex:**
  ```
  cd "/Users/ravi-imac/Library/CloudStorage/OneDrive-IndianInstituteofScience/Overleaf/SEAL"
  latexmk -pdf TRL.tex
  ```
  Running `pdflatex` alone skips the bibtex step and citations will render as `?`.
  If a rebuild looks stale (e.g., an old title still showing), force a clean rebuild:
  `latexmk -C TRL.tex` then rebuild — OneDrive sync + latexmk incremental state can
  occasionally desync.
- Generated PDFs (`TRL.pdf`, `Figures/positioning_figure.pdf`) are gitignored;
  regenerate locally with the command above.

## 5. Working preferences (how to collaborate on this repo)

- **No `Co-Authored-By: Claude` trailer** on commit messages in this repo — the
  user explicitly rejected a commit specifically to remove it. Keep commit messages
  plain.
- Never fabricate experimental numbers — use visible `\todo{...}` markers instead
  and wait for real data.
- Preserve all existing `\cite{}` keys, `\ref{}` labels, and figure files unless a
  change is explicitly agreed.
