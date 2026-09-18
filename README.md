# ninfer-serve 413 문제 수정 빌드

NInfer-3090 서버에서 이미지 요청이 413으로 거부되던 문제를 고쳐 다시 만든 ninfer-serve 빌드입니다.

## 처음에 겪은 문제

이미지를 넣어 요청을 보내면 `413 request_too_large`가 돌아왔습니다. `--max-request-mib`를 큰 값으로 올려도 달라지지 않았고, 텍스트만 있는 요청은 잘 됐습니다.

## 왜 그랬나

curl로 `-d`나 `--data-binary`를 쓰면 요청이 `Content-Type: application/x-www-form-urlencoded`로 보내집니다. ninfer에 포함된 cpp-httplib은 이 Content-Type의 요청에만, `--max-request-mib`와는 별개로 8KB짜리 하드 한도를 걸어뒀습니다. 서버는 Content-Type을 신경 쓰지 않고 body를 그냥 JSON으로 파싱하므로, JSON을 헤더 한 장 없이 보낸 요청이 8KB만 넘어도 413이 됐습니다.

거기에 서버가 413을 "request body exceeds the configured payload limit of N bytes"라고 출력하는데, 여기 N이 우리가 설정한 값 그대로 보여서 "옵션은 반영됐는데 왜 계속 안 되지" 싶은 상황이 됐습니다. 실제로 걸리는 한도는 8KB 고정이라 옵션과 상관이 없었습니다.

## 어떻게 고쳤나

`third_party/cpp-httplib/httplib.h`의 `Server::read_content`에서 form-urlencoded 전용 8KB 체크 분기를 지웠습니다. 이제 요청 body는 `--max-request-mib`로 정한 한 가지 한도로만 제한되고, 기존 curl 명령을 그대로 써도 통과합니다. 8KB보다 작았던 기존 요청들은 전과 똑같이 처리됩니다.

## 들어 있는 파일

- `ninfer-serve.exe` — 패치된 서버 (CUDA 13.4, MSVC 14.44로 빌드)
- `avformat-62.dll`, `avcodec-62.dll`, `avutil-60.dll`, `swscale-9.dll`, `swresample-6.dll` — FFmpeg 런타임
- `libcurl.dll`, `z.dll`

## 적용 방법

기존 실행 폴더의 `ninfer-serve.exe`와 DLL들을 이 파일들로 교체한 뒤 평소대로 서버를 띄우면 됩니다. 이미지를 보내던 요청도 그대로 보내면 됩니다.

## 직접 다시 빌드하려면

툴체인: VS 2022 Build Tools(MSVC C++), CUDA 12.8 이상(이 빌드는 13.4), CMake 3.28 이상, git, vcpkg.

```powershell
$env:PATH = "D:\ninfer-tools\cmake-3.31.6-windows-x86_64\bin;C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v13.4\bin;" + $env:PATH
$env:CUDA_PATH = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v13.4"

cmake -S <repo> -B build-windows -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE=<vcpkg>\scripts\buildsystems\vcpkg.cmake `
  -DVCPKG_TARGET_TRIPLET=x64-windows `
  -DCMAKE_CUDA_ARCHITECTURES=86

cmake --build build-windows --config Release --target ninfer-serve --parallel
```

vcpkg는 저장소의 `vcpkg.json`(curl, ffmpeg[zlib], pkgconf)을 따르고, `cd <repo>`에서 `vcpkg install --triplet x64-windows`로 먼저 받아두면 됩니다.

## 주의할 점

이 빌드는 CUDA 13.4로 컴파일해서 기존 v0.6.0(CUDA 12.8 기반)보다 높은 NVIDIA 드라이버가 필요합니다. 서버 GPU 드라이버가 570 미만이면 CUDA 초기화에서 실패할 수 있습니다. 그런 환경이면 드라이버를 올리거나 CUDA 12.8로 재빌드하세요.

## 아직 확인 못 한 것

이 머신에는 모델 파일(.ninfer)과 GPU가 없어서 실제 서버에서 413이 사라지는 것까진 확인하지 못했습니다. `--help`가 정상 출력되는 것과 빌드가 되는 것까지만 확인했습니다. 실서버에서 기존 curl을 다시 보내 거부가 없어지는지가 마지막 확인입니다.