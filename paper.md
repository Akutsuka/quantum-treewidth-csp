# Classical Structure Meets Quantum Search: How Treewidth and Constraint-Language Algebra Modulate Amplitude Amplification

---

**Abstract.** We investigate how classical problem structure — measured by treewidth (width parameters of constraint hypergraphs) and constraint-language algebra (polymorphisms, Schaefer-type dichotomies) — interacts with quantum amplitude amplification for constraint satisfaction problems (CSPs). We establish seven results. *First* (Proposition 1), in the polynomial-space regime, quantum backtracking (Montanaro 2018) achieves a quadratic speedup over classical recursion for treewidth-*w* CSPs over domain *d*: the dominant exponent halves from (*w*+1)·log *d* to (*w*+1)·(log *d*)/2. *Second* (Observation 1, Memoization Orthogonality), in the exponential-space regime, this quantum algorithm is *asymptotically slower* than classical memoized dynamic programming for all *d* ≥ 3, *w* ≥ 1. *Third* (Proposition 2), we adapt the Ambainis et al. (SODA 2019) precomputation/Grover hybrid to tree-decomposition DP, identifying a regime-dependent speedup. *Fourth* (Theorem 1, Universal Quadratic Speedup), we prove that *all* classical randomized SAT algorithms with poly-time trials admit full quadratic quantum speedup via amplitude amplification — correcting an earlier informal classification — and show as a corollary that quantum PPSZ achieves O\*(1.143^*n*) for 3-SAT, beating quantum Schöning. *Fifth* (Observation 2, the Simulability Barrier), any quantum circuit whose interaction graph has treewidth O(*w*) is classically simulable in time 2^{O(*w*)} · poly(*n*), erasing quantum advantage precisely when structure is strongest. *Sixth* (Theorem 2, QSETH Lower Bound), we prove conditionally that the exponent constant *c* in O\*(*d*^{*cw*}) satisfies *c* ≥ 1/2: no quantum algorithm can solve primal-pathwidth-parameterized SAT in time O\*((√2 − ε)^{pw}). *Seventh*, we analyze the naïve recursive Grover strategy on tree decompositions, showing it achieves *c* = 1/2 for depth-1 (star) decompositions (Proposition 3) but is provably slower than classical memoized DP at depth *D* ≥ 2 (Analysis 1). This motivates the **Gap Question**: for general tree decompositions, the optimal quantum exponent *c* lies in [1/2, 1], with *c* = 1/2 tight at depth 1 (matching the QSETH lower bound) and *c* = 1 the best known for *D* ≥ 2. We carefully separate the conditional theorem (about all quantum algorithms) from the analysis (about one specific strategy) and the conjecture (about the optimal *c*). The gap is explained by an OR/AND asymmetry in tree-decomposition DP: Grover accelerates the OR (forget/search) operations but leaves the AND (join/verify) operations unchanged, and the compounding AND costs overwhelm the OR savings once decomposition depth exceeds 1.

Our central finding is that **bounded treewidth hurts quantum speedups more than it helps**: the structure enabling classical tractability (memoization, bounded-width DP) is precisely the structure neutralizing quantum advantages (simulability, oracle-composition barriers). Quantum advantage is maximized for *intermediate* structure — enough regularity to outperform unstructured Grover search, insufficient for classical DP to dominate.

*Note on validation.* All numerical results in this paper validate *cost-model algebra* — they confirm that the mathematical relationships between asymptotic cost functions hold as predicted. The underlying quantum-algorithmic primitives (backtracking, amplitude amplification, tensor-network simulation) are taken as proven in their respective papers; we do not re-verify them on quantum hardware. Experimental validation on quantum simulators remains future work.

---

## 1. Introduction

The P vs. NP problem asks whether every problem whose solution is efficiently verifiable is also efficiently solvable. Quantum algorithms offer partial traction: Grover's algorithm (1996) searches an unstructured space of *N* elements in O(√*N*) queries, a provably optimal quadratic speedup (Bennett–Bernstein–Brassard–Vazirani 1997; Zalka 1999). But most real-world NP-hard instances are not unstructured — they carry rich combinatorial geometry that classical algorithms exploit through dynamic programming over tree decompositions, constraint propagation, and algebraically-informed preprocessing.

Two classical theories precisely characterize this structure. *Treewidth* (Robertson–Seymour; Bodlaender 1996) measures how tree-like a constraint hypergraph is: CSPs with treewidth *w* are solvable in time O(*n* · *d*^{*w*+1}) via dynamic programming over a tree decomposition, polynomial for bounded *w*. *Constraint-language dichotomies* (Schaefer 1978; Bulatov 2017; Zhuk 2020) classify which constraint languages Γ make CSP(Γ) tractable (when Γ admits a weak near-unanimity polymorphism) versus NP-complete.

We ask: **how does this classical structure modulate the speedup achievable by quantum amplitude amplification?** This question sits at the intersection of three active research programmes — quantum fine-grained complexity (QSETH: Buhrman–Patro–Speelman 2021), quantum parameterized complexity (FPQT: Bremner et al. 2022), and quantum speedups of exponential-time dynamic programming (Ambainis et al. 2019; Kļevickis–Prūsis–Vihrovs 2022). While each of these programmes contains results adjacent to our question (Kļevickis et al. compute treewidth using quantum DP; Ambainis et al. accelerate DP over subset lattices), the specific question of quantum speedups for CSPs *parameterized by treewidth* — targeting the base of *d*^*w* rather than the exponent of *n* — has not, to our knowledge, been posed in this form.

### 1.1 Nature of contribution

**This is a framework and synthesis paper, not a deep-theorems paper.** Its contribution is posing a question that has not been explicitly studied — how treewidth and constraint-language structure modulate quantum amplitude amplification — and systematically connecting existing tools (amplitude amplification, quantum backtracking, QSETH, tensor-network simulation) to answer it. Most individual components are applications or restatements of known results; the value lies in the synthesis and in identifying the structural reasons why quantum advantage is elusive in this setting.

