# Legal notes

**This is a summary, not legal advice.** It says what Pi-Desktop is made of, under which terms, and where the
source is. If something here is wrong or missing, [open an issue](../../issues).

## Names and trademarks

Pi-Desktop is an independent project by defcom5-rockchip. It names other projects and companies only to say what
it is built from or runs on (nominative use). None of them is affiliated with Pi-Desktop or endorses it.

- **Ubuntu** is a trademark of Canonical Ltd. Pi-Desktop is built from Ubuntu 26.04 packages and keeps Ubuntu's
  package identity (`ID=ubuntu`) so that Ubuntu tooling and PPAs keep working. It is not Ubuntu, and Canonical has
  not reviewed or approved it.
- **Armbian** is a trademark of the Armbian project. Pi-Desktop is built with the Armbian build framework and uses
  Armbian's kernel branch and board-support packages as its base. It is not an Armbian image and does not use
  Armbian's logo.
- **Rockchip** and **RK3588** are marks of Rockchip Electronics Co., Ltd., the maker of the SoC, whose kernel
  branch, media libraries and boot binaries are used.
- **Arm** and **Mali** are marks of Arm Limited. The image ships Arm's Mali user-space driver under Arm's licence
  (below). Pi-Desktop does not use Arm's or Mali's marks to promote itself.
- **Orange Pi** is a mark of Shenzhen Xunlong Software Co., Ltd., the maker of the Orange Pi 5B that Pi-Desktop
  targets. Pi-Desktop is not made or endorsed by Xunlong.
- **Raspberry Pi** is a trademark of Raspberry Pi Ltd. **Pi-Desktop is not a Raspberry Pi product.** It does not
  run on Raspberry Pi boards and has nothing to do with "Raspberry Pi Desktop".
- **Google Chrome** (Google LLC), **Firefox** and **Thunderbird** (Mozilla Foundation), **LibreOffice** (The Document
  Foundation), **GIMP**, **Inkscape** and **VSCodium** (their respective projects) are offered as first-click
  installers that download each program from its own repository. Pi-Desktop does not ship them and uses their
  names only to label the installer. **VS Code** and **Microsoft** are marks of Microsoft Corporation; VS Code is
  not offered.
- **Chromium** in the image is the xtradeb team's build of the open-source Chromium browser from their Launchpad
  PPA.
- **GNOME** is a trademark of the GNOME Foundation. **Linux** is a registered trademark of Linus Torvalds.

## How the image is built

**Built on Armbian, by hook.** Pi-Desktop images are built from pinned Armbian build-framework inputs. Armbian owns
the RK3588 boot chain: U-Boot, partition layout, kernel branch, device trees, modules, firmware, board-support
packages and the first-boot filesystem resize. Pi-Desktop changes one thing there, eleven kernel patches applied
through Armbian's own `userpatches` mechanism, all published with the source. Everything else is a userspace layer
applied through Armbian's late image hook, `customize-image.sh`, and every line of it is in the recipe repository.
The images are credential-free: there is no default user or password, the first boot creates yours.

## Licence of Pi-Desktop's own work

- **Code** — the recipe (build config, customize script, overlay scripts, systemd units, polkit actions, udev
  rules), the first-click installers, the screensaver daemon, the Files add-on, the Bluetooth and audio services,
  and the package build scripts — is **GPL-3.0-or-later** ([LICENSE](LICENSE)). The kernel patches are under the
  kernel's GPL-2.0-only; a patch to another package is under that package's licence.
- **Artwork** — the Plaid wallpaper, the penguin-drop screensaver video and the script that generates it, the
  tartan Pi-Desktop logo and the installer icons — is **CC BY-SA 4.0**. Attribution: "Pi-Desktop artwork by
  defcom5-rockchip, CC BY-SA 4.0".
- Everything else in this repository is under [LICENSE](LICENSE) unless a file says otherwise.

## Complete corresponding source

