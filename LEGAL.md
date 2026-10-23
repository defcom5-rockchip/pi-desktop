# Legal notes

**This is a summary, not legal advice.** It says what Plaid Pi is made of, under which terms, and where the
source is. If something here is wrong or missing, [open an issue](../../issues).

## Names and trademarks

Plaid Pi (until 5.0 called Pi-Desktop) is an independent project by defcom5-rockchip. The source and recipe repositories, and the package, service and file names inside the image, keep the pi-desktop / pidesktop name internally. It names other projects and companies only to say what
it is built from or runs on (nominative use). None of them is affiliated with Plaid Pi or endorses it.

- **Plaid** in Plaid Pi is the pattern, after the tartan logo. Plaid Pi is not affiliated with Plaid Inc. or any other
  company of that name.
- **Ubuntu** is a trademark of Canonical Ltd. Plaid Pi is built from Ubuntu 26.04 packages and keeps Ubuntu's
  package identity (`ID=ubuntu`) so that Ubuntu tooling and PPAs keep working. It is not Ubuntu, and Canonical has
  not reviewed or approved it. **Ubuntu Pro** is Canonical's subscription service; Pro Updates names it to report
  what the Ubuntu Pro client lists, and is not affiliated with or endorsed by Canonical.
- **Armbian** is a trademark of the Armbian project. Plaid Pi is built with the Armbian build framework and uses
  Armbian's kernel branch and board-support packages as its base. It is not an Armbian image and does not use
  Armbian's logo.
- **Rockchip** and **RK3588** are marks of Rockchip Electronics Co., Ltd., the maker of the SoC, whose kernel
  branch, media libraries and boot binaries are used.
- **Arm** and **Mali** are marks of Arm Limited. The image ships Arm's Mali user-space driver under Arm's licence
  (below). Plaid Pi does not use Arm's or Mali's marks to promote itself.
- **Orange Pi** is a mark of Shenzhen Xunlong Software Co., Ltd., the maker of the Orange Pi 5 and 5B that Plaid Pi
  targets. Plaid Pi is not made or endorsed by Xunlong.
- **Raspberry Pi** is a trademark of Raspberry Pi Ltd. **Plaid Pi is not a Raspberry Pi product.** It does not
  run on Raspberry Pi boards and has nothing to do with "Raspberry Pi Desktop".
- **Google Chrome** (Google LLC), **Firefox** and **Thunderbird** (Mozilla Foundation), **LibreOffice** (The Document
  Foundation), **GIMP**, **Inkscape** and **VSCodium** (their respective projects) are offered as first-click
  installers that download each program from its own repository. Plaid Pi does not ship them and uses their
  names only to label the installer. **VS Code** and **Microsoft** are marks of Microsoft Corporation; VS Code is
  not offered. For Firefox, Plaid Pi ships one small library of its own, libpidesktop-ffmali (LGPL-2.1-or-later),
  which its Firefox launcher loads so the unmodified Firefox can use the hardware video decoder with the Mali driver;
  it is not part of Firefox and is not made or endorsed by Mozilla.
- **Chromium** in the image is the xtradeb team's build of the open-source Chromium browser from their Launchpad
  PPA.
- **GNOME** is a trademark of the GNOME Foundation. **Linux** is a registered trademark of Linus Torvalds.

## How the image is built

**Built on Armbian, by hook.** Plaid Pi images are built from pinned Armbian build-framework inputs. Armbian owns
the RK3588 boot chain: U-Boot, partition layout, kernel branch, device trees, modules, firmware, board-support
packages and the first-boot filesystem resize. Plaid Pi changes the kernel there: from 5.0 it merges upstream stable
fixes into Armbian's branch and applies its own patches through Armbian's `userpatches` mechanism, all published
with the source. Everything else is a userspace layer
applied through Armbian's late image hook, `customize-image.sh`, and every line of it is in the recipe repository.
The images are credential-free: there is no default user or password, the first boot creates yours.

## Licence of Plaid Pi's own work

