# Contributing to Python deltalake package

## Workflow

Most of the workflow is based on the `Makefile`, [`maturin`](https://www.maturin.rs/) CLI tool and [`uv`](https://docs.astral.sh/uv/).

### Setup your local environment with virtualenv

```bash
make setup
```

### Activate it

```bash
source ./.venv/bin/activate
```

### Ready to develop with maturin

[maturin](https://github.com/PyO3/maturin) is used to build the python package.

Install delta-rs in the current virtualenv:

```bash
make develop
```

Then, list all the available tasks

```bash
make help
```

Format:

```bash
make format
```

Check:

```bash
make check-python
```

Unit test:

```bash
make unit-test
```

## Release process

1. Make a new PR to update the version in `python/Cargo.toml`.
2. Once merged, push a tag of the format `python-vX.Y.Z`. This will trigger CI
   to create and publish release artifacts.
3. In GitHub, create a new release based on the new tag. For release notes,
   use the generator at a starting point, but please revise them for brevity.
   Remove anything that is dev-facing only (chores), and bring all important
   changes to the top, leaving less important changes (such as dependabot
   updates) at the bottom.
4. Once the artifacts are showing up in PyPI, announce the release in the delta-rs
   Slack channel. Be sure to give a shout-out to the new contributors.

### Windows wheels

The release workflow builds and publishes both x64 and ARM64 wheels automatically.
The GitHub-hosted ARM64 runner includes LLVM; no manual setup is needed for releases.

#### Local ARM64 build

Install native ARM64 Python 3.11+, `uv`, Rust, Visual Studio 2022 C++ ARM64 build
tools, a Windows SDK, and LLVM. Unlike x64, the ARM64 crypto dependencies require
Clang. Use stable Rust to match Windows CI and satisfy current dependency requirements.

From the repository root, in a developer PowerShell configured for ARM64
(adjust the LLVM path if installed elsewhere):

```powershell
rustup toolchain install stable
$env:RUSTUP_TOOLCHAIN = "stable"
rustup target add aarch64-pc-windows-msvc --toolchain stable
$env:PATH = "$env:ProgramFiles\LLVM\bin;$env:PATH"
$env:CC_aarch64_pc_windows_msvc = "clang-cl"
$env:CXX_aarch64_pc_windows_msvc = "clang-cl"
uvx maturin build -m .\python\Cargo.toml --target aarch64-pc-windows-msvc --interpreter python --profile python-release-arm
```

The wheel is written to `target\wheels`. Install the `win_arm64` wheel with native
ARM64 Python; x64 Python under emulation needs the `win_amd64` wheel instead.
Installing a prebuilt wheel does not require Rust, Visual Studio, or LLVM.
