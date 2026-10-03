# Installing to eMMC on the Orange Pi 5B — and cleaning it first

The 5B's onboard eMMC is the best reason to own the board: faster and more reliable than any microSD.
Most "I installed it and the board misbehaves" reports trace back to **one mistake**, and it's easy to make.

> ## ⚠️ The trap, in one paragraph
> On RK3588 the **bootloader and the operating system can come from different places.** The boot ROM
> loads the bootloader from **eMMC before it looks at the SD card**, and Armbian's bootloader then boots
> whichever system it finds first — a microSD card wins when one is inserted. So with an old loader on the
> eMMC you can be running a **brand-new OS on top of years-old bootloader firmware**. That firmware
> initialises your RAM and manages power; stale versions are a documented cause of **crashes and
> spontaneous reboots under heavy load**, and because the loader is shared, *the problem follows you
> across distros* — which is why people report "it does it on Armbian too."
>
> **Copying a root filesystem onto the eMMC does not install a bootloader.** Only an installer that
> writes the loader, or writing the **whole image**, does.

---

## Which loader am I actually running?

The loader sits in the first megabytes of the device, before the partitions. Read its version string
(replace `mmcblk0` with the eMMC if yours is numbered differently; see the `lsblk` step below):

```sh
sudo head -c 16M /dev/mmcblk0 | strings -n 10 | grep -m1 'U-Boot SPL'
```

A Pi-Desktop 4.0 loader reports Armbian's build, which looks like
`U-Boot SPL 2017.09_armbian-2017.09-S39cd-…-Bxxxx (Sep 14 2026 - …)`. The "2017.09" is what Rockchip's
U-Boot fork calls itself; the date and the `armbian` tag are what matter. An Android or Rockchip vendor
string, a much older date, or nothing at all means you are booting a foreign loader — use **Method A**.

The package that owns the loader on a Pi-Desktop system is `linux-u-boot-orangepi5b-vendor`; it is held,
so updates never replace it silently.

---

## Method A — the supported install, from a Pi-Desktop running on microSD

Boot Pi-Desktop from a microSD card first. Then identify the eMMC. **Do this carefully:**

```sh
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
findmnt /
```

- Your **microSD** is the one with `/` mounted on it — usually `mmcblk1`.
- The **eMMC** is the *other* `mmcblkN` with nothing mounted — usually `mmcblk0`.
- ⚠️ **If you're not certain which is which, stop.** Writing to the wrong one destroys the card you are
  running from.

Then pick one of the two routes.

### A1 — Armbian's installer (copies the system you are running)

```sh
sudo apt install armbian-config
sudo armbian-install
```

Choose *boot from eMMC — system on eMMC*. Armbian's installer writes the bootloader to the eMMC and copies
your running system onto it, including anything you have installed and set up since the first boot. When
it finishes: shut down, **remove the microSD**, power on.

(`armbian-config` is not in the Pi-Desktop image; this installs it from Armbian's repository for the
occasion. It can be removed again afterwards.)

### A2 — the exact release image (a clean copy, nothing carried over)

```sh
zstd -dc Pi-Desktop-4.0-Plaid.img.zst | sudo dd of=/dev/mmcblk0 bs=4M status=progress conv=fsync
sync
```

This writes bootloader, kernel and root filesystem exactly as released; the first boot on the eMMC runs the
setup wizard again. The `flash-emmc.sh` script published with each release does the same with guard rails:
it refuses the device you are running from, checks the image's SHA-256 first, asks for a typed confirmation
and reads the eMMC back to verify the write.

**From a PC instead:** put the board in **maskrom mode** (hold MASKROM while applying power, USB-C to the
PC) and write the image with Rockchip's `rkdeveloptool` or the RKDevTool GUI. Use this when the board won't
boot at all.

---

## Method B — wipe the eMMC first (when in doubt, or when things are weird)

If the eMMC previously held Android or another distro and you want a genuinely clean slate, zero the front
of the device before installing. That removes the partition table, the first-stage loader (`idbloader`,
at sector 64) and U-Boot (at sector 16384) in one pass.

From a microSD boot, **after** confirming the device with `lsblk` and `findmnt /`:

```sh
# ⚠️ CHECK TWICE. This is not reversible.
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=64 status=progress
sudo sync
```

Then follow **Method A**.

> **Why 64 MB?** It comfortably covers the GPT, `idbloader`, U-Boot and any leftover Android metadata,
> without spending time zeroing the whole device (which is unnecessary).

---

## Method C — the recovery net: maskrom

If the board won't boot at all — including after a bad flash — it is almost certainly **not bricked**.
RK3588 has a hardware recovery mode:

1. Power the board off, connect **USB-C to a PC**.
2. Hold the **MASKROM** button, apply power, release after a few seconds.
3. On the PC: `rkdeveloptool ld` should list the device in maskrom mode.
4. Erase and reflash with Rockchip's tooling (`rkdeveloptool ef`, then the loader and the image).

Boards recovered this way come back fully. Keep this section in mind *before* you experiment.

---

## After installing

- **The first boot takes longer** — the filesystem expands to fill the eMMC — and with route A2 the
  setup wizard runs again (user, password, computer name).
- Verify the loader with the **"Which loader am I actually running?"** command above. If it still reports
  something foreign, the loader sectors weren't written — repeat with **Method B**, then **A**.
- Keep a microSD with a known-good image around. It is the fastest way to rescue or re-flash an eMMC.

## Still crashing after a clean install?

Then it isn't the eMMC, and the next suspect is **power**. The 5B draws hard when the CPU, GPU and video
decoder spike together (rapid video seeking is the classic trigger). Use a supply rated for the board — a
marginal phone charger browns out under exactly that load, and it looks identical to a software crash. A
quick way to separate the two:

```sh
sudo apt install stress-ng
stress-ng --cpu 8 --timeout 600s
```

If the board reboots during that — with no video playing at all — it's power or firmware, not software.

---

*Questions or a board that won't cooperate: open an
[issue](https://github.com/defcom5-rockchip/pi-desktop/issues). Include your power supply, how you
installed, and the output of `lsblk` — that's usually enough to spot it.*
