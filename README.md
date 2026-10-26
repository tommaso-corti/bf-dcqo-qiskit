# BF-DCQO for cardinality-constrained portfolio selection

This repository contains an independent Qiskit implementation of bias-field digitized
counterdiabatic quantum optimization (BF-DCQO) [BF], applied to the
cardinality-constrained Markowitz problem in the formulation of [LS]. It is a personal
project and is not affiliated with Kipu Quantum or IonQ.

The notebook was written as a companion to [this Medium article](https://medium.com/@corti.tommaso/i-rebuilt-a-quantum-algorithm-from-the-equations-up-it-kept-trying-to-fool-me-c8262f7bef1a),
which describes the project for a broader audience.

The notebook implements the algorithm step by step (QUBO and Ising formulation,
closed-form counterdiabatic coefficient, circuit construction, iterative bias-field
update) and applies it to a problem of 25 assets of the S&P 500, for which the exact
optimum is known by exhaustive enumeration. The decomposition pipeline of [LS], which
splits a 250-asset universe into clusters that fit the available qubits, is not
implemented: the problem studied here corresponds to a single cluster.

## Summary of results

On the 25-asset problem ($K=12$, $2^{25}$ states, 5,200,300 valid portfolios), BF-DCQO
finds the exact optimum in all five independent runs, on average at iteration 2.6,
i.e. after about 13,000 samples; at the last iteration about 2.4% of the shots are the
optimal portfolio. With the bias field switched off, the optimum is never sampled, which
isolates the contribution of the feedback loop. Pruning 60% of the gates leaves the
results unchanged, while two Trotter steps instead of four still find the optimum with
fewer valid samples. On a 14-asset sub-problem, depolarizing noise reduces the
probability of sampling the optimum from 15.3% to 0.12%, and pruning raises it to
0.60%; on IonQ's cloud simulator with the Aria 1 noise model the same configuration
reaches 3.4%.

At this problem size, simulated annealing finds the optimum in about 44 ms, so the
results do not indicate a quantum advantage. Their purpose is to characterize the
algorithm on an instance with known solution. All results refer to classical
simulation of the circuits; no experiment was run on quantum hardware.

## Contents

- `bfdcqo_qiskit_github.ipynb`: the notebook, with all outputs.
- `results_main_run.json`, `results_experiments.json`: checkpoint files with the
  results of the BF-DCQO runs (energies, bitstrings and per-iteration metrics). When
  present, the notebook loads them instead of repeating the runs.

## Requirements

The notebook was run with Python 3.11, Qiskit 2.2.1, Qiskit Aer 0.17.2, NumPy, Pandas,
Matplotlib and openpyxl (to read the price file). Section 11 additionally requires
`qiskit-ionq` and an IonQ account; the API key is read from the environment variable
`IONQ_API_TOKEN`, so that it never appears in the notebook.

## Data

The price data (daily prices of the largest S&P 500 positions, 2023-06-20 to
2026-06-17, source Bloomberg) are not included because of licensing restrictions. To
run the notebook with other data, provide an Excel file `SP_Daily_prices.xlsx` whose
first column contains the dates and whose other columns contain the daily prices of
one asset each, ordered by index weight; the notebook uses the first 25 columns.
Expected returns and covariances are estimated from simple daily returns, which is
the natural choice for the Markowitz model; [LS] uses log-returns.

## Changes with respect to the previous version

The previous version of the notebook contained an error in the preparation of the
initial state. The two reference papers use opposite sign conventions for the bias
field: [BF] writes the driver as $\sum_i(h^x_i X_i - h^b_i Z_i)$ with
$h^b_i \leftarrow +\langle\sigma^z_i\rangle$, whereas [LS] writes $\sum_j(-X_j + h^b_j Z_j)$
with $h^b_j \leftarrow -\langle\sigma^z_j\rangle$. The expression for the preparation
angle in Eq. (S26) of [BF] corresponds to the second convention, while the notebook
followed the first one. As a consequence, the bias field pushed the initial state away
from the best samples instead of towards them. The angle is now computed from the
eigenvector of Eq. (S25), and a test checks that the prepared state is the ground state
of the same driver used for the counterdiabatic term.

A second error concerned the random seeds: iteration $r$ of run `seed` used the
simulator seed `seed + r`, so that different runs shared random streams. The seed is
now `1000*seed + r`.

After the corrections, all results were recomputed. Since the optimum is now found in
almost every run, the final gap no longer discriminates between configurations, and the
experiments use metrics that do not saturate (first-hit iteration, samples to solution
and probability of the optimum). The notebook was also restructured so that all
experiments use a single BF-DCQO function, with optional noise model and external
backend, instead of separate copies. The bias-field ablation, the classical benchmark
and the comparison with IonQ's simulator are new.

## Notes

The circuits are executed with the V1 `Sampler` primitive of Qiskit Aer, which is being
deprecated in favour of `SamplerV2`. Moving to `SamplerV2` would change the random
number generation, and therefore the numerical results, and is left for a future
version.

Transpiling the 14-qubit circuit for IonQ's backend with Qiskit reduces the number of
CX gates from 1,456 to 1,092. This count refers to the gate set accepted by the
backend; the further compilation into IonQ's native gates is performed on IonQ's
side and is not visible from the notebook.

## References

- [BF] A. Gomez Cadavid, A. Dalal, A. Simen, E. Solano, N. N. Hegade, *Bias-field
  digitized counterdiabatic quantum optimization*, arXiv:2405.13898 (2024); Phys. Rev.
  Res. 7, L022010 (2025).
- [LS] A. Gomez Cadavid et al., *Large-scale portfolio optimization on a trapped-ion
  quantum computer*, arXiv:2602.23976 (2026).
- [STA] N. N. Hegade et al., *Shortcuts to adiabaticity in digitized adiabatic quantum
  computing*, Phys. Rev. Appl. 15, 024038 (2021).
- P. W. Claeys, M. Pandey, D. Sels, A. Polkovnikov, *Floquet-engineering
  counterdiabatic protocols in quantum many-body systems*, Phys. Rev. Lett. 123,
  090602 (2019).
