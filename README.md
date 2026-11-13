# xnu-haydn

Apple's XNU (ios-sweet generation, xnu-8796 / iOS 16) built for the **Xiaomi
Redmi K40 Pro** (`haydn`, Qualcomm SM8350 / Snapdragon 888).

This repository exists for one reason: **the kernel can only be built on
macOS**, and the machine doing the porting is a Windows host. It carries the
patch series and a workflow that builds it on a public macOS runner.

## What is here

| Path | What |
| --- | --- |
| `xnu/patches/` | The XNU patch series, applied in order. Taken from [dipa-studio/ios-sweet](https://github.com/dipa-studio/ios-sweet) (BSD-2-Clause-Patent) |
| `.github/workflows/xnu.yml` | Builds the kernel on `macos-14` and publishes it as an artifact |

It is **not** a complete iOS port and it does not ship anything of Apple's. The
IPSW, the root filesystem and the boot shim are inputs the porter supplies.

## Why this and not the whole ios-sweet tree

The patch series is what makes XNU compile for a non-Apple ARM64 machine, and it
is board-specific in exactly three places (physical timer trapped by QHEE, UART
framebuffer console, panel geometry). Keeping it in its own repository means a
haydn change is a haydn diff, and the CI runs on every push to `xnu/patches/`.

## The build

```sh
gh workflow run xnu.yml -R inonyu/xnu-haydn
gh run download RUN_ID -R inonyu/xnu-haydn -n xnu-kernel -D xnu/out
```

Output: `kernel.development.*` (a Mach-O), plus `SHA256SUMS`.

The runner is `macos-14` on purpose: it ships Xcode 15.x, whose `iig` matches
the xnu-8796 IORPC ABI. Xcode 16's `iig` emits `IORPC.kernelContent`, which
this xnu does not have, and the build fails in IOKit's generated headers.

## Status

The series builds for `sweet` (Redmi Note 10 Pro, SM7150) on the upstream
repository. Retargeting it to `haydn` (SM8350) is the work in progress: the
board differences are recorded in
[`docs/hardware.md`](https://github.com/inonyu/haydn-device/blob/main/docs/hardware.md)
of the device repository.
