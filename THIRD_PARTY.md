# Third-party components

- Magisk 25.2 (25200), upstream: https://github.com/topjohnwu/Magisk/tree/v25.2 . License: GPL-3.0. The release includes the upstream tag source archive and license text. No new Magisk code changes were made for this publication. The init binary matches the locally retained Magisk 25.2 ARM distribution.
- Magisk source archive: https://github.com/topjohnwu/Magisk/archive/refs/tags/v25.2.tar.gz . Git submodules are referenced by the upstream tree; a GitHub tag archive alone does not embed all submodule working trees. Build instructions: https://github.com/topjohnwu/Magisk/blob/v25.2/docs/build.md .
- The TV's original kernel and device-specific boot components are retained. This is a partition backup, not a fresh kernel build. A complete matching kernel source/build environment is not included and has not been identified for this exact firmware. TCL's official source index is https://github.com/TCLOpenSource/TCL_Kernel_OpenSource ; a matching source version is not asserted.
- Android boot image / AVB formats: https://android.googlesource.com/platform/system/tools/mkbootimg/ and https://android.googlesource.com/platform/external/avb/ .

This project is unofficial and is not affiliated with TCL, MediaTek, MStar or the Magisk authors. Existing third-party rights and license conditions are retained; no blanket relicensing of the image is implied.