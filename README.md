# local_manifests_lineage — Lineage 23.2 + kernel 5.4 (akari/tama)

Manifest **terpisah** dari `local_manifests` (yang berbasis Sony-SDM845-5-4/AOSP).
Yang ini: **device tree + vendor Lineage 23.2** + **kernel 5.4** (cubbins) +
**binaries SODP Tama v3 (odm)**.

## Konten (`lineage-akari-5.4.xml`)
- `device/sony/akari`, `device/sony/tama-common` — Lineage 23.2 (dipatch ke 5.4)
- `vendor/sony/akari`, `vendor/sony/tama-common` — vendor Lineage 23.2
- `vendor/sony/tama` — **binaries SODP Tama v3** (`vendor-sony-tama`, odm)
- `kernel/sony/sdm845` — kernel 5.4 (cubbins) + techpack + wlan

## Binaries v3 (odm)
- Tama tidak ada binaries 5.4. Build 5.4 pakai binaries **4.19 v3 (odm)** dari repo
  SODP `vendor-sony-tama` (sudah ke-fork di org kita).
- Repo itu punya `Android.mk` yang menyalin `bin/ etc/ firmware/ lib/ lib64/ system_ext/`
  ke `$(TARGET_OUT_ODM)` — build Android memindai semua `Android.mk`, jadi odm v3
  otomatis terpasang (tidak perlu regenerate vendor Lineage).
- Syarat: `PRODUCT_PLATFORM=tama` (sudah diset di `device/sony/tama-common/common.mk`).

## Build
```sh
repo init -u https://github.com/LineageOS/android -b lineage-23.2
mkdir -p .repo/local_manifests
curl -o .repo/local_manifests/lineage-akari-5.4.xml \
  https://raw.githubusercontent.com/Tama-5-4-lineage/local_manifests_lineage/main/lineage-akari-5.4.xml
repo sync -c --force-sync -j$(nproc)
source build/envsetup.sh
lunch lineage_akari-userdebug     # atau aosp_... sesuai device tree
mka bacon
```

## Catatan
- Kernel 5.4 sudah dipatch untuk **retrofit dynamic** (initramfs skip + force_normal_boot
  + DT fstab vendor) + `AndroidKernel.mk` + fs-verity.
- **Potensi duplikat odm**: vendor Lineage punya `proprietary/odm/lib{,64}/hw/audio.primary.sdm845.so`;
  v3 punya HAL lain (gralloc/sensors/vulkan/bt/keymaster). Overlap minimal — kalau muncul
  error duplikat, hapus entri odm yang bentrok dari vendor Lineage.
- Build from source kernel 5.4 itu eksperimental; ekspektasikan bug (display/camera/audio).
