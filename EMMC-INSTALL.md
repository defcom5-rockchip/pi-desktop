# Installing to eMMC on the Orange Pi 5B — and cleaning it first

The 5B's onboard eMMC is the best reason to own the board: it's faster and more reliable than
any microSD. But most "I installed it and the board misbehaves" reports trace back to **one
mistake**, and it's an easy one to make.

> ## ⚠️ The trap, in one paragraph
> On RK3588 the **bootloader and the operating system can come from different places.** The board's
> boot ROM looks for a bootloader on **eMMC before it looks at the SD card**. So if your eMMC still
> has the factory Android bootloader on it — or an old distro's — you can end up running a **brand
> new OS on top of years-old bootloader firmware**. That firmware initialises your RAM and manages
> power. Stale versions are a documented cause of **crashes and spontaneous reboots under heavy
> load**, and because the bootloader is shared, *the problem follows you across distros* — which is
> why people often report "it does it on Armbian too."
>
> **Copying a root filesystem onto the eMMC does not install a bootloader.** Only writing the
> **whole image** does.

---

## Which loader am I actually running?

Before anything else, find out whether you're on our firmware or someone else's:

```sh
sudo cat /proc/device-tree/chosen/u-boot,version 2>/dev/null
# or, if that's empty:
sudo strings /dev/mmcblk0 2>/dev/null | grep -m5 -i 'U-Boot 20'
```

Ours (as of Pi Desktop v1.0 / Pi Studio v1.4) reports roughly: **`spl-v1.13`, `bl31-v1.45`,
u-boot built 2026.** If you see something much older, an Android/Rockchip vendor string, or nothing
at all, you are booting a foreign loader — go to **Method A** below.

---

## Method A — the right way: write the full image to eMMC

This rewrites everything: bootloader, kernel, and root filesystem. It is the supported install.

**Easiest route — from a running Pi Desktop / Pi Studio booted off microSD:**

1. Boot the board from a microSD with our image on it.
2. Identify the eMMC device. **Do this carefully:**
   ```sh
   lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
   ```
   - Your **microSD** is the one with `/` mounted on it — usually `mmcblk1`.
   - The **eMMC** is the *other* `mmcblkN` with nothing mounted — usually `mmcblk0`.
   - ⚠️ **If you're not certain which is which, stop.** Writing to the wrong one destroys the card
     you are currently running from. Confirm with `findmnt /` and compare.
3. Write the image (replace `mmcblk0` with *your* eMMC device, and use your image's filename):
   ```sh
   xz -dc pi-desktop-v1.0-orangepi-5b.img.xz | sudo dd of=/dev/mmcblk0 bs=4M status=progress conv=fsync
   sync
   ```
4. Shut down, **remove the microSD**, power on. The board now boots entirely from eMMC.

**Alternative — from a PC:** put the board in **maskrom mode** (hold the MASKROM button while
applying power, USB-C to the PC) and write the image with Rockchip's `rkdeveloptool` or the
RKDevTool GUI. Use this when the board won't boot at all.

---

## Method B — wipe the eMMC first (when in doubt, or when things are weird)

If the eMMC previously held Android or another distro and you want a genuinely clean slate, zero the
front of the device before installing. That removes the partition table, the first-stage loader
(`idbloader`, at sector 64) and u-boot (at sector 16384) in one pass.

From a microSD boot, **after** confirming the device with `lsblk` and `findmnt /`:

```sh
# ⚠️ CHECK TWICE. This is not reversible.
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=64 status=progress
sudo sync
```

Then follow **Method A** to write the image.

> **Why 64 MB?** It comfortably covers the GPT, `idbloader`, u-boot and any leftover Android
> metadata, without spending time zeroing the whole device (which is unnecessary).

---

## Method C — the recovery net: maskrom

If the board won't boot at all — including after a bad flash — it is almost certainly **not
bricked**. RK3588 has a hardware recovery mode:

1. Power the board off, connect **USB-C to a PC**.
2. Hold the **MASKROM** button, apply power, release after a few seconds.
3. On the PC: `rkdeveloptool ld` should list the device in maskrom mode.
4. Erase and reflash:
   ```sh
   rkdeveloptool ef                 # erase flash
   # then write the loader + image per Rockchip's tooling
   ```

Boards recovered this way come back fully. Keep this section in mind *before* you experiment.

---

## After installing

- **First boot takes longer** — the filesystem expands to fill the eMMC.
- Verify you're on the expected firmware using the **"Which loader am I actually running?"** check
  above. If it still reports something foreign, the loader sectors weren't written — repeat with
  **Method B**, then **A**.
- Keep a microSD with a known-good image around. It's the fastest way to rescue or re-flash an
  eMMC, and it costs a few dollars.

## Still crashing after a clean install?

Then it isn't the eMMC, and the next suspect is **power**. The 5B draws hard when the CPU, GPU and
video decoder spike together (rapid video seeking is the classic trigger). Use a supply rated for
the board — a marginal phone charger will brown out under exactly that load, and it looks
identical to a software crash. A quick way to separate the two:

```sh
sudo apt install stress-ng
stress-ng --cpu 8 --timeout 600s
```

If the board reboots during that — with no video playing at all — it's power or firmware, not
software.

---

*Questions or a board that won't cooperate: open a
[Discussion](https://github.com/defcom5-rockchip/pi-desktop/discussions). Include your power supply,
how you installed, and the output of `lsblk` — that's usually enough to spot it.*
