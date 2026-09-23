# Known Issues

Pi Desktop is a niche distro with a single maintainer, running on the vendor 6.1 BSP kernel.
This file exists on purpose. We'd rather tell you what's rough than let you find out.

Current as of **v2.0.1**.

---

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
flicker-free *and* hardware-decodes video. Chromium stays available; after 2.0.3 it flickers only
on image-heavy pages. The cure for (2) is a different GL driver, which is what the successor
image does — see *Final release*.

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

## VID-4: 10-bit files in the image's Chromium are transcoded, not hardware-decoded

**Status:** by design for now · **Severity:** convenience (Jellyfin-style setups transcode 10-bit)

The `+rkmpp` Chromium decodes through a V4L2 plug-in (libv4l-rkmpp), not through our VA-API
driver, and that plug-in outputs 8-bit NV12 only. Before 2.0.2 it *advertised* 10-bit anyway
and aborted Chromium's whole GPU process on the first 10-bit frame (a white/black flash). Our
fork of the plug-in, shipped in 2.0.2, hides the 10-bit profiles and removes the abort:
Chromium now reports 10-bit unsupported, so a media server transcodes it and 8-bit HEVC plays
in hardware. Real 10-bit in Chromium needs the same RGA conversion as VID-3, inside the
plug-in. **Firefox is the browser to use for 10-bit** — it direct-plays it in hardware.

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

## Fixed in v2.0.3 "Crystal Blue Persuasion" — the final release on this base

- **VID-1 (1)**: the Chromium typing/text-field flicker — mutter composites browser surfaces instead
  of direct-scanning them out.
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

## Final release

**2.0.3 "Crystal Blue Persuasion" is the last Pi Desktop built on Joshua Riek's `ubuntu-rockchip`.** That project is archived;
this fork kept it alive for the Orange Pi 5B for a year, and it goes out working: 4K, hardware
video in both browsers, Bluetooth that survives WiFi, a kernel patched by hand for the CVEs that
mattered to a desktop. Thank you, Joshua — none of this existed without the base you built.

The successor is **Pi-Desktop 3.0** on the Armbian build framework: Ubuntu 26.04, GNOME 50, and
Rockchip's vendor kernel as Armbian tracks it (6.1.172 at the time of writing, against 6.1.75 here).
That trades a hand-maintained kernel for one that inherits a hundred stable releases of fixes, and
the ARM GL driver for Mesa panfork — which is what makes Chromium flicker-free there. What it does
not do yet: Firefox 8-bit video in hardware (the ARM driver lacks a two-channel 8-bit import format)
and 4K@120 as a default. Follow it in this organisation's repositories when it leaves test.

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
