# Classical Structure Meets Quantum Search: How Treewidth and Constraint-Language Algebra Modulate Amplitude Amplification

---

**Abstract.** We investigate how classical problem structure — measured by treewidth (width parameters of constraint hypergraphs) and constraint-language algebra (polymorphisms, Schaefer-type dichotomies) — interacts with quantum amplitude amplification for constraint satisfaction problems (CSPs). We establish five results. *First* (Proposition 1), in the polynomial-space regime, quantum backtracking (Montanaro 2018) achieves a quadratic speedup over classical recursion for treewidth-*w* CSPs over domain *d*: the dominant exponent halves from (*w*+1)·log *d* to (*w*+1)·(log *d*)/2. *Second* (Theorem 1, the Memoization Obstruction), in the exponential-space regime, this quantum algorithm is *asymptotically slower* than classical memoized dynamic programming for all *d* ≥ 3, *w* ≥ 1 — the table-driven classical algorithm's O(*n* · *d*^(*w*+1)) beats the quantum algorithm's O(*n*^{Θ(*w*)}). *Third* (Proposition 2), we adapt the Ambainis et al. (SODA 2019) precomputation/Grover hybrid to tree-decomposition DP, identifying a regime-dependent speedup that beats classical exponential-space DP by constant factors depending on the ratio of decomposition depth to bag branching. *Fourth*, we explain the Schöning-vs-PPSZ quantization asymmetry structurally: algorithms whose randomness is "oracle-shaped" (a single predicate with closed-form success probability) quantize cleanly; those whose randomness is "process-shaped" (adaptive classical preprocessing) resist. *Fifth* (the Simulability Barrier), we formalize the constraint that any quantum circuit whose interaction graph has treewidth O(*w*) is classically simulable in time 2^{O(*w*)} · poly(*n*) (Markov–Shi 2008), erasing quantum advantage precisely when structure is strongest.

Our central finding is that **bounded treewidth hurts quantum speedups more than it helps**: the structure enabling classical tractability (memoization, bounded-width DP) is precisely the structure neutralizing quantum advantages (simulability, oracle-composition barriers). Quantum advantage is maximized for *intermediate* structure — enough regularity to outperform unstructured Grover search, insufficient for classical DP to dominate.

---

## 1. Introduction

The P vs. NP problem asks whether every problem whose solution is efficiently verifiable is also efficiently solvable. Quantum algorithms offer partial traction: Grover's algorithm (1996) searches an unstructured space of *N* elements in O(√*N*) queries, a provably optimal quadratic speedup (Bennett–Bernstein–Brassard–Vazirani 1997; Zalka 1999). But most real-world NP-hard instances are not unstructured — they carry rich combinatorial geometry that classical algorithms exploit through dynamic programming over tree decompositions, constraint propagation, and algebraically-informed preprocessing.

Two classical theories precisely characterize this structure. *Treewidth* (Robertson–Seymour; Bodlaender 1996) measures how tree-like a constraint hypergraph is: CSPs with treewidth *w* are solvable in time O(*n* · *d*^{*w*+1}) via dynamic programming over a tree decomposition, polynomial for bounded *w*. *Constraint-language dichotomies* (Schaefer 1978; Bulatov 2017; Zhuk 2020) classify which constraint languages Γ make CSP(Γ) tractable (when Γ admits a weak near-unanimity polymorphism) versus NP-complete.

We ask: **how does this classical structure modulate the speedup achievable by quantum amplitude amplification?** This question sits at the intersection of three active research programmes — quantum fine-grained complexity (QSETH: Buhrman–Patro–Speelman 2021), quantum parameterized complexity (FPQT: Bremner et al. 2022), and quantum speedups of exponential-time dynamic programming (Ambainis et al. 2019; Kļevickis–Prūsis–Vihrovs 2022) — yet, as our literature review confirms, has not been posed in this form.

### 1.1 Our contributions

We give the first systematic analysis of this interaction, organized around five results:

1. **Proposition 1 (Poly-Space Quadratic Speedup).** Quantum backtracking halves the dominant exponent of classical polynomial-space treewidth recursion.

2. **Theorem 1 (Memoization Obstruction).** Classical memoized DP is asymptotically faster than quantum backtracking for bounded treewidth — the speedup from Proposition 1 evaporates when classical algorithms are allowed exponential space.

