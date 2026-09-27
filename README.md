# local_manifests_lineage — Lineage 23.2 + kernel 5.4 (akari/tama)

Manifest **terpisah** dari `local_manifests` (yang berbasis Sony-SDM845-5-4/AOSP).
Yang ini: **device tree + vendor Lineage 23.2** (aoi-itsme / romiyusnandar / aoitsme)
+ **kernel 5.4** (cubbins) — untuk build **LineageOS 23.2 (Android 16)**.

## Konten
- `lineage-akari-5.4.xml` — local manifest:
  - `device/sony/akari`, `device/sony/tama-common` (Lineage 23.2)
  - `vendor/sony/akari`, `vendor/sony/tama-common` (Lineage 23.2)
  - `kernel/sony/sdm845` ← `kernel_sony_sdm845-5.4` (cubbins) + techpack + wlan

## Build
```sh
repo init -u https://github.com/LineageOS/android -b lineage-23.2
mkdir -p .repo/local_manifests
curl -o .repo/local_manifests/lineage-akari-5.4.xml \
  https://raw.githubusercontent.com/Tama-5-4-lineage/local_manifests_lineage/main/lineage-akari-5.4.xml
repo sync -c --force-sync -j$(nproc)
source build/envsetup.sh
lunch lineage_akari-userdebug      # atau aosp_... sesuai device tree
mka bacon
```

## Perubahan device tree yang WAJIB (swap 4.9 → 5.4)
Di `device/sony/tama-common/BoardConfigCommon.mk`:
```make
TARGET_KERNEL_VERSION := 5.4
TARGET_KERNEL_SOURCE  := kernel/sony/sdm845
BOARD_KERNEL_TAGS_OFFSET := 0x01E00000
BOARD_RAMDISK_OFFSET     := 0x02000000
```
Di `device/sony/akari/BoardConfig.mk`: `TARGET_KERNEL_CONFIG := <defconfig 5.4>`.

Cmdline (dari SODP 5.4 `PlatformConfig.mk`):
`androidboot.bootdevice=1d84000.ufshc`, `service_locator.enable=1`,
`coherent_pool=8M`, `msm_drm.dsi_display0=somc,default_cmd_panel:config0`.

## ⚠️ Masalah integrasi build kernel
- Kernel 5.4 (cubbins) memakai **AOSP `build.config`** (GKI-style), **bukan
  `AndroidKernel.mk`** yang dipakai device tree Lineage.
- Solusi: (a) build kernel terpisah via `common-kernel/build-kernels-clang.sh`
  lalu pakai sebagai prebuilt (`TARGET_PREBUILT_KERNEL` + `TARGET_PREBUILT_DTB`),
  atau (b) tambahkan dukungan build 5.4 ke mekanisme device tree.
- Referensi config 5.4: `Tama-5-4-lineage/device-sony-{tama,akari,common}` (branch `b-mr1`).

## Vendor / blobs
Vendor Lineage 23.2 (aoitsme) dibuat untuk **4.9**. Kernel 5.4 butuh blobs odm
yang serasi (SODP A16 / 5.4 Tama dari opendevices.sony.net). Ini titik paling
rawan mismatch (display/camera/audio).
