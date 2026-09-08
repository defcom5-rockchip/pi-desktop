# Changelog

## 2.0.3 — in progress
- Video driver v2.1.5: created-depth surface description (Chromium VA-API 10-bit green, driver KI-8);
  libva auto-detection on Panthor/Panfrost stacks (driver KI-9). Recipe updated 2026-09-08.
- Planned: ssh host-key ordering before sshd (BOOT-2); ARMED-orange mpv icon; real `Depends:` for the
  mpv 0.38 package; 4K60 10-bit copy-path mpv profile.

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
