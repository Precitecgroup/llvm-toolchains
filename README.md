# LLVM Toolchain Builder

This repository contains GitHub Actions workflows to build, compile, and package LLVM release toolchains and MemorySanitizer (MSan) libc++ builds for Linux and macOS.

These prebuilt packages are designed to be imported directly by Bazel using rules like `toolchains_llvm` in the `aos` repository.

## Architectures & Features Matrix

| Operating System | CPU Architecture | Release Arch Identifier | MSan libc++ Supported? |
| :--- | :--- | :--- | :--- |
| **Linux** (glibc ≥ 2.34) | `amd64` (x86_64) | `x86_64-linux-gnu-ubuntu-22.04` | Yes |
| **Linux** | `arm64` (aarch64) | `aarch64-linux-gnu` | Yes |
| **macOS** (Darwin) | `arm64` (aarch64) | `aarch64-apple-darwin` | No (unsupported by LLVM on macOS) |

---

## 🚀 GitHub Actions Workflow

We use a GitHub Actions workflow to build the target architectures in parallel, package them, and publish them to a tagged GitHub Release.

### How to trigger a build:
1. Go to the **Actions** tab of the GitHub repository.
2. Select the **Build LLVM Toolchains** workflow.
3. Click the **Run workflow** dropdown on the right.
4. Input the desired options:
   - **LLVM version to build**: e.g., `21.1.1` (defaults to `21.1.1` to match AOS target). This version is also used as the GitHub Release tag.
   - **Force a complete rebuild of Clang**: Check to ignore compiler caches and rebuild Clang/LLVM from source.
   - **Branch, tag, or commit ref of RealtimeRoboticsGroup/aos to retrieve the script**: The ref/branch to check out from the RealtimeRoboticsGroup/aos repo (defaults to `main`).
5. Click **Run workflow**.

During execution, the workflow will automatically create a GitHub Release tagged with the LLVM version (e.g., `21.1.1`) and upload the built `.tar.zst` packages from each platform directly as assets to it.

### Runner Requirements
To ensure builds complete successfully in a reasonable timeframe:
- **Linux amd64**: Runs on standard GitHub-hosted `ubuntu-22.04` runner.
- **Linux arm64**: Runs on native GitHub-hosted `ubuntu-24.04-arm` runner (or equivalent self-hosted).
- **macOS arm64**: Runs on standard GitHub-hosted `macos-14` (Apple Silicon M1) runner.

---

## 🏷️ Creating a New Release

To build a new version of the LLVM toolchains and publish it as a GitHub Release:

1. **Trigger the Workflow**:
   Trigger the **Build LLVM Toolchains** workflow via the **Actions** tab on GitHub (see instructions above). Provide the target LLVM version as the input (e.g., `21.1.1`).

2. **Automated Release Creation**:
   During the workflow run, the runner that completes first will automatically create a new Git tag and a GitHub Release named after the version tag (e.g., `21.1.1`).

3. **Asset Upload**:
   As each parallel platform build (Linux amd64, Linux arm64, macOS arm64) finishes, it compiles its target and uploads its `.tar.zst` packages directly to the release page.

4. **Reviewing the Release**:
   Once all three runner jobs complete, visit the **Releases** page of your repository. You should see the release tagged with the version containing all target packages (e.g. 5 assets total: 3 compiler toolchains and 2 MSan-enabled libc++ builds).

---

## 🛠️ Local Build Instructions

This workspace does not store a duplicate copy of the LLVM helper script. To run builds locally, use the script from your adjacent `aos` repository workspace.

### Prerequisites

Ensure you have the following packages installed on your build machine:
- `git`
- `cmake`
- `ninja`
- `clang`
- `clang++`
- `curl`
- `tar`
- `zstd`
- `ccache` (optional, but highly recommended to speed up compilation)

### Run commands

To build both Clang and the MSan libc++ variant on a Linux host (e.g. amd64):

```bash
../aos/tools/helpers/build_from_llvm.py \
  --llvm-version 21.1.1 \
  --clang \
  --libcxx msan \
  --tempdir /tmp/llvm-build \
  --force-redownload \
  --clang-force-rebuild
```

On macOS, running the same command will build the Clang release and automatically skip building MSan libc++ (printing a warning since MSan is unsupported on Darwin).

---

## 📦 Output Packages & Checksums

The generated build outputs are located on the GitHub Release page, or locally under `--tempdir` (or `/tmp/llvm-build` if using the example commands above):

- **Clang Release Toolchain**: `clang+llvm-<version>-<release_arch>.tar.zst`
- **MSan libc++**: `libcxx-msan-<version>-<release_arch>.tar.zst`

### Integrating with AOS

1. Calculate the SHA256 checksums of the output tarballs:
   ```bash
   shasum -a 256 /tmp/llvm-build/*.tar.zst
   ```
2. Upload the tarballs to your distribution source (if not using GitHub Release directly).
3. Update the hashes and URLs in the `aos` repository:
   - `MODULE.bazel`
   - `WORKSPACE`
