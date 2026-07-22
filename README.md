# Pi Desktop
### The Orange Pi 5B, as it should have shipped.

A clean, fast, self-healing Linux desktop for the Orange Pi 5B (RK3588) — built by
someone who bought the board expecting a real computer, and got tired of waiting for one.

## What it is
- 🖥️ **4K@120 / high-refresh, flicker-free** — a custom-tuned kernel that does the VOP2/dclk work most images skip
- 🎬 **Hardware video decode** — 4K in your browser and players on the RK3588's VPU, CPU barely awake
- 🔵 **Bluetooth that fixes itself** — the AP6275P radio bug that's broken these boards for years, root-caused *and* auto-healed
- 🎛️ **A curated desktop** — hardware-accelerated Chromium, VLC & mpv, LibreOffice, VSCodium. Nothing you didn't ask for
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
**v1.0-beta** — [Releases](../../releases). Flash the `.img.xz` to microSD or eMMC,
verify the SHA256, boot. First boot expands to fill the card.

## The family
- **[Pi Studio](https://github.com/defcom5-rockchip/pi-studio)** — the audio-production sibling (Sonic Pi, Ardour, an offline NPU AI music copilot)
- **Naked Network Pi** — the stripped, headless base both are built on *(coming)*

Built on [Joshua Riek's ubuntu-rockchip](https://github.com/Joshua-Riek/ubuntu-rockchip)
· Ubuntu 24.04 LTS · GPL-3.0 · by defcom5-rockchip
