# Changelog

## 4.0 Plaid — 2026-10-XX

Successor to the retired 2.0.x line, on a new base: Ubuntu 26.04 packages, GNOME 50.1, built with the Armbian
framework on Armbian's Rockchip vendor kernel 6.1.172 (`rk-6.1-rkr7.2`, armbian/linux-rockchip `44bbd021`) plus
eleven Pi-Desktop patches. Release candidate test9: 78/79 first-boot checks on an Orange Pi 5B, 2026-10-03 (the
one fail was a setting the tester had changed). Image about 1.5 GB compressed.

### Kernel (source: tag `pi-desktop-4.0` in defcom5-rockchip/linux-rockchip)
- 0001 `drm/rockchip: vop2: reject AFBC modifier (no-flicker 4K@120 variant)` — the 2.0.x flicker root cause.
- 0002 `arm64: dts: rockchip: orangepi-5: bump CMA pool 256M -> 1024M` and 0005 `arm64: dts: rockchip: orangepi-5:
  source vp0 dclk from v0pll; CMA 512M` — net effect 512 MiB CMA, also set by the `cma=512M` boot argument.
- 0003 `drm/rockchip: vop2: raise dclk ceiling to 1200MHz for RK3588 4K@120Hz`.
- 0004 `drm/rockchip: vop2: keep the HDMI-PHY-PLL dclk decision at the 600 MHz TMDS limit` — without it rkr7.2 keeps
  FRL modes on the PHY PLL and 4K@120 never trains.
- 0006 `net: wireless: bcmdhd: handle NL80211_WPA_VERSION_3 in wl_set_wpa_version()` — WPA3 and WPA2/WPA3 networks
  were refused (deauth reason 13) with wpa_supplicant 2.11; upstream PR armbian/linux-rockchip#561.
- 0007 `drm/rockchip: dw_hdmi_qp: optional cap on the FRL per-lane rate` (knob, off by default).
- 0008 `drm/bridge: dw-hdmi-qp: tunable FLT_update poll interval after FRL training` — polling off by boot argument:
  the 100 ms SCDC polling wedged the monitor's DDC and gave black screens after 14 min–3 h at 4K@120.
- 0009 `drm/rockchip: dw_hdmi_qp: hide HDMI 2.1 FRL modes unless allow_frl=1` — the off switch; the image boots with
  `rockchipdrm.allow_frl=1`.
- 0010 `phy: rockchip: samsung-hdptx: don't touch FFE registers while suspended` — the hard hang on screen-off/wake
  at 120 Hz (register writes to a runtime-suspended PHY); proven on hardware 2026-09-25.
- 0011 `drm/bridge: dw-hdmi-qp: retrain an FRL link after a short HPD drop` — black screen after a monitor
  power-cycle at 120 Hz; proven 2026-09-25 with cable-pull, DPMS-off and 60 Hz negatives.
- Config: RT_GROUP_SCHED off (real-time scheduling for audio under cgroup v2), lockup and hung-task detectors on.
  Kernel, DTB, headers, U-Boot, BSP and firmware packages are held.

### Display
- 4K@120 Hz over HDMI 2.1 FRL offered by default; 4K@60 unchanged. The maintainer's daily driver since 2026-09-25.
- Arm Mali user-space driver (libmali g24p0, wayland-gbm) on the vendor kbase driver; a libmali-hook shim provides
  `EGL_EXT_device_query` so Firefox's graphics probe passes. Chromium and Chrome flicker-free.
- Screensaver instead of screen blank: per-user service (content from `~/Pictures/screensaver`, penguin-drop loop
  video default, idle inhibitor while showing and while locked, crash restart), GNOME blank locked off, GDM greeter
  blank/suspend off, patched gnome-control-center 50.3 (`+pd3.3`) with a Screensaver group in Settings ▸ Power.
  Default delay 15 min; lock on demand (Super+L) kept.
- HDR: mutter's 50.4/50.5 metadata fix tested; the maintainer's monitor stays pale (DSP-2).

### Audio
- PipeWire 1.6.9 / WirePlumber 0.5.17 (Debian packaging rebuilt), PulseAudio daemon gone; rtkit real-time allowed
  for group `audio` regardless of login timing (RR 20 from boot).
- 3.5 mm jack works: the vendor es8323 driver's "OUT1 Switch" defaults off — a boot oneshot turns it on; a user
  service follows the jack (plug → headphones, unplug → previous output). Verified by ear; 44.1/48/96 kHz follow
  the file.
