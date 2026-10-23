# Changelog

## 5.5 — 2026-10-22

Same base as 5.0 (Ubuntu 26.04, GNOME 50, kernel `6.1.189-vendor-rk35xx`, tag `pi-desktop-5.0`).

### Boards
- **The original Orange Pi 5 is now listed as supported** next to the 5B. It uses the same image; we have none to test
  on, so it is supported through issues, pull requests to the recipe and Discussions (KNOWN-ISSUES HW-1). The Orange Pi 5 Plus is
  planned for a later release.

### Added
- **Network Drive** (Utilities, and **Add as Network Drive…** in the Files right-click menu on an `smb://` share or inside one):
  adds a NAS share as a network drive, a kernel CIFS mount under `/mnt/<name>` that reads ahead 32 MiB. In a two-hour
  test, a 4K AV1 film played from a network drive in Firefox with no decoder stall; 32 MiB covers about 30 seconds of
  an 8–9 Mbit/s film, less of a high-bitrate one (KNOWN-ISSUES NET-1). One `/etc/fstab` line per drive after a marker
  comment, mounted on first access by a systemd automount and unmounted after 15 idle minutes; it never blocks boot.
  The login is kept in `/etc/pidesktop/netdrive/<name>.cred` (root only, 0600); the window passes the password to the
  root helper on its standard input, never on a command line. Read-only by default. If the share refuses the login or
  doesn't answer, nothing is changed, and the window asks for the user name when only a password is typed (2026-10-22).

### Changed
- **MPP `+pd34`:** the VP9 fix in the form Rockchip's reviewer asked for in rockchip-linux/mpp PR #976: the frame
  label wraps within 1..0xffff on every SoC and the RK3588 exception is gone, so RK3588 decodes VP9 with MPP's default
  number of frames in flight. `+pd33` forced two. In Rockchip's test decoder (`mpi_dec_test`), synthetic 4K60 clips:
  214 / 233 fps (10-/8-bit) against 149 / 181 with the one-frame guard; 201,600 frames through three wraps with 0 resets; through rockchip-vaapi, ffmpeg, Firefox
  and Chromium past the wrap with 0 resets. Also published for Ubuntu 24.04 and 26.04 as the `pd34` release of
  defcom5-rockchip/mpp (2026-10-20).
- **mpv `+pd2`:** a larger cache for films on network shares (`demuxer-max-bytes=512MiB`,
  `demuxer-max-back-bytes=128MiB`, `cache-pause-wait=10`). mpv already caches GVfs, CIFS and NFS paths; the 150 MiB
  default lasted about 30 seconds at 40 Mbit/s. Local files are unaffected. `+pd2` is built with Ubuntu's standard
  compiler flags (link-time optimisation, frame pointers), like Ubuntu's own mpv; `+pd1` was built without them.
  Side by side on four 4K60 clips the two play the same: same dropped frames and GPU load, pixel-identical pictures
  (output at 1080p60, not 4K@120; 2026-10-21).
- **PipeWire 1.6.10 and WirePlumber 0.5.18** (from 1.6.9 / 0.5.17). PipeWire 1.6.10 is a bugfix release: crashes in the
  JACK layer, pops when draining unsigned formats, a regression in the rear-channel upmix delay, unbounded queuing
  for paused PulseAudio clients and the ALSA plugin's handling of xruns. WirePlumber 0.5.18 adds fixes and small
  improvements: profiles are ranked by which outputs are plugged in, HDMI outputs take the monitor's name, and a
  failing ALSA device is retried at most three times. Upstream 1.6.10 on Debian's 1.6.9-2
  packaging (its CVE-2026-14330 patch kept: the fix is not in 1.6.10) and Debian's WirePlumber 0.5.18-1, rebuilt for
  26.04 as `~pd55` with the same features as 5.0's packages. Tried on the test board before packaging, with Plaid Pi's
  audio settings: HDMI, the 3.5 mm jack, DisplayPort, a USB audio interface and a Bluetooth speaker on SBC-XQ played
  with no errors, and again with the packages in the 5.5 image (2026-10-22).
