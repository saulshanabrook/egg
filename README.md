# <img src="doc/egg.svg" alt="egg logo" height="40" align="left"> egg: egraphs good

[![Crates.io](https://img.shields.io/crates/v/egg.svg)](https://crates.io/crates/egg)
[![Released Docs.rs](https://img.shields.io/crates/v/egg?color=blue&label=docs)](https://docs.rs/egg/)
[![Main branch docs](https://img.shields.io/badge/docs-main-blue)](https://egraphs-good.github.io/egg/egg/)
[![Zulip](https://img.shields.io/badge/zulip-join%20chat-blue)](https://egraphs.zulipchat.com)

egg is a library for e-graphs and equality saturation.
It can be used to build program optimizers, synthesizers, verifiers, and more.
For more information, see these resources:

- [website](https://egraphs-good.github.io/)
- [tutorial](https://docs.rs/egg/latest/egg/tutorials/)
- [API docs](https://docs.rs/egg/)
- [POPL 2021 paper](https://doi.org/10.1145/3434304)

> Also check out the [egglog](https://github.com/egraphs-good/egglog) 
 system that provides an alternative approach to 
 equality saturation based on Datalog.
 It features a language-based design, incremental execution, and composable analyses.
 See also the [paper](//mwillsey.com/papers/egglog) and the [egglog web demo](https://egraphs-good.github.io/egglog).

Are you using egg?
Please cite using this [BibTeX citation](./CITATION.bib) and
 add your project to the 
 ["Awesome E-graphs" page](https://github.com/philzook58/awesome-egraphs)!

<!-- Check out the [egg web demo](https://egraphs-good.github.io/egg-web-demo) for some quick e-graph action! -->

## Using egg

Add `egg` to your `Cargo.toml` like this:
```toml
[dependencies]
egg = "0.11.0"
```

Make sure to compile with `--release` if you are measuring performance!

### Clock selection

The default `quanta` feature uses [Quanta](https://docs.rs/quanta/) for timing.
Its first clock read can spend up to 200 ms calibrating in each process, in
exchange for potentially cheaper subsequent reads. It can suit long-running
applications that read the clock frequently. For short-lived applications,
`std::time::Instant` avoids Quanta's calibration, but clock reads may cost more.
Both clocks support runner time limits and timing reports; benchmark your
platform and workload to choose between them.

In your existing `[dependencies.egg]` entry, keep the `version`, `git`, or `path`
setting and choose one of these feature configurations. To use Quanta explicitly:

```toml
[dependencies.egg]
# Keep your existing dependency source here.
default-features = false
features = ["quanta"] # also enables std
```

To use the standard-library clock:

```toml
[dependencies.egg]
# Keep your existing dependency source here.
default-features = false
features = ["std"]
```

Keep any other features you need in the same list, such as `"deterministic"` or
`"serde-1"`. Leaving default features enabled also selects Quanta. These clock
options require a revision containing this feature; the published 0.11.0 release
does not provide them.

Cargo combines features requested by all dependencies, so another dependency
enabling `egg`'s defaults or `quanta` feature will select Quanta again. Check the
resolved features with `cargo tree -e features -i egg`.

Browser WebAssembly needs Quanta for timing; the `wasm-bindgen` feature selects it.
The standard-library clock is unsupported on `wasm32-unknown-unknown`.
Without `std`, the existing no-op clock still disables time limits and reports
zero elapsed time.

## Developing

It's written in [Rust](https://www.rust-lang.org/).
Typically, you install Rust using [`rustup`](https://www.rust-lang.org/tools/install).

Run `cargo doc --open` to build and open the documentation in a browser.

Before committing/pushing, make sure to run `make`, 
 which runs all the tests and lints that CI will (including those under feature flags).
This requires the [`cbc`](https://projects.coin-or.org/Cbc) solver
 due to the `lp` feature.

### Tests

Running `cargo test` will run the tests.
Some tests may time out; try `cargo test --release` if that happens.

There are a couple interesting tests in the `tests` directory:

- `prop.rs` implements propositional logic and proves some simple
  theorems.
- `math.rs` implements real arithmetic, with a little bit of symbolic differentiation.
- `lambda.rs` implements a small lambda calculus, using `egg` as a partial evaluator.


### Benchmarking

To get a simple csv of the runtime of each test, you set the environment variable
`EGG_BENCH_CSV` to something to append a row per test to a csv.

Example:
```bash
EGG_BENCH_CSV=math.csv cargo test --test math --release -- --nocapture --test --test-threads=1
```

