# Changelog

## Unreleased — post-2.0.2 fixes in the recipe (the line was retired at v2.0.2; nothing below shipped as an image)
- Hardware video in Chrome via a second launcher, "Google Chrome (Hardware Video)" (VA-API + render-node
  override); the default Chrome launcher stays flicker-free (software compositing, CPU video). Chromium moves
  from liujianfeng 132 (V4L2 lane) to xtradeb 153 on the VA-API lane. Measured: HEVC 8-bit, H.264, 10-bit VP9
  at 4K60 on the VPU in both.
- Chromium typing/text-field flicker fixed: mutter composites browser surfaces instead of direct scanout
  (`/etc/environment.d/60-pidesktop-mutter.conf`). The image-thumbnail flicker is ANGLE-on-panfork and stays;
  Firefox remains the default (VID-1).
- Kernel 6.1.0-1027.27 (2026-09-08 build): Ethernet-after-long-sleep resume race fixed (stmmac/phylink reorder
  + YT8531 re-init; upstreamed to Armbian); Bluetooth SCO use-after-free fix re-applied; RGA 1.3.13.
  Suspend stays masked (sleep device tree not baked).
- BOOT-2 fixed: ssh host keys generated before sshd on first boot.
- Video driver v2.1.5 (created-depth surface export; libva auto-detection on Panthor/Panfrost stacks).
- mpv `video-sync=display-resample`: 4K60 zero-copy drops 605/900 → 0.
- Documented the AP6275P Bluetooth fix set (boot race + coexistence values) as canonical.
- Pi Desktop on the ubuntu-rockchip base is retired at v2.0.2 (2026-09-23). Successor: Pi-Desktop 3.0 on Armbian
  (Ubuntu 26.04 / GNOME 50 / vendor 6.1.172).

## 2.0.2 — 2026-09-07
- HEVC hardware-decoded, including in Firefox (driver v2.1.0+, Firefox HEVC policy).
- 10-bit works: HEVC Main10, H.264 High 10, VP9 Profile 2 advertised and displayed zero-copy in Firefox and mpv (driver v2.1.3).
- H.264 B-frame ghosting fixed (driver v2.1.2); 4K surface-pool exhaustion in VA-API Chromes fixed (v2.1.4).
- Chromium no longer loses its GPU process on 10-bit files; plays 8-bit HEVC in hardware (libv4l-rkmpp fork + HEVC flag).
- mpv defaults to zero-copy VA-API; the 10-bit guard profile is gone.
- Kernel 6.1.0-1027 with RGA driver 1.3.13; ES8388 silent-codec rule; `iw` added.
- Leaner: LibreOffice Writer + Calc only; Remmina, Shotwell, btop removed.
- Known: first-boot ssh takes ~4 minutes to come up (BOOT-2).

## 2.0.1 — 2026-09-04
- Odd-resolution VP9 green screen fixed (driver v2.0 "Reframe"); honest codec menu; Firefox default,
  flicker-free, hardware-decoded; mpv 0.38; Betterbird actually installed; Pro Updates in-image.
