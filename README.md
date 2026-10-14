# nothing-spacewar-lineage-bakasu

Kernel for **Nothing Phone (1) (spacewar)** running LineageOS-based ROMs
(lineage-24.0). Fork of
[LineageOS/android_kernel_nothing_sm7325](https://github.com/LineageOS/android_kernel_nothing_sm7325)
(`lineage-24.0`) with BakaSU root, SUSFS hiding, DroidSpaces container
support and NoMount built in.

## Branches

| branch | content |
|---|---|
| `lineage-24.0` | clean base, pinned to upstream `f431f6e13653` |
| `lineage-24.0-bakasu` | base + everything below (CI builds this) |

## What's in `lineage-24.0-bakasu`

- **BakaSU** (ex-ReSukiSU) kernel driver, submodule @ `373303c5`
  (2026-10-07), driver version `35218`. Pair it with a manager built from
  the same commit (versionCode must match, e.g. official CI
  `Manager-release` for that commit).
- **SUSFS v2.3.0** (`gki-android12-5.10` backported to 5.4): hide
  root/traces from apps. 5.4 adaptations: `cmdline.c` spoof (no
  `bootconfig.c` on 5.4), `selinux_state.ss->` conversion, fsnotify 5.4
  API, no `ksu_handle_post_execveat_sucompat`, plus the `ksu_handle_stat`
  call site ported to `vfs_statx` (upstream only hooks `vfs_fstatat`,
  which doesn't exist on 5.4).
- **DroidSpaces** container support (GKI config block). The mandatory
  SYSVIPC/POSIX_MQUEUE kABI patches are hand-ported to 5.4
  (`include/linux/sched.h`, `include/linux/sched/user.h`, KABI slots
  6/7/8 and 1). `CFS_BANDWIDTH`/`CGROUP_PIDS` stay off on purpose: they
  break the kABI and no patch covers them.
- **NoMount** path redirection, built in (`CONFIG_NOMOUNT=y`,
  `fs/nomount/` from upstream `dev`).
- USB HID gadget (`CONFIG_USB_CONFIGFS_F_HID`) is already on, so the
  phone can act as USB keyboard/mouse via configfs.

## Build (CI)

GitHub Actions (`Build kernel`) builds `lineage-24.0-bakasu` on every manual
run: AOSP clang `r563880c` + `LLVM=1 LLVM_IAS=1`, artifact is `Image`.

Local build:

```bash
export PATH="$HOME/toolchains/clang-r563880c/bin:$PATH"
echo "-gf431f6e13653" > .scmversion   # keep vendor release string!
make O=out ARCH=arm64 LLVM=1 LLVM_IAS=1 CC="ccache clang" KSU_VERSION=35218 \
  vendor/lahaina-qgki_defconfig vendor/debugfs.config \
  resukisu.config droidspaces.config nomount.config
make O=out ARCH=arm64 LLVM=1 LLVM_IAS=1 CC="ccache clang" KSU_VERSION=35218 \
  -j"$(nproc)" Image
```

## KMI / vermagic (important)

LineageOS 24 vendor modules expect `5.4.302-qgki-gf431f6e13653`. To keep
them loading:

- never override `CONFIG_LOCALVERSION` (stays `-qgki` from defconfig),
- always write `.scmversion` as above before building,
- commit your changes (a `-dirty` tree changes the release string).

CI checks the release string and fails the build on mismatch.

## Flash

```bash
fastboot boot  lineage-24.0-boot-bakasu-susfs.img   # test first
fastboot flash boot lineage-24.0-boot-bakasu-susfs.img
```

or flash the AnyKernel3 zip (`lineage-24.0-AK3-bakasu-susfs.zip`) from
the BakaSU manager / recovery. To go back: `git checkout lineage-24.0` and
rebuild, or re-flash your stock `boot.img`.

## Upstream references

- https://github.com/LineageOS/android_kernel_nothing_sm7325
- https://github.com/ReSukiSU/ReSukiSU (now BakaSU)
- https://gitlab.com/simonpunk/susfs4ksu
- https://www.droidspaces.org/docs/kernel-configuration.html
- https://github.com/maxsteeel/nomount
