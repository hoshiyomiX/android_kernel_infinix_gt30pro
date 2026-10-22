# Infinix GT 30 Pro (X6873) — Android 15 Linux 6.1 Kernel Source

[![Kernel](https://img.shields.io/badge/kernel-Linux%206.1-blue)](https://android.googlesource.com/kernel/common/)
[![Android](https://img.shields.io/badge/Android-15-green)](https://source.android.com/docs/core/architecture/kernel/gki-overview)
[![SoC](https://img.shields.io/badge/SoC-MediaTek%20MT6897-orange)](https://www.mediatek.com/products/smartphones-2)
[![License](https://img.shields.io/badge/license-GPL--2.0-blueviolet)](LICENSE)

Linux kernel source for the **Infinix GT 30 Pro** — model **X6873**, MediaTek Dimensity 8350 Ultimate (MT6897), Android 15 release — imported byte-for-byte from the official Infinix Mobility open-source portal: <https://opensource.dev.infinixmobility.com>.

This repository carries the **one-chipset MT6897 platform source** — the device focus is the **Infinix GT 30 Pro (kernel project `x6873_h972`)**, carried by the default `X6873` branch (`main` holds the chipset baseline and is not the default view). The MT6897 release is a shared Transsion platform drop — per the portal it also covers the **Infinix NOTE 50 Pro+ 5G**, and the tree carries further MT6897 sibling projects (see [Platform sharing](#platform-sharing--mixed-device-support-under-one-chipset) below).

## Device information

| | |
|---|---|
| Device | **Infinix GT 30 Pro** — model **X6873** (kernel project `x6873_h972`) |
| SoC | MediaTek Dimensity 8350 Ultimate (**MT6897**), 4 nm |
| CPU | Octa-core — 4× Cortex-A715 up to 3.35 GHz · 4× Cortex-A510 up to 2.2 GHz |
| GPU | ARM Mali-G615 MC6 |
| Display | 6.78″ AMOLED, 1.5K (1224×2720), 60/90/120/144 Hz |
| Memory | LPDDR5X · UFS 4.0 |
| Kernel | Linux 6.1 — Android Generic Kernel Image (GKI) 2.0, base `android15-6.1` |
| Kernel type | GKI core (`gki_defconfig`) + MediaTek **MGK** device modules (`mgk_64_k61_defconfig`) |
| Android release | Android 15 |

## Platform sharing — mixed-device support under one chipset

This MT6897 release is a **shared platform tree**: one kernel + one device-module set, with each device defined by a devicetree **overlay** (`/plugin/;` DTBO) on the common `mt6897.dts` base. A single defconfig (`mgk_64_k61_defconfig`) builds the kernel; the target device is selected at build time by its project dtbo.

MT6897 projects present in this drop:

| Project | Family / role |
|---------|---------------|
| `x6873_h972` | **Infinix GT 30 Pro (X6873) — focus of this repository** |
| `x6856_h971` · `x6856b_h971` | Infinix NOTE 50 Pro+ 5G family (covered by the same drop per the portal) |
| `lj9_h972` · `cm8_h973` | other Transsion MT6897 sibling programs |
| `d8300evb_h973a` · `k6897v1_64` | MediaTek EVB / base boards |

The drop also carries base devicetrees for further MediaTek SoCs (MT6878, MT6985, MT6989) that ride the same shared modules tree, exactly as published by the vendor.

**One repository is the correct layout for this release.** Kernel source is chipset-scoped, not device-scoped: the devices above differ only by DTBO overlays and per-project `cust_*` dtsi fragments, all built from the same tree — the same model used by AOSP and LineageOS per-SoC kernel repositories. Splitting into per-device repositories would break the vendor's own two-package assembly (the `kernel/kernel-6.1 → ../kernel-6.1` symlink), duplicate gigabytes of shared platform code per device, and diverge from the official GPL drop. The GT 30 Pro stays this repository's focus through the default `X6873` device branch, this README, and the repository metadata.

## Source origin & integrity

Official two-part MT6897 drop (import date 2026-10-21 · tag [`android15-vendor-drop`](../../releases/tag/android15-vendor-drop)):

| File | Size | SHA-256 |
|------|------|---------|
| `android15_kernel_common_mt6897.tar.gz` | 228,549,822 B | `02d94d3662d102369587b78cba53f674ea201569f5d8c759d36d13c98af3ec17` |
| `android15_kernel_modules_and_devicetree_mt6897.tar.gz` | 1,789,217,399 B | `80f41374b876837414ce4ad36b40a727ad1e4fbaa3c04005b7ca3335144cbcd3` |

Download URLs (CloudFront, as published by the portal):

- https://d1wd32b69guoqm.cloudfront.net/opensource/android15_kernel_common_mt6897.tar.gz
- https://d1wd32b69guoqm.cloudfront.net/opensource/android15_kernel_modules_and_devicetree_mt6897.tar.gz

Import verification: every archive member was checked against the imported tree — **108,891 tracked entries, 0 missing**.

## Repository layout

The two packages assemble exactly as published: package 2 ships the symlink `kernel/kernel-6.1 → ../kernel-6.1` that pulls the GKI common source (package 1) into the MediaTek device workspace. The drop tree is preserved verbatim in this repository.

```text
.
├── kernel-6.1/      # GKI android15-6.1 common kernel (package 1) — 78,836 entries
│   ├── build.config.gki.aarch64 · gki_defconfig (arch/arm64/configs/)
│   └── android/abi_gki_aarch64_mtk          # MediaTek KMI symbol list
├── kernel/          # MediaTek MGK device workspace (package 2) — 19,601 entries
│   ├── WORKSPACE                            # → build/bazel_mgk_rules/kleaf/bazel.WORKSPACE
│   ├── kernel-6.1 → ../kernel-6.1           # assembly symlink to the GKI source
│   ├── kernel_device_modules-6.1/            # device modules + MT6897 devicetrees
│   │   ├── arch/arm64/configs/mgk_64_k61_defconfig
│   │   ├── arch/arm64/boot/dts/mediatek/   # mt6897.dts, x6873_h972 (GT 30 Pro), …
│   │   └── build.config.mtk.aarch64*        # mgk / gcc / kasan / ubsan variants
│   ├── build/       # MGK build system (build.sh · bazel_mgk_rules/)
│   ├── prebuilts/   # prebuilt toolchains & build tools (clang, gcc, JDK, kernel-build-tools)
│   ├── tools/       # bazel wrapper · mkbootimg
│   ├── configs/     # ABI/module config fragments (Android q–v)
│   ├── external/    # Bazel build dependencies (bazel-skylib, stardoc)
│   └── tests/
├── vendor/          # vendor/mediatek/kernel_modules — 10,453 entries
├── LICENSE          # GNU GPL-2.0 (full text)
└── README.md
```

The platform tree is shared across several MediaTek projects: devicetree bases for MT6878/MT6897/MT6985/MT6989 and the customer boards listed above are present. The MT6897 device base is `mt6897.dts` with the `mgk_64_k61_defconfig` configuration; the GT 30 Pro project overlay is `x6873_h972.dts`.

## Building

The drop is self-contained — prebuilt toolchains ship under `kernel/prebuilts/`.

Bazel/Kleaf (primary, as shipped by MediaTek):

```bash
cd kernel
tools/bazel build //kernel_device_modules-6.1:mgk_64_k61
# list all available targets (dist / ABI / sanitiser variants):
tools/bazel query //kernel_device_modules-6.1:*
```

A legacy `build.sh` path is also present (`kernel/build/build.sh` with the `build.config.mtk.aarch64*` family in `kernel/kernel_device_modules-6.1/`).

> The build definitions are the vendor's own (`kernel/build/bazel_mgk_rules/`). `mgk_64_k61` is the MT6897 project target; sibling variants (`mgk_64_aging_k61`, `mgk_64_kasan_k61`, `mgk_64_khwasan_k61`, `mgk_64_pkvm_k61`, `mgk_64_vulscan_k61`, …) are declared in `kernel/build/bazel_mgk_rules/mgk.bzl`.

## Branches & tags — single-commit history

| Ref | Meaning |
|-----|---------|
| `X6873` | **default branch** — Infinix GT 30 Pro (kernel project `x6873_h972`) **device branch**; the repository's landing view |
| `main` | MT6897 **chipset baseline** — the one-chipset platform source, kept as a non-default baseline ref |
| `android15-vendor-drop` | tag pinned to that same commit — marks the vendor release import |

History is kept as **exactly one commit** by repository policy: the pristine-import commit and the standardization commit were squashed into a single root commit. `X6873` and `main` point at that same single-commit tree — the shared MT6897 platform code — so the device focus is carried by the branch structure and this README, while device-specific work continues on `X6873`. The vendor tree itself — including the `kernel/kernel-6.1` assembly symlink — is preserved verbatim from the drop, and pristine provenance is guaranteed by the tarball SHA-256 checksums documented above.
## License

The Linux Kernel is provided under:

`SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note`

Being under the terms of the **GNU General Public License version 2 only**, according with the license text published on the Infinix open-source portal. The full license text is in [LICENSE](LICENSE) and in the kernel tree (`kernel-6.1/COPYING`, `kernel-6.1/LICENSES/`).

## Acknowledgements

- Kernel source © Transsion / Infinix — GPL-2.0 release via the [Infinix open-source portal](https://opensource.dev.infinixmobility.com)
- GKI base © Google / AOSP — [kernel/common](https://android.googlesource.com/kernel/common/) (`android15-6.1`)
- Linux kernel © Linus Torvalds and contributors
