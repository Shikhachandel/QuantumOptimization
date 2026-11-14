Portfolio Optimization with QAOA: From Penalty Constraints to XY Mixers
BasQ Qiskit Fall Fest 2026 · Challenge 2 — Optimization / QAOA

An end-to-end quantum optimization study of selecting exactly 3 assets from 8 while balancing expected return and risk. The notebook develops a beginner-friendly penalty-based QAOA, improves it with a Dicke-state initialization and constraint-preserving XY mixer, and investigates what happens under classical competition, deeper circuits, and simulated hardware noise.
Main finding: On the ideal simulator, XY-QAOA produces 100% budget-valid portfolios and reaches 13.0% probability of sampling the exact optimum, versus 61.9% validity and 6.0% optimal-solution probability for the selected penalty-QAOA baseline at depth 5. However, classical methods win easily on this small instance, and simulated device noise largely removes the quantum improvement.

The problem
We generate a synthetic, reproducible market with 8 assets, expected returns $\boldsymbol{\mu}$, and a correlated covariance matrix $\Sigma$ from a two-factor model. The task is to minimize the mean–variance objective
\[
f(\mathbf{x}) = q\,\mathbf{x}^{T}\Sigma\mathbf{x} - \boldsymbol{\mu}^{T}\mathbf{x}
\]
subject to
\[
\sum_{i=1}^{8} x_i = 3, \qquad x_i \in \{0,1\}.
\]
Here $x_i=1$ means asset $i$ is selected, and $q$ controls risk aversion. The notebook uses $q=1$ and a fixed random seed. The “budget” is an exact number of selected assets, not a dollar-spending limit.

For ordinary QAOA, the hard constraint becomes a penalty:
\[
C_{\lambda}(\mathbf{x})=f(\mathbf{x})+\lambda\left(\sum_i x_i -3\right)^2.
\]
This is converted from QUBO to an Ising cost Hamiltonian, containing Pauli $Z$ and $ZZ$ terms. The quantum circuit optimizes angles in alternating cost and mixer layers.
Challenge levels covered
Level	Implementation
Beginner	8-asset portfolio, exact-budget penalty, QUBO-to-Ising encoding, penalty-versus-depth experiments, and brute-force verification
Intermediate	Constraint-preserving XY-QAOA, Dicke-state initialization, optimizer initialization strategies, optional CVaR objective, classical benchmarks, and gate-count comparison
Advanced (exploratory)	Gradient-variance/barren-plateau experiments for up to 14 qubits and noisy simulation using an IBM Heron fake backend
Appendix	Separate 5-asset implementation cross-checking penalty QAOA against XY-QAOA


Chosen intermediate route: Improved QAOA (not quantum machine learning). Instead of trying to discourage invalid portfolios with a large penalty, the Dicke state starts in the exact-$K$ subspace and the XY mixer preserves that subspace under ideal operation.
Results from the notebook
The classical ground truth is obtained by enumerating all $2^8=256$ binary strings, including $\binom{8}{3}=56$ valid portfolios.

Exact optimal portfolio: BETA, GAMA, THET
Optimal mean–variance cost: approximately −0.3323 (lower is better).

The notebook reports the following seeded, ideal-simulation results (best depth by optimal-solution probability for each method):
Method	Depth $p$	P(valid)	P(optimal)	Quality among valid solutions
Penalty QAOA ($\lambda=\lambda_{crit}$)	5	61.9%	6.0%	0.827
Penalty QAOA ($\lambda=10\lambda_{crit}$)	5	87.5%	1.8%	0.644
XY-QAOA (Dicke + XY mixer)	5	100.0%	13.0%	0.855
XY-QAOA + CVaR(0.1)	3	100.0%	10.1%	0.765


A random 8-bit string has a 1/256 ≈ 0.39% chance of matching the exact optimum; a uniformly random valid portfolio has a 1/56 ≈ 1.79% chance.
Classical comparisons
The notebook compares against exact enumeration, a greedy heuristic, and simulated annealing with budget-preserving swap moves. In the reported run:
- Exact enumeration: identifies the optimum in about 2.4 ms.
- Greedy: also identifies the optimum on this particular instance.
- Simulated annealing (50 swap steps): reaches the optimum in 95.8% of 500 seeded runs.
There is no demonstrated quantum advantage here. The contribution is an empirical comparison of constraint encodings, solution quality, circuit cost, and noise sensitivity—not a claim of speedup.
Gate cost and noise
The XY design has a meaningful overhead. After decomposition to CX gates, the notebook reports 210 CX for XY-QAOA versus 56 CX for penalty QAOA at $p=1$, and 498 versus 280 CX at $p=5$.

