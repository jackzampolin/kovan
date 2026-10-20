# Proofs of kovan

## What is checked, in plain words

Many threads use kovan's data structures at the same time. A bug in that kind of code often
shows up only when the threads' steps happen in one particular order, and a test can run
millions of times without ever hitting that order. A proof does not run the code and hope: it
takes a precise description of the code and checks every case that description allows.

A small example of what that buys. In kovan-map 0.1.20, a remove from the hopscotch map did
its work in two steps: first it emptied the entry's slot, then it cleared the bit that tells
lookups "this slot holds an entry of this bucket". If an insert into the same bucket took the
freed slot between those two steps, it set that same bit, and the remove's second step then
cleared it. The new entry sat in the table where no lookup could ever find it. The model of
the map runs the remove and the insert in every possible order, finds that bad order, and
writes it down step by step (`tla/hopscotch/HS_mut_unguarded_remove_counterexample.txt`).
The code now holds the bucket's writer guard for the whole remove, and the same check finds
no bad order for it.

Two tools do the checking, and kovan uses them together. TLA+ looks at many threads at once,
but only for small sizes. Lean looks at one thread at a time, but for every size and every
sequence of operations.

What a check covers is said with it, and nothing beyond: a model checker run covers every
order of steps for the sizes its configuration sets (a few threads, keys and buckets), not
every size; a Lean proof covers every size, for its one-thread model of the code.

## The tools

| tool | what it checks in kovan | how far a check goes |
|------|-------------------------|----------------------|
| TLA+, checked by TLC (`tla/`) | concurrent interleavings: the memory reclamation protocol of `kovan`, and the atomic steps of both maps of `kovan-map` (one step of the model is one atomic step of the code) | every order of every thread's steps, for the threads, keys, buckets and programs each configuration sets |
| Lean 4 (`lean/`) | functional and arithmetic properties, such as thresholds and bounds on the work a sequence of operations causes | every input and every sequence of operations, for a one-thread model of the code |

## How to run them

TLA+: the models need Java on `PATH` (CI uses Java 21). The runner fetches the pinned TLA+
tools on first use.

```sh
bash proofs/tla/run_tlc.sh hopscotch    # one family: every configuration in proofs/tla/hopscotch
bash proofs/tla/run_tlc.sh all          # every family
```

A run compares each configuration's result with the family's `EXPECTED.txt` and exits non-zero
on any mismatch. CI runs `proofs/tla/run_tlc.sh all` (`.github/workflows/tla.yml`).
`proofs/tla/README.md` explains the models, their configurations and their findings.

Lean: install elan once (`proofs/lean/README.md` says how), then

```sh
bash proofs/lean/check.sh    # fails on a hole in a proof (sorry, admit), then builds every proof
```

CI runs the same (`.github/workflows/lean.yml`).

## Coverage

- `tla/reclaim/`: kovan's memory reclamation (pin, the protected load, retire, the traversal,
  flush, a thread's exit and the hand-over of its id), 69 configurations: 30 that must pass and
  39 mutations and witnesses that must fail.
- `tla/chained/`: `kovan_map::HashMap`, the chained map (the link words, the writes, the walks,
  the conditional writes, the freezing migration and clear), 44 configurations: 26 that must
  pass and 18 that must fail.
- `tla/hopscotch/`: `kovan_map::HopscotchMap` (the guarded insert and remove, displacement, the
  conditional writes, the lookup's rescan, resize, clear, the walk across a table swap), 47
  configurations: 28 that must pass and 19 that must fail.
