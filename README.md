# Cargo Bloat Action :rocket:

Analyze and track your Rust project binary size over time.

This action runs on every pull request and gives you a breakdown of your total binary size,
how much each crate contributes to that size, and a list of changes to your dependency tree.

**Table of Contents**

- [Cargo Bloat Action :rocket:](#cargo-bloat-action-rocket)
  - [Example Workflow](#example-workflow)
  - [Inputs](#inputs)
  - [Tips](#tips)
  - [Why?](#why)
  - [Contribute](#contribute)
    - [Please Note](#please-note)

## Example Workflow

Check out [cargo-bloat-example][cargo-bload-example] for a full example project. 
For an example pull request, see https://github.com/Reloaded-Project/cargo-bloat-example/pull/1.

```yaml
on: # rebuild any PRs and main branch changes
  pull_request:
  push:
    branches:
      - master

name: Compare Binary Size

jobs:
  cargo_bloat:
    runs-on: ubuntu-latest
    steps:
      - name: Run cargo bloat
        uses: Reloaded-Project/devops-cargo-bloat-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          by_function: true
          tree_args: "--no-dev-dependencies"
```

## Inputs

| Input              | Description                                           | Required | Default  |
| ------------------ | ----------------------------------------------------- | -------- | -------- |
| `token`            | GitHub token to use for creating comments             | Yes      |          |
| `by-function`      | Display per-function bloat instead of per-crate bloat | No       | `false`  |
| `bloat-args`       | Custom arguments to pass to `cargo bloat`             | No       |          |
| `tree-args`        | Custom arguments to pass to `cargo tree`              | No       |          |
| `exclude-packages` | Packages to exclude from running `cargo bloat` onto   | No       |          |
| `include-packages` | Packages to include from running `cargo bloat` onto   | No       |          |
| `toolchain`        | The toolchain to use                                  | No       | `stable` |
| `target`           | Target triple to install for the Rust toolchain       | No       |          |
| `pre-script`       | Script to run before invoking `dist/index.js`         | No       |          |
| `pre-script-shell` | Shell to use for running the pre-script               | No       | `bash`   |
| `install-rust`     | Determines if Rust should be installed by the action  | No       | `true`   |

The `pre-script` should ideally be a `.sh`, `.ps1` file, or whatever is relevant for your shell of choice.

By default, this action will install Rust for you, but you can opt out of this by setting `install-rust` to `false`.

If you install rust through the script, any code inside the script, including `pre-script` will use
the toolchain defined in `toolchain`. After the action exits, the toolchain will be reverted to default.

## Tips

Sometimes you may want to further customize how the package is built. Suppose you want to measure
the size of a C library that's minimized for size with `no_std` and `panic = "abort"`.

Here's an example of how you can do that:

```yaml
steps:
  - name: Run cargo bloat
    env:
      RUSTFLAGS: "-C panic=abort -C lto=fat -C embed-bitcode=yes"
    uses: Reloaded-Project/devops-cargo-bloat-action@v1
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      bloat-args: "-Z build-std=std,panic_abort -Z build-std-features=panic_immediate_abort --target x86_64-unknown-linux-gnu --profile profile --crate-type cdylib"
      toolchain: nightly
      target: x86_64-unknown-linux-gnu
```

For completeness, I define the profile in `Cargo.toml` like this:

```toml
# Profile Build
[profile.profile]
inherits = "release"
strip = false # Release profile but with symbols, for cargo-bloat.

# Optimized Release Build
[profile.release]
codegen-units = 1
lto = true
strip = true # This is the default in newer Rust versions
panic = "abort"
```

## Why?

To me, keeping track of dependency size is important.

While it may not be the most important metric for everyone, there is a certain amount of pride
and satisfaction with knowing that you're doing a job as good as possible.

Even when not scaling down to the smallest possible size, I think these metrics are useful in a pull
request that modifies code, in order to have a full picture of how the changes impact the project.

Personally I use this to keep track of the size of the various components of the Reloaded3 modding\
framework, as I want to keep the system as scalable as possible.

## Contribute

All contributions are welcome! 

If you have any ideas for improvements or new features, please open an issue or submit a pull request.

When contributing, please follow these guidelines:

* Write clear, descriptive commit messages.
* If adding a new feature, please document in the readme if necessary.
* Fork the example repository and test your changes against your fork.

### Please Note

This is a fork of [wsxiaoys/cargo-bloat-action][wsxiaoys] which is a fork of the original [orf/cargo-bloat-action][orf].

Since the very original is unmaintained, I've decided to fork it for the time being and do the
bare minimum of maintenance to keep it working. However, I'm not a web dev, and I've never really
worked with TypeScript, Node or JavaScript before. So do keep that in mind. 

[wsxiaoys]: https://github.com/wsxiaoys/cargo-bloat-action
[orf]: https://github.com/orf/cargo-bloat-action
[cargo-bload-example]: https://github.com/Reloaded-Project/cargo-bloat-example