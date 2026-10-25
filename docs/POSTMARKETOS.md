# Installing postmarketOS on the Redmi Note 10 Pro (sweet / sweetin)

Status: **researched and prepared, not run on a phone from here.** Sources are named so each claim can be re-checked.

## What exists (verified from the current official repository, `gitlab.postmarketos.org/postmarketOS/pmaports`, default branch `main`, checked 2026-10-25)
- `device/community/device-qcom-sm7150` (generic SM7150 device package; fastboot flashing, 4096-byte sectors for UFS, `systemd-boot`),
  `firmware-xiaomi-sweet`, `firmware-qcom-sm7150`, `alsa-ucm-conf-qcom-sm7150`, and `linux-postmarketos-qcom-sm7150` = Linux **7.1_rc3**
  from `github.com/sm7150-mainline/linux`, which has `sm7150-xiaomi-sweet.dts`.
- Note: the old `gitlab.com/postmarketOS/pmaports` project is a **frozen mirror** (last commit Nov 2024; description: "Moved to gitlab.postmarketos.org"). Do not use it to judge device support.
- Prebuilt images: `https://images.postmarketos.org/bpo/edge/qcom-sm7150/<ui>/<date>/` for `phosh`, `plasma-mobile`, `gnome-mobile`, `sxmo-de-sway` (newest seen: 20261024).
  Downloads redirect to `images.nura.eco`; I could not independently verify who operates that domain, so treat the published checksums as integrity checks, not proof of origin.
- Boot chain: **U-Boot is a hard requirement** (chainloaded from the stock bootloader via the `boot` partition). OpenRC is unsupported; use the systemd images.

## Feature status (from the device wiki page, unverified third-party page that the postmarketos.org wiki redirects to)
Works: display, touch, 3D acceleration, Wi-Fi, Bluetooth, modem calls/SMS/data, accelerometer, magnetometer, ambient light, USB networking, flashing.
Partial: battery, haptics. Broken: audio, camera, fingerprint, proximity sensor. GPS untested.

## Files
- `artifacts/postmarketos/u-boot-sm7150-xiaomi-sweet.img`: built here from `sm7150-mainline/u-boot` tag `2025-12-02` with the project's CI recipe
  (`qcom_defconfig qcom-phone.config tauchgang.config`, `CONFIG_DEFAULT_DEVICE_TREE=qcom/sm7150-xiaomi-sweet`, gzip(u-boot-nodtb)+dtb as kernel,
  `mkbootimg --base 0x0 --kernel_offset 0x8000 --pagesize 4096`). The CI uses osm0sis' C `mkbootimg`; this one used AOSP's python `mkbootimg.py` (same header v0 layout; not compared byte-for-byte). The project also publishes prebuilt U-Boot images with its releases.
  The sweet tree has a single panel variant (unlike `davinci`/`surya`), so no panel-vendor suffix is needed *(inferred from the DT list; unconfirmed)*.

## Install (UNTESTED; erases userdata; needs an unlocked bootloader and fastboot)
Back up anything you need. This replaces `boot` and `userdata`; Android will not boot afterwards until you restore stock `boot` (and a working `userdata`/ROM).

    sudo pacman -S android-tools xz                      # Arch: adb + fastboot
    curl -LO https://images.postmarketos.org/bpo/edge/qcom-sm7150/phosh/20261024-0125/20261024-0125-postmarketOS-edge-phosh-qcom-sm7150.img.xz
    echo "ef8b1b58f144224d68dd4a301661c196b9fd9b847d4321392a7ebe69d38d563a  20261024-0125-postmarketOS-edge-phosh-qcom-sm7150.img.xz" | sha256sum -c
    xz -dk 20261024-0125-postmarketOS-edge-phosh-qcom-sm7150.img.xz
    fastboot erase dtbo
    fastboot flash boot u-boot-sm7150-xiaomi-sweet.img
    fastboot flash userdata 20261024-0125-postmarketOS-edge-phosh-qcom-sm7150.img
    fastboot reboot

Keep `vbmeta` / `vbmeta_system` flashed with `artifacts/vbmeta-disabled.img` (see README). To go back to Android, restore your stock `boot`, `dtbo`, `vbmeta*` and reflash the stock ROM.
After boot, USB networking should appear (postmarketOS convention: `telnet 172.16.42.1` in the initramfs, `ssh user@172.16.42.1` when booted; unverified for this build).
Alternative to the prebuilt image: `pmbootstrap init` (vendor `qcom`, device `sm7150`) then `pmbootstrap install` and `pmbootstrap flasher flash_rootfs`.