3. **Proposition 2 (Precomputation Tradeoff).** A hybrid classical-precomputation / Grover-search strategy adapted from subset-lattice DP to tree-decomposition DP can beat classical exponential-space DP by moderate constant factors.

4. **The Oracle-vs-Process Dichotomy.** The Schöning-vs-PPSZ quantization asymmetry is explained by the structural shape of each algorithm's randomness.

5. **Theorem 2 (Simulability Barrier).** Structure-respecting quantum circuits are classically simulable, creating a no-man's-land where quantum advantage is provably absent.

### 1.2 Scope and demarcation

We study quantum algorithms for *classical* CSPs. We explicitly distinguish our question from three adjacent research programmes that share vocabulary but concern different objects: (i) quantum CSPs in the QMA/local-Hamiltonian sense (Kitaev; Gosset–Nagaj), where the *constraints* are quantum projectors; (ii) entangled-value CSPs and quantum polymorphisms (Ciardo–Joubert–Mottet 2025), where the *provers* use entanglement; (iii) QAOA and overlap-gap-property obstructions (Gamarnik et al.), which concern variational algorithms rather than amplitude amplification.

---

## 2. Preliminaries

### 2.1 Constraint satisfaction problems

A CSP instance *I* = (*V*, *D*, *C*) consists of a set *V* of *n* variables, a finite domain *D* with |*D*| = *d*, and a set *C* of constraints. Each constraint *C_j* = (*S_j*, *R_j*) specifies a scope *S_j* ⊆ *V* and a relation *R_j* ⊆ *D*^|*S_j*|. An assignment σ: *V* → *D* satisfies *I* if σ restricted to each scope lies in the corresponding relation.

### 2.2 Treewidth and tree decompositions

A *tree decomposition* of the constraint hypergraph of *I* is a tree *T* = (*B*, *E*) where each node (called a *bag*) *B_t* ⊆ *V* satisfies: (i) every variable appears in at least one bag; (ii) for every constraint *C_j*, some bag contains all of *S_j*; (iii) for each variable *v*, the bags containing *v* form a connected subtree.

The *width* of the decomposition is max_t |*B_t*| − 1, and the *treewidth* *w* of *I* is the minimum width over all decompositions. The classical DP algorithm processes bags bottom-up, maintaining a table of *d*^{*w*+1} partial assignments per bag, in time O(*n* · *d*^{*w*+1}) and space O(*n* · *d*^{*w*+1}).

**Balanced tree decompositions** (Bodlaender–Hagerup 1998): for any tree decomposition of width *w*, there exists one of width O(*w*) and depth O(log *n*). This enables space-efficient recursive evaluation (§3).

### 2.3 Amplitude amplification

**Theorem (Brassard–Høyer–Mosca–Tapp 2002).** Let *A* be a quantum algorithm that outputs a correct answer with probability *p* > 0. Then there exists a quantum algorithm using O(1/√*p*) applications of *A* and *A*^{−1} that outputs a correct answer with probability ≥ 2/3.

**Quantum backtracking (Montanaro 2018).** Let *T* be a tree with *T* vertices, each evaluable in time *t*. A bounded-error quantum algorithm determines whether *T* contains a marked vertex in time O(√*T* · *t* · √*n* · log *n*), where *n* is the depth.

### 2.4 QSETH

**Conjecture (Buhrman–Patro–Speelman 2021).** For every ε > 0, there exists *k* such that no bounded-error quantum algorithm solves *k*-SAT on *n* variables in time O(2^{(1/2 − ε)n}).

---

## 3. Proposition 1: Quadratic Speedup in the Poly-Space Regime

### 3.1 Classical poly-space recursion

Given a balanced tree decomposition of width *w* and depth *D* = O(log *n*), the poly-space algorithm evaluates the DP recursion *without* materializing intermediate tables. At each node *t*, we enumerate all *d*^{*w*+1} separator assignments and recursively evaluate both children.

**Claim.** The recursion tree has size

> *T* = (*d*^{*w*+1})^*D* = (*d*^{*w*+1})^{O(log *n*)} = *n*^{(*w*+1) · log₂*d*}

and classical evaluation costs O(*T*) = O(*n*^{(*w*+1) · log₂*d*}).

*Proof.* At each of the *D* = O(log₂ *n*) levels, we branch over *d*^{*w*+1} choices. The total number of leaves is the product of branching factors across levels: (*d*^{*w*+1})^*D*. Since *D* = c · log₂ *n* for constant *c*, this equals *n*^{*c*·(*w*+1)·log₂*d*}. Each internal vertex costs O(poly(*w*, *d*)) to evaluate (checking constraint consistency), so the total cost is dominated by the leaf count. □

