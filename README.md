# BF-DCQO with Qiskit

Implementation and benchmark of **Bias-Field Digitized Counterdiabatic Quantum Optimization (BF-DCQO)** applied to Portfolio Optimization on real S&P 500 data, built as a companion to [this Medium article](https://medium.com/@corti.tommaso/i-rebuilt-a-quantum-algorithm-from-the-equations-up-it-kept-trying-to-fool-me-c8262f7bef1a?postPublishedType=repub).

## What this is

A from-scratch Qiskit implementation of the quantum core of Cadavid et al. (2026), validated against brute-force on a 25-asset, K=12 portfolio instance. The goal is not to reproduce the full 250-asset pipeline, but to stress-test the BF-DCQO circuit itself against a certified classical optimum.

## Structure

The notebook follows the same four-stage logic as the article:

| Section | What it does |
|---|---|
| 2–3 | QUBO construction, Ising mapping, closed-form α₁ (with correction to Eq. S23 of the framework paper, verified numerically) |
| 4 | Trotterized gate synthesis — weight-1 and weight-2 Pauli terms only |
| 5 | Full BF-DCQO loop: circuit → shots → post-selection → bias update |
| 6–9 | Experiments: shot budget, Trotter depth, gate pruning, noise comparison |

Each section has a derivation cell above the code. The implementation is meant to be read against the math, not instead of it.

## Data

Requires `SP_Daily_prices.xlsx` (Bloomberg daily prices, 2023–2026), not included due to licensing. The notebook will fail at Section 6 without it. All upstream sections (QUBO, Ising, circuit construction, unit tests) run without the data file.

## Requirements
```bash
pip install qiskit qiskit-aer numpy scipy matplotlib
```

## Papers

- Cadavid et al., *Bias-Field Digitized Counterdiabatic Quantum Optimization*, Phys. Rev. Res. 7, L022010 (2025)
- Cadavid et al., *Large-Scale Portfolio Optimization on a Trapped-Ion Quantum Computer*, arXiv:2602.23976 (2026)
