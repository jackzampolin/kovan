# Lean proofs of kovan

## What this is, in plain words

Lean is a program that checks proofs. We write down a fact about a small, exact model of one
piece of kovan ("after this happens, that can never happen"), then a proof of it, step by
step, and Lean checks every step. If one step does not follow from the ones before, the build
fails. A test tries some inputs; a checked proof covers every input its statement allows, of
any size.

The models here are sequential: one thread doing one operation at a time. That suits facts
about arithmetic and bookkeeping, such as a threshold or a bound on how much work any sequence
of operations can cause. How threads interleave is checked by the TLA+ models in `../tla/`.

## What is proved

Nothing yet. Each proof is a module under `Kovan/`, imported by `Kovan.lean`, and listed here
with what it proves.

## How to check it

The toolchain is pinned in `lean-toolchain` (Lean 4 v4.35.0, core Lean only: no Mathlib, no
other package). Install elan, the Lean toolchain manager, once:

```sh
curl -sSfL https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh \
  | sh -s -- -y --default-toolchain none
```

and put `~/.elan/bin` on `PATH`. Then, from the repository root:

```sh
bash proofs/lean/check.sh          # both checks below
bash proofs/lean/check.sh sorry    # fails if `sorry` or `admit` (a hole in a proof) appears
bash proofs/lean/check.sh build    # `lake build --wfail`: every proof, failing on any warning
```

The first build downloads the pinned Lean release. CI runs the same two checks
(`.github/workflows/lean.yml`), with an elan release pinned by its SHA-256.
