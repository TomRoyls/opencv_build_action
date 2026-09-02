# opencv_build_action

GitHub Actions workflow that builds a custom, minimal **static OpenCV 5.0.0** for
Windows, macOS, iOS, Android, and Linux — no image/video codecs, no CUDA, no
language bindings. C++ only.

[![Build OpenCV](https://github.com/TomRoyls/opencv_build_action/actions/workflows/build-opencv.yml/badge.svg)](https://github.com/TomRoyls/opencv_build_action/actions/workflows/build-opencv.yml)

## Artifacts

| Artifact | Platform | Notes |
|---|---|---|
| `opencv-<ver>-windows-x64-mt` | Windows x64 (MSVC, `/MT`) | static CRT |
| `opencv-<ver>-windows-x64-md` | Windows x64 (MSVC, `/MD`) | dynamic CRT |
| `opencv-<ver>-apple-arm64-xcframework` | iOS 16+, iOS Simulator, macOS 14+ | static `opencv2.xcframework`, arm64 only |
| `opencv-<ver>-android-arm64-v8a` | Android (NDK 27, API 24+) | `ANDROID_STL=c++_static` |
| `opencv-<ver>-linux-x64` | Linux (Ubuntu 24.04) | requires glibc >= 2.39 |

All five are attached to each [release](../../releases).

## Module set

Built: `core imgproc flann geometry features calib stereo photo stitching video
objdetect ml xphoto` (names follow OpenCV 5.x: `features2d` → `features`,
`calib3d` → `calib`; `ml` and `xphoto` come from `opencv_contrib`; `aruco`
ships inside `objdetect`).

Excluded: `imgcodecs`, `videoio`, `highgui`, `dnn`, `ptcloud`, `ts`, all
bindings (Java/Python/ObjC/JS), `world`. Also off: CUDA, OpenCL, IPP, ITT,
Lapack, TBB, protobuf, and all bundled codec libraries. Eigen 3.4.0 is
vendored and enabled (`WITH_EIGEN=ON`).

## Usage

**Publish a release** — push a version tag, the tag name is the OpenCV version:

```bash
git tag 5.0.1 && git push origin 5.0.1        # builds 5.0.1, publishes release
git tag 5.0.1-r2 && git push origin 5.0.1-r2  # rebuild iteration of the same version
```

**Test build without releasing** — Actions → *Build OpenCV* → *Run workflow*
(optional `opencv_ref` input overrides the version).

## Consuming the libraries

- **Windows**: pick `mt` or `md` to match your application's CRT setting
  (`/MT` vs `/MD`), otherwise LNK2038.
- **Apple**: drop `opencv2.xcframework` into Xcode; minimum targets are
  iOS 16 / macOS 14 (set in the workflow).
- **Android**: link with `ANDROID_STL=c++_static`; static libs are in
  `sdk/native/staticlibs/arm64-v8a/` plus `3rdparty/libs/`. `find_package(OpenCV)`
  works from the installed CMake config on Windows/Linux/Android.
- **Linux**: build links against glibc >= 2.39 (Ubuntu 24.04 baseline).

## Customizing

Everything lives in [`.github/workflows/build-opencv.yml`](.github/workflows/build-opencv.yml):

- `BUILD_MODULES` — the module whitelist (deps are auto-resolved by OpenCV's
  `BUILD_LIST`).
- Per-job CMake flags — one shared block plus platform additions; note some
  flags are deliberately Windows-only (`WITH_PTHREADS_PF`, `WITH_OPENCL_D3D11_NV`).
- Deployment targets — `--iphoneos_deployment_target` / `--macosx_deployment_target`
  in the macOS job.