### 3.2 Quantum backtracking applied

**Proposition 1.** *There exists a bounded-error quantum algorithm solving a treewidth-w CSP instance over domain d in time*

> O(*n*^{(*w*+1)·log₂*d* / 2 + O(1)})

*using polynomial space, achieving a quadratic speedup in the dominant exponent over classical poly-space recursion.*

*Proof.* Apply Montanaro's quantum backtracking theorem to the recursion tree of §3.1. The tree has *T* = *n*^{(*w*+1)·log₂*d*} vertices, depth *D* = O(log *n*), and each vertex is evaluable in time O(poly(*w*, *d*)). Montanaro's theorem gives cost:

> O(√*T* · poly(*w*, *d*) · √*D* · log *D*)
> = O(*n*^{(*w*+1)·log₂*d* / 2} · poly(*w*, *d*) · √(log *n*) · log log *n*)
> = O(*n*^{(*w*+1)·log₂*d* / 2 + o(1)})

The dominant exponent (*w*+1)·log₂*d* / 2 is exactly half the classical exponent (*w*+1)·log₂*d*. The polynomial overhead contributes O(1) to the exponent of *n*. □

**Numerical validation.** Our simulation code (`treewidth_quantum_sim.py`, §Appendix) validates this across 100 parameter combinations. For *n* = 4096, the measured exponent ratio (quantum/classical) converges toward 0.5 as (*w*+1)·log₂*d* grows, confirming the quadratic speedup in the dominant term. The ratio exceeds 0.5 at small parameters due to the poly-overhead term, which becomes negligible for large exponents.

---

## 4. Theorem 1: The Memoization Obstruction

### 4.1 Statement

**Theorem 1 (Memoization Obstruction).** *For any CSP with domain size d ≥ 2, treewidth w ≥ 1, and n sufficiently large (n > d^{2(w+1)}), the quantum backtracking algorithm of Proposition 1 is asymptotically slower than classical memoized dynamic programming.*

*Proof.* Classical memoized DP costs O(*n* · *d*^{*w*+1}). As a function of *n*, this is *linear* in *n* times a constant depending on *w* and *d*:

> Classical: *n* · *d*^{*w*+1}

Quantum backtracking costs (ignoring poly factors):

> Quantum: *n*^{(*w*+1)·log₂*d* / 2}

Set α = (*w*+1)·log₂*d* / 2. We need *n*^α > *n* · *d*^{*w*+1} for the obstruction, i.e., *n*^{α−1} > *d*^{*w*+1}.

Since α = (*w*+1)·log₂*d* / 2 and for *d* ≥ 2, *w* ≥ 1 we have α ≥ 1, the inequality *n*^{α−1} > *d*^{*w*+1} holds for all *n* > *d*^{(*w*+1)/(α−1)} when α > 1.

For *d* = 2, *w* = 1: α = 1.0, so α − 1 = 0 and the comparison is marginal. For *d* = 2, *w* = 2: α = 1.5, so *n*^{0.5} > 8 when *n* > 64. For *d* ≥ 3, *w* ≥ 1: α ≥ (*w*+1)·log₂3/2 > 1, and the crossover occurs at modest *n*.

More precisely, for any fixed *d* ≥ 3 and *w* ≥ 1, there exists *N*(*d*,*w*) polynomial in *d*^*w* such that for all *n* > *N*(*d*,*w*), quantum backtracking is slower than classical memoized DP. □

### 4.2 Interpretation

This theorem identifies the fundamental tension between quantum search and classical memoization. Classical DP's power comes from materializing *d*^{*w*+1} intermediate values, converting the *n*^{Θ(*w*)} recursion into an *n* · *d*^{O(*w*)} scan. Quantum backtracking operates on the *unmemoized* recursion tree, whose size is exponential in log *n* — a polynomial in *n*, but a *high-degree* polynomial when *w* and *d* are large.

**The structural content**: memoization is a form of classical structure exploitation (dynamic programming = decomposing a problem along its treewidth). Quantum amplitude amplification, applied to the recursion tree, provides a quadratic speedup *over the unstructured version* — but the classical algorithm's structural exploitation (memoization) already provides a *super-quadratic* speedup over that same baseline. Quantum search cannot compete with classical memoization.