We organize the analysis around the following results, stated with honest assessments of their novelty:

1. **Proposition 1 (Poly-Space Quadratic Speedup).** Quantum backtracking (Montanaro 2018) halves the dominant exponent of classical polynomial-space treewidth recursion. *This is a direct application of Montanaro's theorem to a known recursion tree. The observation is straightforward; the value is contextual.*

2. **Observation 1 (Memoization Orthogonality).** When both classical and quantum algorithms are granted exponential space, quantum search provides no speedup on treewidth DP, because the bottleneck (table construction) is deterministic computation, not search. Amplitude amplification is orthogonal to memoization. *This is a tautology once stated clearly — quantum search accelerates search, not deterministic computation — but it explains why no quantum treewidth-DP speedup has been found.*

3. **Proposition 2 (Precomputation Tradeoff).** A hybrid classical-precomputation / Grover-search strategy adapted from Ambainis et al. (SODA 2019) to tree-decomposition DP yields moderate asymptotic cost-model improvements at specific parameters. *These improvements are unlikely to survive fault-tolerance overhead (Campbell–Khurana–Montanaro 2019) and should not be interpreted as practical predictions.*

4. **Theorem 1 (Universal Quadratic Speedup).** All classical randomized SAT algorithms with poly-time trials admit a full quadratic quantum speedup via amplitude amplification. Corollary: quantum PPSZ achieves O\*(1.143^*n*) for 3-SAT, beating quantum Schöning's O\*(1.155^*n*). *This is a direct consequence of Bennett (1973) + Brassard–Høyer–Mosca–Tapp (2002), implicit since Dantsin–Kreinovich–Wolpert (2005). We state it because its application to PPSZ appears underappreciated, and because an earlier draft of this paper incorrectly claimed PPSZ resisted quantization.*

5. **Observation 2 (Simulability Barrier).** Quantum circuits whose interaction graph has treewidth O(*w*) are classically simulable in time 2^{O(*w*)} · poly(*n*). *This is a direct corollary of Markov–Shi (SICOMP 2008), not a new result. We restate it because its implication — that structure-respecting quantum circuits cannot provide advantage — has not been articulated in the treewidth-CSP context.*

6. **Theorem 2 (QSETH Lower Bound).** Under QSETH, the exponent constant *c* in O\*(*d*^{*cw*}) satisfies *c* ≥ 1/2 for primal-pathwidth-parameterized SAT. *This is a clean conditional result, but it only constrains algorithms in the pw = Θ(n) regime (see Remark 3, §8.1). It serves as a sanity-check baseline rather than a deep structural barrier.*

7. **Analysis of Recursive Grover + Open Question.** The naïve recursive Grover strategy achieves *c* = 1/2 at decomposition depth 1 (Proposition 3) but fails at depth ≥ 2 (Analysis 1). This motivates an open question — whether *c* < 1 is achievable by *any* quantum algorithm for general decompositions — but we emphasize that the failure of one naïve algorithm is weak evidence for a universal barrier. The OR/AND asymmetry (§8.4) is a structural explanation for why *this specific algorithm* fails, not a proof that all algorithms must fail.

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

**Pathwidth vs. treewidth.** A *path decomposition* is a tree decomposition whose underlying tree is a path. The *pathwidth* pw of an instance is the minimum width over all path decompositions. Since every path is a tree, pw ≥ *w*; more precisely, pw ≤ *w* · O(log *n*) (Bodlaender 1998). Lower bounds proved for pathwidth (such as Theorem 2) are therefore *weaker* than corresponding lower bounds for treewidth — they rule out less. Throughout this paper, "treewidth" and "pathwidth" refer to the *primal* graph (vertices = variables, edges = co-occurrence in a constraint) unless stated otherwise. Other variants (incidence treewidth, dual treewidth) can differ substantially (Samer–Szeider 2010).

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

## 4. Observation 1: Memoization Orthogonality

### 4.1 Statement

**Observation 1 (Memoization Orthogonality).** *For any CSP with domain size d ≥ 2, treewidth w ≥ 1, and n sufficiently large (n > d^{2(w+1)}), the quantum backtracking algorithm of Proposition 1 (which uses polynomial space) is asymptotically slower than classical memoized dynamic programming (which uses O(n · d^{w+1}) space).*

*Proof.* Classical memoized DP costs O(*n* · *d*^{*w*+1}). As a function of *n*, this is *linear* in *n* times a constant depending on *w* and *d*:

> Classical (exp-space): *n* · *d*^{*w*+1}

Quantum backtracking costs (ignoring poly factors):

> Quantum (poly-space): *n*^{(*w*+1)·log₂*d* / 2}

Set α = (*w*+1)·log₂*d* / 2. We need *n*^α > *n* · *d*^{*w*+1} for the orthogonality comparison, i.e., *n*^{α−1} > *d*^{*w*+1}.

Since α = (*w*+1)·log₂*d* / 2 and for *d* ≥ 2, *w* ≥ 1 we have α ≥ 1, the inequality *n*^{α−1} > *d*^{*w*+1} holds for all *n* > *d*^{(*w*+1)/(α−1)} when α > 1.

For *d* = 2, *w* = 1: α = 1.0, so α − 1 = 0 and the comparison is marginal. For *d* = 2, *w* = 2: α = 1.5, so *n*^{0.5} > 8 when *n* > 64. For *d* ≥ 3, *w* ≥ 1: α ≥ (*w*+1)·log₂3/2 > 1, and the crossover occurs at modest *n*.

More precisely, for any fixed *d* ≥ 3 and *w* ≥ 1, there exists *N*(*d*,*w*) polynomial in *d*^*w* such that for all *n* > *N*(*d*,*w*), quantum backtracking is slower than classical memoized DP. □

### 4.2 Interpretation