- Outputs named (HDMI / Headphones (3.5 mm) / DisplayPort (USB-C)); HDMI is the default sink.
- Rhythmbox 3.5.1 (newest upstream, all 13 meson features, dark by default; held) with GStreamer codecs; Audacity
  3.7.8 (xtradeb); Brasero + cdrkit + cdrdao for burning.
- Bluetooth: the 2.0.x AP6275P fixes (boot race; coexistence `btc_params`) ported with the bt-sentinel; bluez
  mpris-proxy disabled (it muted Bluetooth speakers ~10 s at every YouTube ad).

### Video
- rockchip-vaapi 2.2.0, MPP snapshot `a8b19653` (2026-08-05) and librga `26a50ef` rebuilt for 26.04; codec and
  2D-engine device nodes opened to group `video` (root-only on an Armbian rootfs).
- **10-bit HEVC decodes on the video engine in Chromium** (measured on the release image 2026-10-03: 4K Main 10,
  595 frames, 0 dropped, ~21 % CPU across its processes). Firefox 156 falls back to software for 10-bit on this
  build — the reverse of 2.0.x; see KNOWN-ISSUES VID-5.
- Chromium 154 (xtradeb) is the default browser, with VA-API flags and a render-node override (the NPU registers as
  a second render node and Chromium picked it).
- mpv 0.41: `video-sync=display-resample` for files; audio-sync (plus deinterlace for DVD) profiles for DVD and
  Blu-ray — the 123 drops/min at 120 Hz on DVDs were the every-vsync redraw, not the CPU.
- VLC removed (its video path on this image is software-only under XWayland, a worse duplicate of mpv; Armbian's
  desktop list re-installs it, the recipe purges it).

### Network
- WPA3 (patch 0006): verified WPA2/WPA3 mixed, WPA3-only + PMF, 2.4 and 5 GHz, Wi-Fi 6 rates.
- `.local` names resolve (mdns4_minimal in nsswitch); gvfs-fuse and cifs-utils for NAS shares from Files.
- SSH: root login key-only (Armbian ships "yes").

### Desktop
- First login asks for a computer name (suggests `<user>-pidesktop`); login screen at every boot after the first
  (Armbian's autologin off-switch never fired); Log Out shown in the power menu.
- GUI password prompts: polkit ≥ 126's socket-activated agent helper needs `SO_PEERPIDFD` (Linux 6.5); on 6.1
  every prompt hung and the retries ate the CPU. Socket masked, classic setuid helper restored via
  dpkg-statoverride. Proven by six first-click installs on a fresh image.
- Files runs in its own user unit and unwritten data is capped at 256 MB, so a big copy can no longer get the
  session killed by systemd-oomd (systemd#35270). Proven with a multi-GB NAS→SD copy.
- "Open in Terminator" in Files; Plaid Terminator look and fastfetch greeting; "Pi-Desktop How-To.txt" on the
  desktop (Chrome, the first-click apps, screen scaling at 120 Hz).
- Plaid wallpaper default; dock layout; persistent journal (200 MB).
- All overlay files installed root-owned (test3 had six user-writable `/etc` files, including `modprobe.d`).

### Apps on first click; image size
- Desktop tier "mid": Firefox, LibreOffice (Writer/Calc/Impress/Draw/Math, no Base/Java), GIMP, Inkscape,
  Thunderbird (Mozilla Team PPA), VSCodium (own repo, key pinned) and Google Chrome (Google's repo, key pinned) are
  placeholders that install on first click (pkexec; the placeholder is removed on success). test8 with the full
  tier was 2.26 GB, over GitHub's 2 GiB release limit; test9 is 1.55 GB. Six installers measured on hardware
  2026-10-03 (GIMP 45 s, VSCodium 297 s, LibreOffice 99 s, Inkscape 69 s, Thunderbird 138 s, Chrome 45 s).
- VS Code (Microsoft build, telemetry on) not shipped; VSCodium offered instead.

### Licensing and identity
- Google Chrome no longer in the image (Google's terms: personal licence, no redistribution) — first-click installer
  instead.
- Firefox no longer in the image (Mozilla's distribution policy covers unaltered builds only) — first-click
  installer; the Mali libGL shim, wrapper and VA-API prefs are applied as it installs.
- Code GPL-3.0-or-later, artwork CC BY-SA 4.0; new LEGAL.md (corresponding-source table), PRIVACY.md and
  SECURITY.md; the os-release home and privacy URLs point to the project website and About shows a Pi-Desktop logo.
- Kernel source published as tag `pi-desktop-4.0`; the recipe published as defcom5-rockchip/pi-desktop-recipe.

### Still not done
- Suspend stays masked (PWR-1). 10-bit video remains Firefox-only (VID-5). Video over USB-C untested (DSP-1).

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
