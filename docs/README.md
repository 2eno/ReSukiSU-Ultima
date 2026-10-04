# ReSukiSU Ultima
<img align='right' src='ReSukiSU_blue.svg' width='220px' alt="ReSukiSU Ultima Icon">


**English** | [简体中文](./zh/README.md)

A based-on [`SukiSU-Ultra/SukiSU-Ultra`](https://github.com/SukiSU-Ultra/SukiSU-Ultra) fork, added some interesting changes, also make it more stable and build easily.

[![Latest release](https://img.shields.io/github/v/release/spacealtctrl/ReSukiSU-Ultima?label=Release&logo=github)](https://github.com/spacealtctrl/ReSukiSU-Ultima/releases/latest)
[![Kernel License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-orange.svg?logo=gnu)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![Other part License：GPL v3](https://img.shields.io/github/license/spacealtctrl/ReSukiSU-Ultima?logo=gnu)](/LICENSE)

## Features

1. Kernel-based `su` and root access management
2. Module system based on [metamodules](https://kernelsu.org/guide/metamodule.html): Pluggable infrastructure for systemless modifications.
3. [App Profile](https://kernelsu.org/guide/app-profile.html): Lock up the root power in a cage
4. Support non-GKI and GKI 1.0
5. KPM Support
6. Tweaks to the manager theme and the built-in susfs management tool.
7. Multi manager support, for default [Official KernelSU](https://github.com/tiann/KernelSU)/[RKSU](https://github.com/rsuntk/KernelSU)/[MKSU](https://github.com/5ec1cff/KernelSU)/[SukiSU](https://github.com/SukiSU-Ultra/SukiSU-Ultra) is supported work as manager with ReSukiSU Ultima's kernel
8. **Sentinel**: kernel-side root-probe detection with per-app cloaking and optional su-request notifications.
9. **Built-in Zygisk**: a self-contained, ptrace-based Zygisk implementation - run Zygisk modules without a separate Zygisk provider.

## Compatibility Status

- ReSukiSU Ultima officially supports Android GKI 2.0 devices (kernel 5.10+).

- Older kernels (3.4+) are also compatible, but the kernel will have to be built manually.

- Currently, only `arm64-v8a`, `armeabi-v7a` and `X86_64`are supported.

- [SuSFS](https://gitlab.com/simonpunk/susfs4ksu) in this project is **Only** support backport to kernel 4.3+

- `SuSFS Inline Hook` requires the kernel side of [SuSFS](https://gitlab.com/simonpunk/susfs4ksu) **v2.3.0** or newer, the build stops with older susfs patches

- `Tracepoint Syscall Redirect hook` is only support with GKI2(5.10+) kernel

## Hook Mode
- `Tracepoint Syscall Redirect hook` The default hook mode, from [upstream](https://github.com/tiann/KernelSU), but its only support GKI2 kernel with `arm64-v8a` or `x86_64` ABI
- `Manual Hook` The most compatible Hook, support from Linux kernel 3.4 to Linux kernel 6.18
- `SuSFS Inline Hook` An hook from [SuSFS](https://github.com/simonpunk/susfs4ksu), like `Manual Hook`, but provide from `SuSFS` project, not this project

## Integration

See the [repository](https://github.com/spacealtctrl/ReSukiSU-Ultima).

### Prebuilt GKI kernel with SUSFS

The `Build GKI Kernel (ReSukiSU Ultima + SUSFS)` workflow (`.github/workflows/build-gki-susfs.yml`) builds an `android15-6.6` GKI kernel from the Android Common Kernel with ReSukiSU Ultima built in and the SUSFS inline hooks enabled. By default it targets the Pixel 10 Pro XL (`mustang`). Run it from the Actions tab, then pick the ACK release branch and the device codename.

- The `…-AnyKernel3` artifact is already a flashable zip. Flash it with the ReSukiSU manager or Kernel Flasher.
- The `…-Image` artifact contains `Image`, `Image.lz4` and `Image.gz`, so you can repack your stock `boot.img` with `magiskboot`.
- Pair the kernel with a ReSukiSU **Ultima** manager. The upstream ReSukiSU manager uses another signing key and a newer UAPI, so the kernel does not recognize it. If you sign your own manager build through the `KEYSTORE`, `KEYSTORE_PASSWORD`, `KEY_ALIAS` and `KEY_PASSWORD` secrets, the kernel workflow trusts that key automatically.

## KPM Support

- Based on KernelPatch, we removed features redundant with KSU and retained only KPM support.
- Work in Progress: Expanding APatch compatibility by integrating additional functions to ensure compatibility across different implementations.

**Open-source repository**: [https://github.com/ShirkNeko/SukiSU_KernelPatch_patch](https://github.com/ShirkNeko/SukiSU_KernelPatch_patch)

**KPM template**: [https://github.com/udochina/KPM-Build-Anywhere](https://github.com/udochina/KPM-Build-Anywhere)

> [!Note]
>
> 1. Requires `CONFIG_KPM=y`
> 2. Non-GKI devices requires `CONFIG_KALLSYMS=y` and `CONFIG_KALLSYMS_ALL=y`
> 3. For kernels below `4.19`, backporting from `set_memory.h` from `4.19` is required.

## Sentinel

Sentinel is a kernel-side root-probe detector. It watches for apps that probe for
root - e.g. `access("/system/bin/su")`, magisk/ksu path checks, `packages.list`
enumeration - and streams those probe events to the manager, where you can review
them and **cloak** the offending app. Cloaking hides root from that app by reusing
the existing App Profile enforcement (umount + root-deny), so a detector sees a
clean, unrooted device.

- **Recent Probes**: a live feed on the Sentinel screen showing which apps probed
  for root and what they looked for.
- **Auto-cloak**: automatically cloak any new app that probes for root.
- **Notify on root requests**: instead of auto-cloaking, get a notification when a
  non-cloaked app probes for su, with one-tap **Grant** / **Cloak** / **Ignore**. A
  cloaked app never notifies. (Auto-cloak and Notify are mutually exclusive.)

Detection is opt-in at runtime from the manager's Sentinel screen; kernel overhead
is negligible when it is off.

> [!Note]
>
> 1. Requires `CONFIG_KSU_SENTINEL=y` (enabled by default; `depends on KSU`).
> 2. Cloak enforcement reuses App Profile, so no extra config is needed.

## Built-in Zygisk

ReSukiSU Ultima ships its own self-contained Zygisk implementation
("Zygisk-Ultima"), so Zygisk modules work without installing a separate Zygisk
provider. It is a ptrace-based injector deployed under `ksud` (not as a module) and
is **off by default** - enable it from the manager's settings, then reboot. The
launch hook installed in `post-fs-data.d` doubles as a kill-switch: if anything ever
goes wrong, deleting it disables Zygisk on the next boot.

This engine is adapted from [**ReZygisk**](https://github.com/PerformanC/ReZygisk) by
[The PerformanC Organization](https://github.com/PerformanC) (GPL-3.0) - huge thanks
to them for keeping it open source.

> [!Note]
>
> 1. Off by default; toggle it in the manager and reboot to apply.
> 2. No extra kernel config is required.

## Sponsor

- [ShirkNeko](https://afdian.com/a/shirkneko) (maintainer of SukiSU)
- [weishu](https://github.com/sponsors/tiann) (author of KernelSU)

<details>
<summary>ShirkNeko's sponsorship list</summary>

- [Ktouls](https://github.com/Ktouls) Thanks so much for bringing me support.
- [zaoqi123](https://github.com/zaoqi123) Thanks for the milk tea.
- [wswzgdg](https://github.com/wswzgdg) Many thanks for supporting this project.
- [yspbwx2010](https://github.com/yspbwx2010) Many thanks.
- [DARKWWEE](https://github.com/DARKWWEE) 100 USDT
- [Saksham Singla](https://github.com/TypeFlu) Provide and maintain the website
- [OukaroMF](https://github.com/OukaroMF) Donation of website domain name
</details>

## License

- The file in the “kernel” directory is under [GPL-2.0-only](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) license.
- The images of the files `ic_launcher(?!.*alt.*).*` with anime character sticker are copyrighted by [怡子曰曰](https://space.bilibili.com/10545509), the Brand Intellectual Property in the images is owned by [明风 OuO](https://space.bilibili.com/274939213), and the vectorization is done by @MiRinChan. Before using these files, in addition to complying with [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt), you also need to comply with the authorization of the two authors to use these artistic contents.
- Except for the files or directories mentioned above, all other parts are under [GPL-3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html) license.

## Credit

- [SukiSU-Ultra/SukiSU-Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)： upstream
- [ReZygisk](https://github.com/PerformanC/ReZygisk) by [The PerformanC Organization](https://github.com/PerformanC): the open-source (GPL-3.0) ptrace-based Zygisk implementation our built-in Zygisk engine is adapted from. Thank you for keeping it open source.

<details>
<summary>SukiSU's credit</summary>

- [KernelSU](https://github.com/tiann/KernelSU): upstream
- [MKSU](https://github.com/5ec1cff/KernelSU): Magic Mount
- [RKSU](https://github.com/rsuntk/KernelsU): support non-GKI
- [susfs](https://gitlab.com/simonpunk/susfs4ksu): An addon root hiding kernel patches and userspace module for KernelSU.
- [KernelPatch](https://github.com/bmax121/KernelPatch): KernelPatch is a key part of the APatch implementation of the kernel module
</details>

<details>
<summary>KernelSU's credit</summary>

- [Kernel-Assisted Superuser](https://git.zx2c4.com/kernel-assisted-superuser/about/): The KernelSU idea.
- [Magisk](https://github.com/topjohnwu/Magisk): The powerful root tool.
- [genuine](https://github.com/brevent/genuine/): APK v2 signature validation.
- [Diamorphine](https://github.com/m0nad/Diamorphine): Some rootkit skills.
</details>