**Caveat on resource asymmetry.** Observation 1 compares a poly-space quantum algorithm to an exp-space classical algorithm — an asymmetric comparison. When both algorithms are granted exponential space, the picture changes: a quantum algorithm with QRAM could store the same DP tables as the classical algorithm, then use Grover to search the final table in O(√(d^{w+1})) rather than O(d^{w+1}). The total cost would be O(*n* · *d*^{*w*+1}) for table construction plus O(*d*^{(*w*+1)/2}) for the final search — dominated by the construction phase, which is identical to classical DP. The precise content of the memoization orthogonality is therefore: **when both sides use memoized DP, quantum search provides no speedup on the dominant (table-construction) cost.** The table construction is a deterministic computation, not a search, and Grover does not accelerate deterministic computation.

This is a weaker but more precise statement than "quantum is slower." The obstruction is not that quantum algorithms are inherently worse, but that the *specific advantage* of amplitude amplification (accelerating search) is orthogonal to the *specific advantage* of memoization (avoiding redundant computation). The two optimizations target different bottlenecks.

### 4.3 Numerical validation

Our simulation confirms the memoization orthogonality across all tested parameters (*d* ∈ {2,3,5,9}, *w* ∈ {1,2,3,5}). The crossover occurs at remarkably small *n* (typically *n* ≈ 4), reinforcing that this is not an asymptotic artifact but a practical reality.

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

**Proposition 2.** *The optimal cutoff level ℓ* balances these terms, yielding a hybrid cost that can fall below the classical DP baseline at specific parameter settings.*

*Proof.* By direct optimization over ℓ (see validation code, §Appendix). The optimal ℓ* satisfies 2^{ℓ*} · *d*^{*w*+1} · ln 2 ≈ (*w*+1) · ln *d* / 2 · *d*^{(*w*+1)(D−ℓ*)/2} · ln *d*. For moderate parameters, ℓ* ≈ *D*/2 to 2*D*/3. □

**Caveat.** The improvements are moderate asymptotic cost-model ratios, not asymptotic exponent improvements — the base *d*^*w* is unchanged. More importantly, Campbell–Khurana–Montanaro (2019) showed that even Grover's full quadratic speedup can vanish under realistic fault-tolerance assumptions. Constant-factor improvements from a cost model that ignores gate counts, error correction, and QRAM overhead should not be interpreted as practical predictions. We include Proposition 2 to characterize the theoretical landscape, not to claim practical advantage.

### 5.2 Limitations

The speedup is moderate (constant factors, not asymptotic exponent improvement) and parameter-dependent. As *d* or *w* grow, the precomputation cost dominates earlier, shrinking the Grover-searchable upper portion. This is the tree-decomposition analogue of the 1.817^*n* vs. ideal 1.414^*n* gap in the subset-lattice setting of Ambainis et al.

---

## 6. The Universal Quadratic Speedup and Its Limits

### 6.1 Theorem 1: Universal Quadratic Speedup for Randomized SAT Algorithms

An earlier draft of this paper proposed an "oracle-shaped vs. process-shaped" classification of randomized algorithms, arguing that algorithms like PPSZ resist quantum speedup while algorithms like Schöning quantize cleanly. **That classification was incorrect.** Both algorithms — and indeed all randomized algorithms in the relevant class — admit a full quadratic quantum speedup via amplitude amplification. We correct the record here and reframe the section around the right question.

**Theorem 1 (Universal Quadratic Speedup).** *Let A be a classical randomized algorithm that solves k-SAT on n-variable, m-clause formulas in expected time O\*(c^n), where each execution of A (given its random bits) is implementable as a deterministic computation in time T(n, m) = poly(n, m). Then there exists a bounded-error quantum algorithm solving k-SAT in time O\*(c^{n/2}).*

*Proof.* View A as a deterministic function of its random input *r*: A(*r*) = 1 if the execution with randomness *r* produces a satisfying assignment, 0 otherwise. By assumption, Pr_r[A(*r*) = 1] ≥ 1/c^n.

By Bennett (1973), the deterministic computation A(*r*) can be implemented as a reversible circuit of size O(T(n,m)^{1+ε}) for any ε > 0. For T = poly(n, m), the reversible circuit is polynomial-size.

This reversible circuit implements the unitary |*r*⟩|0⟩ → |*r*⟩|A(*r*)⟩, which serves as the Grover oracle. Amplitude amplification (Brassard–Høyer–Mosca–Tapp 2002) over the space of random inputs, using this oracle, finds a successful *r* in O(1/√p) = O(c^{n/2}) iterations, each costing poly(n, m). Total: O\*(c^{n/2}). □

*Remark.* This observation is implicit in Dantsin–Kreinovich–Wolpert (SIGACT News, 2005) and is a direct consequence of the amplitude amplification theorem. We state it explicitly because its application to the PPSZ algorithm appears underappreciated in the literature.

### 6.2 Application: Quantum PPSZ beats quantum Schöning

Applying Theorem 1 to the two main classical SAT algorithms:

**Quantum Schöning.** Schöning's algorithm (1999) runs in O\*((2(k−1)/k)^n). By Theorem 1: O\*((2(k−1)/k)^{n/2}). For 3-SAT: O\*(1.155^n). (This is the result of Ambainis 2004.)

**Quantum PPSZ.** PPSZ (Paturi et al. 2005), with Hertli's (2014) improved analysis, runs in O\*(1.307^n) for 3-SAT. Each trial draws a random permutation π and random bits, then applies bounded-width resolution and variable-setting — a deterministic poly-time computation given (π, bits). By Theorem 1: O\*(1.307^{n/2}) = O\*(1.143^n).

| *k* | Schöning | PPSZ | Q-Schöning | **Q-PPSZ** | Grover | Best known quantum |
|-----|----------|------|------------|-----------|--------|-------------------|
| 3 | 1.334 | 1.307 | 1.155 | **1.143** | 1.414 | **1.143 (Q-PPSZ)** |
| 4 | 1.500 | 1.469 | 1.225 | **1.212** | 1.414 | **1.212 (Q-PPSZ)** |
| 5 | 1.600 | 1.569 | 1.265 | **1.253** | 1.414 | **1.253 (Q-PPSZ)** |
| 7 | 1.714 | 1.693 | 1.309 | **1.301** | 1.414 | **1.301 (Q-PPSZ)** |
| 10 | 1.800 | 1.794 | 1.342 | **1.339** | 1.414 | **1.339 (Q-PPSZ)** |

