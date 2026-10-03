# Pi-Desktop
### The Orange Pi 5B, as a desktop computer.

**Pi-Desktop 4.0 "Plaid"** · [Download](#download) · [Known issues](KNOWN-ISSUES.md) · [Changelog](CHANGELOG.md) ·
[Privacy](PRIVACY.md) · [Legal](LEGAL.md) · [Security](SECURITY.md) · [Website](https://defcom5-rockchip.github.io/pi-desktop/)

> The 2.0.x line on `ubuntu-rockchip` was retired at v2.0.2 (2026-09-23); 4.0 is its successor on a new base — see the [CHANGELOG](CHANGELOG.md).

A GNOME 50 desktop image for the Orange Pi 5B (RK3588S), built from Ubuntu 26.04 packages with the Armbian
build framework, on Armbian's Rockchip vendor kernel 6.1.172 plus eleven Pi-Desktop patches. Not affiliated
with or endorsed by Canonical or Armbian. Not a Raspberry Pi product.

## What it is

Everything below was checked on an Orange Pi 5B running the release candidate (test9, 2026-10-03: 78 of 79
first-boot checks passed; the one miss was a setting the tester had changed himself).

- **4K at 120 Hz over HDMI, out of the box.** The vendor kernel reaches HDMI 2.1 FRL with our clock patches.
  The three ways it failed on the way there — a black screen after 14 minutes to 3 hours, a hard hang when the
  screen turned off at 120 Hz, a black screen after the monitor was power-cycled — are fixed by kernel patches
  0008, 0010 and 0011, each re-tested on hardware. One boot argument hides 120 Hz again; 4K@60 needs none of this.
- **A flicker-free desktop and browser.** Arm's Mali user-space driver instead of Mesa panfork, and AFBC scanout
  rejected in the kernel (patch 0001). The Chromium typing and thumbnail flicker of 2.0.x is gone.
- **Hardware video.** Our [rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi) driver, 2.2.0:
  8-bit H.264, HEVC and VP9 in Chromium (and in Chrome once installed); 10-bit HEVC and VP9 in Firefox once
  installed; mpv 0.41 through VA-API, with profiles that keep DVDs and Blu-rays from stuttering at 120 Hz.
  AV1 is software.
- **Wi-Fi joins WPA3 and WPA2/WPA3 networks.** Ubuntu 26.04's wpa_supplicant signals WPA3 in a way the vendor
  Wi-Fi driver never handled, so every modern home network refused the board. Patch 0006 fixes the driver
  ([armbian/linux-rockchip#561](https://github.com/armbian/linux-rockchip/pull/561)); Wi-Fi 6 rates measured on 5 GHz.
- **Bluetooth that survives Wi-Fi.** The two AP6275P fixes from 2.0.x — the boot race and the coexistence
  timing — are carried over with the self-healing sentinel. The ten-second mute at every YouTube ad on a
  Bluetooth speaker is gone (bluez's mpris-proxy was telling the speaker "stopped"; it is off).
- **Audio.** PipeWire 1.6.9 and WirePlumber 0.5.17 with real-time priority from boot. The 3.5 mm jack works
  (the vendor codec driver shipped it switched off) and follows plugging like a laptop; HDMI is the default
  output and the outputs are named. Rhythmbox 3.5.1 — newest upstream, every feature on — Audacity 3.7.8 and
  Brasero.
- **A screensaver instead of screen blanking**, with its own group in Settings ▸ Power; lock on demand; a login
  screen at every boot after the first; Log Out back in the power menu.
- **Fixes you would otherwise find yourself.** GUI password prompts hang on Ubuntu 26.04 with a 6.1 kernel
  (polkit's new helper wants Linux 6.5) — fixed. A large file copy could get the whole session killed by
  systemd-oomd — fixed. The first boot asks for a computer name instead of calling every board "pidesktop".
- **A small image; apps on first click.** About 1.5 GB compressed. Firefox, LibreOffice, GIMP, Inkscape,
  Thunderbird, VSCodium and Google Chrome sit in the app grid (Chrome and Firefox in the dock too) as
  placeholders that download the real app from its own repository on the first click, then step aside. Six of
  the seven were timed on hardware, 45 seconds to five minutes; Firefox's is the newest and still owes that
  run. VS Code is not offered: Microsoft's build ships with telemetry on.
- **No snaps. No telemetry of our own.** What the Ubuntu and Armbian base still talks to is listed, with what we
  found, in [PRIVACY.md](PRIVACY.md).
- **Updates that cannot undo the work.** Ubuntu packages update from Ubuntu's repositories. The kernel, device
  tree, bootloader, BSP and firmware packages are held and change only with Pi-Desktop releases — otherwise
  Armbian's next release would replace the kernel and silently drop all eleven patches.

## Honest scope

One maintainer, a vendor 6.1 kernel, best effort. It ships a [KNOWN-ISSUES](KNOWN-ISSUES.md) file on purpose —
we'd rather tell you what's rough than let you find out. Suspend is disabled. 10-bit video is Firefox-only.
HDR on the maintainer's monitor stays pale, and that is the monitor. The board has two display outputs, HDMI 2.1
and DisplayPort over USB-C, nothing else; a passive HDMI-to-DisplayPort adapter cannot work on any computer
([DSP-1](KNOWN-ISSUES.md#dsp-1-two-display-outputs-a-passive-hdmi-to-displayport-adapter-cannot-work)).

Security updates for the Ubuntu packages come from Ubuntu's repositories for the life of 26.04. The kernel and
boot packages are built from Armbian's branch and refreshed with Pi-Desktop releases for as long as Armbian
builds that branch. Pi-Desktop's own packages are updated with releases. There is no guaranteed support period
and no commercial product behind this. [SECURITY.md](SECURITY.md) says how to report a problem.

## Download

**4.0 Plaid** — [Releases](../../releases): one compressed image and its `.sha256`. Verify the checksum, write
the image to a microSD card or the eMMC (GNOME Disks, balenaEtcher or `dd`), boot. The first boot grows the
filesystem to the card, then asks for a user, a password and a computer name. Step by step on the
[download page](https://defcom5-rockchip.github.io/pi-desktop/download.html).

**Installing to eMMC? Read [EMMC-INSTALL.md](EMMC-INSTALL.md) first.** Copying a root filesystem onto eMMC does
*not* install a bootloader — the guide covers the correct install, wiping a factory-Android eMMC, and maskrom
recovery.

## The recipe is public

**Built on Armbian, by hook.** Pi-Desktop images are built from pinned Armbian build-framework inputs. Armbian owns
the RK3588 boot chain: U-Boot, partition layout, kernel branch, device trees, modules, firmware, board-support
packages and the first-boot filesystem resize. Pi-Desktop changes one thing there, eleven kernel patches applied
through Armbian's own `userpatches` mechanism, all published with the source. Everything else is a userspace layer
applied through Armbian's late image hook, `customize-image.sh`, and every line of it is in the recipe repository.
The images are credential-free: there is no default user or password, the first boot creates yours.

Don't trust the image; audit the recipe.

- **Recipe** — the Armbian `userpatches/` layer: build config, customize script, overlay (every service, rule
  and script), kernel patches, package build scripts:
  [defcom5-rockchip/pi-desktop-recipe](https://github.com/defcom5-rockchip/pi-desktop-recipe).
- **Kernel** — Armbian's `rk-6.1-rkr7.2` at commit `44bbd021` with the eleven patches applied, tag
  `pi-desktop-4.0`: [defcom5-rockchip/linux-rockchip](https://github.com/defcom5-rockchip/linux-rockchip).
- **Video driver** — [defcom5-rockchip/rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi).

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