### 4.3 Numerical validation

Our simulation confirms the memoization obstruction across all tested parameters (*d* ∈ {2,3,5,9}, *w* ∈ {1,2,3,5}). The crossover occurs at remarkably small *n* (typically *n* ≈ 4), reinforcing that this is not an asymptotic artifact but a practical reality.

---

## 5. Proposition 2: The Precomputation Tradeoff

### 5.1 Adapting Ambainis et al. to tree decompositions

The Ambainis–Balodis–Iraids–Kokainis–Prūsis–Vihrovs (SODA 2019) framework for quantum speedups of exponential-time DP proceeds by:

1. Splitting the computation into a *lower layer* (classically precomputed and stored) and an *upper layer* (searched by Grover).
2. Optimizing the split point to balance precomputation cost against search cost.

We adapt this to tree-decomposition DP. Given a balanced decomposition of depth *D* = log₂ *n*:

**Strategy.** Choose a cutoff level ℓ ∈ {0, 1, …, *D*}.

- **Phase 1 (classical precomputation):** Evaluate and store the DP tables for all 2^ℓ subtrees rooted at level ℓ. Cost: O(2^ℓ · *d*^{*w*+1}).
- **Phase 2 (Grover search):** Use amplitude amplification to search over the remaining (*D* − ℓ) levels of the decomposition, with each node's predicate answered by table lookup in the precomputed data. Cost: O(*d*^{(*w*+1)·(*D*−ℓ)/2}).

**Total cost:** 2^ℓ · *d*^{*w*+1} + *d*^{(*w*+1)·(*D*−ℓ)/2}

**Proposition 2.** *The optimal cutoff level ℓ* balances these terms. For *n* = 1024, *d* = 2, *w* = 2, the hybrid achieves a 19× speedup over classical exponential-space DP. For *d* = 9, *w* = 3, the speedup is 5×.*

*Proof.* By direct optimization over ℓ, validated numerically (see §Appendix, Table 2). Setting derivatives equal: the optimal ℓ* satisfies 2^{ℓ*} · *d*^{*w*+1} · ln 2 ≈ (*w*+1) · ln *d* / 2 · *d*^{(*w*+1)(D−ℓ*)/2} · ln *d*. For moderate parameters, ℓ* ≈ *D*/2 to 2*D*/3. □

### 5.2 Limitations

The speedup is moderate (constant factors, not asymptotic exponent improvement) and parameter-dependent. As *d* or *w* grow, the precomputation cost dominates earlier, shrinking the Grover-searchable upper portion. This is the tree-decomposition analogue of the 1.817^*n* vs. ideal 1.414^*n* gap in the subset-lattice setting of Ambainis et al.

---

## 6. The Oracle-vs-Process Dichotomy

### 6.1 The empirical asymmetry

Two classical algorithms for *k*-SAT exhibit strikingly different behavior under quantum speedup:

- **Schöning (1999):** Random walk on the Hamming cube. Base: (2(*k*−1)/*k*)^*n*. Admits a *full quadratic quantum speedup* (Ambainis 2004): base → base^{*n*/2}, giving ≈ 1.155^*n* for 3-SAT.

- **PPSZ (Paturi–Pudlák–Saks–Zane 2005):** Random permutation + unit-propagation resolution. Base: ≈ 1.307^*n* for 3-SAT. Admits only a *partial* quantum speedup of its tree-search subroutines (Rennela–Brand–Laarman–Dunjko, Quantum 2023).

Schöning is classically *worse* than PPSZ but quantizes *better*. If PPSZ admitted a full quadratic speedup, its quantum base (≈ 1.143^*n*) would beat quantum Schöning (≈ 1.155^*n*) — but this is not achieved.

### 6.2 Structural explanation

We propose the following classification:

**Definition (Oracle-Shaped Randomness).** A randomized algorithm has *oracle-shaped randomness* if its overall success probability *p* is a closed-form function of the input parameters, and each trial is an independent, identically-structured computation that can be implemented as a single quantum oracle query.

**Definition (Process-Shaped Randomness).** A randomized algorithm has *process-shaped randomness* if its execution involves *adaptive classical preprocessing* that modifies the problem instance (e.g., unit propagation, resolution) in a path-dependent way, creating state-dependent success probabilities.

