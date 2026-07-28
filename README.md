# Classical Structure Meets Quantum Search

**How Treewidth and Constraint-Language Algebra Modulate Amplitude Amplification**

A framework and synthesis paper connecting existing tools (amplitude amplification, quantum backtracking, QSETH, tensor-network simulation) to answer a question that has not been explicitly studied: how does classical problem structure interact with quantum speedups for constraint satisfaction problems?

## Quick Start

```bash
# Core analysis (Propositions 1-2, Observations 1-2):
python code/treewidth_quantum_sim.py --plot

# QSETH bounds and gap analysis (Theorems 1-2, Analysis 1):
python code/qseth_bounds.py --plot

# Quick sanity check:
python code/treewidth_quantum_sim.py --quick
```

## Results

| # | Name | Type | What it says |
|---|------|------|-------------|
| P1 | Poly-space quadratic speedup | Application of Montanaro 2018 | Quantum backtracking halves the exponent of unmemoized treewidth DP |
| O1 | Memoization orthogonality | Observation | AA doesn't speed up deterministic table construction; quantum search is orthogonal to memoization |
| P2 | Precomputation tradeoff | Application of Ambainis et al. 2019 | Hybrid classical/quantum yields moderate cost-model improvements (not practical predictions) |
| **T1** | **Universal quadratic speedup** | **Theorem (Bennett + BHMT)** | **All randomized SAT algorithms with poly-time trials quantize to √(base); quantum PPSZ = O\*(1.143^n)** |
| O2 | Simulability barrier | Corollary of Markov-Shi 2008 | Structure-respecting circuits are classically simulable |
| **T2** | **QSETH lower bound** | **Conditional theorem** | **c ≥ 1/2 in O\*(d^{cw}) for pathwidth-parameterized SAT (bites at pw=Θ(n) only)** |
| P3+A1 | Depth analysis + Gap Question | Analysis of one algorithm | Recursive Grover achieves c=1/2 at depth 1, fails at depth ≥ 2; c ∈ [1/2, 1] is open |

## The central finding

**Bounded treewidth hurts quantum speedups more than it helps.** Three mechanisms:

1. **Memoization orthogonality**: Classical DP exploits tree structure via memoization (deterministic table-building). Quantum search accelerates search, not deterministic computation. The two optimizations target different bottlenecks.

2. **Simulability barrier**: Quantum circuits that mirror instance treewidth are classically simulable (Markov-Shi). Exploiting structure requires structure-respecting circuits, but those are exactly the ones classical simulation catches.

3. **OR/AND asymmetry**: Tree-decomposition DP interleaves forget nodes (OR — Grover helps) and join nodes (AND — no speedup). At depth ≥ 2, compounding AND costs overwhelm Grover savings.


## File structure

```
├── README.md
├── CHANGELOG.md
├── paper.md                               # Full paper
├── code/
│   ├── treewidth_quantum_sim.py           # Validates P1, O1, P2, O2
│   └── qseth_bounds.py                   # Validates T2, P3, A1, Gap Question
└── results/
    ├── validation_results.json
    ├── scaling_comparison.png             # Figure 1
    ├── schoning_ppsz_asymmetry.png        # Figure 2 (pre-correction; shows old table)
    ├── precomputation_tradeoff.png        # Figure 3
    └── qseth_gap_analysis.png            # Figure 4
```

## Requirements

Python 3.8+, numpy. Optional: matplotlib (for `--plot`).