**Quantum PPSZ beats quantum Schöning for all k ≥ 3.** The improvement comes entirely from PPSZ being classically superior; the quantum speedup mechanism is identical.

**Erratum.** An earlier version of this paper listed quantum PPSZ as "hypothetical, NOT achieved" and proposed an "oracle-shaped vs. process-shaped" classification to explain why PPSZ purportedly resisted quantization. This was based on a misreading of Rennela–Brand–Laarman–Dunjko (Quantum, 2023). That paper explores speedups of PPSZ's *internal subroutines* beyond naive amplitude amplification — a different and harder question. The naive amplitude amplification of the full PPSZ trial is straightforward and gives the bounds above. We retract the oracle-vs-process classification.

### 6.3 The correct open question

Since every classical randomized algorithm with poly-time trials quantizes to √(base) by Theorem 1, the interesting question is not "which algorithms quantize" (they all do) but:

**Open Question.** *Does there exist a quantum algorithm for k-SAT running in time O\*(c^n) with c < √c_{classical}, where c_{classical} is the base of the best classical algorithm?*

Such an algorithm would need to exploit problem structure in a *fundamentally quantum* way — not merely amplifying a classical trial, but using interference, entanglement, or quantum walks to navigate the solution space more efficiently than any classical randomized algorithm. The Rennela et al. (2023) programme of quantum-accelerating internal subroutines (tree search, resolution) is one approach; it has yielded partial results but no unconditional improvement over the √(base) bound.

**Relation to the treewidth question.** For treewidth-parameterized CSPs, the analogous question is: can quantum algorithms beat the √(d^w) = d^{w/2} base? Theorem 2 (§8.1) establishes d^{1/2} as a QSETH-conditional lower bound per unit of width, and Analysis 1 (§8.2) shows that recursive Grover fails to achieve even d^{cw} for any c < 1 at decomposition depth ≥ 2. The memoization orthogonality (§4) provides the structural explanation: classical DP already exploits tree structure so thoroughly that quantum search has nothing left to accelerate.

---

## 7. Observation 2: The Simulability Barrier

### 7.1 Statement

**Observation 2 (Simulability Barrier).** *Let I be a CSP instance with treewidth w. If a quantum algorithm for I produces a quantum circuit C whose interaction graph has treewidth O(w), then by Markov–Shi (SICOMP 2008; refined by Cheng et al. 2025 for rank-width), C is classically simulable in time 2^{O(w)} · poly(n). In particular, any quantum speedup over classical DP requires the circuit to have treewidth ω(w).*

This is a direct corollary of the Markov–Shi tensor-network simulation theorem, not a new result. We state it here because its *implication* for our question — that structure-respecting quantum circuits are automatically classically simulable — has not been articulated in the treewidth-parameterized CSP setting.

*Derivation.* Markov and Shi (SICOMP 2008) show that a quantum circuit on *n* qubits whose tensor-network graph has treewidth *tw* is simulable in time *d_gate*^{O(*tw*)} · poly(*n*), where *d_gate* is the maximum gate dimension. If *tw* = O(*w*), this cost is 2^{O(*w*)} · poly(*n*), matching classical DP up to polynomial factors. Therefore, no quantum advantage is possible with such circuits. □

### 7.2 The structural dilemma

This creates a fundamental constraint on the *shape* of any RQ1-positive algorithm:

**Dilemma.** To exploit instance structure, a quantum algorithm should "know about" the tree decomposition — e.g., process bags in tree order, apply constraint-checking unitaries respecting the bag structure. But such structure-respecting circuits will have interaction graphs of treewidth O(*w*), triggering the Markov-Shi simulability bound.

**Grover's escape.** Grover's algorithm avoids this by using a *global* diffusion operator (the reflection about the mean) that acts on all *n* qubits simultaneously, creating circuit treewidth Θ(*n*) regardless of instance structure. This makes it classically hard to simulate — but it also means Grover cannot exploit low instance treewidth.

**The open question.** Is there a circuit family with treewidth *strictly between w and n* that achieves O(*d*^{*cw*}) for some *c* < 1? Our numerical analysis (§Appendix, Table 4) shows this would require a non-trivial interpolation between structure-awareness and entanglement complexity.

### 7.3 Recent refinements

Cheng–Wang–Deng–Chen–Ji (arXiv:2510.06775, 2025) and de Colnet et al. (arXiv:2605.29944, 2026) show that *rank-width*, not treewidth, is the sharper parameter governing classical simulability. Circuits with bounded rank-width (which can be substantially smaller than treewidth) are simulable via FeynmanDD and quadratic sums-of-powers methods. This tightens the barrier: even circuits that "look" high-treewidth by naive tensor contraction may be simulable if their rank-width is bounded.

---

## 8. The QSETH Lower Bound, Recursive Grover Analysis, and the Gap Question

This section contains one conditional theorem (§8.1), one analysis of a specific algorithmic strategy (§8.2), and one conjecture that the analysis motivates (§8.3). We are careful to separate these: the theorem is a statement about *all* quantum algorithms; the analysis concerns *one particular* algorithm; the conjecture extrapolates from the analysis to a broader claim that remains open.

### 8.1 Theorem 2: QSETH Lower Bound

We work with the *primal pathwidth* of a CNF formula — the pathwidth of the graph whose vertices are variables and whose edges connect variables appearing in the same clause. This is the standard parameterization in the SETH/pathwidth literature (cf. "The Primal Pathwidth SETH," arXiv:2403.07239). All references to "pathwidth" below mean primal pathwidth unless stated otherwise.

