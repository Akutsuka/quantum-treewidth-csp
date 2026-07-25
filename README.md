# Classical Structure Meets Quantum Search

Research package for the paper: *"How Treewidth and Constraint-Language Algebra Modulate Amplitude Amplification"*

## Quick Start

```bash
# 1. Run the full validation suite (no dependencies beyond numpy):
python code/treewidth_quantum_sim.py

# 2. Generate plots (requires: pip install matplotlib):
python code/treewidth_quantum_sim.py --plot

# 3. Save all numerical results:
python code/treewidth_quantum_sim.py --json results/validation_results.json

# 4. Quick sanity check (small parameters, runs in seconds):
python code/treewidth_quantum_sim.py --quick
```

## What the code validates

| Paper result | What the code checks | Key output |
|---|---|---|
| **Proposition 1** (poly-space quadratic speedup) | Exponent ratio α_quantum/α_classical → 0.5 | Table of ratios across (d, w, n) |
| **Theorem 1** (memoization obstruction) | Quantum backtracking slower than classical DP for d≥3, w≥1 | Crossover points |
| **Proposition 2** (precomputation tradeoff) | Optimal cutoff level ℓ and speedup vs classical DP | Speedup factors (5–32×) |
| **§6** (Schöning vs PPSZ) | Exponent bases for k-SAT algorithms | Comparison table |
| **Theorem 2** (simulability barrier) | Markov-Shi cost vs Grover vs classical DP | Barrier identification |

## File structure

```
├── README.md                              # This file
├── paper.md                               # The research paper
├── code/
│   └── treewidth_quantum_sim.py           # Validation suite (Python 3.8+)
└── results/
    ├── validation_results.json            # Numerical results (auto-generated)
    ├── scaling_comparison.png             # Figure 1: three-regime cost scaling
    ├── schoning_ppsz_asymmetry.png        # Figure 2: k-SAT quantization gap
    └── precomputation_tradeoff.png        # Figure 3: hybrid classical/quantum
```

## The central finding

**Bounded treewidth hurts quantum speedups more than it helps.** Three mechanisms reinforce each other:

1. **Memoization obstruction**: Classical DP with memoization converts n^{Θ(w)} recursion into n·d^{O(w)}, beating quantum backtracking's n^{Θ(w/2)}.

2. **Simulability barrier**: Quantum circuits whose interaction graph has treewidth O(w) are classically simulable in 2^{O(w)}·poly(n), erasing quantum advantage.

3. **Oracle-composition challenge**: The precomputation/Grover hybrid achieves moderate constant-factor speedups, but cannot reach the ideal c=1/2 in O*(d^{cw}).

Quantum advantage peaks at **intermediate structure** — enough for oracle-shaped algorithms (like quantum Schöning for k-SAT) to exploit, but not enough for classical DP to dominate.

## Requirements

- Python 3.8+
- numpy (standard; `pip install numpy`)
- matplotlib (optional, for plots; `pip install matplotlib`)