A Pi-Desktop image is mostly other people's free software. For every copyleft component that Pi-Desktop builds
or changes, the complete corresponding source — including the scripts that control compilation and
installation — is published at the places below.

| Component | In 4.0 | Licence | Source |
|---|---|---|---|
| Linux kernel | 6.1.172, Armbian `rk-6.1-rkr7.2` (armbian/linux-rockchip `44bbd021d7808cab93d1f2d64c1f092038257576`) plus 11 Pi-Desktop patches | GPL-2.0-only | tag `pi-desktop-4.0` in [defcom5-rockchip/linux-rockchip](https://github.com/defcom5-rockchip/linux-rockchip); the patches, `.config` fragment and build hooks are also in the recipe |
| U-Boot | Armbian's build of [radxa/u-boot](https://github.com/radxa/u-boot) branch `next-dev-v2024.10`, commit `39cd993e5d6296635438e84f4576b3a9bf76f86e`, unmodified by Pi-Desktop (package `linux-u-boot-orangepi5b-vendor`) | GPL-2.0-or-later | that repository and commit; Armbian's build configuration in [armbian/build](https://github.com/armbian/build) |
| The recipe | Armbian `userpatches/`: `config-pidesktop3.conf`, `customize-image.sh`, `overlay/`, `kernel/`, `bootenv/`, package build scripts | GPL-3.0-or-later | [defcom5-rockchip/pi-desktop-recipe](https://github.com/defcom5-rockchip/pi-desktop-recipe) |
| gnome-control-center | `1:50.3-0ubuntu0.2+pd3.3` — Ubuntu's source plus one patch (the Screensaver group in Power) | GPL-2.0-or-later | Ubuntu source package `1:50.3-0ubuntu0.2`; `pidesktop-power-screensaver.patch` and the build script `rebuild-settings.sh` in the recipe (`gcc/`) |
| Rhythmbox | `3.5.1-1~pd3` — the GNOME 3.5.1 release tarball, one patch (prefer the dark theme), one package replacing Ubuntu's split packages | GPL-2.0-or-later | the GNOME tarball (SHA-256 recorded), `0001-prefer-dark-theme.patch` and `build-rhythmbox-deb.sh` in the recipe (`rhythmbox/`) |
| PipeWire / WirePlumber | `1.6.9-2~pd31` / `0.5.17-1~pd31` — Debian's source packages `1.6.9-2` and `0.5.17-1` rebuilt for Ubuntu 26.04 arm64; no code change, one changelog entry | MIT | the Debian source packages; the recipe holds the source packages or pointers to them |
| rockchip-vaapi | `2.2.0-1~pd3` — Pi-Desktop's VA-API driver for the Rockchip video engine | LGPL-2.1-or-later | [defcom5-rockchip/rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi) |
| librockchip-mpp (Rockchip MPP) | `1.5.0+git20260805.a8b19653+ds-0ubuntu1~rk1+pd31` — Rockchip MPP snapshot `a8b19653` (2026-08-05) with the packaging's VP9 presentation-queue fix, rebuilt for 26.04 | Apache-2.0 | source package in the recipe; upstream [rockchip-linux/mpp](https://github.com/rockchip-linux/mpp) |
| librga | `2.2.0+git20260725.26a50ef-0ubuntu1~rk1` — librga fork at `26a50ef`, rebuilt for 26.04 | Apache-2.0 | source package in the recipe |
| libmali-hook shim | the GPL part of the libmali package (`hook/*`), patched so Firefox's graphics probe passes (`EGL_EXT_device_query`) | GPL-2.0-or-later | `0001-libmali-hook-EGL_EXT_device_query-shim-for-Firefox-gfxtest.patch` in the recipe (`overlay/gpu/`); upstream [rockchip-linux/libmali](https://github.com/rockchip-linux/libmali) |
| Pi-Desktop services and scripts | screensaver daemon, first-click installers, hostname prompt, OUT1 and headphone-follow services, Bluetooth hciattach/sentinel, Files add-on, os-release keeper | GPL-3.0-or-later | the recipe overlay |
| Artwork | wallpaper, screensaver video and generator, logo, icons | CC BY-SA 4.0 | the recipe overlay (`overlay/pidesktop/backgrounds/`, `…/screensaver/`, `…/branding/`, the installer icons) |

Everything not in the table is an unmodified binary package from Ubuntu 26.04 (`ports.ubuntu.com`), the
`xtradeb/apps` and `mozillateam/ppa` PPAs on Launchpad, or Armbian (`apt.armbian.com`, source at
[github.com/armbian](https://github.com/armbian)), at the versions listed in the manifest published with each
release (also `/etc/pi-desktop-manifest.txt` in the image). Their source is available from those archives
(`apt source <package>`). If a source package named here is missing from the recipe repository, open an issue
and it will be added.

## Third-party binary components

| Component | What it is | Terms |
|---|---|---|
| Arm Mali user-space driver | `libmali-valhall-g610-g24p0-wayland-gbm` 1.9-1: Rockchip's packaging of Arm's proprietary graphics libraries (OpenGL ES, EGL, Vulkan) for the Mali-G610, plus a GPL-2+ hook library | Arm's End User Licence Agreement for the Mali userspace driver (LES-PRE-20769, 25 November 2015), reproduced in `/usr/share/doc/libmali-valhall-g610-g24p0-wayland-gbm/copyright` in the image. It allows redistribution with the notices intact and grants no right to use Arm's marks. The driver itself is binary-only; the hook source is at [rockchip-linux/libmali](https://github.com/rockchip-linux/libmali). |
| Rockchip boot binaries | the DDR-initialisation blob and the BL31 (Arm Trusted Firmware) binary embedded in `idbloader.img` and `u-boot.itb`, from Rockchip's `rkbin`, as bundled by Armbian's U-Boot package | Rockchip's terms in [rockchip-linux/rkbin](https://github.com/rockchip-linux/rkbin) (binary redistribution). |
| Radio and GPU firmware | package `armbian-firmware` 26.11.0-trunk — Armbian's bundle of vendor firmware for many boards (Broadcom/Cypress/Infineon, Realtek and others). On the Orange Pi 5B the files in use are the AP6275P Wi-Fi/Bluetooth firmware (`fw_bcm43752a2_pcie_ag.bin`, `clm_bcm43752a2_pcie_ag.blob`, `nvram_AP6275P.txt`, `BCM4362A2.hcd`) and Arm's Mali CSF firmware (`mali_csffw.bin`) | Each file under its vendor's redistribution terms. The package ships no licence texts — inherited from Armbian, same as every Armbian image. Pi-Desktop appends three Bluetooth-coexistence parameters to the AP6275P NVRAM text file (diverted, so firmware updates keep them). |

## No warranty

Pi-Desktop is distributed under the GNU General Public License, which says (version 3, section 15):

> THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW. EXCEPT WHEN OTHERWISE STATED
> IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY OF ANY KIND,
> EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
> FOR A PARTICULAR PURPOSE. THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD
> THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING, REPAIR OR CORRECTION.

Section 16 (Limitation of Liability) applies likewise; the full text is in [LICENSE](LICENSE). The same holds for
the image as a whole, to the extent the law allows. Flashing an image can destroy the data on the target device;
read [EMMC-INSTALL.md](EMMC-INSTALL.md) before writing to eMMC.

## Support

Pi-Desktop has one maintainer and is maintained on a best-effort basis. Security updates for the Ubuntu packages
come from Ubuntu's repositories for the life of Ubuntu 26.04. The kernel, device tree, bootloader and
board-support packages are built from Armbian's branch, held in the image, and refreshed with Pi-Desktop releases
for as long as Armbian builds that branch. Pi-Desktop's own packages are updated with releases. There is no
guaranteed support period and this is not a commercial product. Problems: [issues](../../issues); security
problems: [SECURITY.md](SECURITY.md).