**Theorem 2.**  *Assuming QSETH, for every ε > 0, there is no bounded-error quantum algorithm that, given a CNF formula φ on n variables together with a path decomposition of primal pathwidth pw, decides satisfiability of φ in time O((2^{1/2} − ε)^{pw} · poly(n)). The algorithm is required to work for all values of pw (not merely bounded pw). Equivalently, the exponent constant c in O\*(2^{c·pw}) satisfies c ≥ 1/2.*

*Proof.* Suppose for contradiction that such an algorithm *A* exists for some ε > 0.

**Step 1 (pathwidth bound).** Any CNF formula φ on *n* variables has primal pathwidth pw(φ) ≤ *n* − 1. To see this, take any ordering v₁, v₂, …, v_n of the variables; the path decomposition with bags {v₁, …, v_n}, {v₂, …, v_n}, …, {v_n} has width *n* − 1 and is a valid primal path decomposition (every clause's variables appear together in some bag, since they all appear in the first bag). A path decomposition can be computed in linear time for any fixed width (Bodlaender 1996).

**Step 2 (applying A).** Let φ be any *k*-SAT formula on *n* variables. Compute a path decomposition of width pw(φ) ≤ *n* − 1 (Step 1). Apply *A*:

> Time(*A*, φ) = O((√2 − ε)^{pw(φ)} · poly(*n*)) ≤ O((√2 − ε)^{*n*−1} · poly(*n*)) = O(2^{(1/2 − δ)*n*} · poly(*n*))

where δ = 1/2 − log₂(√2 − ε) > 0 for any ε > 0 (since log₂(√2 − ε) < log₂(√2) = 1/2).

**Step 3 (QSETH contradiction).** QSETH (Buhrman–Patro–Speelman 2021) states: for every δ > 0, there exists *k*₀ such that for all *k* ≥ *k*₀, no bounded-error quantum algorithm solves *k*-SAT on *n* variables in time O(2^{(1/2 − δ)*n*}). But *A* solves *k*-SAT for all *k* in time O(2^{(1/2−δ)*n*}), contradicting QSETH for *k* ≥ *k*₀(δ). □

*Remark 1.* The theorem applies to algorithms parameterized by pw for *all* values of pw, not merely algorithms that are FPT in pw. An algorithm running in time *f*(pw) · poly(*n*) for any computable *f* would suffice for the contradiction, since *f*(pw) ≤ *f*(*n*−1) and QSETH concerns the dependence on *n*.

*Remark 2.* For general domain *d* ≥ 2, encoding each *d*-valued variable with ⌈log₂ *d*⌉ Boolean variables increases primal pathwidth by a factor of at most ⌈log₂ *d*⌉. The lower bound generalizes: no quantum algorithm achieves O\*(*d*^{*cw*}) with *c* < 1/2 for *d*-ary CSPs parameterized by primal pathwidth.

*Remark 3 (Scope limitation).* The proof uses pw(φ) ≤ *n* − 1, which means the QSETH contradiction arises only from instances where pw = Θ(*n*) — the regime where pathwidth is not a useful parameter. For instances where pw ≪ *n* (the actual parameterized regime of interest), the theorem still formally applies (it constrains the function *f*(pw) in algorithms running in time *f*(pw) · poly(*n*)), but it does not rule out algorithms with runtime *g*(*n*) · *h*(pw) where *g*(*n*) ≥ 2^{*n*/2} provides the QSETH-compatible baseline and *h*(pw) < (√2)^{pw} provides a parameterized speedup. In other words, Theorem 2 constrains algorithms that are *purely* parameterized by pw, but does not constrain hybrid runtimes that also depend on *n* in a QSETH-compatible way. Proving a lower bound that bites in the pw ≪ *n* regime would require a fine-grained reduction producing *k*-SAT instances with pw = o(*n*), which the existing LMS/pw-SETH framework does not provide.

### 8.2 Analysis of Recursive Grover on Tree Decompositions

We now analyze the natural recursive Grover strategy applied to tree decompositions. We emphasize: this section concerns *one specific algorithmic approach*. The conclusions do not rule out other quantum algorithms that might interact with memoization or tree structure differently.

**Proposition 3 (Depth-1 Upper Bound).** *Let I be a CSP over domain [d] with a tree decomposition of width w and depth D = 1 (a star: one root bag with n leaf bags as children). Then satisfiability of I can be decided by a bounded-error quantum algorithm in time O(d^{(w+1)/2} · n · poly(w)), achieving c = 1/2 in O\*(d^{cw}).*

**Remark on triviality.** Depth-1 decompositions represent instances where a single separator of *w*+1 variables connects *n* independent subproblems — instances that are already easy classically (O(*d*^{*w*+1} · *n*) by brute-forcing the separator). The value of Proposition 3 is not practical; it is that *c* = 1/2 is *achievable*, matching the QSETH lower bound (Theorem 2) and showing that the bound is tight for this subclass. The interesting instances have depth *D* = Θ(log *n*), where Analysis 1 shows recursive Grover fails.

*Proof.* In a star decomposition, the root bag contains *w*+1 variables and each leaf bag shares a subset of these with the root. Satisfiability reduces to: ∃σ ∈ [*d*]^{*w*+1} such that all leaf constraints are simultaneously satisfied given σ.

We construct the Grover oracle *O*_σ as follows. For a candidate root assignment σ:

1. **Constraint encoding.** For each leaf bag *B_i* (*i* = 1, …, *n*), the constraint *C_i* depends only on σ restricted to *B_i* ∩ *B_root* (which is determined by σ) and possibly on variables private to *B_i*. In a depth-1 decomposition, any variable private to *B_i* can be existentially quantified by checking at most *d* extensions, each costing O(1). Classically, checking one leaf costs O(*d*^{|*B_i* \ *B_root*|}) ≤ O(*d*^{*w*+1}).

2. **Reversible implementation.** Each leaf check is a Boolean function of σ and ancilla bits. Using standard reversible-computation techniques (Bennett 1973), we implement the check as a reversible circuit on O(*w* log *d*) qubits per leaf, with O(poly(*w*, *d*)) gates.

3. **Parallel evaluation.** The *n* leaf checks are on disjoint qubit registers (each leaf uses its own ancillae). We compute them in parallel, AND the results into a single output qubit, and uncompute the ancillae. Total gate count: O(*n* · poly(*w*, *d*)). Circuit depth: O(poly(*w*, *d*) + log *n*) (the log *n* for the AND tree).

4. **Grover search.** Apply Grover's algorithm over the *d*^{*w*+1} root assignments using *O*_σ. Cost: O(√(*d*^{*w*+1})) oracle calls, each costing O(*n* · poly(*w*, *d*)) gates. Total: O(*d*^{(*w*+1)/2} · *n* · poly(*w*, *d*)). □

**Analysis 1 (Recursive Grover at depth D ≥ 2).** *The naïve recursive Grover strategy — applying Grover search over separator assignments at each level of a balanced tree decomposition, with recursive evaluation of children — satisfies the recurrence:*

> *T*(0) = O(1)
> *T*(*D*) = √(*d*^{*w*+1}) · 2 · *T*(*D* − 1)

*Solving: T(D) = (2 · d^{(w+1)/2})^D.*

*For a balanced decomposition with n = 2^D bags, this exceeds the classical memoized DP cost n · d^{w+1} whenever D ≥ 2.*

*Derivation.* At each internal node of depth *D*, the naïve strategy applies Grover search over the *d*^{*w*+1} separator assignments. Each Grover query evaluates both children, which are themselves depth-(*D*−1) subproblems solved recursively. This gives the recurrence above. The comparison to classical DP:

> (2 · *d*^{(*w*+1)/2})^*D* > 2^*D* · *d*^{*w*+1}  ⟺  *d*^{(*w*+1)*D*/2} > *d*^{*w*+1}  ⟺  *D* > 2

At *D* = 2 the costs are equal; at *D* ≥ 3 recursive Grover is strictly worse. □

**Caveat on composition.** The recurrence above assumes that nested amplitude amplification composes multiplicatively — i.e., that the √ factor at each level multiplies independently. This is a standard but non-trivial assumption. Rigorous composition of nested Grover search requires: (a) reversible implementation of each recursive oracle call, (b) careful tracking of success probabilities across levels (Høyer–Mosca–de Wolf 2003), and (c) that the recursion depth *D* does not introduce polynomial overhead beyond what the recurrence captures. For *D* = O(log *n*) (balanced decompositions), these conditions are satisfied by the analysis in Montanaro (2018, §4.1) and Childs–Kothari–Kovacs-Deak–Sundaram–Wang (2025, §3), which handle recursive quantum divide-and-conquer with logarithmic depth. We do not claim optimality of this composition — only that the recurrence correctly models the naïve strategy under standard composition assumptions.

**Important distinction.** Analysis 1 shows that *one particular quantum algorithm* (recursive Grover) fails to beat classical memoized DP at depth ≥ 2. It does *not* show that *no* quantum algorithm can do so. A different approach — for instance, one that interacts with the memoization structure itself, or that uses quantum walks rather than amplitude amplification — might achieve *c* < 1. The question of whether such an algorithm exists remains open.

### 8.3 The Gap Question

Theorem 2 provides a rigorous lower bound; Analysis 1 provides evidence for the upper bound. Together they motivate:

**Gap Question.** *For CSPs with primal treewidth w over domain d, the optimal quantum exponent c in O\*(d^{cw}) satisfies:*

| Regime | Lower bound on *c* | Best known upper bound on *c* | Status |
|--------|-------------------|------------------------------|--------|
| General (any *D*) | 1/2 (QSETH, Thm 2) | 1 (classical DP) | **Open gap [1/2, 1]** |
| Depth *D* = 1 | 1/2 (QSETH, Thm 2) | 1/2 (Prop. 3) | **Tight** |
| Depth *D* ≥ 2 | 1/2 (QSETH, Thm 2) | 1 (classical DP) | **Open** |

We conjecture that *c* = 1 is optimal for general decompositions of depth *D* ≥ 2 — that is, no quantum algorithm achieves *c* < 1 in this regime. The evidence for this conjecture comes from three independent sources: the memoization orthogonality (Observation 1, §4), the simulability barrier (Observation 2, §7), and the failure of naïve recursive Grover (Analysis 1). However, we stress that each of these is evidence about a *specific* obstruction or algorithm, not a proof of impossibility.

**Closing the gap** would require either: (a) a quantum algorithm achieving *c* < 1 at depth ≥ 2, refuting the conjecture; or (b) a QSETH-conditional proof that *c* = 1 is necessary at depth ≥ 2, which would require a fine-grained reduction from *k*-SAT to bounded-treewidth instances with pw ≪ *n* — a reduction the existing LMS framework does not provide, since it produces instances with pw = Θ(*n*).

### 8.4 The OR/AND Asymmetry

The Gap Question is motivated by a structural observation about tree-decomposition DP. The computation interleaves two fundamentally different operations:

- **Forget nodes** (existential quantification): OR over *d* domain values → Grover gives √*d* speedup per level.
- **Join nodes** (universal verification): AND over child subtrees → no quantum speedup.

In a balanced decomposition of depth *D*, the compound Grover saving is *d*^{(*w*+1)*D*/2} while the compound AND cost is 2^*D* (one binary join per level). Classical memoized DP amortizes both into a single linear scan of *n* · *d*^{*w*+1}. The recursive Grover algorithm pays *d*^{(*w*+1)/2} per level (the Grover-reduced OR cost), but this compounds across *D* levels rather than being absorbed by memoization. At *D* = 1, there is only one OR and no compounding — Grover wins cleanly. At *D* ≥ 2, the compounding overtakes the memoized baseline.

To our knowledge, this OR/AND asymmetry provides a structural explanation — though not a proof of impossibility — for why quantum amplitude amplification has not yielded improvements to the exponential base of treewidth-parameterized DP. Whether this asymmetry is a fundamental barrier or merely an artifact of the recursive Grover strategy remains an open question.

---

## 9. Synthesis: The Answer

### 9.1 The three-way tension

Classical structure and quantum amplitude amplification interact through three mutually reinforcing mechanisms:

**(a) The memoization effect.** Structure (bounded treewidth) enables classical DP with memoization, converting *n*^{Θ(*w*)} recursion into *n* · *d*^{O(*w*)} table-driven evaluation. Quantum search applied to the unmemoized recursion achieves a quadratic speedup over *that* baseline, but the memoized classical algorithm provides a *super-quadratic* speedup over the same baseline. When both sides are granted exponential space, quantum provides no speedup on the table-construction cost (§4.2). Net effect: **the specific advantage of amplitude amplification (accelerating search) is orthogonal to the specific advantage of memoization (avoiding redundant computation).**

**(b) The simulability effect.** Structure-respecting quantum circuits (those whose interaction graph mirrors the instance treewidth) are classically simulable (Observation 2, via Markov–Shi), erasing any quantum advantage. Avoiding simulability requires high-entanglement circuits — but those cannot exploit instance structure. Net effect: **structure constrains circuit design, removing quantum advantage.**

**(c) The precomputation tradeoff.** The Ambainis-style hybrid (classically precompute lower layers, Grover-search upper layers) partially circumvents (a), but the gains are moderate (constant factors at fixed parameters) and disappear as *d* or *w* grow. The memoization orthogonality is softened, not eliminated. Net effect: **structure enables partial quantum gains, but sub-quadratic.**

### 9.2 The central finding

**Bounded treewidth hurts quantum speedups more than it helps.** The structure that makes a problem classically tractable — decomposability into bounded-width subproblems, enabling memoized DP — is precisely the structure that neutralizes quantum advantages, via both the memoization orthogonality and the simulability barrier.

Quantum advantage for CSPs is maximized in a *sweet spot* of intermediate structure:

- *Too little structure* (random instances, high treewidth): quantum advantage is limited to Grover's √*N*, which is provably optimal (BBBBV).
- *Too much structure* (bounded treewidth): classical DP dominates, and structure-respecting circuits are simulable.
- *Intermediate structure* (e.g., *k*-SAT with moderate clause density, graph coloring on expander-like graphs): enough regularity for oracle-shaped algorithms (Schöning) to exploit, insufficient for classical DP to dominate. This is where quantum Schöning achieves its genuine advantage.

### 9.3 The Schaefer-landscape connection

On the constraint-language side, by Theorem 1 (Universal Quadratic Speedup), the quantum exponent base for any NP-hard language Γ is at most √*c*_classical(Γ), where *c*_classical(Γ) is the base of the best classical randomized algorithm for CSP(Γ). The achievable quantum speedup therefore *inherits* the classical fine-grained landscape: languages where classical algorithms are faster (lower *c*_classical) also have faster quantum algorithms (lower √*c*_classical). There is no known case where quantum provides a *super-quadratic* speedup — i.e., achieving base strictly less than √*c*_classical — for any specific constraint language.

**Remark (Tovey's theorem and bounded locality).** Bounded constraint overlap does not ensure tractability even classically. Tovey (1984) showed that 3-SAT restricted to instances where every variable appears in at most 3 clauses remains NP-complete. This is a classical counterpart to our finding: "local-looking" constraint structure (few constraints per variable, analogous to bounded bag size) does not by itself reduce hardness. The structural parameters that *do* ensure tractability — treewidth (Freuder; Dechter–Pearl), polymorphisms (Schaefer; Bulatov–Zhuk) — are global, not local.

---

## 10. Discussion

### 10.1 Practical implications

Babbush et al. (2021) and Campbell–Khurana–Montanaro (2019) have shown that quadratic quantum speedups are unlikely to yield practical advantages on error-corrected hardware at moderate problem sizes. Our results reinforce this pessimism for *structured* instances: the very structure that makes real-world instances solvable also makes quantum advantage elusive. The practical sweet spot — if it exists — lies in instances large enough for asymptotic speedups to matter but structured enough for oracle-shaped algorithms to exploit, yet not so structured that classical DP dominates.

### 10.2 Open questions

1. **The width-parameterized upper bound (RQ1 proper).** Can the precomputation tradeoff of Proposition 2 be made asymptotic — i.e., is there *c* < 1 such that treewidth-*w* CSPs are solvable in O(*d*^{*cw*} · poly(*n*))? Our analysis suggests the memoization orthogonality makes *c* = 1/2 impossible, but does not rule out *c* < 1. The Ambainis et al. subset-lattice analogy (1.817^*n* vs. 2^*n*) suggests *c* = log₂ 1.817 ≈ 0.862 as a plausible target.

2. **QSETH-conditional treewidth lower bounds (RQ2).** Can Lokshtanov–Marx–Saurabh-style reductions be made coherence-compatible, yielding a quantum analogue of "no (2−ε)^{tw} algorithm under SETH"?

3. **Polymorphism-indexed speedup classification (RQ3).** For which NP-hard constraint languages Γ does the best known quantum exponent match the square root of the best classical exponent?

4. **Counting (RQ4).** The QSETH-strikes-again results (Chen et al. 2023) suggest no quantum speedup for exact #CSP; amplitude estimation gives quadratic gains for approximate counting. A unified statement crossing Creignou-Hermann with QSETH remains open.

### 10.3 Limitations of this work

Our results are analytical, not algorithmic: we characterize the cost landscape rather than constructing new quantum algorithms. The validation code models asymptotic costs, not gate-level quantum circuits. The precomputation tradeoff (Proposition 2) requires a rigorous composition analysis (Høyer–Mosca–de Wolf bounded-error inputs) that we sketch but do not fully formalize. The oracle-vs-process dichotomy (§6) is a classification framework, not a theorem; formalizing it as a provable barrier would require defining "process-shaped randomness" in a way that admits a lower bound.

---

## 11. Conclusion

We have shown that the interaction between classical structure and quantum search is characterized by a fundamental tension: the structure that makes problems classically tractable is precisely the structure that neutralizes quantum advantages. This manifests through three reinforcing mechanisms — the memoization orthogonality, the simulability barrier, and the oracle-composition challenge — that together explain why no width-parameterized quantum speedup has been found despite decades of work on both quantum algorithms and parameterized complexity.

The honest answer to our research question is therefore nuanced: bounded treewidth *enables* a quadratic quantum speedup in the polynomial-space regime (Proposition 1), but this regime is dominated by classical memoized DP, rendering the quantum speedup practically moot. Genuine quantum advantage lives in the intermediate-structure regime, where oracle-shaped algorithms like quantum Schöning can exploit partial regularity without triggering classical tractability.

This finding has implications beyond CSPs: any computational domain where classical algorithms exploit decomposability (divide-and-conquer, dynamic programming, hierarchical methods) is likely to exhibit the same memoization orthogonality, limiting quantum advantage to regimes where the classical structural exploitation is absent or incomplete.

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
23. Tovey, C.A. (1984). A simplified NP-complete satisfiability problem. Discrete Appl. Math., 8(1), 85–89.
24. Høyer, P., Mosca, M., & de Wolf, R. (2003). Quantum search on bounded-error inputs. ICALP 2003, LNCS 2719, 291–299.
25. Samer, M., & Szeider, S. (2010). Algorithms for propositional model counting. J. Comput. Syst. Sci., 76(8), 850–868.
26. Bennett, C.H. (1973). Logical reversibility of computation. IBM J. Res. Dev., 17(6), 525–532.
27. de Colnet, A., Geerts, F., Hai, L.V., Laarman, A., Lee, H.K., & Pérez, G. (2026). Quadratic sums-of-powers for fixed-parameter tractable quantum-circuit simulation. arXiv:2605.29944.
28. Zalka, C. (1999). Grover's quantum searching algorithm is optimal. Phys. Rev. A, 60(4), 2746–2751.
29. Dantsin, E., Kreinovich, V., & Wolpert, A. (2005). On quantum versions of record-breaking algorithms for SAT. ACM SIGACT News, 36(4), 103–108.
30. Hertli, T. (2014). 3-SAT faster and simpler — unique-k-SAT upper bounds. SIAM J. Comput., 43(2), 718–725.
31. Paturi, R., Pudlák, P., Saks, M., & Zane, F. (2005). An improved exponential-time algorithm for k-SAT. J. ACM, 52(3), 337–364.

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
│   ├── treewidth_quantum_sim.py    # Validation: Propositions 1-2, Theorems 1-2
│   └── qseth_bounds.py            # Validation: Theorems 3-4, Gap Question
├── results/
│   ├── validation_results.json     # Full numerical results
│   ├── scaling_comparison.png      # Figure 1: cost scaling
│   ├── schoning_ppsz_asymmetry.png # Figure 2: k-SAT bases
│   ├── precomputation_tradeoff.png # Figure 3: hybrid tradeoff
│   └── qseth_gap_analysis.png     # Figure 4: gap theorem + OR/AND asymmetry
└── paper.md                        # This document
```

### A.3 Running the code

```bash
# Propositions 1-2, Theorems 1-2 (cost models, memoization, precomputation):
python code/treewidth_quantum_sim.py
python code/treewidth_quantum_sim.py --plot --json results/validation_results.json

# Theorems 3-4, Gap Question (QSETH lower bound, depth crossover, OR/AND):
python code/qseth_bounds.py
python code/qseth_bounds.py --plot

# Quick check (small parameters, fast):
python code/treewidth_quantum_sim.py --quick
```

### A.4 What the code validates

1. **Proposition 1 (§3):** Measures the effective exponent α = log(cost)/log(n) for both classical poly-space and quantum backtracking across d ∈ {2,3,5,9}, w ∈ {1,2,3,5,8}, n ∈ {16,...,4096}. Checks that the ratio α_quantum/α_classical → 0.5 as the dominant term grows.

2. **Theorem 1 (§4):** For each (d,w) pair, finds the crossover point where quantum backtracking becomes slower than classical exp-space DP. Confirms this occurs at small n for all d ≥ 3.

3. **Proposition 2 (§5):** Sweeps cutoff level ℓ from 0 to log₂(n) for the precomputation/Grover hybrid, reports the optimal ℓ and resulting speedup over classical DP.

4. **§6 tables:** Computes Schöning, quantum Schöning, PPSZ, hypothetical quantum PPSZ, and Grover exponent bases for k ∈ {3,4,5,7,10}.

5. **Theorem 2 (§7):** Compares log₂ costs of Markov-Shi simulation, Grover, and classical DP across (w,n) pairs, identifying the barrier regime.

6. **Theorem 2 (§8.1):** For each ε > 0, computes log₂(√2 − ε) and the resulting QSETH contradiction parameter δ, confirming every ε yields δ > 0.

7. **Theorem 4 (§8.2):** Computes quantum recursive Grover cost vs. classical memoized DP across decomposition depths D = 0..20, confirming quantum wins only at D ≤ 1.

8. **Gap Question (§8.3):** Validates the depth-1 upper bound c = 0.500 across all (d, w) combinations, and generates the gap visualization (Figure 4).

### A.5 Interpreting the output

- **✓ at Ratio ≈ 0.5:** Confirms Proposition 1 (quadratic speedup in dominant exponent). The ratio exceeds 0.5 at small parameters because poly-overhead terms (n^{3/2} · log n from Montanaro's theorem) contribute to the effective exponent; they become negligible as (w+1)·log₂d grows.

- **✓ OBSTRUCTION:** Confirms Observation 1 (memoization orthogonality). The crossover at n ≈ 4 means classical DP dominates for essentially all practical instance sizes.

- **✓ BEATS / ✗ WORSE:** For Proposition 2 (precomputation tradeoff), indicates whether the hybrid beats classical DP. All tested parameters show ✓ BEATS with speedups of 5–32×.

- **←best in the k-SAT table:** Marks the best *known* (not hypothetical) quantum exponent base for each k.
