[![Test action](https://github.com/brndnmtthws/rust-action/actions/workflows/test.yml/badge.svg)](https://github.com/brndnmtthws/rust-action/actions/workflows/test.yml)

# All-in-one GitHub action for Rust projects

This Action provides a simple (but opinionated) way to handle Rust projects with GitHub Actions.
The goal of this Action is to make it relatively easy to setup a decent
workflow without a ton of boilerplate. This Action prefers convention over
configuration, but it does provide a few knobs you can tweak if you wish.

Out of the box features include:

- ⚡️ Caching for fast builds
- 🛠️ Most of the tools you need, including:
  - Tarpaulin
  - Clippy
  - Rustfmt
  - Sccache (not enabled by default)
- 😎 Good vibes

## Quickstart

```yaml
name: Build & test
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
env:
  CARGO_TERM_COLOR: always
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Setup Rust toolchain with caching
        uses: brndnmtthws/rust-action@v1
      - run: cargo build
      - run: cargo test
        env:
          RUST_BACKTRACE: 1
```

## About caching

By default, this action caches the Cargo registry, and the `target/` directory
produced by Cargo.

This action also includes [sccache](https://github.com/mozilla/sccache) for
caching build outputs, which provides another way to cache builds, instead of
caching `target/` and relying on Rust's [incremental
builds](https://blog.rust-lang.org/2016/09/08/incremental.html). The sccache
package is not installed by default, and its usage is not enabled, but it can be
enabled with `enable-sccache: 'true'`.

It's recommended that you benchmark caching with sccache versus target caching,
as the effectiveness of either will depend on your circumstances.

Additionally, Cargo's registry is cached which can speed up fetching of
packages from [crates.io](https://crates.io/).

You can disable all of the built-in caching if you choose, refer to the
[Disable all caching](#disable-all-caching) recipe.

## Inputs

| Input | Description | Default |
|---|---|---|
| `toolchain` | Specify a Rust toolchain. | `stable` |
| `components` | Rust components to install along with the toolchain. | `clippy, rustfmt` |
| `cargo-packages` | The default set of cargo packages to install | `cargo-tarpaulin` |
| `disable-cargo-registry-cache` | If set to 'true', the Cargo registry cache will not be enabled. | unset |
| `disable-cargo-target-cache` | If set to 'true', the Cargo target cache will not be enabled. | unset |
| `enable-sccache` | If set to 'true', sccache will be enabled. | unset |
| `target-cache-key` | The cache key to use for caching the target dirs. | `cargo-target-${{runner.os}}-${{ inputs.toolchain }}-${{ hashFiles('**/Cargo.lock', '**/Cargo.toml') }}` |
| `target-cache-restore-keys` | The cache restore keys to use for caching the target dirs. | <code>cargo-target-\${{runner.os}}-\${{ inputs.toolchain }}<br />cargo-target-\${{runner.os}}-</code> |
| `registry-cache-key` | The cache key to use for caching the cargo registry. | `cargo-registry-${{ inputs.toolchain }}-${{ hashFiles('**/Cargo.lock', '**/Cargo.toml') }}` |
| `registry-cache-restore-keys` | The cache restore keys to use for caching the cargo registry. | <code>cargo-registry-\${{ inputs.toolchain }}-<br />cargo-registry-</code> |
| `sccache-cache-key` | The cache key to use for caching sccache. | `sccache-${{runner.os}}-${{ inputs.toolchain }}-${{ hashFiles('**/Cargo.lock', '**/Cargo.toml') }}` |
| `sccache-cache-restore-keys` | The cache restore keys to use for caching sccache. | <code>sccache-\${{runner.os}}-\${{ inputs.toolchain }}-<br />sccache-\${{runner.os}}-</code> |

## Recipes

### The kitchen sink: build, test, lint with Clippy, check formatting with Rustfmt, run on Ubuntu, macOS, Windows, and multiple feature flags

<details>
  <summary>Show me the code</summary>

```yaml
name: Build & test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  CARGO_TERM_COLOR: always

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    strategy:
      matrix:
        rust-toolchain:
          - stable
          - beta
          - nightly
        features:
          - serde
          - default
        os:
          - ubuntu-latest
          - macos-latest
          - windows-latest
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Setup ${{ matrix.rust-toolchain }} Rust toolchain with caching
        uses: brndnmtthws/rust-action@v1
        with:
          toolchain: ${{ matrix.rust-toolchain }}
      - run: cargo build --features ${{ matrix.features }}
      - run: cargo test --features ${{ matrix.features }}
        env:
          RUST_BACKTRACE: 1
      - run: cargo fmt --all -- --check
      - run: cargo clippy --features ${{ matrix.features }} -- -D warnings
```

</details>

### Disable all caching

<details>
  <summary>Show me the code</summary>

```yaml
- uses: brndnmtthws/rust-action@v1
  with:
    disable-cargo-registry-cache: 'true'
    disable-cargo-target-cache: 'true'
    enable-sccache: 'false'
```

</details>

### Upload coverage report to codecov.io

<details>
  <summary>Show me the code</summary>

```yaml
name: Coverage

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    name: coverage
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Setup nightly Rust toolchain with caching
        uses: brndnmtthws/rust-action@v1
        with:
          toolchain: nightly
      - run: cargo tarpaulin --features nightly --out Xml
      - name: Upload to codecov.io
        uses: codecov/codecov-action@v3
        with:
          fail_ci_if_error: true
```

</details>
