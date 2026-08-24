# TDGFEM
# GPU-TDG — 1D Cosserat rod benchmark

Minimal reproducible code for the paper

> **GPU-Accelerated Implicit and Explicit Time-Discontinuous Galerkin Finite
> Element Methods for Large-Scale Transient Stress Wave Propagation in
> Cosserat Media**

The classic 1D Cosserat-rod benchmark (impact load, classical elasticity
`lc = 0`) is solved with four methods — the implicit and explicit
time-discontinuous Galerkin finite element method (TDGFEM), the implicit
Newmark method, and the central difference method (CDM, i.e. explicit
Newmark) — each with a CPU and a GPU variant, all under a **mixed-precision**
strategy (stiffness matrix in double, state vectors in single).

## Requirements

- MATLAB **R2024b**
- Parallel Computing Toolbox
- a CUDA-enabled **NVIDIA GPU** (for `useGPU = true`; CPU runs need no GPU)

## Structure

```
.
├── run_1D_benchmark.m        # minimal example: implicit vs explicit TDGFEM
├── README.md
├── src/                      # shared assembly / load / post-processing
└── methods/
    ├── implicit_TDGFEM/      # GMRES / BiCGSTAB / TFQMR (useGPU flag)
    ├── explicit_TDGFEM/      # predictor–corrector, 4 SpMV/step (useGPU flag)
    ├── Newmark/              # implicit Newmark, beta=0.25, gamma=0.5
    └── CDM/                  # central difference, beta=0, gamma=0.5
```

## Run

Minimal example (runs the implicit and explicit TDGFEM and compares the
axial-displacement wave):

```matlab
run_1D_benchmark
```

Each method individually (edit `useGPU = true/false` and, for the implicit
TDGFEM, `solver = 'GMRES' | 'BiCGSTAB' | 'TFQMR'`):

```matlab
cd methods/explicit_TDGFEM; run_explicit_TDGFEM   % or implicit_TDGFEM / Newmark / CDM
```

## Benchmark

- **Geometry**: `50 m × 1 m` thin strip (a 1D rod), 4-node bilinear
  quadrilaterals, `ndivx = 2000`, `ndivy = 40`.
- **Material**: isotropic Cosserat with `E = 10 GPa`, `ν = 0`, `ρ = 2500 kg/m³`,
  characteristic length `lc = 0` (classical elasticity).
- **Load**: compressive impact `F0 = -1000 N` at the right end (step load of
  duration `T_impact = 0.005 s`); the left end is fixed.
- **Time step**: `dt = 1e-5 s` for the explicit schemes (CFL-limited) and
  `dt = 1e-4 s` for the implicit schemes; total time `T_sim = 0.01 s`.
- **Stress**: the axial stress `σ_xx` is computed along the centerline as the
  sum of the top- and bottom-edge stresses (`nodeSS(:,8)` after nodal
  averaging), following the 1D convention used in the paper.

All four methods share the same mesh, material, boundary conditions and load,
so their results are directly comparable; they agree to within the numerical
dissipation of each time integrator.

## Mixed precision

The stiffness matrix `K` is assembled and stored in double precision (MATLAB
GPU sparse linear algebra requires double), while the state vectors
(displacement, velocity, acceleration) are kept in single precision. Matrix–
vector products use `single(K * double(x))`.