- **Code** — the recipe (build config, customize script, overlay scripts, systemd units, polkit actions, udev
  rules), the first-click installers, the screensaver daemon, the Files add-on, the Bluetooth and audio services,
  and the package build scripts — is **GPL-3.0-or-later** ([LICENSE](LICENSE)). The kernel patches are under the
  kernel's GPL-2.0-only; a patch to another package is under that package's licence.
- **Artwork** — the Plaid wallpaper, the penguin-drop screensaver video and the script that generates it, the
  tartan Plaid Pi logo and the installer icons — is **CC BY-SA 4.0**. Attribution: "Plaid Pi artwork by
  defcom5-rockchip, CC BY-SA 4.0".
- Everything else in this repository is under [LICENSE](LICENSE) unless a file says otherwise.

## Complete corresponding source

A Plaid Pi image is mostly other people's free software. For every copyleft component that Plaid Pi builds
or changes, the complete corresponding source — including the scripts that control compilation and
installation — is published at the places below.

| Component | Version (5.5; 5.0 and 4.0 where noted) | Licence | Source |
|---|---|---|---|
| Linux kernel | 5.0 and 5.5: `6.1.189-vendor-rk35xx`: Armbian `rk-6.1-rkr7.2` (armbian/linux-rockchip `44bbd021d7808cab93d1f2d64c1f092038257576`) merged with upstream stable v6.1.189 (merge `7b6195db`, vendor-driver interface fixes `f3411423`), plus the Plaid Pi patches 0001–0017 (the display, Wi-Fi and Bluetooth fixes, Armbian's rtw88 Wi-Fi backport and two Wi-Fi build fixes); Armbian's build also adds its out-of-tree Wi-Fi drivers. 4.0: 6.1.172, the same Armbian commit plus 11 Plaid Pi patches | GPL-2.0-only | [defcom5-rockchip/linux-rockchip](https://github.com/defcom5-rockchip/linux-rockchip): tag `pi-desktop-5.0` (5.0, the merge plus the in-tree patches as commits), tag `pi-desktop-4.0` (4.0); all patches also in the recipe (`kernel/`, including the fix for the Armbian-added drivers, which exist only inside Armbian's build); the Armbian-added drivers come from the Armbian build framework (`lib/functions/compilation/patch/drivers_network.sh`) |
| U-Boot | Armbian's build of [radxa/u-boot](https://github.com/radxa/u-boot) branch `next-dev-v2024.10`, commit `39cd993e5d6296635438e84f4576b3a9bf76f86e`, unmodified by Plaid Pi (package `linux-u-boot-orangepi5b-vendor`) | GPL-2.0-or-later | that repository and commit; Armbian's build configuration in [armbian/build](https://github.com/armbian/build) |
| The recipe | Armbian `userpatches/`: `config-pidesktop3.conf`, `customize-image.sh`, `overlay/`, `kernel/`, `bootenv/`, package build scripts | GPL-3.0-or-later | [defcom5-rockchip/pi-desktop-recipe](https://github.com/defcom5-rockchip/pi-desktop-recipe) |
| gnome-control-center | `1:50.3-0ubuntu0.2+pd3.4` (4.0: `+pd3.3`) — Ubuntu's source plus one patch (the Screensaver group in Power) | GPL-2.0-or-later | Ubuntu source package `1:50.3-0ubuntu0.2`; `pidesktop-power-screensaver.patch` and the build script `rebuild-settings.sh` in the recipe (`gcc/`) |
| Rhythmbox | `3.5.1-1~pd3` — the GNOME 3.5.1 release tarball, one patch (prefer the dark theme), one package replacing Ubuntu's split packages | GPL-2.0-or-later | the GNOME tarball (SHA-256 recorded), `0001-prefer-dark-theme.patch` and `build-rhythmbox-deb.sh` in the recipe (`rhythmbox/`) |
| PipeWire / WirePlumber | `1.6.10-1~pd55` / `0.5.18-1~pd55` (5.0: `1.6.9-2~pd31` / `0.5.17-1~pd31`) — upstream PipeWire 1.6.10 on Debian's `1.6.9-2` packaging (with Debian's CVE-2026-14330 patch, which 1.6.10 does not contain) and Debian's WirePlumber source package `0.5.18-1`, rebuilt for Ubuntu 26.04 arm64 | MIT | the source packages (`.dsc`, `.orig` tarball, `.debian.tar.xz`) published with the 5.5 release; Debian's packaging at salsa.debian.org/utopia-team |
| rockchip-vaapi | `3.5.0-1` (Number Nine "Pepper SGT", tag `v3.5.0`; 4.0: `2.2.0-1~pd3`) — Plaid Pi's VA-API driver for the Rockchip video engine | LGPL-2.1-or-later | [defcom5-rockchip/rockchip-vaapi](https://github.com/defcom5-rockchip/rockchip-vaapi), release [v3.5.0](https://github.com/defcom5-rockchip/rockchip-vaapi/releases/tag/v3.5.0) |
| libpidesktop-ffmali (5.0) | `1.1` — library Plaid Pi's Firefox launcher loads so the unmodified Firefox can use the hardware video decoder with the Mali driver | LGPL-2.1-or-later | `/usr/share/pi-desktop/firefox/` in the image; source, build script and test in the recipe (`userpatches/overlay/pidesktop/firefox/ffmali/`) |
| mpv | 5.5: `0.41.0-2ubuntu4+pd2` (5.0: `+pd1`; mpv, libmpv2) — Ubuntu's mpv source package `0.41.0-2ubuntu4` plus two patches (integer-format and refused video plane imports; a precision fix in `vo=gpu`'s integer-texture pass), Plaid Pi's `/etc/mpv/mpv.conf` (from 5.5 with a larger cache for network files), PipeWire audio output off | mpv GPL-2.0-or-later; as built (linked with libcdio, GPL-3.0+) conveyed under GPL-3.0-or-later; the patches LGPL-2.1-or-later | the patched source package (`.dsc`, Ubuntu's `.orig.tar.gz`, our `.debian.tar.xz`) published with the 5.5 release; patches and `build-mpv-deb.sh` in the recipe (`mpv/`) |
| gst-plugins-good | 5.0 and 5.5: `1.28.2-2ubuntu0.4+pd1` (gstreamer1.0-plugins-good, -gtk3, -pulseaudio) — Ubuntu's source plus upstream commit `ef0f9938` (flacparse: resync after a seek) | LGPL-2.1-or-later | Ubuntu source package `1.28.2-2ubuntu0.4`; the patch and `build-gst-plugins-good-deb.sh` in the recipe (`gst-plugins-good/`); the patched source package is published with the 5.0 release |
| librockchip-mpp (Rockchip MPP) | `1.5.0+git20260805.a8b19653+ds-0ubuntu1~rk1+pd34` (librockchip-mpp1, librockchip-mpp-dev, librockchip-vpu1; 5.0: `+pd33`, 4.0: `+pd31`) — Rockchip MPP snapshot `a8b19653` (2026-08-05) with the packaging's VP9 presentation-queue fix, rebuilt for 26.04, plus eight Plaid Pi patches: seven for AV1 (a padding-OBU tile-offset fix and the deferred reference refresh that rockchip-vaapi 3.5.0 uses; two added exported functions, soname unchanged) and one for VP9 (the frame label wraps within 1..0xffff on every SoC, so RK3588 decodes VP9 on both cores with MPP's default number of frames in flight: the form Rockchip's reviewer asked for in [rockchip-linux/mpp#976](https://github.com/rockchip-linux/mpp/pull/976), accepted by that reviewer) | Apache-2.0 (no source obligation; offered anyway) | upstream [rockchip-linux/mpp](https://github.com/rockchip-linux/mpp) at `a8b19653`; the patches (`debian/patches/0001`–`0008`) and the packaging in the source package published with the 5.5 release and in the recipe's SOURCES.md |
| librga | `2.2.0+git20260725.26a50ef-0ubuntu1~rk1` — librga fork at `26a50ef`, rebuilt for 26.04 | Apache-2.0 (no source obligation) | the librga fork at `26a50ef`, see the recipe's SOURCES.md |
| libmali-hook shim | the GPL part of the libmali package (`hook/*`), patched so Firefox's graphics probe passes (`EGL_EXT_device_query`) | GPL-2.0-or-later | `0001-libmali-hook-EGL_EXT_device_query-shim-for-Firefox-gfxtest.patch` in the recipe (`overlay/gpu/`); upstream [rockchip-linux/libmali](https://github.com/rockchip-linux/libmali) |
| Pro Updates | 5.0 and 5.5: `1.5.0` — the update window, its command-line tool and root helper (`pro-updates`, `pro-updates-gui`, `/usr/libexec/pro-updates/pro-updates-helper`), with its own icon | GPL-3.0-or-later | [defcom5-rockchip/pro-updates](https://github.com/defcom5-rockchip/pro-updates), tag `v1.5.0`; the `.deb` carries its copyright file |
| Plaid Pi services and scripts | screensaver daemon, first-click installers, hostname prompt, OUT1 and headphone-follow services, Bluetooth hciattach/sentinel, Files add-on, os-release keeper; 5.0: Killswitch (mount list and stale-mount detach) and the Plaid Panel top-bar extension; 5.5: Network Drive (window, root helper, polkit action, Files menu entry) | GPL-3.0-or-later | the recipe overlay |
| Dhruva (the dock, 5.0) | 3.1 by NarkAgni, [narkagni/dhruva](https://github.com/narkagni/dhruva) commit `95462d4`, with one Plaid Pi change (`src/core/autohide/EdgeDetection.js`, 2026-10-13: the dock's pressure barrier is rebuilt when the monitor geometry changes), built with the project's own `manage.sh pack` plus its LICENSE file | GPL-3.0-or-later | that repository at that commit; the source tarball of that commit (including the translation sources behind the compiled `.mo` files) and the Plaid Pi change as a patch (`dhruva-plaid-pi-EdgeDetection.patch`) are published with the 5.0 release; `/usr/share/pi-desktop/look/README` records how it was built |
| Plaid icon theme (5.0) | the Adwaita icon theme's folder icons (GNOME Project, adwaita-icon-theme 50.0) recoloured to the Plaid palette, plus 16 px renders | CC BY-SA 3.0 US or LGPL-3.0, as Adwaita | `/usr/share/icons/Plaid` and its COPYING; the recolouring script `/usr/share/pi-desktop/look/make-plaid-icons.sh` |
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
| Radio and GPU firmware | package `armbian-firmware` 26.11.0-trunk — Armbian's bundle of vendor firmware for many boards (Broadcom/Cypress/Infineon, Realtek and others). On the Orange Pi 5B the files in use are the AP6275P Wi-Fi/Bluetooth firmware (`fw_bcm43752a2_pcie_ag.bin`, `clm_bcm43752a2_pcie_ag.blob`, `nvram_AP6275P.txt`, `BCM4362A2.hcd`) and Arm's Mali CSF firmware (`mali_csffw.bin`) | Each file under its vendor's redistribution terms. The package ships no licence texts — inherited from Armbian, same as every Armbian image. Plaid Pi appends three Bluetooth-coexistence parameters to the AP6275P NVRAM text file (diverted, so firmware updates keep them). |

## No warranty

Plaid Pi is distributed under the GNU General Public License, which says (version 3, section 15):

> THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW. EXCEPT WHEN OTHERWISE STATED
> IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY OF ANY KIND,
> EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
> FOR A PARTICULAR PURPOSE. THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD
> THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING, REPAIR OR CORRECTION.

Section 16 (Limitation of Liability) applies likewise; the full text is in [LICENSE](LICENSE). The same holds for
the image as a whole, to the extent the law allows. Flashing an image can destroy the data on the target device;
read [EMMC-INSTALL.md](EMMC-INSTALL.md) before writing to eMMC.

## Support

Plaid Pi has one maintainer and is maintained on a best-effort basis. Security updates for the Ubuntu packages
come from Ubuntu's repositories for the life of Ubuntu 26.04. The kernel is Armbian's Rockchip vendor tree (in 5.0
merged with upstream stable 6.1.189 by Plaid Pi); it, the device tree, bootloader and board-support packages are
held in the image and refreshed only with Plaid Pi releases. Plaid Pi's own packages are updated with releases. There is no
guaranteed support period and this is not a commercial product. Problems: [issues](../../issues); security
problems: [SECURITY.md](SECURITY.md).
