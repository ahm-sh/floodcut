# Development

Setup, commands, demo modes, conventions and limitations. The project overview is in the [README](../README.md).

## Setup

Tested with Python 3.14.0 on Windows 11. `requirements.txt` pins the direct dependencies, and `requirements-lock.txt` is the full freeze.

```bash
python -m venv .venv
```

```bash
.venv\Scripts\activate          # Windows;  source .venv/bin/activate on macOS/Linux
```

```bash
pip install -r requirements.txt
```

```bash
pip install -e .
```

## Commands

| task | command | time |
|---|---|---|
| run the tests (83 checks, including an end-to-end quantum smoke test and headless UI tests) | `pytest` | ~30 s |
| validate the scenario and QUBO on every feasible plan | `python -m floodcut validate` | 1 s |
| one quick live comparison of all four methods | `python -m floodcut quick` | ~3 s |
| quick benchmark preset | `python -m floodcut bench --preset quick` | ~1 s |
| full benchmark (48 QAOA + 40 SA runs, weight sweep, SA budget study) | `python -m floodcut bench --preset full` | ~100 s |
| regenerate the report figures from saved results | `python -m floodcut figures` | ~10 s |
| build the DOCX report from the saved results | `python report/build_report.py` | ~3 s |
| start the interactive demo | `streamlit run app/streamlit_app.py` | — |

`quick` accepts `--seed --n --k --wH --wF --wR --tau --p --init {dicke,basis} --maxiter --shots --opt-seed`.

## Demo

`streamlit run app/streamlit_app.py` opens two clearly separated modes:

- **Live quantum run:** runs exact, greedy, annealing and QAOA on the current sidebar settings, using the shared solver code. Results are hidden as soon as any setting changes.
- **Replay saved benchmark:** shows a saved run only when the scenario fingerprint, k, weights and τ match the sidebar exactly. Otherwise it shows a "Saved results do not match" banner and a button that switches to a live run. Provenance (timestamp and versions) is shown.

The page includes:
- the synthetic map, with selected sites and illustrative spill areas for each method;
- a side-by-side table with threshold badges;
- district burdens;
- the exact Pareto plot;
- a circuit panel showing qubits, depth, CX counts, the sample histogram, P(opt) and leakage.




## Conventions worth knowing

- **Bit order:** Qiskit counts keys are little-endian, so `x_i = int(key[n-1-i])` (see `objective.bitstring_to_plan`).
- **Angles:** `U_C(γ) = exp(−iγH_C)` is applied as `RZ(2γh_i)` and `RZZ(2γJ_ij)`. Each mixer edge applies `RXX(β)·RYY(β) = exp(−iβ/2·(XX+YY))`. Parameters are ordered `[γ1, β1, …, γp, βp]`, with a leading `β0` for the basis start.
- **Mixer:** a parity-ordered product over the ring. It is *not* the exact exponential of the full ring XY Hamiltonian, and a test checks that they differ.
- **Displayed QAOA plan:** the **most frequent sample**. "Best observed" is shown only with a caveat, because with many shots over few plans it approaches exhaustive search.
- **Normalizers:** min and range over all feasible plans for the scenario and k, held fixed across weights. This needs enumeration, so it only works for small instances.

## What was not tested / limitations

- **Hardware and noise:** no runs on quantum hardware and no noise models. All results are ideal statevector simulation, and transpilation assumes all-to-all connectivity.
- **Model scope:** In brief: no hydrodynamics, synthetic and uncertain inputs, pairwise-only interactions, a simplified fairness proxy, no safety guarantee, instances small enough to solve exactly, QAOA parameter sensitivity, and expert review required before any real-world use.



