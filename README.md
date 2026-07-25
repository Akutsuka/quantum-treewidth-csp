# Classical Structure Meets Quantum Search

Research package for the paper: *"How Treewidth and Constraint-Language Algebra Modulate Amplitude Amplification"*

## Quick Start

```bash
# Propositions 1-2, Theorems 1-2 (cost models, memoization, precomputation):
python code/treewidth_quantum_sim.py --plot --json results/validation_results.json

# Theorems 3-4, Gap Theorem (QSETH lower bound, depth crossover):
python code/qseth_bounds.py --plot

# Quick sanity check:
python code/treewidth_quantum_sim.py --quick
```

## Results (7 theorems/propositions)

| # | Result | What it says | Validated by |
|---|--------|-------------|--------------|
| P1 | Poly-space quadratic speedup | Quantum backtracking halves the dominant exponent of unmemoized treewidth DP | `treewidth_quantum_sim.py` |
| T1 | Memoization obstruction | Classical memoized DP beats quantum backtracking for d≥3, w≥1 | `treewidth_quantum_sim.py` |
| P2 | Precomputation tradeoff | Ambainis-style hybrid gives moderate constant-factor speedups | `treewidth_quantum_sim.py` |
| §6 | Oracle-vs-process dichotomy | Schöning quantizes fully; PPSZ resists (structural explanation) | `treewidth_quantum_sim.py` |
| T2 | Simulability barrier | Structure-respecting circuits are classically simulable (Markov-Shi) | `treewidth_quantum_sim.py` |
| **T3** | **QSETH lower bound** | **c ≥ 1/2: no quantum algorithm beats O*((√2)^pw) for pathwidth-parameterized SAT** | `qseth_bounds.py` |
| **T4** | **Depth-dependent crossover** | **c = 1/2 at depth 1; quantum WORSE than classical at depth ≥ 2** | `qseth_bounds.py` |

## The Gap Theorem

```
  ┌─────────────────────────────────────────────────────────┐
  │  LOWER BOUND (QSETH):          c ≥ 1/2                │
  │  UPPER BOUND (depth-1):         c = 1/2  (achieved)    │
  │  UPPER BOUND (general, D ≥ 2):  c = 1    (classical)   │
  │                                                         │
  │  GAP:  c ∈ [1/2, 1]  for general decompositions       │
  │  STATUS: OPEN — no c < 1 known for D ≥ 2              │
  └─────────────────────────────────────────────────────────┘
```

**Why the gap exists (OR/AND asymmetry):** Tree-decomposition DP interleaves forget nodes (OR — Grover gives √d) and join nodes (AND — no quantum speedup). At depth 1 there's one OR and no compounding; at depth ≥ 2 the compound AND costs overwhelm the Grover savings.

## File structure

```
├── README.md
├── paper.md                               # Full paper (Sections 1-12)
├── code/
│   ├── treewidth_quantum_sim.py           # Validates P1, T1, P2, §6, T2
│   └── qseth_bounds.py                   # Validates T3, T4, Gap Theorem
└── results/
    ├── validation_results.json
    ├── scaling_comparison.png             # Figure 1
    ├── schoning_ppsz_asymmetry.png        # Figure 2
    ├── precomputation_tradeoff.png        # Figure 3
    └── qseth_gap_analysis.png            # Figure 4
```

## Requirements

- Python 3.8+, numpy
- matplotlib (optional, for `--plot`)
