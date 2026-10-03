# Security

## Reporting a vulnerability

Use GitHub's private vulnerability reporting:
**<https://github.com/defcom5-rockchip/pi-desktop/security/advisories/new>**

Please don't open a public issue for something exploitable. Include the image version (`cat
/etc/pi-desktop-release`), how to reproduce it, and what you think the impact is. The same contact is published
machine-readably at <https://defcom5-rockchip.github.io/pi-desktop/.well-known/security.txt>.

## What to expect

One maintainer, best effort. You get an acknowledgement when the report has been read, a fix as a recipe change
and, when it matters, a new image, and an entry in [CHANGELOG.md](CHANGELOG.md) / [KNOWN-ISSUES.md](KNOWN-ISSUES.md)
saying what it was. Credit in the changelog if you want it. There is no bounty, no response-time guarantee and no
embargo process beyond what GitHub's advisory workflow provides.

## Scope

**In scope** — what Pi-Desktop adds or changes:

- the eleven kernel patches;
- the recipe and overlay: services, the first-click installers (key pinning, repository files, the polkit
  actions), polkit rules, udev rules, sshd and apt configuration, the os-release keeper;
- the rebuilt packages: gnome-control-center `+pd3`, Rhythmbox 3.5.1, PipeWire/WirePlumber, MPP, rockchip-vaapi,
  librga, the libmali-hook shim;
- the website.

**Out of scope — report upstream instead:** bugs in unmodified Ubuntu, Armbian, GNOME, Chromium, Rockchip or Arm
components, and in Google Chrome, Firefox, Thunderbird, LibreOffice, GIMP, Inkscape and VSCodium (upstream
builds installed on first click). If Pi-Desktop's configuration is what exposes an upstream bug, tell us too.

## Security-relevant defaults (verified on the release candidate)

- **SSH** is on and listening from the first boot. Root may log in with a key only; your own account can log in
  with its password. Choose a strong password at the setup wizard, or set `PasswordAuthentication no` in
  `/etc/ssh/sshd_config.d/` if you don't need it.
- **No firewall** is installed (no `ufw`).
- **Ubuntu security updates install automatically** (`unattended-upgrades`). The kernel, device tree, bootloader,
  BSP and firmware packages are held and change only with Pi-Desktop releases — kernel fixes arrive with
  releases, not daily. Armbian's kernel branch and Ubuntu's advisories are where to watch.
- **polkit:** the socket-activated agent helper of polkit ≥ 126 needs `SO_PEERPIDFD` (Linux 6.5), which the 6.1
  kernel lacks, so the socket is masked and the classic setuid-root helper is used instead (`dpkg-statoverride`).
  That is the pre-126 configuration every Ubuntu release before 26.04 ran with.
- **Third-party repositories** added by the first-click installers use pinned signing keys: Google's
  `EB4C1BFD4F042F6DDDCCEC917721F63BD38B4796`, VSCodium's `1302DE60231889FE1EBACADC54678CF75A278D9C`. A key that
  does not match is refused and nothing is installed.
- **Hardware video** device nodes (`/dev/mpp_service`, `/dev/rga`) are opened to group `video` so the desktop
  user can decode in hardware; they are root-only on a stock Armbian image.
- **The journal is persistent** (200 MB cap) so crash evidence survives a reboot.
- **Closed components we cannot audit:** Arm's Mali user-space driver, Rockchip's boot binaries, the radio and GPU
  firmware (see [LEGAL.md](LEGAL.md)).

## Known hazards that are not vulnerabilities

The vendor Wi-Fi driver bug fixed by patch 0006 failed closed (the association was refused); it never joined a
network with weaker security. Nothing in Pi-Desktop's documentation asks you to lower your router's security
to work around it.
