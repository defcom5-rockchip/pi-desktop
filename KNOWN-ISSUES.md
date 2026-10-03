# Known Issues

Pi-Desktop is a niche distro with a single maintainer, running on the Rockchip vendor 6.1 kernel.
This file exists on purpose. We'd rather tell you what's rough than let you find out.

Current as of **4.0 Plaid** (release candidate test9 on an Orange Pi 5B, 2026-10-03). The 2.0.x
sections further down are the retired line's history and stay as they were.

---

## 4.0 Plaid

### PWR-1: No suspend

**Status:** disabled on purpose · **Severity:** feature missing

Suspend, hibernate and hybrid sleep are masked and GNOME's automatic suspend is off, as in 2.0.x.
The 2.0.x Ethernet-after-sleep fix is in this kernel, but the sleep device-tree configuration and a
resume soak were never done on this image, and shipping sleep untested is worse than shipping none.
Shut down, or let the screensaver run.

---

### VID-5: 10-bit video is hardware-decoded in Chromium, software in Firefox

**Status:** measured on the 4.0 release image, 2026-10-03 · **Severity:** performance · **Affects:** 10-bit files

This is the reverse of what 2.0.x did, so it is worth stating with the numbers. A 4K HEVC **Main 10** file, played
full-screen for 20 s on the release image (driver 2.2.0):

| | Chromium 154 (in the image) | Firefox 156 (first-click install) |
|---|---|---|
| frames decoded / dropped | **595 / 0** | 571 / **26** |
| CPU | **~21 % across its processes** | 229 % + 121 % |
| video engine in use | **yes** | no |

- **Chromium and Chrome decode 10-bit HEVC on the video engine.** So does 8-bit HEVC, H.264 and VP9. Jellyfin's web
  client direct-plays 4K Main 10 in Chromium with no server transcode — nothing to install, it is the default browser.
- **Firefox 156 decodes 10-bit in software.** Its own log says `IsHardwareAccelerated=false` → "Using preferred
  software codec hevc", then drops frames. The VA-API support is compiled in and libva is installed, so something in
  this build or in Firefox 156 refuses the path that Firefox 155 took on 2.0.x. Under investigation; use Chromium for
  video until it is fixed.
- **AV1** is software in both; the chip's AV1 block has no browser path. Firefox has AV1 switched off on purpose so
  YouTube serves VP9, which is hardware-decoded.
- **HDR** files play but look pale in both browsers: neither does HDR tone mapping on Linux (DSP-2). mpv is the HDR
  player.

Evidence, including the test clip and the decoder log, is kept with the project notes.

---

### DSP-2: HDR looks pale — on the monitor we have, it is the monitor

**Status:** monitor-side · **Severity:** cosmetic

GNOME's HDR switch produces a correct HDR10 signal (EOTF ST 2084, BT.2020, static metadata — read back
from the driver), but the maintainer's 32-inch 300-nit "HDR ready" monitor (EDID name "XHS XR32UMH")
never switches into HDR and the picture goes pale. GNOME 50.4/50.5's HDR-metadata fix was backported
onto Ubuntu's mutter and tested: the metadata is now complete, the monitor still does not enter HDR.
Leave HDR off. A report from a monitor that is known to enter HDR from another source would settle it.

---

### USB-1: Bus-powered optical and hard drives need the combo USB port or a powered hub

**Status:** hardware · **Severity:** informational

Measured with a bus-powered USB Blu-ray writer: the USB-C port and the top USB 3.0 port could not power
it; the lower "USB 2.0 / shared with Type-C" port can — and that port is USB 2.0 only. From the
schematic: the combo port's 5 V comes straight from the board rail with no limiter; the stacked
USB 3.0 + USB 2.0 pair shares one 1 A limit; USB-C gets about 1.45 A. Not fixable in software. For
full speed and for burning, use a powered USB 3 hub.

---

### DSP-1: Two display outputs; a passive HDMI-to-DisplayPort adapter cannot work