The advanced section uses Qiskit Aer with FakeFez, a simulated IBM Heron backend with a device-derived noise model; it does not execute on a real quantum processor. At $p=5$, XY-QAOA feasibility falls from 100% ideally to about 28.5% under noise. After conditioning on valid samples, the best noisy settings are roughly tied at 5.3% probability of the optimum. The notebook also explores readout mitigation and gradient-variance scaling.
How to run
Requirements: Python 3.12 was used for the saved run; Qiskit 2.x APIs are used.
1. Clone this repository and open challenge_2_portfolio_final.ipynb in JupyterLab, VS Code, or qBraid.
2. Install the dependencies in the same Python environment as your notebook kernel:
   python -m pip install "qiskit>=2.0" qiskit-aer qiskit-ibm-runtime scipy numpy matplotlib pandas pylatexenc
   Alternatively, uncomment and run the notebook's first %pip install cell.
3. Restart the kernel and Run All cells in order. The notebook defines variables and functions across successive sections; running later cells first can cause NameError or stale results.
4. Inspect the plots, printed comparison tables, and generated figures/ and results/ directories.
The notebook notes a run time of approximately 10 minutes on qBraid, with the parameter sweep and gradient experiments taking the most time. Runtime will vary by environment.
Repository layout
.
├── challenge_2_portfolio_final.ipynb   # Complete notebook (Beginner → Advanced + appendix)
├── README.md                           # This file
├── figures/                            # Generated plots (created on execution)
└── results/                            # Generated CSV comparisons (created on execution)
Examples of generated artifacts include figures/assets.png, figures/comparison.png, figures/appendix_crosscheck.png, results/intermediate_summary.csv, results/classical_baselines.csv, and results/appendix_crosscheck.csv. These folders/files are generated by running the notebook and are not required to open it.
Evaluation methodology
Results use probabilities from the full final QAOA state, rather than counting only the best observed bitstring:
- P(valid): probability of measuring exactly 3 selected assets.
- P(optimal): probability of measuring the globally optimal valid portfolio.
- Quality: normalized conditional solution quality among valid measured portfolios (1 is best).
- Classical reference: brute-force exhaustive enumeration.
- Correctness checks: compare the Hamiltonian diagonal to classical costs, verify simulator agreement with Qiskit, verify Dicke-state preparation, and compare sampled versus predicted probabilities.
Important limitation: ideal-simulator validity guarantees do not survive physical noise. The synthetic asset data are educational and are not investment recommendations.
Potential next steps
- Reduce XY-mixer routing and Dicke-state preparation overhead with a hardware-aware circuit.
- Test a warm start from a feasible portfolio instead of preparing a uniform Dicke state.
- Explore larger portfolio instances with stronger classical heuristic baselines.
- Validate noise-mitigation approaches on actual hardware when available.
Acknowledgments, references, and AI-use disclosure
This project builds on B. Tirado's Combinatorial Optimization with QAOA: Max-Cut tutorial for BasQ Qiskit Fall Fest 2026 (the notebook cites CC BY 4.0 for tutorial text and Apache 2.0 for tutorial code). The appendix identifies an independent implementation by team member Shikha.

Key references discussed in the notebook include Markowitz (1952) on portfolio selection; Farhi, Goldstone & Gutmann (2014) on QAOA; Hadfield et al. (2019) on alternating-operator/XY mixers; Bärtschi & Eidenbenz (2019) on Dicke states; Zhou et al. (2020) on QAOA initialization; Barkoutsos et al. (2020) on CVaR; and McClean et al. (2018), Larocca et al. (2022), and Ragone et al. (2024) on barren plateaus.

AI assistance: The source notebook discloses substantial use of Claude (Anthropic) through Claude Code for code, explanatory text, and local testing under team direction. It says the team selected the challenge/route and reviewed runs and results on qBraid. The team should complete the outstanding individual-contributions statement in the notebook/report before submitting or publishing.
Educational quantum optimization experiment · BasQ Qiskit Fall Fest 2026, Challenge 2.
