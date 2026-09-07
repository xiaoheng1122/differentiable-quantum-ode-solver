# Differentiable Quantum ODE Solver

This repository contains a compact implementation of a differentiable quantum
circuit (DQC) solver for first-order ordinary differential equations. Solution
components are represented by expectation values of a parameterized circuit,
the differential equation is enforced at collocation points, and a classical
trajectory is computed afterwards for an independent accuracy check.

The implementation is intended for local state-vector validation and for
adaptation as an example in the OriginQ quantum-algorithm library. It does not
submit cloud jobs, require an API key, or access a real quantum processor.

## Repository contents

The executable research material consists of two files:

- `differentiable_ode_solver.py` contains the complete command-line solver,
  including the state-vector circuit, derivative evaluation, residual losses,
  optimizers, classical references, metrics, and plotting utility.
- `differentiable_ode_solver.ipynb` presents the same workflow in executable
  Notebook form. Markdown cells introduce the equations and circuit, and code
  cells run the benchmarks and display the numerical comparison.

This README provides the project description, mathematical notation, usage
instructions, and bibliographic references.

## Mathematical formulation

For each output component, let $f_\theta(x)$ denote the expectation value of a
parameterized quantum circuit. The initial-value condition is imposed with the
floating construction

$$
\widehat{u}_\theta(x)=u_0+f_\theta(x)-f_\theta(x_0).
$$

The input coordinate is encoded by a Chebyshev tower. For qubit $q$,

$$
\phi_q(x)=2(q+1)\arccos(x), \qquad q=0,\ldots,n-1.
$$

The feature rotations are followed by trainable `RZ-RX-RZ` blocks and a
nearest-neighbour CNOT chain. The measured observable is the total
$Z$-magnetization. Since the feature gates are RY rotations, the coordinate
derivative is evaluated with the parameter-shift identity

$$
\partial_x f_\theta(x)=
\sum_q \frac{\partial\phi_q}{\partial x}
\frac{f_\theta(\phi_q+\pi/2)-f_\theta(\phi_q-\pi/2)}{2}.
$$

For collocation coordinates $x_j$, the optimizer minimizes the equation
residual rather than a solution label:

$$
\mathcal{L}(\theta)=\frac{1}{N}\sum_{j=1}^{N}
\left\|R\!\left(x_j,\widehat{\boldsymbol{u}}_\theta(x_j),
\partial_x\widehat{\boldsymbol{u}}_\theta(x_j)\right)\right\|_2^2.
$$

The classical reference is used only after optimization. This separation makes
the residual loss and the post-training accuracy audit explicit.

## Differential-equation benchmarks

Three small problems are included.

### Damped rotating mode

$$
\boldsymbol{u}'(x)=
\begin{bmatrix}-2 & -20\\ 20 & -2\end{bmatrix}\boldsymbol{u}(x),
\qquad \boldsymbol{u}(0)=\begin{bmatrix}1\\0\end{bmatrix}.
$$

The reference is $\bigl(e^{-2x}\cos(20x),e^{-2x}\sin(20x)\bigr)$.

### Coupled linear system

$$
\boldsymbol{u}'(x)=
\begin{bmatrix}3 & 5\\ -5 & -3\end{bmatrix}\boldsymbol{u}(x),
\qquad \boldsymbol{u}(0)=\begin{bmatrix}0.5\\0\end{bmatrix}.
$$

The reference is evaluated from the closed form of the matrix exponential.

### Nonlinear Riccati equation

$$
u'-4u+6u^2-\sin(50x)-u\cos(25x)+\tfrac12=0,
\qquad u(0)=0.75.
$$

Its independent reference is generated with a fixed-step fourth-order
Runge–Kutta integrator.

The reported metrics are the training residual, residual RMS on the holdout
grid, aggregate RMSE, component RMSE, and maximum absolute error.

## Backends and dependencies

The command-line option `--backend auto` selects the Torch state-vector path
when PyTorch is installed and otherwise uses the dependency-light NumPy path.
Both paths explicitly manipulate CPU state vectors and use the same feature
map, ansatz, observable, and coordinate derivative. The option
`--backend pyqpanda3` selects OriginQ's local `CPUQVM` implementation when
`pyqpanda3` is available.

The NumPy path requires Python and NumPy. Matplotlib is optional and is used
only for `--plot`; PyTorch and `pyqpanda3` are optional backends.

## Usage

Run all three examples from the repository directory:

```bash
python -m pip install numpy matplotlib
python differentiable_ode_solver.py --plot
```

Run one benchmark or shorten the local smoke test:

```bash
python differentiable_ode_solver.py --case coupled --steps 20 --points 16
python differentiable_ode_solver.py --backend pyqpanda3 --case lambda20
```

The Notebook can be opened in Jupyter and executed from top to bottom. It
locates the sibling Python file, runs a deterministic state-vector experiment,
prints the metric table, and draws the DQC/classical trajectories when
Matplotlib is installed.

## OriginQ integration

The solver has no imports from `pyqpanda_alg` or other project-local modules,
so it can be copied into an OriginQ algorithm repository as a standalone
example. The native branch follows the current QPanda3 calling convention for
`CPUQVM`, `VQCircuit`, `RY`, `RZ`, `RX`, and `CNOT`. The separation between the
backend interface and the residual definitions also leaves a clear extension
point for a future fake backend or hardware adapter.

## References

[1] O. Kyriienko, A. E. Paine, and V. E. Elfving, “Solving nonlinear
differential equations with differentiable quantum circuits,” *Physical Review
A* **103**, 052416 (2021). DOI:
[10.1103/PhysRevA.103.052416](https://doi.org/10.1103/PhysRevA.103.052416).

[2] OriginQ, “QPanda3 documentation,” public documentation repository,
[github.com/OriginQ/QPanda3-doc](https://github.com/OriginQ/QPanda3-doc).

[3] A. Kandala *et al.*, “Hardware-efficient variational quantum eigensolver
for small molecules and quantum magnets,” *Nature* **549**, 242–246 (2017).
DOI: [10.1038/nature23879](https://doi.org/10.1038/nature23879).