**Status:** hardware, not a bug · **Severity:** informational · **Affects:** DisplayPort-only monitors

The Orange Pi 5B has exactly two display outputs: the **HDMI 2.1 port** and **DisplayPort over
the USB-C port** (DP alt mode). There is no eDP: the chip's eDP lanes share the HDMI pins and go
to the HDMI connector (schematic page 8, "HDMI TX/eDP1.3 MUX Port0").

- **HDMI port → DisplayPort monitor: a passive adapter cannot work**, on this or any computer.
  HDMI cannot speak DisplayPort. You need an **active HDMI-to-DisplayPort converter** (usually
  USB-powered). 4K@120 through such a converter is not something we have verified.
- **USB-C → DisplayPort cable** is the native DP path. **USB-C → HDMI** adapters are active
  converters and ride the same path. On the RK3588 that path is **DP 1.4**: 4K@60 RGB, or
  4K@120 only at 4:2:0. 4K@120 RGB is the HDMI port's job.
- **We have not yet tested video over USB-C** on Pi-Desktop. The driver is enabled and probes
  the port on hotplug. Reports with cable or adapter model welcome.

---

### BOOT-3: The first boot logs you in without asking; every boot after that shows a login screen

**Status:** by design (Armbian's setup wizard) · **Severity:** cosmetic

The first-login wizard enables autologin for the session it creates; a Pi-Desktop service turns it
off at the next boot. Verified on the release candidate: the wizard boot auto-logs-in, the second boot
asks for the password. If you need the login screen before ever rebooting, reboot once.

---

### APP-1: What is not in the image, and why

**Status:** decisions · **Severity:** informational

- **VLC** is out: on this image VLC 3's video path is software-only (it runs under XWayland without the GPU),
  so it only duplicated mpv, badly. mpv is the video player, Rhythmbox the music player.
- **PeaZip** is not shipped (a build exists; decision pending). Archive Manager (file-roller) is in.
- **VS Code** is not offered: Microsoft's build has telemetry on by default. VSCodium is the
  first-click alternative.
- **Firefox, LibreOffice, GIMP, Inkscape, Thunderbird, VSCodium and Google Chrome** install on first
  click from their own repositories and need the network for that. Six of the seven installers ran on
  hardware (45 s to 5 min on the test card); the Firefox installer is the newest and has not been
  timed yet. LibreOffice comes without Base (it drags in Java); `sudo apt install libreoffice-base`
  adds it.
- **libdvdcss** and AACS keys are not shipped. If a DVD refuses to play: `sudo apt install libdvd-pkg`.

---

### APP-2: mpv cannot open smb:// links

**Status:** Ubuntu build option · **Severity:** minor

Ubuntu builds mpv without SMB support. Opening the same file from Files works: Files hands mpv a local
gvfs path.

---

### DSP-3: GNOME picks 125% scale on a 32-inch 4K monitor; at 120 Hz that costs video frames

**Status:** GNOME default kept · **Severity:** performance at 4K@120 only

GNOME's own maths (138 dpi against a 110 dpi target) chooses 125%. At 4K@120, fractional scaling
made mpv drop about 211 frames in 19 s against 1 at 100%. The How-To on the desktop shows the
alternative: Scale 100% plus Settings ▸ Accessibility ▸ Seeing ▸ Text Size.

---

### PWR-2: Screensaver and lock notes

**Status:** open, low · **Severity:** cosmetic

- GNOME's own screen blank is locked off (Settings ▸ Power shows a Screensaver group instead). A
  blank at 4K@120 hard-hung the board before kernel patch 0010; the fix is in, the fence stays.
- Locking — Super+L, or when the screensaver ends with "Ask for Password Afterwards" on — turns the
  monitor off for a few seconds and relights it on the next input. That is stock GNOME, and since 0010
  it is safe at 120 Hz (this exact path used to hang the board; re-tested 2026-09-25).
- The screensaver's mpv has crashed after many hours of looping, with a kernel "GPU activity takes
  longer than time interval" message alongside. The service restarts it (at most 3 times per 10 min).
  Cause not found.

---

### AUD-1: Analog jack: 192 kHz is resampled to 96 kHz; headset microphone untested

**Status:** hardware ceiling / untested · **Severity:** informational

The ES8388 codec tops out at 96 kHz; 44.1, 48 and 96 kHz follow the file, 192 is resampled. The
jack's headset-microphone input has not been tested (needs a 4-pole headset).

---

### BT-1: Bluetooth media buttons may not reach the browser

**Status:** untested side effect · **Severity:** minor

bluez's mpris-proxy is disabled because it relayed every YouTube ad as "stopped" and the Bluetooth
speaker muted for ten seconds. mpris-proxy is also what forwards play/pause from Bluetooth headsets to
desktop players, so those buttons may no longer work. Not measured; reports welcome.

---

### SYS-1: systemd-oomd margin on slow SD cards

**Status:** mitigated · **Severity:** rare

A big copy in Files used to get the whole session killed (Files ran inside the session bus's cgroup;
systemd issue #35270). Files now runs in its own unit and unwritten data is capped at 256 MB. In the
proof run — a multi-GB NAS-to-SD copy with Rhythmbox starting mid-copy — the session survived, but
memory pressure peaked at 44% against oomd's 50% threshold. Thin margin on a slow card; eMMC not
measured.

---

### SYS-2: Kernel log noise

**Status:** cosmetic · **Severity:** none

The vendor kernel logs the same harmless lines on every boot (venc devfreq, rkvdec2 "niu" resets,
debugfs duplicates, dhd prealloc, HS200 clock, vop2 OPP), and `dw-dp fde50000.dp: AUX timeout` about
ten times per HDMI hotplug (it probes the empty USB-C DisplayPort). None of these is an error.

---

## 2.0.x — the retired line (history, unchanged)

## VID-1: Chromium-family browsers flicker; Firefox doesn't — so Firefox is the default

**Status:** two causes found; one fixed in 2.0.3, one is the GPU driver · **Severity:** cosmetic · **Affects:** Chromium, Chrome

This was two different faults that looked like one, which is why every single flag "almost" fixed it:

1. **Typing / text-field flicker — fixed in 2.0.3.** It only appeared when mutter put the browser's
   buffer straight onto a hardware plane (direct scanout). 2.0.3 composites those surfaces
   instead (`/etc/environment.d/60-pidesktop-mutter.conf`, `MUTTER_DEBUG=disable-direct-scanout`).
   Measured on hardware 2026-08-27; the split was confirmed 2026-09-22 when the same browser ran
   clean on a stack that composites by default. Cost: one extra composite pass for fullscreen
   windows. Delete the file to get direct scanout back.
2. **Thumbnail / image flicker — not fixable on this stack.** Modern Chromium renders through
   ANGLE (mandatory since ~M117), and ANGLE on panfork Mesa 23 mis-presents partial updates. The
   same Chromium on the ARM blob driver is clean, so it is the GL driver, not the browser and not
   the kernel (the kernel logged nothing during 30 s of reproduced flicker, and Chromium 114 was
   clean on the same kernel where 132 flickered).

**What we did about it:** Firefox is the default — Gecko/WebRender doesn't use ANGLE, so it's
flicker-free *and* hardware-decodes video. **Chrome ships flicker-free by default** (GPU compositing
off, which also means no hardware video in that launcher — 4K60 VP9 there is CPU decode). A second
launcher, **"Google Chrome (Hardware Video)"**, runs with GPU compositing and VA-API decode (see VID-4)
and shows (2) on image-heavy pages. Chromium 153 runs the hardware-video way and shows (2) too. The
cure for (2) is a different GL driver, which is what the successor image uses — see *Final release*.

---

## VID-2: AV1 is software-decoded in browsers

**Status:** won't fix on this stack · **Severity:** performance

The browser hardware paths (rkmpp V4L2 and our VA-API driver) carry H.264, VP8 and VP9 — no AV1.
YouTube increasingly serves AV1; if a 4K video pins your CPU, check *Stats for nerds* for `av01`.

**Workaround:** an extension that blocks AV1 (enhanced-h264ify, tick only "Block AV1") keeps
streams on the hardware decoder. The RK3588's AV1 block is real — the ffmpeg-rockchip stack can
use it — but no browser path reaches it on this kernel.

---

## VID-3: 10-bit video at 4K60 is not yet smooth in Firefox

**Status:** open, next driver phase · **Severity:** performance (10-bit 4K60 only)

As of 2.0.2, 10-bit HEVC and VP9 Profile 2 are hardware-decoded and displayed zero-copy in
Firefox and mpv (see *Fixed in v2.0.2*). One cost remains: the driver still repacks every
10-bit frame from the video engine's packed format to P010 on the CPU. That fits 4K at 30 fps
(4K Main10 features play in hardware end to end) and 1080p at any rate; at 4K **60** it can
stutter. Moving that step to the RGA hardware is measured at under 5 ms per 4K frame and is
the next driver phase. 8-bit content is unaffected.

Also expected, not a bug: HDR (PQ/BT.2020) files look pale in Firefox because Firefox on Linux
does no HDR tone mapping, whichever decoder produced the pixels. mpv tone-maps them correctly.

---

## VID-4: 10-bit HEVC in Chrome/Chromium is not hardware-decoded; 10-bit VP9 is

**Status:** partly fixed in 2.0.3 · **Severity:** performance · **Affects:** Chrome 153 (Hardware Video launcher), Chromium 153

2.0.3 moves Chromium from the V4L2 plug-in lane (132) to the same VA-API lane as Chrome and
Firefox (xtradeb 153). Measured on hardware: HEVC 8-bit, H.264 and **VP9 Profile 2 (10-bit) at
4K60** decode on the VPU in both browsers. HEVC Main10 is the exception: Chromium's own policy
does not attempt hardware HEVC 10-bit on Linux, so those files fall back to software or, in
Jellyfin, to a transcode. Firefox and mpv play HEVC Main10 in hardware, zero-copy.

---

## BOOT-2: On the very first boot, ssh takes about four minutes to come up

**Status:** fixed in 2.0.3 · **Severity:** first boot only

The image ships without ssh host keys (correct: every install gets its own), and a service
generates them on first boot. sshd starts before that finishes, fails, and systemd backs off
("start request repeated too quickly") until the keys exist, then it starts and stays up. On
image 5's first boot that window was 17:47 to 17:52; the second boot had zero failures. If you
install headless and ssh refuses at first, wait five minutes before assuming the worst. 2.0.3
makes key generation a hard prerequisite of sshd (the 2.0.1 unit only ordered itself `Before=`,
which socket-activated sshd ignored).

---

## BOOT-1: First boot is busy for a couple of minutes

**Status:** open · **Severity:** cosmetic

First boot runs the one-time setup and indexing. One systemd unit (`oem-config`) reports
"failed" on that first boot as it tears itself down — it's gone by the second boot, which comes
up with **zero failed units**. Give it a few minutes once; it does not recur.

---

## GPU-1: Rare GPU driver crash under heavy compositing

**Status:** open, upstream (vendor driver) · **Severity:** rare but disruptive

The vendor Mali kernel driver (`kbase`) has a use-after-free on GPU-context teardown that can
crash the desktop session. Observed once, recovered on its own. Exposure exists on every RK3588
image with this kernel; the permanent fix is mainline's Panthor driver.

---

## Fixed after v2.0.2, in the recipe only (never released — the line was retired first)

**v2.0.2 is the last published Pi Desktop image on this base.** The fork's final commit carries the
fixes below for anyone who builds it (`./build.sh --board=orangepi-5b --suite=noble --flavor=desktop`);
none of them was baked into a released image.


- **VID-1 (1)**: the Chromium typing/text-field flicker — mutter composites browser surfaces instead
  of direct-scanning them out.
- **Hardware video in Chrome and Chromium** (VID-4): a new "Google Chrome (Hardware Video)" launcher
  with the VA-API flag set plus a render-node override (the NPU registers as a second render node and
  Chrome picked it); the default Chrome launcher stays flicker-free. Chromium moves to xtradeb 153 on
  the same VA-API lane. HEVC 8-bit, H.264 and 10-bit VP9 at 4K60 measured on the VPU in both.
- **Ethernet dead after a long sleep** (kernel): `stmmac_resume()` started phylink before the MAC
  reset, so the Motorcomm YT8531 PHY came up with a corrupt advertisement (ANAR 0x0de0) and never
  linked. Backported the upstream reorder plus a PHY re-init on resume; 7/7 long-sleep wakes clean at
  1 Gbps on hardware (2026-09-08). Merged into Armbian's vendor kernel from our report the next day.
  *Suspend itself stays masked in this release* — the sleep-capable device tree it was soaked with
  was not baked in time, and shipping suspend without it would be untested. The fix is in the
  kernel for anyone who enables sleep by hand.
- **Bluetooth SCO use-after-free** (CVE fix that had been reverted in 1.x for a build conflict):
  re-applied, open-coded for this kernel. Kernel 6.1.0-1027.27 build of 2026-09-08.
- **BOOT-2**: sshd (service and socket) now hard-depends on the host-key generation unit, so it
  cannot start before the keys exist — a fresh install answers ssh on the first boot.
- **Video driver v2.1.5**: surfaces described by their created bit depth until decoded (fixes the
  "garbled, then green" 10-bit start in VA-API Chromium builds); libva finds the driver on
  Panthor/Panfrost stacks without `LIBVA_DRIVER_NAME`.
- **mpv**: `video-sync=display-resample` — 4K60 zero-copy went from 605 dropped frames in 900 to 0.
- **AP6275P Bluetooth**: the boot-race fix and the coexistence timing values, unchanged since 1.0,
  now documented as the canonical set (they were nearly lost in the successor image).

Not done, and now closed with the release: the RGA repack for 10-bit at 4K60 in Firefox (VID-3),
real 10-bit in the Chromium plug-in (VID-4), the ARMED-orange mpv icon.

---

## Final release — retired at v2.0.2

**Pi Desktop on Joshua Riek's `ubuntu-rockchip` is retired. v2.0.2 is its last image.** That project
is archived upstream; this fork kept it alive for the Orange Pi 5B for a year and goes out working:
4K, hardware video in both browsers, Bluetooth that survives WiFi, a kernel patched by hand for the
CVEs that mattered to a desktop. Thank you, Joshua — none of this existed without the base you built.

The successor is **Pi-Desktop 3.0** on the Armbian build framework: Ubuntu 26.04, GNOME 50, and
Rockchip's vendor kernel as Armbian tracks it (6.1.172 at the time of writing, against 6.1.75 here).
That trades a hand-maintained kernel for one that inherits a hundred stable releases of fixes — the
Ethernet-after-sleep fix from this fork is already in it, merged upstream — and Mesa panfork for the
ARM GL driver, which is what makes Chromium flicker-free there. What it does not do yet: Firefox 8-bit
video in hardware (the ARM driver lacks a two-channel 8-bit import format), and 4K@120 as a default.
Follow it in this organisation's repositories when it leaves test.

## Scope and horizon

Pi Desktop 2.x is built on the **vendor 6.1 BSP kernel** (6.1.75 base), because that is what did
4K@120 and hardware video on this SoC when the line started. The vendor line continues in
Pi-Desktop 3.0 on Armbian's much newer drop of the same kernel; the mainline world (Panthor, current
Mesa, merged RK3588 decoders) remains the long-term destination and is where VID-1 (2), VID-4 and
GPU-1 all fall at once.

---

## Fixed in v2.0.2

*Release soak: two boots on an Orange Pi 5B, swept against the 2.0.1 journal baseline — zero hand repairs, 0 failed units on the second boot, error-level message count identical to 2.0.1.*

- **HEVC is hardware-decoded — including in the browser.** The driver gained a real HEVC
  bitstream assembler (v2.1.0), verified pixel-identical to software decode; Firefox ships
  with HEVC enabled by policy. A 4K Main10 feature plays in hardware start to finish.
- **10-bit works: HEVC Main10, H.264 High 10 and VP9 Profile 2 are advertised and display
  zero-copy** in Firefox and mpv. The "GPU stack can't render 16-bit" story from 2.0.1 was
  wrong: the driver exported the chroma plane with a mistyped format code ("GR16", which does
  not exist). One byte (driver v2.1.3). Jellyfin's web client direct-plays 10-bit in Firefox
  in hardware; measured, not assumed.
- **H.264 ghosting with B-frames** (broadcast-style streams drifted after the first dozen
  frames): the synthesized PPS hard-coded a reference-count default real encoders don't use.
  Fixed in driver v2.1.2 by learning the value from the stream.
- **Chromium no longer loses its GPU process on 10-bit files** (see VID-4) and now plays 8-bit
  HEVC in hardware: our fork of the libv4l-rkmpp plug-in plus the HEVC flag.
- **Chrome/Chromium with the VA-API decoder ran out of decode surfaces at 4K** (garbled, then
  green). Pool raised, allocation failures now fail cleanly (driver v2.1.4).
- **mpv defaults to zero-copy VA-API** (`hwdec=vaapi,vaapi-copy`); the 10-bit "guard" profile
  from 2.0.1 is gone — it also fired on hardware-decoded 10-bit and quietly converted it to
  8-bit on the CPU.
- **Silent audio after a Bluetooth drop:** the board's unused analog codec (ES8388, silent on
  this kernel) could become the default sink. It is now deprioritized in WirePlumber.
