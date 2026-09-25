# ninfer-3090

**Hyper optimised Qwen3.8-27B inference on one RTX 3090 (sm_86)**

Fork of [Neroued/ninfer](https://github.com/Neroued/ninfer) (⭐2389) with patches for RTX 3090 deployment.

## Patches included

| Patch | File | What |
|-------|------|------|
| 🔧 Model name relaxation | `src/serve/openai_common.cpp` | **llama.cpp-compatible**: accept any `model` name in API requests. Respond with the actually loaded model. No more 404 when agents send `model: "gpt-4"`. |
| 🔧 413 fix | `third_party/cpp-httplib/httplib.h` | Remove the hardcoded **8 KB form-urlencoded body limit**. JSON payloads sent via `curl --data-binary` were rejected with 413 regardless of `--max-request-mib`. |

## Upstream sources

This repo tracks the latest `Neroued/ninfer` master with the patches above cherry-picked on top. See `patches/` for individual diffs.

## Build

```powershell
$env:PATH = "D:\ninfer-tools\cmake-3.31.6-windows-x86_64\bin;C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v13.4\bin;" + $env:PATH
$env:CUDA_PATH = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v13.4"

cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE=<vcpkg>\scripts\buildsystems\vcpkg.cmake `
  -DVCPKG_TARGET_TRIPLET=x64-windows `
  -DCMAKE_CUDA_ARCHITECTURES=86

cmake --build build --config Release --target ninfer-serve --parallel
```

Requirements:
- VS 2022 Build Tools (MSVC C++)
- CUDA 12.8+ (this build: 13.4)
- CMake 3.28+
- NVIDIA driver ≥ 570
- vcpkg

## Apply patches (if updating from upstream)

```bash
git clone https://github.com/Neroued/ninfer.git
cd ninfer
for p in patches/*.patch; do git apply "$p"; done
```
