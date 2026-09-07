# Differentiable Quantum ODE Solver — State-Vector Validation

This release is intentionally small. The executable content is exactly one
Python file and one Jupyter Notebook; neither imports the historical project
tree, archived result folders, or local helper modules.

## Files

- `differentiable_ode_solver.py` is the complete command-line implementation.
  Its `auto` backend selects the Torch state-vector path when PyTorch is
  available and otherwise uses the dependency-light NumPy state-vector
  simulator. If `pyqpanda3` is available, `--backend pyqpanda3` evaluates the
  same circuit with OriginQ's `CPUQVM`.
- `differentiable_ode_solver.ipynb` is an executable companion. It explains the
  construction in Markdown, runs all three examples, prints the error table,
  and draws the comparison figure.

The README is repository metadata only; the two files above are the complete
reproducible solver deliverables.

## Run the Python file

The default path needs NumPy. Matplotlib is needed only for a saved figure.

```bash
python -m pip install numpy matplotlib
python differentiable_ode_solver.py --plot
```

The command trains the three cases using ODE residuals and the initial value,
then reports the post-training comparison with a classical reference. A single
case or a shorter smoke run can be selected as follows:

```bash
python differentiable_ode_solver.py --case coupled --steps 20 --points 16
python differentiable_ode_solver.py --backend pyqpanda3 --case lambda20
```

The native command requires a working `pyqpanda3` installation. It does not
require an API key, a cloud job, or a real quantum processor.

## Run the Notebook

Open `differentiable_ode_solver.ipynb` in Jupyter and execute all cells. The
Notebook searches for the sibling Python file, loads its definitions, and then
executes a deterministic local state-vector run. It therefore starts from a
clean checkout without the original multi-directory package.

## Method

For each output component, the circuit expectation is denoted by
\(f_\theta(x)\). The floating initial-value transform is

\[
\widehat u_\theta(x)=u_0+f_\theta(x)-f_\theta(x_0),
\]

so the boundary value is satisfied by construction. The input is encoded by a
Chebyshev tower,

\[
\phi_q(x)=2(q+1)\arccos(x), \qquad q=0,\ldots,n-1,
\]

followed by `RZ-RX-RZ` rotations and a nearest-neighbour CNOT chain. The
observable is the total \(Z\)-magnetization. The coordinate derivative is
obtained from the RY parameter-shift identity, not from a finite-difference
grid:

\[
\partial_x f_\theta(x)=\sum_q \frac{\partial\phi_q}{\partial x}
\frac{f_\theta(\phi_q+\pi/2)-f_\theta(\phi_q-\pi/2)}{2}.
\]

With collocation coordinates \(x_j\), training minimizes

\[
\mathcal L(\theta)=\frac{1}{N}\sum_j
\left\|R\!\left(x_j,\widehat{\boldsymbol u}_\theta(x_j),
\partial_x\widehat{\boldsymbol u}_\theta(x_j)\right)\right\|_2^2,
\]

where \(R\) is the equation residual. The classical trajectory is not used
inside this loss. It is generated after training for an independent audit.

The included equations are:

1. Damped rotation: \(\boldsymbol u'=\begin{bmatrix}-2&-20\\20&-2\end{bmatrix}\boldsymbol u\), \(\boldsymbol u(0)=(1,0)\).
2. Coupled linear system: \(\boldsymbol u'=\begin{bmatrix}3&5\\-5&-3\end{bmatrix}\boldsymbol u\), \(\boldsymbol u(0)=(0.5,0)\).
3. Nonlinear Riccati equation: \(u'-4u+6u^2-\sin(50x)-u\cos(25x)+1/2=0\), \(u(0)=0.75\).

The first two references are closed forms. The Riccati reference is produced
with an independent fixed-step RK4 integrator. Metrics include residual RMS,
aggregate RMSE, component RMSE, and maximum absolute error.

## OriginQ integration

The file has no imports from `pyqpanda_alg` and can be copied into an OriginQ
algorithm repository as a standalone example. The optional `pyqpanda3` branch
uses `CPUQVM`, `VQCircuit`, `RY`, `RZ`, `RX`, and `CNOT` with the current QPanda3
calling convention. The NumPy branch is useful for a dependency-light review;
the two branches share the same feature map, ansatz, observable, and
parameter-shift derivative.

## References

[1] O. Kyriienko, A. E. Paine, and V. E. Elfving, “Solving nonlinear
differential equations with differentiable quantum circuits,” *Physical Review
A* **103**, 052416 (2021). DOI: [10.1103/PhysRevA.103.052416](https://doi.org/10.1103/PhysRevA.103.052416).

[2] OriginQ, “QPanda3 documentation,” public documentation repository,
[github.com/OriginQ/QPanda3-doc](https://github.com/OriginQ/QPanda3-doc).

[3] A. Kandala *et al.*, “Hardware-efficient variational quantum eigensolver
for small molecules and quantum magnets,” *Nature* **549**, 242–246 (2017).
DOI: [10.1038/nature23879](https://doi.org/10.1038/nature23879).

