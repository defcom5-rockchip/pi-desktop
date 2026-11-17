# Plaid Pi (formerly Pi-Desktop)
### The Orange Pi 5, 5B and 5 Plus, as a desktop computer.

**Plaid Pi Plus 6.5** (Orange Pi 5 Plus) · **Plaid Pi 6.0** (Orange Pi 5B and 5) · [Download](#download) · [Known issues](KNOWN-ISSUES.md) · [Changelog](CHANGELOG.md) ·
[Privacy](PRIVACY.md) · [Legal](LEGAL.md) · [Security](SECURITY.md) · [Website](https://defcom5-rockchip.github.io/pi-desktop/)

> The 2.0.x line on `ubuntu-rockchip` was retired at v2.0.2 (2026-09-23); 4.0 was its successor on a new base;
> 5.0 (2026-10-19) continued it under the new name, then 5.5 and 6.0 (the Orange Pi 5B image); 6.5 (2026-11-17) adds the
> Orange Pi 5 Plus with its own KDE Plasma image — see the [CHANGELOG](CHANGELOG.md).

Two desktop images built from Ubuntu 26.04 packages with the Armbian build framework, on Armbian's Rockchip vendor kernel
merged with upstream stable Linux plus the Plaid Pi kernel patches: **Plaid Pi Plus 6.5** for the Orange Pi 5 Plus (RK3588; KDE
Plasma 6.6 on Mesa's open Panthor GPU driver, kernel 6.1.191) and **Plaid Pi 6.0** for the Orange Pi 5B and Orange Pi 5
(RK3588S; GNOME 50 on Arm's Mali driver, kernel 6.1.189). Not affiliated with or endorsed by Canonical or Armbian. Not a Raspberry Pi product.

![The 6.0 GNOME image. Plaid Pi on a 32-inch 4K monitor at 120 Hz: tartan-canopied penguins parachuting over a dusk horizon under the Plaid top bar; the dock auto-hides until the pointer reaches the bottom edge.](docs/img/desktop.png)

## What it is

Everything below was checked on an Orange Pi 5B: on the 5.0 release image (2026-10-19: 133 of 133 first-boot checks
passed, and again after all seven first-click apps and Pro Updates had run), again on the 5.5 release image
(2026-10-22: 147 of 147) and on the 6.0 release image (2026-10-26: 152 of 152), unless an item says it was measured on 4.0.
The 6.5 image was checked on an Orange Pi 5 Plus (2026-11-17: 150 of 150 first-boot checks, and again after Pro Updates
and all seven first-click apps); items marked 6.5 were measured there.

- **An Orange Pi 5 Plus image (new in 6.5): KDE Plasma 6.6 on the open Panthor GPU driver.** The same video stack, the
  Plaid look on Plasma, Dolphin with the Network Drive and disc menus, 4K at 120 Hz on its HDMI 2.1 output; tested on one
  5 Plus here (what was not: [KNOWN-ISSUES HW-2](KNOWN-ISSUES.md#hw-2-orange-pi-5-plus--the-board-of-the-65-image-what-was-not-tested)).
- **DVDs and Blu-rays copied to folders play from the desktop (new in 6.5).** Right-click the folder in Dolphin ▸ *Play DVD
  or Blu-ray*, or double-click its VIDEO_TS.IFO; mpv opens the folder in place (0.42 adds ISO images and the disc menus). In a
  headless probe from the build PC, 53 of the 55 DVD folders on the maintainer's NAS opened as discs (the two that did not are
  incomplete copies); Streets of Fire was watched on the 5 Plus. Plex does not support such folders (its documentation), and
  Jellyfin 12.4 glues the wrong VOB files together (from its source).
- **A disc in a USB drive plays with its menu (new in 6.5).** Cinemosk's *Play the disc in the drive* starts a DVD in the
  disc's own menu (mpv 0.42). Bought DVDs need libdvdcss, which Plaid Pi cannot include: Cinemosk offers to build it on the
  computer. On the 5 Plus it took 1 min 50 s from Cinemosk's *Disc support* and 51 s from the first-time offer (one
  run each; it includes downloads, so it varies), and a bought DVD then played from the drive with its menu. Blu-ray discs
  go through the same button, and *Disc support* adds Java 21 for Blu-ray menus written in Java; no Blu-ray disc was
  tested. Bought Blu-rays stay encrypted.
- **Cinemosk (new in 6.5):** a picture-settings window for films. Smooth, Balanced or Sharp enlargement, with the GPU cost
  of each measured on the board shown next to the choice; banding clean-up, screen fill and even 24 fps motion (measured at
  120 Hz) for DVDs;
  per-folder overrides from inside mpv (Alt+C), a meter (Alt+M).
- **4K at 120 Hz over HDMI, out of the box** (the 5B since 4.0, the 5 Plus in 6.5). The vendor kernel reaches HDMI 2.1 FRL with our clock patches.
  The three ways it failed on the way there — a black screen after 14 minutes to 3 hours, a hard hang when the
  screen turned off at 120 Hz, a black screen after the monitor was power-cycled — are fixed by kernel patches
  0008, 0010 and 0011, each re-tested on hardware. One boot argument hides 120 Hz again; 4K@60 needs none of this.
- **A flicker-free desktop and browser.** On the 6.0 image the GPU's proprietary user-space driver instead of Mesa panfork, on
  the 6.5 image Mesa's open Panthor driver, and AFBC scanout rejected in the kernel (patch 0001) on both. The Chromium typing and thumbnail flicker of 2.0.x is gone.
- **Hardware video, including 10-bit and AV1, in Chromium** (shipped on the 6.0 image, a first-click install on the 6.5
  KDE image). Our
  [rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi) driver, 4.0.0, on our build of Rockchip's
  MPP library. Measured on 4.0 (2026-10-03): Chromium decodes a 4K HEVC **Main 10** file on the video engine — 595
  frames, none dropped, about a fifth of the CPU (21 % across its processes) — so Jellyfin direct-plays 10-bit
  without transcoding, out of the box; Chromium is 155 in 5.0. H.264, 8-bit HEVC and VP9 likewise, and VP9 now
  runs on both decoder cores (see Dual-core VP9 below).
  **AV1** is new in 5.0: Firefox (first-click install; on the GNOME image with our small Mali library loaded by its
  launcher, on the KDE image on Mesa with nothing extra) plays
  YouTube 4K60 AV1 on the video engine with 1.2–1.5 % of frames dropped after start-up, where driver 3.0.0 dropped
  91–95 %; Chromium and Chrome start AV1 playback clean in 10 of 10 measured first plays (VID-5). mpv (0.41 in 6.0, 0.42 in 6.5) goes
  through VA-API, with profiles that keep DVDs and Blu-rays from stuttering at 120 Hz.
- **The Number Nine Pepper video driver.** Our VA-API driver, rockchip-vaapi 4.0.0 "Pepper Pro", hands video to the
  RK3588's video engine, so Chromium, Chrome, Firefox and mpv decode H.264, HEVC, VP9 and AV1, 8- and 10-bit, in
  hardware (KNOWN-ISSUES lists the exceptions). 4.0.0 rebuilds the H.264 and HEVC headers per picture, so most of the
  streams earlier releases decoded wrong now decode bit-exact (VID-8: H.264 conformance 74 → 111 of 135, HEVC 17 → 141 of 147). For AV1 it rebuilds the headers MPP needs from the VA-API parameters,
  and with our MPP build each picture is decoded once. A fork of Eduardo García-Mádico Portabella's driver:
  [defcom5-rockchip/rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi). *Number Nine* is a tribute to
  the 1980s graphics-card maker, whose Pepper co-processor boards name the releases; not affiliated with or endorsed
  by Number Nine or the successors to its assets.
- **Dual-core VP9.** The RK3588's two video decoder cores work on VP9 together again. A frame label that the hardware
  compares in 16 bits made both cores reset after 65,536 frames (about 18 minutes of 60 fps video), so upstream MPP
  and Plaid Pi 4.0 decoded VP9 on one core. Our fix wraps the label where the hardware does; Rockchip's reviewer
  accepted it ([rockchip-linux/mpp#976](https://github.com/rockchip-linux/mpp/pull/976)). In Rockchip's test decoder,
  4K60 VP9 runs 44 % faster for 10-bit and 29 % for 8-bit than on one core.
- **Full-screen video capable.** 4K video decoded on the video engine — H.264, HEVC, VP9 and AV1, 8- and 10-bit —
  and drawn zero-copy by mpv, full screen on a 4K monitor at 120 Hz; on the GNOME image the screensaver already loops a
  film full screen.
  10-bit 4K at 60 fps uses most of the GPU ([KNOWN-ISSUES VID-6](KNOWN-ISSUES.md#vid-6-10-bit-4k60-in-mpv-leaves-little-gpu-headroom)),
  and 4K60 10-bit AV1 drops most frames in mpv for now ([VID-9](KNOWN-ISSUES.md#vid-9-av1-at-4k60-10-bit-drops-most-frames-in-mpv)).
- **Netflix and Prime Video in the browsers (new in 6.0).** Install Google Chrome with one click; Chromium and Firefox
  then use Chrome's Widevine module too, so keep Chrome installed (nothing is copied; it stays Google's install).
  Checked on the 6.0 test builds: Netflix plays in Chrome, Chromium and Firefox, Prime Video in Firefox and Chrome, and
  the sign-ins survive a reboot; other services using Widevine were not tried. Linux gets Widevine's software level,
  so the video is decoded by the CPU, not the video engine, and services choose the quality: Netflix sent 1080p at
  24 fps, Prime Video standard definition.
- **Films from a NAS without skips (new in 5.5).** **Network Drive** (Utilities, or right-click a share in Files ▸
  *Add as Network Drive…*) adds the share as a drive that reads 32 MiB ahead, so a short pause on the NAS doesn't
  interrupt the film; in a two-hour test a 4K AV1 film played from one in Firefox with no decoder stall
  ([KNOWN-ISSUES NET-1](KNOWN-ISSUES.md#net-1-films-played-straight-from-a-nas-share-smb-can-skip-with-the-sound-cutting-out)).
- **Wi-Fi joins WPA3 and WPA2/WPA3 networks.** Ubuntu 26.04's wpa_supplicant signals WPA3 in a way the vendor
  Wi-Fi driver never handled, so every modern home network refused the board. Patch 0006 fixes the driver, and
  Armbian merged it into its vendor-kernel branch on 2026-10-21
  ([armbian/linux-rockchip#561](https://github.com/armbian/linux-rockchip/pull/561)); Wi-Fi 6 rates measured on 5 GHz.
- **Bluetooth that survives Wi-Fi.** The two AP6275P fixes from 2.0.x — the boot race and the coexistence
  timing — are carried over with the self-healing sentinel. The ten-second mute at every YouTube ad on a
  Bluetooth speaker is gone (bluez's mpris-proxy was telling the speaker "stopped"; it is off).
- **Audio.** PipeWire 1.6.10 and WirePlumber 0.5.18 with real-time priority from boot; GStreamer 1.28.9 in 6.5. The 3.5 mm jack works
  (the vendor codec driver shipped it switched off) and follows plugging like a laptop; HDMI is the default
  output and the outputs are named. Rhythmbox 3.5.1 — newest upstream, every feature on — Audacity (3.7.8 on 6.0, 4.0.2 on
  6.5) and Brasero, on both images.
- **A screensaver instead of screen blanking** on the GNOME image, with its own group in Settings ▸ Power (the KDE image
  uses Plasma's own power management); lock on demand; a login screen at every boot, with the Plaid Pi logo on it (GDM
  on the GNOME image, SDDM on the KDE image); Log Out back in GNOME's power menu.
- **Fixes you would otherwise find yourself.** GUI password prompts hang on Ubuntu 26.04 with a 6.1 kernel
  (polkit's new helper wants Linux 6.5) — fixed. A large file copy could get the whole session killed by
  systemd-oomd — fixed. The first boot asks for a computer name instead of every board having the same name.
- **A small image; apps on first click.** 6.0: 1.39 GiB compressed (1,496,576,700 bytes); 6.5: 1.48 GiB (1,590,814,467 bytes). Firefox, LibreOffice, GIMP,
  Inkscape, Thunderbird, VSCodium and Google Chrome sit in the app grid or the Application Launcher (Chrome and Firefox in
  the GNOME dock; Chrome, Chromium and Firefox in the KDE panel) as placeholders that download the real app from its own
  repository on the first click, then step aside; on the KDE image Chromium is one more such install. All seven
  ran on the 5.0 image on hardware (Firefox 158 from the Mozilla Team PPA, Chrome 155); six were timed on 4.0,
  45 seconds to five minutes. VS Code is not offered: Microsoft's build ships with telemetry on.
- **No snaps. No telemetry of our own.** What the Ubuntu and Armbian base still talks to is listed, with what we
  found, in [PRIVACY.md](PRIVACY.md). Armbian's browser policies (its start page and bookmarks, which made Chromium
  and Chrome say "Managed by your organization") are not applied since 5.0.
- **No default passwords.** The first boot has you set your own; root cannot log in over SSH with a password, only
  with a key; system changes ask for your password, and Network Drive logins are stored readable by root only.
- **Updates that cannot undo the work.** Ubuntu's security updates and the browsers' own repositories update daily
  by themselves; Pro Updates 1.5.0 shows and applies the rest when you open it. The kernel, device
  tree, bootloader, BSP and firmware packages are held and change only with Plaid Pi releases — otherwise
  Armbian's next release would replace the kernel and silently drop the patches. On 4.0 the daily automatic
  updates were switched off (Armbian's default): run `sudo apt update && sudo apt upgrade`, or turn them on as
  [SECURITY.md](SECURITY.md#if-you-run-pi-desktop-40) describes.

## Honest scope

**Supported boards:** the **Orange Pi 5 Plus** (the 6.5 image is built for it and tested on it), the **Orange Pi 5B**,
which the 6.0 image is built for and tested on, and the original **Orange Pi 5**, which uses the 5B image: same chip, and the 5B's device tree is the 5's plus the eMMC and the
Wi-Fi/Bluetooth module, minus the SPI flash. We have no Orange Pi 5 to test on, so the 5 is supported through
[issues](../../issues), pull requests to the [recipe](https://github.com/defcom5-rockchip/pi-desktop-recipe) and [Discussions](../../discussions); readers
reported that the earlier 2.0.x image boots on it, and nobody has reported 4.0 or later on a 5 yet (see [KNOWN-ISSUES HW-1](KNOWN-ISSUES.md#hw-1-orange-pi-5-the-original-not-the-b--supported-runs-the-5b-image-untested-by-us)).
**Orange Pi 5 Plus:** its own image from 6.5 (KDE Plasma; the full RK3588, two 2.5 GbE ports), tested on one board here;
[KNOWN-ISSUES HW-2](KNOWN-ISSUES.md#hw-2-orange-pi-5-plus--the-board-of-the-65-image-what-was-not-tested) lists what was not tested (eMMC, the Broadcom Wi-Fi module).

One maintainer, a vendor 6.1 kernel, best effort. It ships a [KNOWN-ISSUES](KNOWN-ISSUES.md) file on purpose —
we'd rather tell you what's rough than let you find out. Suspend is disabled. 10-bit video is hardware-decoded in
Chromium, Chrome and mpv, and in Firefox (on the GNOME image through our launcher library; VID-5, VID-7); AV1 in Chromium
can show a rare stale frame mid-stream.
HDR on the maintainer's monitor stays pale, and that is the monitor. The Orange Pi 5B and 5 have two display outputs,
HDMI 2.1 and DisplayPort over USB-C, nothing else (the 5 Plus adds a second HDMI output and an HDMI input, untested here:
HW-2); a passive HDMI-to-DisplayPort adapter cannot work on any computer
([DSP-1](KNOWN-ISSUES.md#dsp-1-two-display-outputs-a-passive-hdmi-to-displayport-adapter-cannot-work)).

Security updates for the Ubuntu packages come from Ubuntu's repositories for the life of 26.04. The kernel and
boot packages are built from Armbian's branch and refreshed with Plaid Pi releases: a new kernel means installing
the new image. Plaid Pi's own packages are updated with releases. There is no guaranteed support period
and no commercial product behind this. [SECURITY.md](SECURITY.md) says how to report a problem.

## Download

**Plaid Pi Plus 6.5 for the Orange Pi 5 Plus** — [Releases](../../releases): `Plaid-Pi-Plus-6.5-orangepi5-plus.img.zst` (1.48 GiB;
7.14 GiB written; a 16 GB card or more) and its `.sha256`.
**Plaid Pi 6.0 for the Orange Pi 5B and 5** — `Plaid-Pi-6.0-orangepi5b.img.zst` (1.39 GiB; 6.50 GiB written) and its `.sha256`.
Verify the checksum, write
the image to a microSD card or the eMMC (GNOME Disks, balenaEtcher or `dd`), boot. The first boot grows the
filesystem to the card, then asks (in a text console) for a root password, then a user, a password and a computer name. Step by step on the
[download page](https://defcom5-rockchip.github.io/pi-desktop/download.html).

**Installing to eMMC? Read [EMMC-INSTALL.md](EMMC-INSTALL.md) first** (written for the 5B's onboard eMMC; untested on the 5 Plus). Copying a root filesystem onto eMMC does
*not* install a bootloader — the guide covers the correct install, wiping a factory-Android eMMC, and maskrom
recovery.

## The recipe is public

**Built on Armbian, by hook.** Plaid Pi images are built from pinned Armbian build-framework inputs. Armbian owns
the RK3588 boot chain: U-Boot, partition layout, kernel branch, device trees, modules, firmware, board-support
packages and the first-boot filesystem resize. Plaid Pi changes the kernel there: upstream stable 6.1.191 (6.5) or 6.1.189
(6.0) merged into Armbian's branch, and its own patches (25 in 6.5, numbered 0001–0028 with gaps; 0001–0017 in 6.0) applied
through Armbian's own `userpatches` mechanism, all published with the source. Everything else is a userspace layer
applied through Armbian's late image hook, `customize-image.sh`, and every line of it is in the recipe repository.
The images are credential-free: there is no default user or password, the first boot creates yours.

Don't trust the image; audit the recipe.

- **Recipe** — the Armbian `userpatches/` layer: build config, customize script, overlay (every service, rule
  and script), kernel patches, package build scripts:
  [defcom5-rockchip/pi-desktop-recipe](https://github.com/defcom5-rockchip/pi-desktop-recipe).
- **Kernel** — Armbian's `rk-6.1-rkr7.2` at commit `44bbd021` merged with upstream stable v6.1.191 (6.5: tag
  `pi-desktop-6.5`, the Plaid Pi patches applied from the recipe at build time) or v6.1.189 (5.0–6.0: tag `pi-desktop-5.0`,
  all but 0016 also as commits on the tag; 0016 patches drivers Armbian adds at build time; 4.0: tag `pi-desktop-4.0`):
  [defcom5-rockchip/linux-rockchip](https://github.com/defcom5-rockchip/linux-rockchip).
- **Video driver** — [defcom5-rockchip/rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi),
  release [v4.0.0](https://github.com/defcom5-rockchip/rockchip-vaapi/releases/tag/v4.0.0).

[LEGAL.md](LEGAL.md) has the complete corresponding-source table, the licences and the trademark notes.

## The family

- **[Pi Studio](https://github.com/defcom5-rockchip/pi-studio)** — the audio-production sibling (Sonic Pi, Ardour,
  an offline NPU AI music copilot); its next release moves onto this kernel base.
- **Naked Network Pi** — the stripped, headless base *(coming)*.

## Credits

Built from Ubuntu 26.04 packages with the Armbian build framework, on Armbian's Rockchip vendor kernel; not
affiliated with or endorsed by Canonical or Armbian. The 1.x and 2.0.x line stood on the
[ubuntu-rockchip](https://github.com/Joshua-Riek/ubuntu-rockchip) project — thank you.
Code GPL-3.0-or-later, artwork CC BY-SA 4.0. By defcom5-rockchip, with Claude as co-engineer.
