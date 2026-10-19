# Known Issues

Plaid Pi (formerly Pi-Desktop) is a niche distro with a single maintainer, running on the Rockchip vendor 6.1 kernel.
This file exists on purpose. We'd rather tell you what's rough than let you find out.

Current as of **Plaid Pi 5.0** (release image on an Orange Pi 5B, 2026-10-19: 133 of 133 first-boot checks), with
the 4.0 findings kept where they still apply. The 2.0.x
sections further down are the retired line's history and stay as they were.

---

## 4.0 and 5.0

Each entry applies to both 4.0 and 5.0 unless its status line says otherwise. Problems that 5.0 fixes are listed
under [Fixed in 5.0](#fixed-in-50), with the one-time fix for 4.0 where there is one.

### PWR-1: No suspend

**Status:** disabled on purpose · **Severity:** feature missing

Suspend, hibernate and hybrid sleep are masked and GNOME's automatic suspend is off, as in 2.0.x.
The 2.0.x Ethernet-after-sleep fix is in this kernel, but the sleep device-tree configuration and a
resume soak were never done on this image, and shipping sleep untested is worse than shipping none.
Shut down, or let the screensaver run.

---

### VID-5: 10-bit video in the browsers — hardware in Chromium and Chrome; in Firefox from 5.0 (software on 4.0)

**Status:** measured on the 4.0 release image, 2026-10-03; Firefox changes in 5.0 (last bullet) · **Severity:** performance · **Affects:** 10-bit files

This is the reverse of what 2.0.x did, so it is worth stating with the numbers. A 4K HEVC **Main 10** file, played
full-screen for 20 s on the release image (driver 2.2.0):

| | Chromium 154 (in the 4.0 image) | Firefox 156 (first-click install) |
|---|---|---|
| frames decoded / dropped | **595 / 0** | 571 / **26** |
| CPU | **~21 % across its processes** | 229 % + 121 % |
| video engine in use | **yes** | no |

- **Chromium and Chrome decode 10-bit HEVC on the video engine.** So does 8-bit HEVC, H.264 and VP9. Jellyfin's web
  client direct-plays 4K Main 10 in Chromium with no server transcode — nothing to install, it is the default browser.
- **Firefox 156 decodes 10-bit in software — and that one is our bug, not Firefox's** (found 2026-10-05). The image
  ships a shim library so Firefox's graphics probe passes on the Mali driver, but the stock copy was parked in the same
  library directory under the same name and the loader picks it first, so the shim never loads and Firefox falls back
  to software for everything. Fixed in 5.0. On a 4.0 install, the fix is to move the parked stock file out of
  `/usr/lib/aarch64-linux-gnu/` and run `ldconfig`; after that Firefox hardware-decodes 10-bit HEVC and VP9 zero-copy.
  **Don't apply that by hand for now:** with the shim loaded, 10-bit plays zero-copy and draws near-black in Firefox and
  in mpv (VID-7), and 4.0's mpv settings then hit the screensaver decoder leak that 5.0 fixes.
- **Firefox decodes 8-bit video (H.264, VP9 profile 0 — YouTube) in software, and that one is Firefox's**
  ([Mozilla Bug 2078820](https://bugzilla.mozilla.org/show_bug.cgi?id=2078820), filed 2026-10-06 with a tested patch).
  Firefox imports the chroma plane of an 8-bit frame in a two-channel format the Arm Mali driver does not offer, the
  import fails, and the decoder re-initialises in software. 10-bit uses formats the driver does offer, which is why it
  imports once the shim loads (and then draws wrong, VID-7). As of 2026-10-18 a third version of the patch is with Mozilla's reviewer (the triage reviewer agreed with the approach on 2026-10-15); not merged. When
  Mozilla ships the fix it arrives as an ordinary Firefox update; whether 5.0's library (below) is then still needed
  will be re-tested.
- **AV1** is software in both on 4.0: the driver had no AV1 path, and Firefox has AV1 switched off so YouTube serves
  VP9. 5.0 ships rockchip-vaapi 3.5.0, which decodes AV1 on the video engine, and Firefox has it switched on (below).
  Measured on the development board: Firefox 157 playing YouTube 4K60 AV1 dropped 91–95 % of its frames with driver
  3.0.0, about 8 % with 3.5.0, and 1.2–1.5 % after start-up with the image's `media.video-queue.hw-accel-size=10`
  (about 4 % over a whole run); that needs the image's MPP build (`+pd33`; on a stock MPP the driver decodes AV1
  correctly but Firefox cannot keep 4K60, driver KNOWN-ISSUES KI-14). Chromium and Chrome: no green or stale frames at
  playback start in 10 of 10 measured first plays; a rare stale frame for one or two screen refreshes mid-stream, when
  the browser runs out of decoded pictures, remains open (driver KI-16). Firefox, mpv and ffmpeg are not affected by
  that one.
- **HDR** files play but look pale in both browsers: neither does HDR tone mapping on Linux (DSP-2). "Correct colours"
  for the browsers below means the picture is right apart from that, not the dark VID-7 pattern. mpv is the HDR player.

- **5.0: Firefox decodes on the video engine.** Plaid Pi's Firefox launcher loads libpidesktop-ffmali, a small LGPL
  library of ours, as Firefox's EGL/GL library; it works around both Mali import problems (the refused 8-bit chroma
  plane; VID-7's integer brightness plane) without modifying Firefox. Measured 2026-10-12 with Firefox 157.0 from the
  Mozilla Team PPA: 1080p 24 fps H.264 (8-bit), VP9 and AV1 (8- and 10-bit) test clips decode on the video engine;
  4K60 AV1 was measured afterwards with driver 3.5.0 (the AV1 bullet above). Not yet measured in Firefox: 10-bit 4K60
  smoothness, HEVC, and the picture against a reference (checked by eye
  only). `PIDESKTOP_FFMALI=0` (with Firefox closed) turns it off and restores 4.0's behaviour. The Firefox installer
  on the 5.0 image installs the Mozilla Team PPA's current Firefox (158 at release) and this library.

Evidence, including the test clip and the decoder log, is kept with the project notes. The picture is right too:
Chromium and Chrome both played a 10-bit HDR VP9 4K60 file on the video engine with correct colours, checked by eye on
2026-10-09. They probably hand the GPU the whole frame at once, which the Mali driver draws correctly (VID-7).

---

### VID-6: 10-bit 4K60 in mpv leaves little GPU headroom

**Status:** measured 2026-10-09 (5.0, mpv `0.41.0-2ubuntu4+pd1`) · **Severity:** performance · **Affects:** 10-bit
4K video at 50-60 fps in mpv. On 4.0, mpv plays 10-bit through copy-back, which runs 10-bit 4K60 at about half speed.

5.0's mpv draws 8-bit and 10-bit video zero-copy (VID-7). Measured on the development board, full screen on a 4K@120 output,
with the image's settings, 20 s after a 5 s settle (frames dropped out of about 1,200; GPU load median):

| Video | Zero-copy (5.0 default) | Copy-back |
|---|---|---|
| 10-bit HDR10 HEVC 4K60 (a real HDR test clip) | 6-18 dropped, GPU 94-95 % | half speed (11 s of video in 20 s), 338-350 dropped |
| 10-bit HDR10 VP9 4K60 (the same clip re-encoded) | 16-48 dropped, GPU 89-96 % | half speed (11 s in 20 s), 268-347 dropped |
| 10-bit HLG VP9 4K24 (a concert, 30 s) | 1 dropped, GPU 38 % | not measured |
| 8-bit SDR H.264 4K60 (synthetic test pattern) | 4-12 dropped, GPU 28-36 % | 248 dropped |

At 10-bit 4K60 the GPU is close to its limit, and settings that add GPU work push it over (10-bit HDR10 HEVC 4K60):
`--deband=yes` 278 dropped; mpv's default colour scaler, lanczos, instead of the image's bilinear: 123; the older
`--vo=gpu` renderer: 396; `--video-sync=display-resample` instead of the image's audio sync: 13.5 s of video in 20 s.
10-bit at 24-30 fps has plenty of room. Chromium and Chrome also play 10-bit 4K60 on the video engine with a correct
picture (checked by eye, 2026-10-09).

Plaid Pi's mpv is held. If Ubuntu's mpv replaces it anyway (after `apt-mark unhold mpv` and an upgrade), dpkg
installs Ubuntu's own `/etc/mpv/mpv.conf` without asking, because the file still matches Plaid Pi's package. That
file says `hwdec=vaapi` (10-bit near-black, VID-7) and has no `gpu-api=opengl` (mpv falls back to software
rendering). Fix it by setting `hwdec=vaapi-copy` and adding `gpu-api=opengl` and `video-sync=audio`, or reinstall
Plaid Pi's mpv package from the release.

---

### VID-7: Zero-copy 10-bit video draws near-black on the Mali driver

**Status:** found and root-caused 2026-10-09; open in the driver, worked around in Plaid Pi's mpv and, from 5.0, for
Firefox · **Severity:** wrong picture · **Affects:** Firefox on 4.0 with the manual fix in VID-5, and on 5.0 only if
its library is switched off (the launcher then hides 10-bit); Ubuntu's mpv when zero-copy is on (4.0's settings once
the shim loads, or set by hand)

*Zero-copy* hands the video engine's frame straight to the GPU. *Copy-back* copies it into ordinary memory first:
slower, but it always draws correctly.

Seen first as a 10-bit HDR test file that played "really dark with reds, never a bright clean screen". Colour bars
show the pattern: yellow, green and cyan go almost black while red, blue and magenta keep their hue. That is a picture
with no brightness (luma) and only colour (chroma) left.

The cause is in the Arm Mali driver (`libmali` g24p0), measured with a stand-alone probe that leaves the video driver
and the players out. A 10-bit frame reaches the GPU as two planes: brightness as 16-bit samples (`DRM_FORMAT_R16`),
colour as pairs of 16-bit samples (`GR1616`). The Mali driver imports the colour plane as an ordinary normalised
texture, which samples correctly, but imports the brightness plane as an *integer* texture (`GL_R16UI`). Video shaders
read textures as fractions between 0 and 1, and an integer texture read that way returns 0. The data itself is intact:
read as integers, every sample is exact, and the same bytes uploaded as an ordinary texture (copy-back) draw correctly.
Every VA-API driver version tested shows it (2.1.5, 2.2.0, 2.2.1~pre2), and so does Ubuntu's stock mpv: it is the Mali driver's import, not the VA-API driver's.

What the image does about it now:
- mpv: Plaid Pi's own mpv samples the integer-format plane as what it is (and copies the refused 8-bit chroma
  plane), so 8- and 10-bit draw zero-copy (8-bit identical to copy-back; 10-bit within one 8-bit step on about 5 % of
  pixels, no bias). It is held, because Ubuntu's mpv would
  draw 10-bit near-black with zero-copy on. With any other mpv the recipe keeps copy-back and a profile that switches
  10-bit files to copy-back; the first-boot check confirms which case the image is in.