- **Chromium and Chrome** start with Chromium's media buffer setting at 60 seconds instead of 10
  (`JointMediaOptimizer:SrcMediaMultiBufferPreloadSeconds/60`, present in Chromium 155). Per Chromium's source that is
  up to 50 MiB ahead, about 50 seconds of a typical 8–9 Mbit/s film and less of a high-bitrate one; not yet measured
  on the board (2026-10-21).

### Security
- **skb_zerocopy (ZDI-CAN-33367), local privilege escalation, no upstream fix yet:** the two modules that call it,
  `nfnetlink_queue` and `openvswitch`, are blocked in the CVE-mitigation modprobe file. Any user could get both loaded
  (openvswitch without even a user namespace) and use them inside a user namespace; checked on the board.
  KNOWN-ISSUES SYS-5 (2026-10-22).

### Upstream
- **armbian/linux-rockchip#561 merged (2026-10-21):** patch 0006, WPA3 and WPA2/WPA3 networks in the AP6275P's
  `bcmdhd` driver, is merged into Armbian's vendor-kernel branch `rk-6.1-rkr7.2`. Plaid Pi keeps 0006 until its kernel base
  moves past the merge.

## 5.0 — 2026-10-19

A 4.5 was built on 2026-10-11 and passed its image checks but was never published: the same day the project was
renamed Plaid Pi and AV1 hardware decoding was pulled forward, so everything since 4.0 ships together as 5.0.

Ubuntu 26.04 and GNOME 50.1 as in 4.0, on a newer kernel: Armbian's Rockchip vendor tree merged with upstream stable
6.1.189, plus the Plaid Pi patches. Image `Plaid-Pi-5.0-orangepi5b.img.zst`, 1.38 GiB compressed (6.46 GiB
written), built 2026-10-19 on Armbian 26.11.0-trunk. Unless an item says otherwise, each was verified on the
development board. The 6.1.189 kernel (`6.1.189-vendor-rk35xx`) has run on that board since 2026-10-11: 4K at
120 Hz, hardware video decoding, Wi-Fi and Bluetooth. The release image passed its 49 build-time checks and, on
the board, the first-boot check (133 of 133), again 133 of 133 after installing all seven first-click apps and
running Pro Updates; the login screen and the branding were checked by eye. Dates are when each item was found and
fixed. The release documents went through a multi-agent review before publishing.
Four review passes went over the recipe and these documents before publishing: recipe code and privacy, build
reproducibility, every claim against the measurements, and wording. They found no privacy leak; they did find that the
shipped copyright file called the Dhruva dock unmodified although it carries one Plaid Pi change (fixed, and the image
was rebuilt), that the recipe's input downloader asked GitHub for file names it does not use (fixed), and about a dozen
wrong or stale statements in these documents, including a mis-cited Mali commit and the kernel patch count (fixed).

**The project is now called Plaid Pi.** The tartan logo had become the identity, so the name follows it: what would
have been "Pi-Desktop 5.0 Plaid" is "Plaid Pi 5.0", and "Plaid" is no longer a release codename. Only what people see
changed (the OS name, the greeting, the default computer name `plaidpi` and the `<user>-plaidpi` suggestion, the
desktop How-To, the image file name, the documents and the website); the services, scripts and settings inside the
image keep their `pidesktop` names, as do the repositories and their URLs, so nothing that worked on 4.0 breaks.
Earlier releases keep their names in history. The Settings ▸ Power sentence about the screensaver was rebuilt into
our Settings package with the new name (+pd3.4, 2026-10-11).

### Fixed

- **Chromium and Chrome opened on armbian.com and said "Managed by your organization".** Armbian's desktop branding
  adds browser policies: armbian.com as the homepage and startup page, and a read-only "Armbian" bookmarks folder,
  which is what made the browsers report they were managed. 5.0 moves these files out of the policy directories
  (to `/usr/share/pi-desktop/parked-policies/`), so both browsers start with their own defaults; the Chrome installer
  does the same on older installs. Renaming them in place is not enough: Chrome and Chromium read every file in
  `policies/managed` and `policies/recommended`, whatever its name. On 4.0: `sudo mkdir -p /root/parked-policies &&
  sudo mv /etc/chromium/policies/*/armbian.json* /etc/opt/chrome/policies/*/armbian.json* /root/parked-policies/`,
  then quit and restart the browser.