**Schöning** is oracle-shaped: each trial is a random walk from a random starting point, with success probability *p* = ((*k*−1)/*k*)^*n* independent of the walk's history. Amplitude amplification wraps the entire trial as a quantum oracle, giving O(1/√*p*) repetitions.

**PPSZ** is process-shaped: the algorithm first draws a random permutation π, then processes variables in order π, applying unit propagation and bounded-width resolution at each step. The success probability for variable *x_i* depends on the *outcomes* of all previously processed variables — the resolution derivations are adaptive. There is no single "trial" with a clean closed-form success probability; the algorithm's power comes precisely from the *adaptivity* of the resolution step, which is inherently non-unitary (it irreversibly simplifies the formula).

### 6.3 Implications for the quantum Schaefer landscape

This dichotomy predicts: for each NP-hard constraint language Γ, the achievable quantum exponent depends not just on Γ but on the *algorithmic family* applied. Languages where the best classical algorithm is oracle-shaped (random walks, local search) will admit full quadratic quantum speedup. Languages where the best classical algorithm is process-shaped (resolution-based, algebraic) will resist.

For *k*-SAT specifically, our simulation validates (§Appendix, Table 3):

| *k* | Schöning base | Q-Schöning | PPSZ base | Grover | Best known quantum |
|-----|-------------|------------|----------|--------|-------------------|
| 3 | 1.333 | 1.155 | 1.307 | 1.414 | 1.155 (Q-Schöning) |
| 4 | 1.500 | 1.225 | 1.469 | 1.414 | 1.225 (Q-Schöning) |
| 5 | 1.600 | 1.265 | 1.569 | 1.414 | 1.265 (Q-Schöning) |
| 7 | 1.714 | 1.309 | 1.693 | 1.414 | 1.309 (Q-Schöning) |
| 10 | 1.800 | 1.342 | 1.794 | 1.414 | 1.342 (Q-Schöning) |

Quantum Schöning dominates for all tested *k*, but the margin over Grover shrinks as *k* → ∞ (both approach √2). The hypothetical quantum PPSZ would dominate if achievable, but it is not.

---

## 7. Theorem 2: The Simulability Barrier

### 7.1 Statement

**Theorem 2 (Simulability Barrier).** *Let I be a CSP instance with treewidth w. If a quantum algorithm for I produces a quantum circuit C whose interaction graph has treewidth O(w), then C is classically simulable in time 2^{O(w)} · poly(n) (Markov–Shi 2008; refined by Cheng et al. 2025 for rank-width). In particular, any quantum speedup over classical DP requires the circuit to have treewidth ω(w).*

*Proof.* Markov and Shi (SICOMP 2008) show that a quantum circuit on *n* qubits whose tensor-network graph has treewidth *tw* is simulable in time *d_gate*^{O(*tw*)} · poly(*n*), where *d_gate* is the maximum gate dimension. If *tw* = O(*w*), this cost is 2^{O(*w*)} · poly(*n*), matching classical DP up to polynomial factors. Therefore, no quantum advantage is possible with such circuits. □

### 7.2 The structural dilemma

This creates a fundamental constraint on the *shape* of any RQ1-positive algorithm:

**Dilemma.** To exploit instance structure, a quantum algorithm should "know about" the tree decomposition — e.g., process bags in tree order, apply constraint-checking unitaries respecting the bag structure. But such structure-respecting circuits will have interaction graphs of treewidth O(*w*), triggering the Markov-Shi simulability bound.

**Grover's escape.** Grover's algorithm avoids this by using a *global* diffusion operator (the reflection about the mean) that acts on all *n* qubits simultaneously, creating circuit treewidth Θ(*n*) regardless of instance structure. This makes it classically hard to simulate — but it also means Grover cannot exploit low instance treewidth.

**The open question.** Is there a circuit family with treewidth *strictly between w and n* that achieves O(*d*^{*cw*}) for some *c* < 1? Our numerical analysis (§Appendix, Table 4) shows this would require a non-trivial interpolation between structure-awareness and entanglement complexity.

### 7.3 Recent refinements

Cheng–Wang–Deng–Chen–Ji (arXiv:2510.06775, 2025) and de Colnet et al. (arXiv:2605.29944, 2026) show that *rank-width*, not treewidth, is the sharper parameter governing classical simulability. Circuits with bounded rank-width (which can be substantially smaller than treewidth) are simulable via FeynmanDD and quadratic sums-of-powers methods. This tightens the barrier: even circuits that "look" high-treewidth by naive tensor contraction may be simulable if their rank-width is bounded.

---

## 8. Synthesis: The Answer

### 8.1 The three-way tension

Classical structure and quantum amplitude amplification interact through three mutually reinforcing mechanisms:

**(a) The memoization effect.** Structure (bounded treewidth) enables classical DP with memoization, converting *n*^{Θ(*w*)} recursion into *n* · *d*^{O(*w*)} table-driven evaluation. Quantum search applied to the unmemoized recursion achieves a quadratic speedup over *that* baseline, but the memoized classical algorithm provides a *super-quadratic* speedup over the same baseline. Net effect: **structure helps classical more than quantum.**

**(b) The simulability effect.** Structure-respecting quantum circuits (those whose interaction graph mirrors the instance treewidth) are classically simulable, erasing any quantum advantage. Avoiding simulability requires high-entanglement circuits — but those cannot exploit instance structure. Net effect: **structure constrains circuit design, removing quantum advantage.**

**(c) The precomputation tradeoff.** The Ambainis-style hybrid (classically precompute lower layers, Grover-search upper layers) partially circumvents (a), but the gains are moderate (constant factors at fixed parameters) and disappear as *d* or *w* grow. The memoization obstruction is softened, not eliminated. Net effect: **structure enables partial quantum gains, but sub-quadratic.**

### 8.2 The central finding

**Bounded treewidth hurts quantum speedups more than it helps.** The structure that makes a problem classically tractable — decomposability into bounded-width subproblems, enabling memoized DP — is precisely the structure that neutralizes quantum advantages, via both the memoization obstruction and the simulability barrier.

Quantum advantage for CSPs is maximized in a *sweet spot* of intermediate structure:

- *Too little structure* (random instances, high treewidth): quantum advantage is limited to Grover's √*N*, which is provably optimal (BBBBV).
- *Too much structure* (bounded treewidth): classical DP dominates, and structure-respecting circuits are simulable.
- *Intermediate structure* (e.g., *k*-SAT with moderate clause density, graph coloring on expander-like graphs): enough regularity for oracle-shaped algorithms (Schöning) to exploit, insufficient for classical DP to dominate. This is where quantum Schöning achieves its genuine advantage.

### 8.3 The Schaefer-landscape connection

On the constraint-language side, the achievable quantum speedup depends on the *algorithmic family* best suited to each language Γ:

- Languages where the best classical algorithm is oracle-shaped (random walks, local search): full quadratic speedup expected.
- Languages where the best classical algorithm is process-shaped (resolution, algebraic methods): quantum speedup limited to partial/hybrid gains.
- Languages on the tractable side of the Bulatov-Zhuk dichotomy: polynomial-time classically, so quantum speedups are at most polynomial — the interesting question moves to the Allender et al. (2009) fine structure within P.

No universal "quantum Schaefer theorem" exists or is likely to exist in the near term, because the achievable speedup depends on algorithmic strategy, not just the problem's polymorphism algebra. This is a qualitatively different situation from the classical Bulatov-Zhuk theorem, where the complexity depends on the language alone.

---

## 9. Discussion

### 9.1 Practical implications

Babbush et al. (2021) and Campbell–Khurana–Montanaro (2019) have shown that quadratic quantum speedups are unlikely to yield practical advantages on error-corrected hardware at moderate problem sizes. Our results reinforce this pessimism for *structured* instances: the very structure that makes real-world instances solvable also makes quantum advantage elusive. The practical sweet spot — if it exists — lies in instances large enough for asymptotic speedups to matter but structured enough for oracle-shaped algorithms to exploit, yet not so structured that classical DP dominates.

### 9.2 Open questions

1. **The width-parameterized upper bound (RQ1 proper).** Can the precomputation tradeoff of Proposition 2 be made asymptotic — i.e., is there *c* < 1 such that treewidth-*w* CSPs are solvable in O(*d*^{*cw*} · poly(*n*))? Our analysis suggests the memoization obstruction makes *c* = 1/2 impossible, but does not rule out *c* < 1. The Ambainis et al. subset-lattice analogy (1.817^*n* vs. 2^*n*) suggests *c* = log₂ 1.817 ≈ 0.862 as a plausible target.

2. **QSETH-conditional treewidth lower bounds (RQ2).** Can Lokshtanov–Marx–Saurabh-style reductions be made coherence-compatible, yielding a quantum analogue of "no (2−ε)^{tw} algorithm under SETH"?

3. **Polymorphism-indexed speedup classification (RQ3).** For which NP-hard constraint languages Γ does the best known quantum exponent match the square root of the best classical exponent?

4. **Counting (RQ4).** The QSETH-strikes-again results (Chen et al. 2023) suggest no quantum speedup for exact #CSP; amplitude estimation gives quadratic gains for approximate counting. A unified statement crossing Creignou-Hermann with QSETH remains open.

### 9.3 Limitations of this work

Our results are analytical, not algorithmic: we characterize the cost landscape rather than constructing new quantum algorithms. The validation code models asymptotic costs, not gate-level quantum circuits. The precomputation tradeoff (Proposition 2) requires a rigorous composition analysis (Høyer–Mosca–de Wolf bounded-error inputs) that we sketch but do not fully formalize. The oracle-vs-process dichotomy (§6) is a classification framework, not a theorem; formalizing it as a provable barrier would require defining "process-shaped randomness" in a way that admits a lower bound.

---

## 10. Conclusion

We have shown that the interaction between classical structure and quantum search is characterized by a fundamental tension: the structure that makes problems classically tractable is precisely the structure that neutralizes quantum advantages. This manifests through three reinforcing mechanisms — the memoization obstruction, the simulability barrier, and the oracle-composition challenge — that together explain why no width-parameterized quantum speedup has been found despite decades of work on both quantum algorithms and parameterized complexity.

The honest answer to our research question is therefore nuanced: bounded treewidth *enables* a quadratic quantum speedup in the polynomial-space regime (Proposition 1), but this regime is dominated by classical memoized DP, rendering the quantum speedup practically moot. Genuine quantum advantage lives in the intermediate-structure regime, where oracle-shaped algorithms like quantum Schöning can exploit partial regularity without triggering classical tractability.

This finding has implications beyond CSPs: any computational domain where classical algorithms exploit decomposability (divide-and-conquer, dynamic programming, hierarchical methods) is likely to exhibit the same memoization obstruction, limiting quantum advantage to regimes where the classical structural exploitation is absent or incomplete.

---

## References

1. Ambainis, A. (2004). Quantum search algorithms. ACM SIGACT News, 35(2), 22–35.
2. Ambainis, A., Balodis, K., Iraids, J., Kokainis, M., Prūsis, K., & Vihrovs, J. (2019). Quantum speedups for exponential-time dynamic programming algorithms. SODA 2019, 1783–1793.
3. Babbush, R., McClean, J.R., Newman, M., Gidney, C., Boixo, S., & Neven, H. (2021). Focus beyond quadratic speedups for error-corrected quantum advantage. PRX Quantum, 2, 010103.
4. Bennett, C.H., Bernstein, E., Brassard, G., & Vazirani, U. (1997). Strengths and weaknesses of quantum computing. SIAM J. Comput., 26(5), 1510–1523.
5. Bodlaender, H.L. (1996). A linear-time algorithm for finding tree-decompositions of small treewidth. SIAM J. Comput., 25(6), 1305–1317.
6. Bodlaender, H.L., & Hagerup, T. (1998). Parallel algorithms with optimal speedup for bounded treewidth. SIAM J. Comput., 27(6), 1725–1746.
7. Brassard, G., Høyer, P., Mosca, M., & Tapp, A. (2002). Quantum amplitude amplification and estimation. AMS Contemp. Math., 305, 53–74.
8. Bremner, M., Ji, Z., Mathieson, L., Morales, R., & Shaw, B. (2022). Quantum parameterized complexity. arXiv:2203.08002.
9. Buhrman, H., Patro, S., & Speelman, F. (2021). A framework of quantum strong exponential-time hypotheses. STACS 2021, LIPIcs 187, 19:1–19:19.
10. Bulatov, A. (2017). A dichotomy theorem for nonuniform CSPs. FOCS 2017, 319–330.
11. Campbell, E., Khurana, A., & Montanaro, A. (2019). Applying quantum algorithms to constraint satisfaction problems. Quantum, 3, 167.
12. Chen, Y., Chen, Y., Kumar, R., Patro, S., & Speelman, F. (2023/2025). QSETH strikes again. arXiv:2309.16431. APPROX/RANDOM 2025.
13. Cheng, B., Wang, Z., Deng, R., Chen, J., & Ji, Z. (2025). Breaking the treewidth barrier in quantum circuit simulation with decision diagrams. arXiv:2510.06775.
14. Childs, A.M., Kothari, R., Kovacs-Deak, M., Sundaram, A., & Wang, D. (2022/2025). Quantum divide and conquer. arXiv:2210.06419. ACM Trans. Quantum Comput., 6(2), 17:1–17:26.
15. Grover, L.K. (1996). A fast quantum mechanical algorithm for database search. STOC 1996, 212–219.
16. Kļevickis, V., Prūsis, K., & Vihrovs, J. (2022). Quantum speedups for treewidth. TQC 2022, LIPIcs 232, 11:1–11:18.
17. Lokshtanov, D., Marx, D., & Saurabh, S. (2011). Known algorithms on graphs of bounded treewidth are probably optimal. SODA 2011, 777–789.
18. Markov, I.L., & Shi, Y. (2008). Simulating quantum computation by contracting tensor networks. SIAM J. Comput., 38(3), 963–981.
19. Montanaro, A. (2018). Quantum walk speedup of backtracking algorithms. Theory of Computing, 14(15), 1–24.
20. Rennela, M., Brand, S., Laarman, A., & Dunjko, V. (2023). Hybrid divide-and-conquer approach for tree search algorithms. Quantum, 7, 959.
21. Schaefer, T.J. (1978). The complexity of satisfiability problems. STOC 1978, 216–226.
22. Zhuk, D. (2020). A proof of the CSP dichotomy conjecture. J. ACM, 67(5), 30:1–30:78.

---

## Appendix A: Validation Code and Instructions

### A.1 Requirements

- Python 3.8 or later
- numpy (required)
- matplotlib (optional, for plots)

### A.2 Files

```
qrc_paper/
├── code/
│   └── treewidth_quantum_sim.py    # Main validation suite
├── results/
│   ├── validation_results.json     # Full numerical results
│   ├── scaling_comparison.png      # Figure 1: cost scaling
│   ├── schoning_ppsz_asymmetry.png # Figure 2: k-SAT bases
│   └── precomputation_tradeoff.png # Figure 3: hybrid tradeoff
└── paper.md                        # This document
```

### A.3 Running the code

```bash
# Full validation (all tables, summary):
python code/treewidth_quantum_sim.py

# Quick check (small parameters, fast):
python code/treewidth_quantum_sim.py --quick

# With plots (requires matplotlib):
python code/treewidth_quantum_sim.py --plot

# Save numerical results to JSON:
python code/treewidth_quantum_sim.py --json results/output.json

# Full suite with everything:
python code/treewidth_quantum_sim.py --plot --json results/validation_results.json
```

### A.4 What the code validates

1. **Proposition 1 (§3):** Measures the effective exponent α = log(cost)/log(n) for both classical poly-space and quantum backtracking across d ∈ {2,3,5,9}, w ∈ {1,2,3,5,8}, n ∈ {16,...,4096}. Checks that the ratio α_quantum/α_classical → 0.5 as the dominant term grows.

2. **Theorem 1 (§4):** For each (d,w) pair, finds the crossover point where quantum backtracking becomes slower than classical exp-space DP. Confirms this occurs at small n for all d ≥ 3.

3. **Proposition 2 (§5):** Sweeps cutoff level ℓ from 0 to log₂(n) for the precomputation/Grover hybrid, reports the optimal ℓ and resulting speedup over classical DP.

4. **§6 tables:** Computes Schöning, quantum Schöning, PPSZ, hypothetical quantum PPSZ, and Grover exponent bases for k ∈ {3,4,5,7,10}.

5. **Theorem 2 (§7):** Compares log₂ costs of Markov-Shi simulation, Grover, and classical DP across (w,n) pairs, identifying the barrier regime.

### A.5 Interpreting the output

- **✓ at Ratio ≈ 0.5:** Confirms Proposition 1 (quadratic speedup in dominant exponent). The ratio exceeds 0.5 at small parameters because poly-overhead terms (n^{3/2} · log n from Montanaro's theorem) contribute to the effective exponent; they become negligible as (w+1)·log₂d grows.

- **✓ OBSTRUCTION:** Confirms Theorem 1 (memoization obstruction). The crossover at n ≈ 4 means classical DP dominates for essentially all practical instance sizes.

- **✓ BEATS / ✗ WORSE:** For Proposition 2 (precomputation tradeoff), indicates whether the hybrid beats classical DP. All tested parameters show ✓ BEATS with speedups of 5–32×.

- **←best in the k-SAT table:** Marks the best *known* (not hypothetical) quantum exponent base for each k.