- Firefox (5.0): the launcher loads libpidesktop-ffmali, which copies a plane the driver imported as an integer
  texture into an ordinary 16-bit texture on the GPU as Firefox binds it (the copy described below); Firefox is
  unmodified. In a stand-alone test that imports the planes the way Firefox does, every sampled texel then matches the
  source; with the library off the brightness plane reads ~0. 1080p 10-bit VP9 and AV1 decode on the video engine in
  Firefox 157 (2026-10-12); the picture was checked by eye only and 10-bit 4K60 is not measured. The copy costs about
  2.2–2.5 ms per 4K frame in the test. With `PIDESKTOP_FFMALI=0` the launcher hides 10-bit as on 4.0 (software,
  correct picture, verified 2026-10-09). Firefox's own setting for turning zero-copy off does not help: its copy goes
  through the same texture import (checked with screenshots); 5.0 locks that setting on.
- Chromium and Chrome are not affected: they draw 10-bit correctly (checked by eye with a 10-bit HDR file), probably
  because they hand the GPU the whole frame at once, which the Mali driver converts correctly (shown in the probe).

The fix in a player: either sample the plane as the integer texture it is, with the right scaling (what Plaid Pi's
mpv now does for the brightness plane: no copy), or copy it on the GPU into an ordinary 16-bit texture before drawing
(a few milliseconds per 4K frame). The copy gives a picture bit-identical to copy-back; integer sampling is within one
8-bit step of it on about 5 % of pixels with no bias (rounding between the GPU's normalised read and an exact integer
read). Sampling it in mpv's older `vo=gpu` renderer exposed a precision bug in that renderer's integer path (a 1/65535
factor folded into a multiplier that is printed with six decimals: 2 % dark); the patch fixes that too.

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
- **We have not yet tested video over USB-C** on Plaid Pi. The driver is enabled and probes
  the port on hotplug. Reports with cable or adapter model welcome.