- **Updates never installed by themselves.** Armbian's BSP package, which the image holds, switches apt's daily job
  off (`APT::Periodic::Enable "0"`), and with it unattended-upgrades: a 4.0 install never upgraded on its own (a 4.0
  test board had sudo, libxml2, libpng, poppler, librsvg, BlueZ, LibreOffice and Chrome updates waiting). 5.0 turns
  the job on and lets it install Ubuntu's security updates plus new versions of anything installed from the xtradeb
  and Mozilla Team PPAs and Google's repository (Chromium, Firefox, Thunderbird, Chrome …), without Armbian's own
  packages (2026-10-09). 4.0 users: three steps in [SECURITY.md](SECURITY.md#if-you-run-pi-desktop-40).

- **The terminal greeting wrapped over its logo with larger text.** fastfetch added the configured logo width on top
  of its own measurement, so the greeting was 131 columns wide and fitted only Terminator's default window at normal
  text size; with Large Text (or any narrower window) its lines wrapped across the logo. It is now 97 columns wide
  and switches to the logo on top in narrower terminals. Terminator's default window is taller, so the top of the
  logo no longer scrolls off with Large Text, and the greeting waits for the window's real size before choosing a
  layout (Terminator can start the shell at 80×24 and resize it a moment later) (2026-10-11).

- **Bluetooth off and airplane mode did not stay off.** A rule that clears a stale Bluetooth block when the radio
  appears also ran on every later block event and undid it. It now runs only when the device appears (2026-10-09).

- **Bluetooth could stay dead after being switched off and on.** The Bluetooth switch is the chip's power pin:
  turning it off and on restarts the AP6275P without its firmware, the kernel keeps talking to it at the old speed,
  and the adapter never comes back by itself (the kernel logged "Frame reassembly failed" about 420,000 times in
  15 s while trying). Plaid Pi's Bluetooth watchdog service (`bt-sentinel`) was meant to reload the firmware but
  only looked at the switch's state after each change, so a quick off-and-on went unseen, including the one the udev
  rule in the previous item caused on 4.0. The 5.0 watchdog acts on the switch events themselves: turn Bluetooth on and
  you see "Starting Bluetooth..." then, about a quarter of a minute later, "Bluetooth is on."; the log flood is a few thousand lines.
  A `tail -f` on `/var/log/bt-revive.log` no longer switches the watchdog off (its "repair already running" check
  matched any command line containing `bt-revive`; it now matches the process name). Patch 0017 rate-limits the
  kernel's error line (ten lines per burst instead of thousands, measured). On 4.0:
  `sudo systemctl restart bluetooth-hciattach.service` brings the adapter back in about a quarter of a minute (2026-10-11).

- **Orange Pi 5 (the original, without the B).** Readers of the CNX Software article reported that the earlier 2.0.x
  image boots on it; nobody has reported 4.x yet, whose bootloader is different. From 5.0 it is listed as "supported
  as reported": owners say it works, we have not run it ourselves. It is a fair expectation: same chip, and the 5B's
  device tree is the 5's plus the eMMC and the Wi-Fi/Bluetooth module, minus the SPI flash, so display, video, audio,
  USB and the M.2 NVMe slot are set up the same way. What a 5 owner does not get: onboard Wi-Fi or Bluetooth (use a
  USB adapter) and eMMC (install to microSD; NVMe boot untested). The Bluetooth helpers now notice the missing module
  and stand down instead of showing "Restoring" and "could not be restored" notices every couple of minutes, and the
  system still calls itself "Orange Pi 5B". We do not own a 5; problems will be dealt with as they are reported
  (2026-10-11).

- **Hardware video for a second account.** The video engine's device nodes were opened to the `video` group only; an
  account added in Settings is not in it and fell back to software decoding. The active user now gets access as
  well (`uaccess`), like the GPU's render node (2026-10-09).

- **Screensaver green frames after about an hour.** mpv's global configuration listed zero-copy VA-API before
  copy-back. Once the desktop loads the graphics shim, mpv's zero-copy attempt reaches the VA-API driver, and every
  abandoned attempt left a handful of frame buffers mapped in the video decoder's IOMMU. The screensaver reloaded its clip, and so re-created the decoder,
  every 60 s; after roughly 55 loops the IOMMU was full, every later decode failed, and the picture went green. Found and
  measured 2026-10-07 (five leaked buffers per decoder start, zero with copy-back requested directly). The screensaver
  now keeps one decoder for the night (`--loop-file`) and asks for copy-back; until driver 2.5.0 the global mpv
  configuration used copy-back only (with 2.5.0 and Plaid Pi's mpv it tries zero-copy first again, see Changed);
  verified over a full night. 4.0 is not affected because its desktop never loaded the shim; 5.0 loads it on every install, so this fix is required
  there. The leak itself is fixed in the VA-API driver from 2.5.0 on; 5.0 ships 3.5.0.

- **FLAC seeking in Rhythmbox.** Ubuntu 26.04's GStreamer 1.28.2 could stop playback with "Internal data stream
  error" when seeking in a FLAC file without a seek table, which is common for vinyl rips and downloads. 5.0 ships
  Ubuntu's gst-plugins-good with the three-line upstream fix from GStreamer 1.28.8 (`1.28.2-2ubuntu0.4+pd1`). Built and
  tested on the board: the package's 116 tests pass, the parser assertions behind the error go to zero (8 and 2 → 0), other formats are
  unchanged, and a 24-bit/192 kHz album without seek tables seeks cleanly in Rhythmbox (2026-10-08). Not held, so a
  newer Ubuntu security update still installs; the first-boot check reports which build is present. A request for
  Ubuntu to ship the fix to every 26.04 user is drafted.

- **mpv frame timing.** mpv now uses audio-clocked frame timing. The display-resample timing chosen on the 2.0.x
  stack is the wrong default on this one: on a 10-bit HDR 4K60 file it played 13.5 s of video in 20 s, where audio
  sync keeps the right speed (VID-6). Colour is upscaled with `cscale=bilinear` instead of mpv's default
  lanczos, which kept the GPU saturated at 4K60 and dropped frames (VID-6). The renderer is pinned to OpenGL (`gpu-api=opengl`: the Mali driver's Vulkan cannot present, and
  mpv's automatic choice fell through to software rendering), and a hwdec change made by a profile or a key lasts for
  one file only (`reset-on-next-file=hwdec`).

### Changed
- **AV1 is decoded on the video engine, in the browsers too.** VA-API driver 3.5.0 (`rockchip-vaapi 3.5.0-1`,
  Number Nine "Pepper SGT", released 2026-10-19) adds AV1 to the H.264, HEVC and VP9 of 4.0, and the image's MPP
  build (`librockchip-mpp1 … +pd33`) adds the entry point the driver uses to decode each AV1 picture once (a VA-API
  client such as Firefox syncs every picture early, which otherwise costs a second decode). Measured on the
  development board with Firefox 157 (Mozilla Team PPA): YouTube 4K60 AV1 dropped 91–95 % of its frames with driver
  3.0.0, about 8 % with 3.5.0, and 1.2–1.5 % after start-up with the image's Firefox setting
  `media.video-queue.hw-accel-size=10` (about 4 % over a whole run). Chromium and Chrome: no green or stale frames at
  playback start in 10 of 10 measured first plays; a rare stale frame mid-stream remains (driver KNOWN-ISSUES KI-16).
  On a stock MPP build the driver still decodes AV1 correctly, but Firefox cannot keep 4K60 (KI-14). Firefox has AV1
  switched on in 5.0, so YouTube serves it AV1 where it used to get VP9. 3.5.0 also carries 2.5.0's fixes for
  abandoned decoders, several decoders on one display and multi-threaded clients.
- **VP9 decodes on both decoder cores.** MPP's VP9 parser labels frames with a counter that never wrapped, and the
  RK3588's two decoder cores behave as if they compare those labels in 16 bits, so VP9 on both cores broke after 65,536
  frames in one session (`resetting for err 0x23/0x107` in the kernel log, frames silently lost); upstream had hidden
  this by running VP9 on one core only, which is what 4.0 shipped. The `+pd33` MPP wraps the counter at 16 bits and
  runs VP9 on both cores again on the RK3588: 25–29 % more VP9 decode throughput (`mpi_dec_test`, 4K: 197.6 vs
  155.4 fps 10-bit, 229 vs 177 fps 8-bit), 0 resets across the wrap in Chromium, Firefox and mpv soaks; the same
  change ran clean on a second board (RK3588, Orange Pi 5 Plus). Submitted upstream as
  [rockchip-linux/mpp#976](https://github.com/rockchip-linux/mpp/pull/976) (open). Patches and changelog are in the
  MPP source package published with the release (2026-10-17).
- **Chromium 155** (xtradeb PPA, `155.0.8059.79`); its configuration-file prompt is answered at build time.
- **Plaid Pi branding on the login screen.** The greeter showed Armbian's logo and the first user got it as the
  account icon; both are now the Plaid Pi logo. Armbian's Chromium and Chrome policies (start page, bookmarks) are
  out of the policy directories, kept under `/usr/share/pi-desktop/parked-policies/` (see Fixed). The LibreOffice
  installer icon rendered wrong (SVG element order); fixed (2026-10-18).
- **The login screen shows from the first boot.** Armbian's first-login wizard switches GDM automatic login on for
  the user it creates; 5.0 turns that switch off in the wizard at build time, instead of a service undoing it at the
  next boot as on 4.0 (2026-10-19).
- **Pro Updates 1.5.0 is in the app grid.** It lists pending apt and Flatpak updates, shows which repository each
  browser updates from and which packages are held, and applies the updates you tick after asking for your
  password, once per change. Held packages are never upgraded and apt never removes a package from it. It has no
  timer or autostart and connects to nothing until you open it. Its Ubuntu Pro section says "not checked" (the
  Ubuntu Pro client is not on the image), and Flathub is added only for your user, only if you choose a Flatpak
  install. 4.0 meant to ship
  it but its install step never ran; before 5.0 a review found shell strings run as root, a false "all
  clear" and a repair button that wrote a differently ordered pin file. All were fixed in 1.5.0, which now uses one
  root helper with fixed commands and the image's own pin file (2026-10-09).
- **The Plaid look is the default.** The look the maintainer uses on their own desktop, for every new install: the Dhruva dock
  (NarkAgni's GNOME Shell extension, with one Plaid Pi change: the dock's pressure barrier is rebuilt when the
  monitor geometry changes) in the wallpaper's navy and cream with the Plaid Pi logo on the
  Show Apps button, replacing Ubuntu's dock, which stays installed and can be switched back on in Extensions; a top
  bar in the same colours (our small Plaid Panel extension); dark mode with the orange accent, including
  GTK3 apps such as Rhythmbox; peach Plaid folder icons (Adwaita's, recoloured); a clean desktop with Trash and
  mounts on the dock (the dock's music pill is off: it would send each track's title and artist to a lyrics
  service); and the maintainer's dock order, now with Pro Updates and Audacity (2026-10-10).
- **Killswitch** is in the app grid's Utilities folder: it lists mounts, probes each one in a child process with a
  3-second timeout so a dead server shows up as STALE instead of freezing the window, and detaches the mounts you
  choose. It offers only network mounts, GVfs shares and mounts under /media, /mnt and /run/media, never the
  system's own filesystems or your home. A network mount gets one forced, lazy unmount after the administrator
  password, so programs stuck on the dead server get an error, and an automount underneath stays and brings the
  share back on the next access; other mounts get a normal unmount; GVfs shares unmount through gio without a
  password (2026-10-10).
- **The Extensions app is installed**, for the Dhruva dock's settings and the way back to Ubuntu's dock (KNOWN-ISSUES
  DSK-1).
- **GNOME Software is gone.** It duplicated what the daily apt job and Pro Updates do and downloaded updates in the
  background; apps still come from the first-click installers and apt. Flatpak stays installed, with no remotes
  (2026-10-10).
- **apt sources over https.** Ubuntu's ports archive and Armbian's repository were plain http in 4.0; both are https
  now (Armbian redirects to an https mirror; tested on the development board 2026-10-09).
- **Kernel: Linux 6.1.189.** 4.0's kernel was Armbian's vendor tree at Linux 6.1.172 (May 2026), and Armbian's branch
  is still on upstream 6.1.172. 5.0 merges upstream stable 6.1.189 (3 October 2026) into it: 5,900 upstream commits. Of
  4,263 changed files, 21 conflicted with Rockchip's changes; each resolution is recorded, upstream fixes were ported
  by hand where the vendor driver differs (SPI controller removal, USB audio gadget, PDM audio clock order), and four
  vendor drivers were updated for upstream interface changes (the RK806 power chip's pin controller, Winbond SPI
  flash, the Broadcom Wi-Fi driver, Rockchip UFS). The Plaid Pi patches apply on top: ten of 4.0's eleven, five for
  drivers on 6.1.189 (0012–0016) and one new one (0017, below). The CMA-size patch and the CMA part of the v0pll
  patch were dropped: the `cma=` boot argument overrides the device tree, so they never took effect. The VP1/VP2
  clock-limit change was dropped too: only HDMI 4K@120 on VP0 needs the higher limit, and the USB-C DisplayPort on
  VP2 stays at the stock 600 MHz. Every Plaid Pi patch says `Assisted-by: LLM` (0012–0014 are Armbian's and upstream's, unchanged). The kernel also carries Armbian's Realtek rtw88 Wi-Fi backport
  (USB and SDIO dongles) as three patches, one hunk dropped because 6.1.189 already contains it, and two build fixes
  for vendor and Armbian-added Wi-Fi drivers (aic8800, rtl8192eu, rtl8811cu, rtl8812au, rtl88x2bu, uwe5622) whose
  version checks did not expect two interface changes stable backported to 6.1 (the USB driver structure in 6.1.188,
  the TDLS callback in 6.1.189), plus a rate limit on the Bluetooth UART driver's "Frame reassembly failed" line,
  which a power-cycled chip otherwise logs several hundred thousand times (patch 0017, 2026-10-11; measured: ten lines per
  burst). The kernel also turns
  on Ubuntu's usual hardening (Yama ptrace restrictions, hardened usercopy, slab freelist hardening and
  randomisation, zeroing on allocation), and the Mali GPU kernel driver is now a release build instead of a debug
  build. Kernel updates still mean a new image.
- **Hardware watchdog on.** If the kernel or systemd itself hangs for 30 seconds, the board resets itself instead
  of staying frozen. A frozen desktop on an otherwise running system does not trigger it.
- **rpcbind off by default.** Port 111 listened from the first boot; it now starts only when an NFSv3 share is
  mounted.
- **Removed: `armbian-imager`** (Armbian's branded SD-card writer) and Ubuntu's text boot theme. A guard that checks
  what a removal would take with it had been refusing every removal since 4.0 because it misread apt's output.
- **mpv rebuilt for the Mali driver** (`mpv 0.41.0-2ubuntu4+pd1`: Ubuntu's mpv with two patches). The Mali GL driver
  refuses one video plane format and imports another as an integer texture (KNOWN-ISSUES VID-7); this build handles
  both, so 8-bit and 10-bit video draw zero-copy. 8-bit pictures are identical to copy-back; 10-bit ones are within one
  8-bit step on a few per cent of pixels, with no bias, because the brightness plane is sampled as the integer texture
  it is rather than copied each frame. The second patch fixes a precision bug in mpv's older `vo=gpu` renderer that drew
  such planes 2 % dark. With driver 2.5.0 or later (5.0 ships 3.5.0) the recipe turns zero-copy on and drops the
  10-bit copy-back profile.
  Frame-drop measurements: VID-6. Everything else is Ubuntu's build; the PipeWire audio output is off (audio goes
  through pipewire-pulse). Patches, build script and the patched source package: recipe `mpv/` and the release. Held:
  an Ubuntu update of mpv would replace it and draw 10-bit near-black with Plaid Pi's settings, so Ubuntu's mpv
  fixes arrive with Plaid Pi rebuilds (the libraries mpv uses, FFmpeg and libplacebo included, still update).
- **Firefox decodes 8- and 10-bit video on the video engine, Firefox unmodified** (KNOWN-ISSUES VID-5, VID-7). The
  launcher loads libpidesktop-ffmali (LGPL), which retries the 8-bit chroma plane the Mali driver refuses (Mozilla Bug
  2078820) in a format it accepts and copies the 10-bit brightness plane it imports as an integer texture. Measured
  2026-10-12, Firefox 157.0 (Mozilla Team PPA): 1080p 24 fps H.264 8-bit, VP9 and AV1 8/10-bit decode on the video
  engine; in a stand-alone test every sampled texel matches for all five plane formats. 4K60 AV1 (YouTube) in Firefox was
  measured later with driver 3.5.0 (see Changed: 1.2–1.5 % dropped after start-up). Not yet measured: 10-bit 4K60 in
  Firefox, and the picture against a reference (checked by eye). `PIDESKTOP_FFMALI=0` returns to 4.0's
  behaviour (10-bit hidden from the hardware decoder). mpv draws 10-bit correctly with Plaid Pi's build. Chromium and
  Chrome are not affected. The Firefox installer gets the Mozilla Team PPA's current Firefox (158 at release) and
  installs the library with it.
- First-boot check now has 133 checks, up from 79 at the 4.0 release (shim resolution, vendor-only browser sources,
  session real-time limits, the FLAC fix, the VA-API driver with RGA, Plaid Pi's mpv, mpv's hwdec matching the driver and
  mpv build, mpv drawing 10-bit correctly, the Firefox Mali video library in place, the 6.1.189 kernel, automatic updates,
  the watchdog, nothing on port 111, the Bluetooth rule, the hold list, https-only apt sources, Pro Updates, the
  Plaid look with the music pill off, Killswitch, GNOME Software gone). Checks that passed without testing anything
  were fixed, and one that always failed was removed. On the release image: 133 of 133 on the board, and 133 of 133
  again after all seven first-click installers and Pro Updates had run.

### Kernel (source: tag `pi-desktop-5.0` in defcom5-rockchip/linux-rockchip)
- `6.1.189-vendor-rk35xx`: Armbian `rk-6.1-rkr7.2` at `44bbd021` merged with upstream stable v6.1.189 (merge commit
  `7b6195db`, vendor-driver interface fixes `f3411423`), plus the Plaid Pi patches 0001–0017 (0002 was dropped, see
  Changed; 0012–0016 are the rtw88 backport and the two Wi-Fi build fixes; 0017 the Bluetooth rate limit). All but 0016 are
  also commits on the tag; 0016 lives only in the recipe because it patches drivers Armbian adds at build time.
  Running on the development board since 2026-10-11.
- The kernel base is pinned to the commit 4.0 was built from (Armbian `rk-6.1-rkr7.2` @ `44bbd021`) plus the stable
  merge, instead of following Armbian's branch head. Armbian's newer commits include an Orange Pi 5 change that renames
  the Ethernet interface from `end1` to `end0` (armbian/linux-rockchip #575); 5.0 does not take it, so existing
  static network configurations keep working.

## 4.0 Plaid — 2026-10-03

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
- Suspend stays masked (PWR-1). 10-bit video is hardware-decoded in Chromium, Chrome and mpv; Firefox decodes it in software (VID-5). Video over USB-C untested (DSP-1).

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
