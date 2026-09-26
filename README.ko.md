# ninfer-sm86

**모든 NVIDIA Ampere (sm_86) GPU를 위한 NInfer 포크 — RTX 3050/3060/3070/3080/3090 및 전문가용 Ampere 카드**

[Don-Chad/ninfer-3090](https://github.com/Don-Chad/ninfer-3090) v0.6.1 기반 포크 (원본: [Neroued/ninfer](https://github.com/Neroued/ninfer) ⭐2389)

NInfer는 **Qwen3.8-27B**를 단일 24GB GPU에서 실행하도록 최적화된 C++20/CUDA 추론 엔진입니다. 이 포크는 Ampere 아키텍처에 맞게 조정하고 사용성 개선을 추가했습니다.

---

## 이 포크가 해결한 문제

| # | 개선 사항 | 적용 파일 |
|---|----------|----------|
| 1 | **모든 모델명 허용** (llama.cpp 방식). 클라이언트가 보내는 어떤 `model` 값이든 현재 로드된 모델로 응답합니다. | `src/serve/http_server.cpp` |
| 2 | **8KB form-urlencoded 제한 제거**. API 에이전트가 대용량 JSON을 POST해도 413 에러가 발생하지 않습니다. | `third_party/cpp-httplib/httplib.h` |
| 3 | **커스텀 Jinja2 채팅 템플릿**. 하드코딩된 2개 고정 템플릿 대신 원하는 Jinja2 템플릿을 사용할 수 있습니다. | `third_party/llama-jinja/`, `src/text/jinja.cpp`, `chat_template.cpp` |
| 4 | **`--chat-template` CLI 옵션**. 서버 시작 시 커스텀 `.jinja` 파일을 지정할 수 있습니다. | `src/serve/serve_options.cpp` |
| 5 | **llama-jinja 엔진 백포트**. llama.cpp의 완전한 Jinja2 파서/렉서/런타임이 포함되어 있습니다. | `third_party/llama-jinja/` |
| 6 | **MSVC 호환성 패치**. `localtime_s`, `_wgetenv`, `constexpr dim3` 등 Windows 빌드를 위한 5개 파일 수정. | `src/core/`, `src/ops/`, `third_party/` |
| 7 | **Windows CI 파이프라인**. VS2022 + CUDA 12.8 자동 빌드 및 릴리즈 워크플로우. | `.github/workflows/build-windows.yml` |
| 8 | **sm_86 CMake 가드 제거**. upstream 기본값 `120a`(RTX 5090) 외에 `86`(RTX 3090)을 허용합니다. | `CMakeLists.txt` |

---

## 지원 sm_86 GPU

**sm_86** 컴퓨트 기능을 가진 NVIDIA Ampere GPU 전부 지원:

| GPU | VRAM | 비고 |
|-----|:----:|------|
| RTX 3090 / 3090 Ti | 24 GB | ✅ 주 타겟, 완전 검증 완료 |
| RTX 3080 / 3080 Ti | 10-12 GB | ✅ 동작 확인; 소형 모델 설정 권장 |
| RTX 3070 / 3070 Ti | 8 GB | ⚠️ 제한된 컨텍스트, 경량 아티팩트 필요 |
| RTX 3060 / 3060 Ti | 8-12 GB | ⚠️ 제한된 컨텍스트 |
| RTX 3050 | 8 GB | ⚠️ Qwen3.8-27B에는 부적합 |
| RTX A4000 / A5000 | 16-24 GB | ✅ 전문가용 카드, 동작 예상 |

---

## 빠른 시작

### Windows

[Releases](https://github.com/doha-230/ninfer-sm86/releases)에서 최신 릴리즈 압축파일 다운로드

```powershell
Expand-Archive ninfer-sm86-windows.zip
cd ninfer-sm86-windows

# 기본 템플릿 실행
.\ninfer-serve.exe model.ninfer

# 커스텀 채팅 템플릿 실행
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

### 소스 빌드 (Windows)

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE="<vcpkg>\scripts\buildsystems\vcpkg.cmake" `
  -DVCPKG_TARGET_TRIPLET=x64-windows `
  -DCMAKE_CUDA_ARCHITECTURES=86

cmake --build build --config Release --parallel
```

> 필요: Visual Studio 2022, CUDA 12.8+, vcpkg

---

## 지원하지 않는 기능

다음은 Blackwell (sm_120a) 전용 하드웨어 기능으로 sm_86에서는 사용할 수 없습니다:
- NVFP4 W4A4 TMA 텐서코어 실행
- FP8 A8 텐서코어 실행
- FP8 KV-캐시 프로파일

FP8 및 NVFP4 **가중치 로딩**(A16 역양자화)은 여전히 동작하며, 텐서코어 실행 경로만 sm_120a 전용입니다.

---

## 업스트림

이 레포지토리는 [Don-Chad/ninfer-3090](https://github.com/Don-Chad/ninfer-3090) (release v0.6.1-rtx3090)의 포크이며, Don-Chad는 [Neroued/ninfer](https://github.com/Neroued/ninfer) (⭐2389)의 포크입니다.

패치 파일은 `patches/` 디렉토리에서 참조할 수 있습니다.