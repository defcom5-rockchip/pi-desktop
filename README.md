# Pi Desktop
### The Orange Pi 5B, as it should have shipped.

A clean, fast, self-healing Linux desktop for the Orange Pi 5B (RK3588) — built by
someone who bought the board expecting a real computer, and got tired of waiting for one.

## What it is
- 🖥️ **4K@120 / high-refresh, flicker-free** — a custom-tuned kernel that does the VOP2/dclk work most images skip
- 🎬 **Hardware video decode — including in the browser, including 10-bit** — 4K60 YouTube
  and 4K HEVC Main10 run on the RK3588's video engine, not the CPU. The driver doing it is
  **ours**: the [rockchip-vaapi fork](https://github.com/defcom5-rockchip/rockchip-vaapi)
  (v2.1.5) — H.264, HEVC, VP9 at 8 and 10 bits, verified pixel-identical to software decode.
  Jellyfin's web client direct-plays 10-bit in Firefox in hardware. *(AV1 is software; 10-bit
  at 4K60 can stutter until the next driver phase — see [KNOWN-ISSUES](KNOWN-ISSUES.md).)*
- 🦊 **Firefox is the default browser** — flicker-free (no ANGLE) *and* hardware-decoded, out of the
  box, with update-proof policies. **Chrome rides along for one job: DRM streaming** — it bundles
  Google's own Widevine, and ships pre-flagged so Netflix just plays. Chromium (rkmpp) stays for
  those who want it — with our fork of its decode plug-in, so 10-bit files no longer take its
  GPU process down.
- 🖱️ **mpv 0.38** — drag a file onto the player and it plays (a Wayland fix 0.36 never had)
- 🛡️ **Pro Updates** — free security updates without an Ubuntu Pro subscription, green out of the box
- 🔵 **Bluetooth that fixes itself** — the AP6275P radio bug that's broken these boards for years, root-caused *and* auto-healed
- 🔌 **Boots with your USB audio interface plugged in** — U-Boot probed USB before storage; fixed since v1.0
- 📧 **Betterbird mail**, LibreOffice Writer + Calc, VSCodium, VLC, WiiM Play for the hi-fi. Nothing you didn't ask for
- 🚫 **No snaps. No telemetry. No junk drawer.** De-snapped, de-Canonical'd
- 🔒 **Upgrade-proof** — kernel, bootloader, and snap-free state held, so `apt upgrade` can't undo the work

## 🔵 Self-healing Bluetooth — the fix Rockchip-land never shipped
The AP6275P WiFi/BT combo chip has broken Bluetooth on these boards for years:
dead at boot with WiFi active, stuttering audio, a toggle that needed a full reboot
to recover. Pi Desktop ships the root-caused fixes *and* a background service that
**resurrects the adapter in ~10 seconds, automatically — no reboot.** *(Fixes published
upstream: see issue #1.)*

## Honest scope
A niche distro, single maintainer, on the vendor 6.1 BSP kernel. It ships a
**KNOWN-ISSUES** file on purpose — we'd rather tell you what's rough than let you find
out. The video driver follows the same rule: it refuses to advertise codecs that don't
actually work, so players fall back to correct software playback instead of a green wall.
Release notes say what a release *doesn't* fix, too.

## Download
**v2.0.3 "Crystal Blue Persuasion" (final)** — [Releases](../../releases). Flash the `.img.xz` to microSD or eMMC,
verify the SHA256, boot. First boot expands to fill the card.

**Installing to eMMC? Read [EMMC-INSTALL.md](EMMC-INSTALL.md) first.** Copying a root filesystem
onto eMMC does *not* install a bootloader — the guide covers the correct install, wiping a
factory-Android eMMC, and maskrom recovery.

## The recipe is public
Every hook, every flag, every fix — with its reasoning in the comments:
[defcom5-rockchip/ubuntu-rockchip](https://github.com/defcom5-rockchip/ubuntu-rockchip).
Don't trust the image; audit the recipe.

## The family
- **[Pi Studio](https://github.com/defcom5-rockchip/pi-studio)** — the audio-production sibling (Sonic Pi, Ardour, an offline NPU AI music copilot)
- **Naked Network Pi** — the stripped, headless base both are built on *(coming)*

Built on [Joshua Riek's ubuntu-rockchip](https://github.com/Joshua-Riek/ubuntu-rockchip)
· Ubuntu 24.04 LTS · GPL-3.0 · by defcom5-rockchip (with Claude as co-engineer)
