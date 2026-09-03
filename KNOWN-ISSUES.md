# Known Issues

Pi Desktop is a niche distro with a single maintainer, running on the vendor 6.1 BSP kernel.
This file exists on purpose. We'd rather tell you what's rough than let you find out.

Current as of **v2.0.1**.

---

## VID-1: Chromium-family browsers flicker; Firefox doesn't — so Firefox is the default

**Status:** root-caused, unfixable on this GPU stack · **Severity:** cosmetic · **Affects:** Chromium, Chrome (any window size)

We finally ran this to ground: modern Chromium renders through ANGLE (mandatory since ~M117),
and ANGLE on this image's GPU stack (panfork/Mesa 23) flickers — typing carets, UI redraws,
independent of resolution or refresh rate. No flag fixes it without breaking something else.

**What we did about it:** made Firefox the default — Gecko/WebRender doesn't use ANGLE, so it's
flicker-free *and* hardware-decodes video. Chrome ships pre-flagged for its one job (DRM
streaming at DRM's own 720p cap, where its flicker-fix flag costs nothing). Chromium stays
available for those who want it, flicker and all. The real cure is the mainline GPU stack
(Panthor + current Mesa) — see *Scope and horizon*.

---

## VID-2: AV1 is software-decoded in browsers

**Status:** won't fix on this stack · **Severity:** performance

The browser hardware paths (rkmpp V4L2 and our VA-API driver) carry H.264, VP8 and VP9 — no AV1.
YouTube increasingly serves AV1; if a 4K video pins your CPU, check *Stats for nerds* for `av01`.

**Workaround:** an extension that blocks AV1 (enhanced-h264ify, tick only "Block AV1") keeps
streams on the hardware decoder. The RK3588's AV1 block is real — the ffmpeg-rockchip stack can
use it — but no browser path reaches it on this kernel.

---

## VID-3: HEVC and 10-bit content are not hardware-decoded (on purpose, for now)

**Status:** in active development ("Deep Ink") · **Severity:** performance

Straight story: the community VA-API driver this image inherited *claimed* HEVC and 10-bit
support, but had never actually decoded them correctly for anyone — HEVC produced a solid green
frame at every bit depth; 10-bit VP9 produced corruption. Our v2.0 driver stops advertising what
it can't deliver, so players and media servers route to their working fallbacks automatically:
software decode locally (correct picture, more CPU), server-side transcode in Jellyfin-style
setups (the rkmpp transcode path handles HEVC fine).

Progress is real and public: the 10-bit conversion fix is written and hardware-verified
(the NV15→P010 work in the driver repo), and the missing HEVC bitstream assembler is scoped.
They return to the menu one codec at a time, as each is verified on hardware.

---

## VID-4: 10-bit video files can show a blue screen in mpv

**Status:** fix verified, ships in v2.0.2 · **Severity:** playback failure on 10-bit files

The GPU stack advertises 16-bit texture support it can't actually render, so 10-bit frames
uploaded by mpv 0.38 display as a solid blue field. (8-bit content — virtually all web video —
is unaffected.)

**Workaround until 2.0.2:** play the file with the older engine, which converts internally:
`/usr/bin/mpv <file>` — or add `--vf=format=yuv420p` to mpv 0.38. The 2.0.2 config does this
automatically, only for 10-bit content.

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

## Scope and horizon

Pi Desktop is built on the **vendor 6.1 BSP kernel**, because that is what does 4K@120 and
hardware video on this SoC *today*. The mainline world is moving fast — RK3588 H.264/HEVC
decoders are merged, 10-bit HDMI output is in review, and a community forward-port of the vendor
media stack onto Linux 6.18 already exists. When that path matures for this board, VID-1, VID-4
and GPU-1 all fall at once (Panthor + modern Mesa), and this file gets a lot shorter. Until
then, the fork is the reason the features work.

---

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
