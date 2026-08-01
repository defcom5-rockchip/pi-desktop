# Pi Desktop
### The Orange Pi 5B, as it should have shipped.

A clean, fast, self-healing Linux desktop for the Orange Pi 5B (RK3588) — built by
someone who bought the board expecting a real computer, and got tired of waiting for one.

## What it is
- 🖥️ **4K@120 / high-refresh, flicker-free** — a custom-tuned kernel that does the VOP2/dclk work most images skip
- 🎬 **Hardware video decode — including in the browser** — 4K60 YouTube runs on the RK3588's video
  engine, not the CPU (~118% instead of 600%+). mpv and VLC are wired to the VPU via rkmpp too.
  Decode runs on the **Panfrost** stack, so it costs no stability tradeoff. *(AV1 is software-only —
  see [KNOWN-ISSUES](KNOWN-ISSUES.md#vid-2-av1-is-software-decoded-and-always-will-be-on-this-stack).)*
- 🔵 **Bluetooth that fixes itself** — the AP6275P radio bug that's broken these boards for years, root-caused *and* auto-healed
- 🔌 **Boots with your USB audio interface plugged in** — U-Boot probed USB before storage, so a
  bus-powered interface could stall the board before it ever reached the kernel. Fixed in v1.0
- 🎛️ **A curated desktop** — hardware-accelerated Chromium, VLC & mpv, LibreOffice, VSCodium, WiiM Play for the hi-fi. Nothing you didn't ask for
- 🚫 **No snaps. No telemetry. No junk drawer.** De-snapped, de-Canonical'd
- 🔒 **Upgrade-proof** — kernel, bootloader, and snap-free state held, so `apt upgrade` can't undo the work

## 🔵 Self-healing Bluetooth — the fix Rockchip-land never shipped
The AP6275P WiFi/BT combo chip has broken Bluetooth on these boards for years:
dead at boot with WiFi active, stuttering audio, a toggle that needed a full reboot
to recover. Pi Desktop ships the root-caused fixes *and* a background service that
**resurrects the adapter in ~10 seconds, automatically — no reboot.** Toggle it off
and on, count to ten, it's back. The thing that demanded a reboot for nine years now
heals itself while you're not looking. *(Fixes published upstream: see issue #1.)*

## Honest scope
A niche distro, single maintainer, on the vendor 6.1 BSP kernel. It ships a
**KNOWN-ISSUES** file on purpose — we'd rather tell you what's rough than let you find
out. One excellent browser ships (HW-accelerated Chromium); a second browser or mail
client is your choice, and we provide the guides (Widevine, etc.). Release notes say
what a release *doesn't* fix, too.

## Download
**v1.0 — "Vanilla Sky"** — [Releases](../../releases). Flash the `.img.xz` to microSD or eMMC,
verify the SHA256, boot. First boot expands to fill the card.

**Installing to eMMC? Read [EMMC-INSTALL.md](EMMC-INSTALL.md) first.** Copying a root filesystem
onto eMMC does *not* install a bootloader — you can end up running a new OS on years-old firmware,
which causes crashes and reboots under load (and follows you across distros). The guide covers the
correct install, how to wipe a factory-Android eMMC, and maskrom recovery.

## The family
- **[Pi Studio](https://github.com/defcom5-rockchip/pi-studio)** — the audio-production sibling (Sonic Pi, Ardour, an offline NPU AI music copilot)
- **Naked Network Pi** — the stripped, headless base both are built on *(coming)*

Built on [Joshua Riek's ubuntu-rockchip](https://github.com/Joshua-Riek/ubuntu-rockchip)
· Ubuntu 24.04 LTS · GPL-3.0 · by defcom5-rockchip
