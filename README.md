# Pi Desktop

**v1.0 "Vanilla Sky" — first alpha baked 2026-07-19.**

**A clean, fast daily-driver desktop for the Orange Pi 5B — in development.**
*by defcom5-rockchip · based on [Joshua-Riek's ubuntu-rockchip](https://github.com/Joshua-Riek/ubuntu-rockchip) · Ubuntu 24.04 LTS · sibling of [Pi Studio](https://github.com/defcom5-rockchip/pi-studio)*

Pi Studio proved the Orange Pi 5B can be a real instrument. Pi Desktop makes it a real computer.

## What it will be (v1.0, in progress)

- **Throughput-tuned kernel** — 4K@120 flicker-free display (our VOP2 patches), desktop preemption,
  hand-picked security backports. Same forge as Pi Studio, different tuning.
- **Every fix we published, inherited:** the Bluetooth boot-race fix
  ([#1](https://github.com/defcom5-rockchip/ubuntu-rockchip/issues/1)), the BT-audio/WiFi coexistence
  tuning ([#2](https://github.com/defcom5-rockchip/ubuntu-rockchip/issues/2)), hardware-decode Chromium,
  first-boot fixes.
- **Curated, not crowded:** Chromium (HW video) + Vivaldi, VLC, LibreOffice, Betterbird, VSCodium
  (telemetry-free, flicker-fix pre-applied), WiiM Play, btop + a temperature readout where an SBC
  needs one. No snaps. No telemetry. No junk drawer.
- **Local AI, next:** the same on-NPU intelligence as Pi Studio's Muse, aimed at the desktop.

## Why

The Orange Pi 5B's official images are years stale and its ecosystem support is thin. We maintain
the board like it deserves — kernel to desktop, with every limitation documented honestly.

**Status:** spec complete, first image in the works. Watch this repo.
