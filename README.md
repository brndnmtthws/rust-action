[![Build and test](https://github.com/brndnmtthws/rust-action/actions/workflows/build-and-test.yml/badge.svg)](https://github.com/brndnmtthws/rust-action/actions/workflows/build-and-test.yml)

# GitHub action for installing Cargo packages

This action provides a simple way to install
[Cargo packages](https://doc.rust-lang.org/cargo/commands/cargo-install.html)
using [`cargo binstall`](https://github.com/cargo-bins/cargo-binstall). `cargo
binstall` allows us to speed up our CI by using binary packages where
available, rather than compiling everything from source.

This is useful for installing tools such as
[Tarpaulin](https://github.com/xd009642/tarpaulin),
[sccache](https://github.com/mozilla/sccache), or any other Rust package that's
supported by `cargo binstall` in your GitHub Actions.

You can use this action in your workflow directly, or use the [all-in-one
action](https://github.com/brndnmtthws/rust-action) for all your Rust CI needs.
While this action is intended to be used with Rust projects, it's also useful
on its own for non-Rust projects (for example, to use sccache with C++).

## Inputs

| Input      | Description                                                                         | Example                                  |
| ---------- | ----------------------------------------------------------------------------------- | ---------------------------------------- |
| `packages` | A comma or whitespace separated list of packages to install using `cargo binstall`. | `cargo-watch cargo-tree cargo-tarpaulin` |

## Recipes

### Install cargo-tarpaulin and sccache

<details>
  <summary>Show me the code</summary>

```yaml
- uses: brndnmtthws/rust-action@v1
  with:
    packages: cargo-tarpaulin sccache
```

</details>
