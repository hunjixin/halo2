# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to Rust's notion of
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0-beta.3] - 2022-03-22
### Added
- `halo2_proofs::circuit`:
  - `AssignedCell::<Assigned<F>, F>::evaluate -> AssignedCell<F, F>`
  - `Assigned::{is_zero_vartime, double, square, cube}`
  - Various trait impls for `Assigned<F>`:
    - `From<&Assigned<F>>`
    - `PartialEq, Eq`
    - `Add<&Assigned<F>>, AddAssign, AddAssign<&Assigned<F>>`
    - `Sub<&Assigned<F>>, SubAssign, SubAssign<&Assigned<F>>`
    - `Mul<&Assigned<F>>, MulAssign, MulAssign<&Assigned<F>>`

### Removed
- `halo2_proofs::plonk::VerifyingKey::{read, write}` (for details see
  [issue 449](https://github.com/zcash/halo2/issues/449))

## [0.1.0-beta.2] - 2022-02-14
(relative to `halo2 0.1.0-beta.1`)

### Added
- `halo2_proofs::circuit::AssignedCell`, an abstraction for typed `Cell`s that
  track the type (and witnessed value if known) of the assignment.
- `halo2_proofs::plonk`:
  - `VerificationStrategy`
  - `SingleVerifier`, an implementation of `VerificationStrategy` for verifying
    proofs individually.
  - `BatchVerifier`, an implementation of `VerificationStrategy` for verifying
    multiple proofs in a batch.
  - `Column::column_type`
  - `impl {PartialOrd, Ord} for Any`
  - `Error::ColumnNotInPermutation`
- `halo2_proofs::poly::Basis: Copy` bound, and corresponding implementations for
  the provided bases.
- `halo2_proofs::dev`:
  - `FailureLocation` (used in `VerifyFailure::Lookup`)
  - `metadata::VirtualCell` (used in `VerifyFailure::ConstraintNotSatisfied`)
  - `impl From<(usize, &str)> for metadata::Region`

### Fixed
- `halo2_proofs::plonk::Assigned` addition was producing incorrect results in
  some cases due to how the deferred representation of `inv0` was handled. This
  could not cause a soundness error, because `Assigned` is only used during
  witness generation, not when defining constraints. However, it did mean that
  the prover would fail to create a valid proof for some subset of valid
  witnesses. [Fixed in #423](https://github.com/zcash/halo2/issues/423).

### Changed
- Migrated to `rand_core` (instead of `rand`), `pasta_curves 0.3`.
- `halo2_proofs::circuit`:
  - `Region` now returns `AssignedCell` instead of `Cell` or `(Cell, Option<F>)`
    from its assignment APIs, and the result types `VR` of their value closures
    now have the bound `for<'vr> Assigned<F>: From<&'vr VR>` instead of
    `VR: Into<Assigned<F>>`:
    - `assign_advice`
    - `assign_advice_from_constant`
    - `assign_advice_from_instance`
    - `assign_fixed`
- `halo2_proofs::plonk`:
  - `create_proof` now take an argument `R: rand_core::RngCore`.
  - `verify_proof` now takes a `VerificationStrategy` instead of an `MSM`
    directly, and returns `VerificationStrategy::Output` instead of `Guard`.
  - `ConstraintSystem::enable_equality` and `ConstraintSystem::query_any` now
    take `Into<Column<Any>>` instead of `Column<Any>` as a parameter to avoid
    excesive `.into()` usage.
  - `Error` has been overhauled:
    - `Error` now implements `std::fmt::Display` and `std::error::Error`.
    - `Error` no longer implements `PartialEq`. Tests can check for specific
      error cases with `assert!(matches!(..))`, or the `assert_matches` crate.
    - `Error::IncompatibleParams` is now `Error::InvalidInstances`.
    - `Error::NotEnoughRowsAvailable` now stores the current value of `k`.
    - `Error::OpeningError` is now `Error::Opening`.
    - `Error::SynthesisError` is now `Error::Synthesis`.
    - `Error::TranscriptError` is now `Error::Transcript`, and stores the
      underlying `io::Error`.
- `halo2_proofs::poly`:
  - `commitment::Accumulator` had its `challenges_packed` field renamed to
    `u_packed`.
  - `commitment::Guard`, returned by the closure passed into
    `VerificationStrategy::process` (and previously returned from `verify_proof`
    directly), has changed so that values returned from its method `compute_g`
    and expected by its method `use_g` are **NOT backwards compatible** with
    values in previous version (namely `halo2 0.1.0-beta.1`).
  - `commitment::MSM::add_to_h_scalar` was renamed to `MSM::add_to_w_scalar`.
  - `commitment::create_proof` now take an argument `R: rand_core::RngCore`.
  - `multiopen::create_proof` now take an argument `R: rand_core::RngCore`.
- `halo2_proofs::dev`:
  - `CircuitLayout::render` now takes `k` as a `u32`, matching the regular
    parameter APIs.
  - `VerifyFailure` has been overhauled:
    - `VerifyFailure::Cell` has been renamed to `VerifyFailure::CellNotAssigned`.
    - `VerifyFailure::ConstraintNotSatisfied` now has a `cell_values` field,
      storing the values of the cells used in the unsatisfied constraint.
    - The `row` fields of `VerifyFailure::{ConstraintNotSatisfied, Lookup}` have
      been replaced by `location` fields, which can now indicate whether the
      location falls within an assigned region.

### Removed
- `halo2_proofs::arithmetic`:
  - `BatchInvert` (use `ff::BatchInvert` instead).
  - Several parts of the `pasta_curves::arithmetic` API that were re-exported
    here (see the changelog for `pasta_curves 0.3.0` for details).
- `halo2_proofs::poly`:
  - `EvaluationDomain::{add_extended, sub_extended, mul_extended}`
  - `Polynomial::one_minus`
  - `impl Neg, Sub for Polynomial`
  - `impl Mul for Polynomial<_, ExtendedLagrangeCoeff>`
  - `impl Default for Rotation` (use `Rotation::cur()` instead).