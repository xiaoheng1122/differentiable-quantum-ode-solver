# Differentiable Quantum Circuit Solver for Ordinary Differential Equations

This is the final local release package for the PRA differentiable quantum
circuit (DQC) study.  It contains three mature differential-equation examples
that can be validated on a local ideal state-vector simulator:

1. a high-frequency damped oscillation (`lambda=20`);
2. a two-state strongly coupled linear system; and
3. a nonautonomous nonlinear Riccati equation.

The convergent--divergent nozzle experiment is deliberately outside V0.  Its
history remains in the earlier patch folders, but it is not a release example
or an accuracy gate here.

## Scope

- Python 3.10+ with `pyqpanda3`, NumPy, SciPy, PyTorch and Matplotlib;
- local ideal statevector only; no API key, cloud job or real QPU;
- exact/classical trajectories are post-training checks, never training labels;
- all 1001 holdout values are recomputed with native pyqpanda3;
- one foreground compute process at a time and no dense full-unitary materialisation.

## Quick start

Create a Python 3.10+ environment and install the release dependencies:

```powershell
python -m pip install -r requirements.txt
```

Run from this V0 directory with the shared environment:

```powershell
python example/DQC/run_all.py --backend native --profile smoke --run-prefix smoke_v0
```

For the formal budgets, run the three cases separately so each archive has a
clear run id:

```powershell
python example/DQC/run_damped_oscillation.py --backend native --profile standard --run-id lambda20_v0
python example/DQC/run_coupled_linear.py --backend native --profile standard --run-id coupled_v0
python example/DQC/run_riccati.py --backend native --profile standard --run-id riccati_v0
```

`--backend torch` is a deterministic complex128 statevector mirror for
development.  The standard profile uses 6 qubits, depth 5, floating boundary
handling, Adam followed by scaled L-BFGS, and the parameters recorded in
`config/pra_dqc_v0.json`.

## Package map

```text
pyqpanda-algorithm/       OriginQ-compatible DQC core and pyqpanda3 statevector
example/DQC/v0_runner.py  shared training, native parity and classical audit
example/DQC/run_*.py      one entry point per equation and a sequential main
notebooks/                 Jupytext Python and paired .ipynb notebook
docs/                      Markdown and LaTeX/PDF library description
figures/latest/            figures used by the library description
results/runs/              retained formal and review result archives (NPZ/CSV)
tests/test_v0.py           lightweight release checks
```

The notebook defaults to loading the archived evidence.  Set
`RUN_EXPERIMENTS=True` in `notebooks/PRA_DQC_V0.py` for a small smoke run.

The local QA pass is recorded in `VERSION_MANIFEST.json`: five focused tests
passed, and all three cases completed in both native and Torch smoke profiles.
The smoke archives use `review_native_v0_20260905_*` and
`review_torch_v0_20260905_*` run IDs and do not replace the retained formal
records below.

## Current evidence

| Case | Aggregate RMSE | Maximum absolute error | Classical check |
| --- | ---: | ---: | --- |
| lambda=20 damped oscillation | 4.772e-5 | 1.653e-4 | analytic solution and RK4 |
| coupled linear system | 1.316e-5 | 2.550e-5 | analytic matrix solution and RK4 |
| Riccati nonlinear equation | 3.058e-3 | 6.508e-3 | DOP853 and RK4 |

These are retained, fixed-seed local records.  They demonstrate a working
statevector DQC implementation, not a claim of quantum speedup or hardware
advantage.  The upload-ready library description, equations, usage contract
and figure links are in
[`docs/PRA_DQC_V0_GitHub_Library_Description.md`](docs/PRA_DQC_V0_GitHub_Library_Description.md).

The same description is available as editable LaTeX source
[`PRA_DQC_V0_GitHub_Library_Description.tex`](docs/PRA_DQC_V0_GitHub_Library_Description.tex)
and compiled
[`PDF`](docs/PRA_DQC_V0_GitHub_Library_Description.pdf).

The coupled headline is a disclosed positive-scaled L-BFGS continuation; its
independent control archive (`results/runs/coupled_regular_s23_20260904`) has
aggregate RMSE `2.673e-5`.

## Review and upload gate

The package is intentionally left local for review.  Inspect the figures,
metrics, source and notebook first.  No GitHub branch, remote repository or
cloud service was changed while preparing V0.

