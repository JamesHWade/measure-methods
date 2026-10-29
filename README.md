# measure-methods

Current development and delivery of the `measure_methods` Python package live in
[`measure-service`](https://github.com/JamesHWade/measure-service). The service
wheel includes the pure calculation package and its existing import interface.
The calculation package remains independent of MCP, storage, and R.

This repository retains the original standalone distribution and source history
for historical installations and calculation provenance. New service evaluations
identify the installed `measure-service` distribution; stored evaluations that
identify `measure-methods` retain their original metadata.

`measure-methods` is a small, dependency-free Python package for numerical
kernels used in analytical method resource accounting. It owns the calculations;
applications can own their input schemas, normalization, feasibility checks,
storage, and user-facing workflows.

The first kernel integrates solvent use while flow and solvent composition each
vary linearly over a time segment. Inputs use canonical units: duration in minutes
and flow in millilitres per minute. The returned solvent use is in millilitres.

```python
from measure_methods import program_solvent_use

use_ml = program_solvent_use(
    times_min=[0, 2],
    flows_ml_min=[1, 3],
    solvent_fractions=[{"A": 0.2, "B": 0.8}, {"A": 0.8, "B": 0.2}],
)
# {"A": 2.2, "B": 1.8}
```

`program_solvent_use()` integrates a complete sequence of method boundary samples.
`linear_solvent_use()` integrates one segment for a complete mixture, and
`constant_solvent_use()` handles a constant-flow, constant-composition phase.
`linear_product_volume()` exposes the single-solvent segment calculation. Numeric
values must be finite real scalars; booleans and strings are rejected. Durations
and flows must be nonnegative; fractions must be between zero and one. Each mixture
must have nonempty solvent names and sum to one within an absolute tolerance of
`1e-9`. Zero duration and zero flow are valid and produce zero use.

The kernel uses Simpson's rule, which is exact for the quadratic product of two
linear functions (apart from floating-point rounding). The linear path preserves
the prior Simpson evaluation order; the constant path preserves
`duration * (flow * fraction)`. If an intermediate operation would underflow or
overflow, the package retries with exact rational arithmetic over the input floats.
A positive result too small to round to a nonzero float, or a result too large for
a finite float, raises `ValueError`. Results are IEEE-754 double-precision floats,
so they still carry ordinary floating-point rounding limits.
If a program contains an underflowed or subnormal segment, its per-solvent total is
accumulated exactly before the final float conversion.

This package does not convert units, normalize incomplete mixtures, decide whether
a method is feasible, or persist results. Callers must convert quantities to canonical units before calculation.

## Install the current package

With Git and Python 3.10 or later available, install the service distribution:

```bash
python -m pip install "git+https://github.com/JamesHWade/measure-service.git"
```

The example continues to import `program_solvent_use` from `measure_methods`.
The combined distribution installs the service's Python
dependencies; the calculation package itself uses only the standard library.

## Upgrade an existing environment

The historical and combined distributions use the same `measure_methods` import
path. Remove the standalone distribution before force-reinstalling the combined
one in the environment used by the service:

```bash
python -m pip uninstall -y measure-methods
python -m pip install --force-reinstall "git+https://github.com/JamesHWade/measure-service.git"
```

A same-version install can be skipped by pip. Removing the old distribution
after installing the combined wheel can also remove shared package files;
force-reinstalling the combined distribution restores its files. Existing
calculation records retain their original provenance through this change.

## Historical standalone installation

The original standalone distribution remains installable from this repository:

```bash
python -m pip install "git+https://github.com/JamesHWade/measure-methods.git"
```

These commands follow the respective default branches. Installed source revisions
can be retained as calculation evidence without fixing future installations to a
commit. A package move does not rewrite the provenance of a retained calculation.

## Historical source checks

New numerical changes and their reference tests belong in the service repository.
To inspect or verify the standalone source retained here:

```bash
python -m pip install -e '.[dev]'
python -m ruff check .
python -m ruff format --check .
python -m pytest
python -m build
```
