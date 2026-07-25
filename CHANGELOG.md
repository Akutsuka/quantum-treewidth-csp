# Changelog

All notable changes to this project are documented here.

## [v0.4] — 2026-07-26

### Crystallized: Self-Review (11 Issues)

Systematic hostile self-review identifying 2 potentially fatal, 3 serious, and 6 moderate/minor issues. All addressed.

**Potentially fatal — fixed:**
- Theorem 3 (QSETH lower bound): Added Remark 3 explicitly noting the proof only bites when pw = Θ(n). Does not rule out hybrid runtimes g(n)·h(pw) where g(n) ≥ 2^{n/2} and h(pw) < (√2)^pw. Proving a lower bound in the pw ≪ n regime remains open.
- Proposition 3 (depth-1 upper bound): Added remark acknowledging star decompositions are already classically easy. Value is tightness of QSETH bound, not practical speedup.

**Serious — fixed:**
- Memoization obstruction (§4.2): Rewrote interpretation to acknowledge resource asymmetry (poly-space quantum vs exp-space classical). Precise content: "amplitude amplification is orthogonal to memoization" — when both sides get exp-space, quantum doesn't help with table construction because it's deterministic, not search.
- Oracle-vs-process definitions (§6.2): Sharpened to address the PPSZ closed-form issue. Distinction is now product structure of individual trials (Schöning: iid) vs sequential dependence (PPSZ: adaptive). Both definitions explicitly marked as informal.
- Theorem 2 → Observation 2: Downgraded from "Theorem" to "Observation" — it's a direct corollary of Markov-Shi (2008), not a new result. Value is the implication in the treewidth-CSP setting.

**Moderate — fixed:**
- Precomputation speedup numbers (Proposition 2): Added caveat that 19×/5× are asymptotic cost-model ratios, not implementable gains. Cites Campbell-Khurana-Montanaro (2019) on practical overhead.
- Novelty claim softened: Introduction now acknowledges Kļevickis et al. and Ambainis et al. as adjacent work rather than claiming the question is entirely unposed.

**Minor — fixed:**
- Validation = cost models: Added note to abstract clarifying code validates cost-model algebra, not quantum mechanics.
- Pathwidth vs treewidth: Added distinction to Preliminaries with pw ≤ w·O(log n) bound, primal/incidence/dual clarification, and note that pathwidth lower bounds are weaker than treewidth lower bounds.
- Tovey's theorem: Added to §9.3 as classical evidence that bounded local structure (few constraints per variable) does not ensure tractability.
- 6 new references: Tovey (1984), Høyer-Mosca-de Wolf (2003), Samer-Szeider (2010), Bennett (1973), de Colnet et al. (2026), Zalka (1999).

## [v0.3] — 2026-07-26

### Revised: Reviewer Feedback

Addressed detailed peer review strengthening mathematical rigor and separating theorems from algorithmic analyses.

**Theorem 3 (QSETH Lower Bound) — strengthened:**
- Specified *primal pathwidth* as the parameterization (the standard in SETH/pw-SETH literature)
- Added self-contained proof of pw(φ) ≤ n−1 with explicit path decomposition construction
- Added Remark 1 clarifying the algorithm must work for all pw values, not just bounded pw
- Cited the Primal Pathwidth SETH (arXiv:2403.07239) for context

**Former Theorem 4(a) → Proposition 3 — oracle justified:**
- Full four-step Grover oracle construction: constraint encoding, reversible implementation (citing Bennett 1973), parallel evaluation on disjoint qubit registers, Grover wrapper
- Explicit gate counts: O(n · poly(w, d)) gates, depth O(poly(w, d) + log n)

**Former Theorem 4(b) → Analysis 1 — downgraded:**
- Renamed from "Theorem" to "Analysis of Recursive Grover" to avoid claiming impossibility for all quantum algorithms
- Added caveat on nested amplitude amplification composition (citing Høyer–Mosca–de Wolf 2003, Childs et al. 2025)
- Added "Important distinction" paragraph: Analysis 1 shows one algorithm fails, not that all algorithms must fail

**Gap Theorem → Gap Conjecture — reframed:**
- Now explicitly a conjecture motivated by three independent pieces of evidence, not a theorem
- Section 8 preamble separates conditional theorem (§8.1) from algorithmic analysis (§8.2) from conjecture (§8.3)

**Priority claim softened:**
- "First structural explanation" → "To our knowledge, this provides a structural explanation — though not a proof of impossibility"

## [v0.2] — 2026-07-26

### Added: QSETH Lower Bound, Depth Analysis, Gap Theorem

Three new results extending the paper from 5 to 7 results (12 sections total).

**Theorem 3 (QSETH Lower Bound):**
- Under QSETH, c ≥ 1/2 in O*(d^{cw}) for pathwidth-parameterized SAT
- No quantum algorithm beats O*((√2)^pw)
- Proof via contradiction with QSETH using pw(φ) ≤ n

**Theorem 4 (Depth-Dependent Crossover):**
- Quantum recursive Grover achieves c = 1/2 at decomposition depth D = 1
- Provably slower than classical memoized DP at depth D ≥ 2
- Sharp crossover at exactly D = 2

**Gap Theorem:**
- c ∈ [1/2, 1] for general decompositions
- c = 1/2 tight at depth 1 (matching QSETH lower bound)
- Gap explained by OR/AND asymmetry in tree-decomposition DP

**New files:**
- `code/qseth_bounds.py` — validation suite for Theorems 3–4 and Gap Theorem
- `results/qseth_gap_analysis.png` — Figure 4: gap visualization + OR/AND asymmetry plot

## [v0.1] — 2026-07-26

### Initial Release

First complete draft of the paper with 5 results and full validation code.

**Results:**
- Proposition 1: Quadratic quantum speedup in the polynomial-space regime (quantum backtracking halves the dominant exponent of unmemoized treewidth DP)
- Theorem 1 (Memoization Obstruction): Classical memoized DP beats quantum backtracking for d ≥ 3, w ≥ 1
- Proposition 2 (Precomputation Tradeoff): Ambainis-style hybrid gives moderate constant-factor speedups over classical exp-space DP
- §6 (Oracle-vs-Process Dichotomy): Schöning quantizes fully (oracle-shaped randomness); PPSZ resists (process-shaped randomness)
- Theorem 2 (Simulability Barrier): Structure-respecting quantum circuits are classically simulable via Markov-Shi

**Files:**
- `paper.md` — full research paper
- `code/treewidth_quantum_sim.py` — validation suite for Propositions 1–2, Theorems 1–2, Schöning/PPSZ analysis
- `results/scaling_comparison.png` — Figure 1: three-regime cost scaling
- `results/schoning_ppsz_asymmetry.png` — Figure 2: k-SAT quantization gap
- `results/precomputation_tradeoff.png` — Figure 3: hybrid classical/quantum tradeoff
- `results/validation_results.json` — full numerical results
- `lit_review_quantum_structure.md` — systematic literature review (July 2026)

**Central finding:** Bounded treewidth hurts quantum speedups more than it helps — the structure enabling classical tractability is precisely the structure neutralizing quantum advantages.
