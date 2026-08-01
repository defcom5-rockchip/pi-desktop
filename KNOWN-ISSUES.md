# Known Issues

Pi Desktop is a niche distro with a single maintainer, running on the vendor 6.1 BSP kernel.
This file exists on purpose. We'd rather tell you what's rough than let you find out.

Current as of **v1.0 — "Vanilla Sky"**.

---

## VID-1: Chromium can flicker on window resize

**Status:** open · **Severity:** cosmetic · **Affects:** Chromium 132, some setups (notably ultrawide)

Dragging or resizing a Chromium window can produce brief flicker or tearing. It's a quirk of the
newer GL path in Chromium 132 and it does not affect playback correctness or decode.

**Workarounds, any of which is reliable:**
- **Fullscreen (F11) is always clean** — this is the one to use for video.
- **Vivaldi is unaffected** if you want a second browser for long sessions.
- **Chromium 114** remains installable (`apt install chromium-browser`) and is less glitchy on this
  path. It's an older engine, so treat it as a video appliance rather than your daily browser.

If you see flicker at a *fixed* window size, or in fullscreen, that's a different problem — please
open an issue with your monitor model and refresh rate.

---

## VID-2: AV1 is software-decoded, and always will be on this stack

**Status:** won't fix (upstream limitation) · **Severity:** performance

The rkmpp V4L2 plugin implements **H.264, HEVC, VP8 and VP9**. There is no AV1 in it. No flag,
setting, or browser version changes this — the hardware path simply doesn't carry the codec.

YouTube increasingly serves AV1 by default. If a 4K video is pinning your CPU while another plays
cool, AV1 is the likely reason. Check `Stats for nerds` — if the codec reads `av01`, you're on the
software path.

**Workaround:** a browser extension that forces H.264/VP9 (h264ify and similar) keeps more streams
on the hardware decoder. AV1 support would have to come from Rockchip's media stack or from mainline.

---

## BOOT-1: First boot is busy for two to three minutes

**Status:** open, fix queued for v1.0.1 · **Severity:** cosmetic

On the very first boot, `tracker-miner-fs-3`, `packagekitd` and `unattended-upgrades` all start
indexing at once. Video playback can stutter while they work, and the desktop feels heavier than it
is. **Decode holds through it** — this is I/O pressure, not a graphics problem.

**Workaround:** give it a few minutes. It does not recur on later boots. Throttling the indexer is
queued for the next release.

---

## BOOT-2: Two systemd units fail at boot (harmless)

**Status:** open, fix queued for v1.0.1 · **Severity:** cosmetic

`casper-md5check.service` and `oem-config.service` report failed in `systemctl --failed`. Both are
leftovers from the Ubuntu live/OEM installer that have no job to do on an installed system. They
affect nothing. Masking them is queued.

```sh
# if the red text bothers you before then:
sudo systemctl mask casper-md5check.service oem-config.service
```

---

## GPU-1: Rare GPU driver crash under heavy compositing

**Status:** open, upstream (vendor driver) · **Severity:** rare but disruptive

The vendor Mali kernel driver (`kbase`) has a use-after-free on GPU-context teardown that can crash
the desktop session. Observed **once**, on a long-running machine under an ultrawide display, and it
recovered on its own.

Worth being straight about the scope: `kbase` is bound to the GPU on every RK3588 image, this one
included, so the exposure exists regardless of what a browser is doing. Pi Desktop's browser video
decode runs on the **Panfrost** stack rather than the vendor blob, so it does not add to this risk.
The permanent fix is mainline's Panthor driver, which replaces `kbase` entirely.

---

## Scope and horizon

Pi Desktop is built on the **vendor 6.1 BSP kernel**, because that is what does 4K@120 and hardware
video on this SoC *today*. Mainline Linux is catching up — HDMI 2.0 support for the RK3588 is in
active review, FRL PHY support has landed, and the Panthor GPU driver plus a current Mesa will
eventually give a cleaner path than the one we're on. We intend to move when mainline carries
VP9/AV1 and 4K@120. Until then, the fork is the reason the features work.

---

## Fixed in v1.0

- **Browser video is now hardware-decoded.** Earlier documentation stated this was a permanent GL
  driver limitation on this SoC. **That was wrong, and the explanation we published was wrong.** The
  real cause was a packaging accident: upstream Mesa 25.x split its gallium drivers into a new binary
  package that the panfork Mesa build doesn't publish, so a routine `apt upgrade` silently replaced
  the panfork graphics stack with stock Mesa. That mismatch — not any hardware or driver limit — is
  what broke decode. v1.0 pins the correct stack and a build gate fails the image if the wrong Mesa
  ever ships again. Verified on a pristine, cold-booted image: `chrome://gpu` reports *Video Decode:
  Hardware accelerated*, `mpp_service` is held by Chromium, `rkvdec` clocks up, and 4K60 VP9 runs at
  ~118% CPU instead of 600%+.
- **USB boot hang.** The board no longer hangs at boot with a bus-powered USB audio interface
  attached. U-Boot probed the USB bus before storage; boot targets are now ordered storage-first.

## Reporting

Found something not listed here? [Open an issue](../../issues) — include your board revision, how you
installed (SD / eMMC / NVMe), your monitor's resolution and refresh rate, and the output of
`uname -a`. If it's video-related, a screenshot of `chrome://gpu` is worth a thousand words.