- **Kernel: RGA driver 1.3.13** (from Rockchip's develop-6.1), the newest on any RK3588 image.
- **Leaner:** LibreOffice is Writer and Calc (Impress/Draw and an unused icon theme out,
  ~35 MB); Remmina, Shotwell and btop removed; `iw` added.

## Fixed in v2.0.1

- **Odd-resolution VP9 green screen.** Academy-ratio and other non-standard-width VP9 videos
  (e.g. 2970×2160 film-scan uploads) hardware-decoded to a solid green frame. Root cause: the
  driver exported frames at a 16-aligned stride where the GPU importer expects 64. Fixed in our
  driver fork (v2.0 "Reframe"), with a public reproducer.
- **The driver's codec menu now tells the truth** (see VID-3) — no more green walls from codecs
  that never worked; correct fallbacks instead.
- **Firefox default, flicker-free, hardware-decoded** — with enterprise-policy prefs that survive
  Firefox updates. Chrome scoped to DRM duty (Netflix verified). Vivaldi dropped (its Widevine
  can't legally ship; Chrome bundles its own).
- **mpv 0.38** — window drag-and-drop finally works on Wayland.
- **Betterbird actually installed** — v1.0's mail default pointed at a package that never made it
  into the image. Now real.
- **Pro Updates ships in-image** and reports green out of the box (apt sources moved to https).
- **A dormant second OS partition no longer appears in the Files sidebar.**
- *(driver nerd note)* `vaDeriveImage` used to return an empty buffer, so every "copy-back"
  video path silently produced black/zeros. It now fails honestly and clients use the working
  path instead.

## Reporting

Found something not listed here? [Open an issue](../../issues) — include your board revision, how
you installed (SD / eMMC / NVMe), your monitor's resolution and refresh rate, and `uname -a`.
If it's video-related, a screenshot of `about:support` (Firefox) or `chrome://gpu` is worth a
thousand words.
