# ninfer-sm86

**CUDA 12.8+ required** | **sm_86** (RTX 3050–3090, A6000)

**NInfer fork for all NVIDIA Ampere (sm_86) GPUs — RTX 3050/3060/3070/3080/3090, and professional Ampere cards.**

Based on [Don-Chad/ninfer-3090](https://github.com/Don-Chad/ninfer-3090) v0.6.1, which itself is a fork of [Neroued/ninfer](https://github.com/Neroued/ninfer) (⭐2389).

NInfer is a C++20/CUDA inference engine optimized for **Qwen3.8-27B** on a single 24 GB GPU. This fork adapts it for the Ampere architecture and adds quality-of-life improvements.

---

## What this fork fixes & adds

| # | Improvement | Affected file(s) |
|---|------------|-----------------|
| 1 | **Accept any model name** (llama.cpp style). Clients can send any `model` field; the server always responds with whatever model is loaded. | `src/serve/http_server.cpp` |
| 2 | **Remove 8 KB form-urlencoded limit**. API agents that POST large JSON payloads no longer get a 413 error. | `third_party/cpp-httplib/httplib.h` |
| 3 | **Custom Jinja2 chat templates**. Use any Jinja2 template instead of the two hardcoded built-in templates. | `third_party/llama-jinja/`, `src/text/jinja.cpp`, `src/targets/.../chat_template.cpp` |
| 4 | **`--chat-template` CLI option**. Point to a custom `.jinja` file at startup. | `src/serve/serve_options.cpp` |
| 5 | **Backported llama-jinja engine**. Full Jinja2 parser/lexer/runtime from llama.cpp. | `third_party/llama-jinja/` |
| 6 | **MSVC compatibility patches**. `localtime_s`, `_wgetenv`, `constexpr dim3` fixes for Windows builds. | 5 files across `src/core/`, `src/ops/`, `third_party/` |
| 7 | **Windows CI pipeline**. Automated build-and-release with VS2022 + CUDA 12.8. | `.github/workflows/build-windows.yml` |
| 8 | **sm_86 CMake guard removed**. The build now accepts `86` (RTX 3090) alongside the upstream default `120a` (RTX 5090). | `CMakeLists.txt` |

---

## Supported sm_86 GPUs

All NVIDIA Ampere GPUs with compute capability **sm_86**:

| GPU | VRAM | Notes |
|-----|:----:|-------|
| RTX 3090 / 3090 Ti | 24 GB | Primary target, fully validated |
| RTX A6000 (Ampere) | 48 GB | ✅ Fully compatible, higher VRAM |
| RTX 3080 / 3080 Ti | 10-12 GB | Works; smaller model configs recommended |
| RTX 3070 / 3070 Ti | 8 GB | Limited context, use compact artifacts |
| RTX 3060 / 3060 Ti | 8-12 GB | Limited context |
| RTX 3050 | 8 GB | Not recommended for Qwen3.8-27B |
| RTX A4000 / A5000 | 16-24 GB | Professional cards, should work |

---

## Downloads

Each release provides two archive variants:

| Archive | CUDA Runtime | Best for |
|---------|:------------:|----------|
| `ninfer-sm86-cuda12.zip` | CUDA 12.8+ | Users with existing CUDA 12.x setup |
| `ninfer-sm86-cuda13.zip` | CUDA 13.x | Users on newer drivers or CUDA 13.x |

Both archives contain identical binaries compiled for **sm_86** (Ampere). Choose the archive whose CUDA major version matches your system. The CUDA 13 variant uses MSVC's conforming preprocessor.

---

## Quick start

### Windows

Download the latest release archive from [Releases](https://github.com/doha-230/ninfer-sm86/releases).

```powershell
Expand-Archive ninfer-sm86-windows.zip
cd ninfer-sm86-windows

# Run with default template
.\ninfer-serve.exe model.ninfer

# Run with custom chat template
.\ninfer-serve.exe model.ninfer --chat-template my_template.jinja
```

### Linux (Docker)

```bash
docker run --rm -v $PWD:/src nvidia/cuda:12.6.3-devel-ubuntu22.04 bash -c '
  apt-get update && apt-get install -y cmake ninja-build pkg-config \
    libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libcurl4-openssl-dev
  cmake -S /src -B /build -G Ninja \
    -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_ARCHITECTURES=86 -DNINFER_BUILD_APPS=ON
  cmake --build /build --parallel --target ninfer ninfer-serve
'
```

### Build from source (Windows)

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE="<vcpkg>\scripts\buildsystems\vcpkg.cmake" `
  -DVCPKG_TARGET_TRIPLET=x64-windows `
  -DCMAKE_CUDA_ARCHITECTURES=86

cmake --build build --config Release --parallel
```

> **Requires: Visual Studio 2022, CUDA 12.8+, vcpkg**

---

## Unavailable features

The following require Blackwell (sm_120a) hardware and are unavailable on sm_86:
- NVFP4 W4A4 TMA tensor-core execution
- FP8 A8 tensor-core execution
- FP8 KV-cache profile

FP8 and NVFP4 **weight loading** (dequantizing to A16) still works; only the native tensor-core execution paths are exclusive to sm_120a.

---

## Upstream

This repository is a fork of [Don-Chad/ninfer-3090](https://github.com/Don-Chad/ninfer-3090) (release v0.6.1-rtx3090), which is itself a fork of [Neroued/ninfer](https://github.com/Neroued/ninfer) (⭐2389).

Patches are maintained in the `patches/` directory for reference.