---

### BOOT-3: On 4.0 the first boot logs you in without asking; every boot after that shows a login screen

**Status:** 4.0: by design (Armbian's setup wizard); 5.0: the wizard's autologin is switched off at build time ·
**Severity:** cosmetic

Armbian's first-login wizard enables GDM autologin for the user it creates. On 4.0 a Plaid Pi service turns it
off at the next boot; verified on the 4.0 release image on 2026-10-03: the wizard boot auto-logs-in, the second boot
asks for the password. If you need the login screen before ever rebooting on 4.0, reboot once. 5.0 edits the
wizard so it never switches autologin on, and the login screen (with the Plaid Pi logo) is what you get after the
wizard; the service stays as a safety net.

---

### APP-1: What is not in the image, and why

**Status:** decisions · **Severity:** informational

- **VLC** is out: on this image VLC 3's video path is software-only (it runs under XWayland without the GPU),
  so it only duplicated mpv, badly. mpv is the video player, Rhythmbox the music player.
- **PeaZip** is not shipped. Archive Manager (file-roller) is in.
- **VS Code** is not offered: Microsoft's build has telemetry on by default. VSCodium is the
  first-click alternative.
- **Firefox, LibreOffice, GIMP, Inkscape, Thunderbird, VSCodium and Google Chrome** install on first
  click from their own repositories and need the network for that. All seven ran on the 5.0 release image on
  hardware (the first-boot check passed 133 of 133 afterwards); six were timed on 4.0 (45 s to 5 min on the
  test card), the Firefox installer was not timed. LibreOffice comes without Base (it drags in Java);
  `sudo apt install libreoffice-base` adds it.
- **libdvdcss** and AACS keys are not shipped. If a DVD refuses to play: `sudo apt install libdvd-pkg`.

---

### APP-2: mpv cannot open smb:// links

**Status:** upstream · **Severity:** minor

mpv has no SMB support of its own (no smb:// stream in mpv 0.41). Opening the same file from Files works: Files hands
mpv a local gvfs path.

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

### AUD-2: Short chirps in browser audio while the machine is busy

**Status:** cause found and verified 2026-10-07 · **Severity:** annoyance · **Affects:** Chrome/Chromium audio under heavy CPU load

A couple of brief chirps during a YouTube video while a build or a large install was running. The file was clean; the
chirps are buffer underruns. Cause: on this image the real-time permission for audio never reaches the desktop session.
The limits file grants it to a group the user is not in, and on a systemd-managed GNOME session that route does nothing
anyway — the shell, PipeWire and every launched app inherit their limits from the user's `systemd --user`, whose
real-time limit is zero. PipeWire survives on a fallback priority through rtkit; Chrome's audio thread runs as an
ordinary thread and gets starved. Firefox asks rtkit itself and was never affected.

Fixed in 5.0 with one drop-in, which also works on a 4.0 install (takes effect after a reboot):

```
sudo mkdir -p /etc/systemd/system/user@.service.d
printf '[Service]\nLimitRTPRIO=95\nLimitMEMLOCK=infinity\nLimitNICE=-15\n' | sudo tee /etc/systemd/system/user@.service.d/pidesktop-realtime.conf
sudo systemctl daemon-reload
```

After it, PipeWire runs at its configured FIFO 88 and Chrome's audio thread at RR 8, measured on the development board.

---

### SYS-3: Browsers could silently switch to Armbian's copies (4.0)

**Status:** fixed in 5.0; one-time fix below for 4.0 · **Severity:** policy / update source

Armbian's repository mirrors Firefox, Chromium, Chrome, Thunderbird, VS Code and VSCodium, rebranded, and its desktop
module installs a pin that prefers them. On 4.0 that pin outranked ours by sorting first, so an `apt upgrade` could
replace the Mozilla Team Firefox with Armbian's repackaged copy, and Chrome updates would come from Armbian's mirror instead of
Google. Plaid Pi's rule is that these come from their vendors only. On 4.0, run:

```
sudo sed -i 's/ resolute-desktop$//' /etc/apt/sources.list.d/armbian.sources
sudo rm -f /etc/apt/preferences.d/armbian-desktops
sudo apt update && sudo apt upgrade
```

then check `apt-cache policy firefox` shows the Mozilla Team candidate (the `apt upgrade` replaces an Armbian copy
that is already installed). 5.0 does this at the last step of the image build, plus a pin file that keeps Armbian's
copies at priority 1 (a negative pin would poison the identical version strings) and the vendors at 1001.

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

### DSP-4: 4K at 120 Hz is on by default; how to turn it off

**Status:** by design · **Severity:** recovery note · **Affects:** HDMI monitors that accept 4K at 120 Hz

Plaid Pi enables HDMI 2.1 (FRL) modes with the boot argument `rockchipdrm.allow_frl=1`, so a capable monitor gets
4K at 120 Hz. The fixes behind it (kernel patches 0008, 0010, 0011) were tested on one monitor and one cable. If a
monitor or cable shows a black screen or drops out at 120 Hz: on another computer, open the card's boot partition,
edit `armbianEnv.txt` and remove `rockchipdrm.allow_frl=1` from the `extraargs=` line. The board then offers at most
4K at 60 Hz, which needs none of this. Using HDMI at 120 Hz and DisplayPort over USB-C at the same time has not been
tested; both outputs draw their pixel clock from the same PLL (DSP-1).

---

### DSK-1: The Plaid dock and top bar; how to get Ubuntu's dock back (5.0)

**Status:** by design · **Severity:** note

5.0 uses the Dhruva dock (a GNOME Shell extension by NarkAgni) and our Plaid Panel top bar instead of
Ubuntu's dock. Dhruva's own settings are in the Extensions app (Dhruva → settings). To go back to Ubuntu's dock, switch
off Dhruva and Plaid Panel and switch on Ubuntu Dock in the Extensions app, or run
`gnome-extensions disable dhruva@narkagni; gnome-extensions disable plaid-panel@pidesktop; gnome-extensions enable ubuntu-dock@ubuntu.com`.
Apps pinned to the dock are not shown in the app grid on this desktop, which is why Killswitch sits in Utilities
rather than on the dock. The dock's music pill is off by default: when switched on, it looks up lyrics for every
track you play by sending the artist, title and length to lrclib.net (PRIVACY.md). Bugs in the dock itself belong
upstream at github.com/narkagni/dhruva; tell us too.

---

### HW-1: Orange Pi 5 (the original, not the B) — reported to boot; untested by us

**Status:** reported working by readers; not tested here · **Severity:** informational

The image is built for the Orange Pi 5B. Readers reported (CNX Software comments, 2026) that the 2.0.x image boots on
the original Orange Pi 5; 4.x has not been reported yet, and its bootloader differs (Armbian's vendor U-Boot built for
the 5B). That it should work fits: same SoC, and the 5B's device tree is the 5's plus the eMMC and the Wi-Fi/Bluetooth
module, minus the SPI flash, so display, video, audio, USB and the 5's M.2 NVMe slot are set up identically.
"Supported as reported" means: nothing here is tested on a 5; reports from owners are taken seriously and fixes that
can be made blind will be made; we will not buy a 5. What is different on a 5:

- No onboard Wi-Fi or Bluetooth. Use a USB adapter (the kernel carries the common Realtek USB drivers). On 5.0 the
  Bluetooth helpers notice the missing module (no Broadcom device on PCIe and no eMMC) and stand down;
  `cat /run/pidesktop-bt-absent` shows that they did. On 4.0 they retry for ever: a "Restoring Bluetooth adapter..."
  notice and, 45 s later, "Bluetooth could not be restored", about every two minutes while you are logged in.
- No eMMC. Install to microSD; [EMMC-INSTALL.md](EMMC-INSTALL.md) does not apply. NVMe can hold the root filesystem
  with the card left in for the bootloader (`armbian-install`); untested by us.
- The 5 has a 16 MB SPI flash for the bootloader, which the 5B does not; the 5B device tree does not describe it, so
  this image cannot see, write or erase it, and booting from SPI is not offered. If a bootloader is already in your
  5's SPI flash it runs before the one on the card; if the board will not boot our card, erase the SPI flash with
  Orange Pi's tools first (untested by us).
- The system still calls itself "Orange Pi 5B" (the device-tree model, shown by the terminal greeting and
  Settings ▸ About).
- The USB-C port is the same design on both boards (the same FUSB302 power-delivery controller, the same
  device-tree description), so expect the behaviour described in DSP-1.

We do not own a 5; if something is wrong there, report it and it will be looked at.

---

## Fixed in 5.0

- **SYS-4: 4.0 never installed updates by itself.** Armbian's BSP package switches apt's daily job off
  (`APT::Periodic::Enable "0"`), which also stops unattended-upgrades. 5.0 turns it on and keeps Armbian's own
  packages out of the automatic updates. On 4.0: the SYS-3 fix first, then the file and the check in
  [SECURITY.md](SECURITY.md#if-you-run-pi-desktop-40).
- **SYS-3: browsers from Armbian's copies** — see the entry above for the 4.0 fix.
- **Chromium and Chrome opened on armbian.com and said "Managed by your organization".** Armbian's desktop branding
  installs browser policies (homepage, startup page, a read-only bookmarks folder). 5.0 moves them out of the policy
  directories to `/usr/share/pi-desktop/parked-policies/`; renaming them in place is not enough, the browsers read
  every file there. On 4.0: `sudo mkdir -p /root/parked-policies && sudo mv /etc/chromium/policies/*/armbian.json*
  /etc/opt/chrome/policies/*/armbian.json* /root/parked-policies/`, then restart the browser.
- **BOOT-3: the first boot logged you in without asking** — 5.0 switches the wizard's autologin off at build time.
- **Rhythmbox could stop when seeking in FLAC files** ("Internal data stream error") that have no seek table: 5.0
  ships gst-plugins-good with the upstream flacparse fix (`ef0f9938`, GStreamer 1.28.8) until Ubuntu ships it.
- **AUD-2: chirps in browser audio under load** — see the entry above for the 4.0 fix.
- **Screensaver green frames after about an hour** (5.0 development only; 4.0 never loaded the graphics shim that
  exposed it) — VA-API driver 2.5.0 or later and a one-decoder screensaver.
- **Bluetooth off and airplane mode did not stay off.** On 4.0, add `ACTION=="add", ` at the start of the last line
  of `/etc/udev/rules.d/99-pidesktop-bt-unblock.rules`.
- **Bluetooth stayed dead after being switched off and on** (the switch cuts the chip's power, and 4.0's Bluetooth
  watchdog could miss a quick off-and-on). On 4.0: `sudo systemctl restart bluetooth-hciattach.service` brings it back
  in about a quarter of a minute. 5.0 restarts it by itself: after you turn Bluetooth on you see "Starting Bluetooth..." and,
  about a quarter of a minute later, "Bluetooth is on." Two more things on 4.0: its watchdog is switched off for as long as any command line contains
  `bt-revive` (a `journalctl -f -t bt-revive` or `tail -f /var/log/bt-revive.log` while troubleshooting), and each
  off/on writes about 200 MB of kernel log to the card before the adapter gives up. A different failure, the adapter
  reporting `command 0x1003 tx timeout` at boot, is not covered by this fix.
- **A second user account got no hardware video.** On 4.0, add the account to the `video` group
  (`sudo usermod -aG video NAME`, then log in again).

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

**Status:** 2.0.x history; possibly fixed in 4.x · **Severity:** rare but disruptive

The vendor Mali kernel driver (`kbase`) has a use-after-free on GPU-context teardown that can
crash the desktop session. Observed once on 2.0.x, recovered on its own. The 4.x and 5.0 kernels carry the vendor fix
"MALI: bifrost: Fix kctx kobj UAF" (Rockchip, June 2025); whether that is the same crash was not proven, and it
has not been seen on 4.x or 5.0. The permanent fix is mainline's Panthor driver.

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

Found something not listed here? [Open an issue](../../issues) — include which board (Orange Pi 5 or 5B) and its revision, how
you installed (SD / eMMC / NVMe), your monitor's resolution and refresh rate, and `uname -a`.
If it's video-related, a screenshot of `about:support` (Firefox) or `chrome://gpu` is worth a
thousand words.
