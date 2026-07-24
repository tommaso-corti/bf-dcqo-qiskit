# BF-DCQO Implementation with Qiskit

An implementation and benchmark of **Bias-Field Digitized Counterdiabatic Quantum Optimization (BF-DCQO)** using Qiskit and `AerSimulator`.

## Key Features
- **QUBO to Ising Mapping**: Penalty-enforced matrix conversions with numerical stability checks.
- **Analytic Counterdiabatic Protocol**: Exact closed-form evaluation of the nested-commutator coefficient $\alpha_1(\lambda)$.
- **Trotterized Gate Synthesis**: Decomposition of 1-qubit and 2-qubit CD interactions ($Y_j$, $Z_i Y_j$, $Y_i Z_j$).
- **Bias-Field Feedback Loop**: Dynamic update of the driver Hamiltonian based on post-selected low-energy shot states.

## Requirements
```bash
pip install qiskit qiskit-aer numpy scipy matplotlib
```
