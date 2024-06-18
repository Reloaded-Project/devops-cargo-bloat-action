# Cargo Bloat Action :rocket:

Analyze and track your Rust project binary size over time.

This action runs on every pull request and gives you a breakdown of your total binary size,
how much each crate contributes to that size, and a list of changes to your dependency tree.

**Table of Contents**

- [Cargo Bloat Action :rocket:](#cargo-bloat-action-rocket)
  - [Example Workflow](#example-workflow)
  - [Inputs](#inputs)
  - [How it works](#how-it-works)
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

name: bloat

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

* `token` (required) - GitHub token to use for creating comments.
* `by_function` - Display per-function bloat instead of per-crate bloat. Default is false.
* `bloat_args` - Custom arguments to pass to `cargo bloat`.
* `tree_args` - Custom arguments to pass to `cargo tree`.
* `exclude_packages` - Packages to exclude from running `cargo bloat` onto.
* `include_packages` - Packages to include from running `cargo bloat` onto.

## How it works

* This action uses `cargo bloat` and `cargo tree` commands to analyze the Rust binary.
* It compares the binary size between the current commit and a base commit (usually the base branch of the PR).
* The difference in binary size and dependency tree is posted as a comment on the Pull Request.

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
      bloat_args: "-Z build-std=std,panic_abort -Z build-std-features=panic_immediate_abort --target x86_64-unknown-linux-gnu --profile profile --crate-type cdylib"
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