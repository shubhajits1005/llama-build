# llama-build

Build for 32 bit device besed on llama.cpp 

​🚀 Running Llama.cpp on 32-bit ARM Android via Cloud Cross-Compilation
​📌 Project Overview
​Deploying modern Large Language Models (LLMs) on resource-constrained, legacy 32-bit ARM (armv7) Android devices presents significant hardware and software challenges. Due to severe RAM limitations, architecture mismatches, and Android kernel sandbox restrictions, direct native compilation on the target device often results in Out-Of-Memory (OOM) crashes or linker failures.
​This project demonstrates a robust, automated workflow to cross-compile llama.cpp into a fully standalone, static musl binary using GitHub Actions and Zig, enabling smooth local inference on 32-bit mobile hardware via Termux.
​🛠️ The Technical Challenges Solved
​Architecture Mismatch:
​The Problem: Modern development hosts (such as 64-bit aarch64 devices or x86_64 cloud runners) default to generating 64-bit binaries that throw an Exec format error on 32-bit targets.
​The Solution: Enforced strict target-specific cross-compilation flags (arm-linux-musleabihf).
​Android Kernel & Loader Limitations (not found errors):
​The Problem: Dynamically linked binaries fail on Android because the standard Linux dynamic linker path (/lib/ld-linux-armhf.so.3) does not exist in the Android userland.
​The Solution: Switched to musl libc combined with static linking to bundle all system dependencies directly into a single self-contained executable.
​Glibc Startup Crashes:
​The Problem: Standard GNU C Library (glibc) binaries trigger a Fatal glibc error: cannot get entropy for arc4random panic on startup when interacting with Android's system call interface.
​The Solution: Used Zig as a drop-in compiler toolchain targeting musl, bypassing glibc security checks entirely.
​⚙️ Automated Build Pipeline (GitHub Actions)
​The project utilizes GitHub Actions to automate the compilation process on high-performance cloud runners, producing a ready-to-use binary in under 90 seconds.

# Running Llama.cpp on 32-bit ARM Android via Cloud Cross-Compilation

A streamlined workflow to cross-compile **`llama.cpp`** into a standalone, static **`musl`** binary using **GitHub Actions** and **Zig**, enabling smooth local LLM inference on 32-bit ARM (`armv7`) Android devices via Termux.

---

## 🛠️ The Technical Challenges Solved
1. **Architecture Mismatch:** Avoids `Exec format error` by explicitly cross-compiling for 32-bit `armv7` targets from 64-bit/x86 hosts.
2. **Android Loader Limitations:** Bypasses missing dynamic linkers (`/lib/ld-linux-armhf.so.3`) using static `musl` linking.
3. **Glibc Startup Crashes:** Completely avoids `Fatal glibc error: cannot get entropy for arc4random` panics by using the Zig toolchain targeting `musl`.

---

## ⚙️ GitHub Actions Workflow (`.github/workflows/build.yml`)
```yaml
name: Build ARM32 Llama.cpp Musl Static

on: [workflow_dispatch]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          repository: ggml-org/llama.cpp
          ref: master

      - name: Set up Zig
        uses: mlugg/setup-zig@v1
        with:
          version: 0.13.0

      - name: Configure CMake with Zig (Musl ARMv7)
        run: |
          cmake -B build \
            -DCMAKE_SYSTEM_NAME=Linux \
            -DCMAKE_SYSTEM_PROCESSOR=armv7 \
            -DCMAKE_C_COMPILER="zig;cc;-target;arm-linux-musleabihf" \
            -DCMAKE_CXX_COMPILER="zig;c++;-target;arm-linux-musleabihf" \
            -DBUILD_SHARED_LIBS=OFF \
            -DLLAMA_BUILD_TESTS=OFF \
            -DGGML_OPENMP=OFF

      - name: Build Llama.cpp
        run: cmake --build build --config Release -j4

      - name: Upload Binary
        uses: actions/upload-artifact@v4
        with:
          name: llama-cli-armv7-musl
          path: build/bin/llama-cli
