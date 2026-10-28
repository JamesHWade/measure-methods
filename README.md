# measure-methods

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

## Install the development version

The initial implementation is in review and is not published on PyPI. With Git
and Python 3.10 or later available:

```bash
python -m pip install "git+https://github.com/JamesHWade/measure-methods.git@codex/linear-program-usage"
```

This follows the active branch HEAD. After the initial PR is merged, the temporary
branch override can be removed. Installed source revisions can be recorded as
calculation evidence without fixing future installations to a commit.

## Development

```bash
python -m pip install -e '.[dev]'
python -m ruff check .
python -m ruff format --check .
python -m pytest
python -m build
```
