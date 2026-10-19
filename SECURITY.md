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

**In scope** — what Plaid Pi adds or changes:

- the kernel: Armbian's Rockchip vendor tree merged with upstream stable 6.1.189, Plaid Pi's patches and its
  configuration changes;
- the recipe and overlay: services, the first-click installers (key pinning, repository files, the polkit
  actions), polkit rules, udev rules, sshd and apt configuration, the os-release keeper;
- the rebuilt packages: gnome-control-center `+pd3`, Rhythmbox 3.5.1, PipeWire/WirePlumber, MPP, rockchip-vaapi,
  librga, the libmali-hook shim, mpv `+pd1` and gst-plugins-good `+pd1`;
- Killswitch (5.0), which detaches a stale network mount with one `umount -f -l` through pkexec and never offers
  the system's own filesystems or your home; the Plaid Panel top-bar extension; the Dhruva dock as configured by
  Plaid Pi (the dock's own code is upstream's apart from one small Plaid Pi change, report its bugs there);
- Pro Updates, the update window, and its root helper (`/usr/libexec/pro-updates/pro-updates-helper`, started with
  pkexec): it can refresh the package lists, upgrade packages, hold packages and release the holds it made, install
  a local `.deb` or the flatpak package, switch named apt sources to https, and restore the browser pin file (which
  also removes the old pin files and drops Armbian's desktop component). Its own repository is
  [defcom5-rockchip/pro-updates](https://github.com/defcom5-rockchip/pro-updates);
- the website.

**Out of scope — report upstream instead:** bugs in unmodified Ubuntu, Armbian, GNOME, Chromium, Rockchip or Arm
components, and in Google Chrome, Firefox, Thunderbird, LibreOffice, GIMP, Inkscape and VSCodium (upstream
builds installed on first click). If Plaid Pi's configuration is what exposes an upstream bug, tell us too.

## Security-relevant defaults (5.0)

- **Kernel:** Linux 6.1.189 (upstream stable, 3 October 2026) merged into Armbian's Rockchip vendor tree, with
  Plaid Pi's patches. It adds Ubuntu's usual hardening: Yama ptrace restrictions, hardened usercopy, slab freelist
  hardening and randomisation, and zeroing memory on allocation. User namespaces stay on, because GNOME's image
  sandbox needs them. The kernel packages are held: kernel fixes arrive only with a new Plaid Pi image, which
  means reflashing (there is no in-place kernel update channel yet).
- **Updates install automatically:** each day apt refreshes its lists and `unattended-upgrades` installs Ubuntu's
  security updates plus new versions of anything installed from the xtradeb and Mozilla Team PPAs and Google's
  repository (Chromium, Firefox, Thunderbird, Chrome …). Armbian's own packages are left out. Armbian's own BSP
  package switches this daily job off; Plaid Pi 5.0 switches it back on (4.0 did not, see below).
- **apt sources use https:** Ubuntu's ports archive and Armbian's repository were plain http on 4.0. Packages
  were always checked against signed indexes either way; https stops the network from seeing which packages and
  versions you download.
- **Pro Updates** (the update window) does nothing until you open it and press a button: no timer, no autostart.
  The window's root work goes through one helper with a fixed set of commands and checked arguments (no shell),
  and it asks for your password for each change (no remembered authorisation); apt never removes a package from
  it. A "Hold back" hold also stops the automatic security updates for that package until you release it; the
  system's own holds cannot be released from the window. It adds Flathub only for your user and only after
  asking. Its Ubuntu Pro section says "not checked": the Ubuntu Pro
  client is not on the image.
- **GNOME Software is not installed** (5.0): it duplicated the update path and downloaded updates in the background.
  Security updates come from the daily apt job; Pro Updates shows and applies the rest when you open it.
- **Killswitch** offers only network mounts, GVfs shares and mounts under /media, /mnt and /run/media, never the
  system's filesystems or your home. Each unmount of a network or local mount asks for the administrator password
  (the standard pkexec prompt for `umount`); GVfs shares unmount through gio without a password.
- **Held packages** do not update until a new Plaid Pi release: the kernel, device tree, boot loader, BSP and
  firmware; PipeWire and WirePlumber; the Mali user-space driver; the video stack (MPP, rockchip-vaapi, librga);
  mpv; GNOME Settings (gnome-control-center); Rhythmbox. `apt-mark showhold` lists them.
- **gst-plugins-good** is rebuilt (`+pd1`, a FLAC seeking fix) but not held: Ubuntu's next security update of it
  replaces Plaid Pi's build, which keeps you on Ubuntu's security fixes and drops the FLAC fix until Ubuntu ships it.
- **SSH** is on and listening from the first boot. Root may log in with a key only; your own account can log in
  with its password. Choose a strong password at the setup wizard, or set `PasswordAuthentication no` in
  `/etc/ssh/sshd_config.d/` if you don't need it.
- **No firewall** is installed (no `ufw`). rpcbind (port 111) is off by default; it starts on demand when you mount
  an NFSv3 share.
- **Real-time audio limits:** every login session may use real-time priority up to 95 and lock unlimited memory,
  so PipeWire can run its audio thread without dropouts. A local program could use this to hog a CPU core or to
  lock a lot of memory; it does not grant any other privilege.
- **polkit:** the socket-activated agent helper of polkit ≥ 126 needs `SO_PEERPIDFD` (Linux 6.5), which the 6.1
  kernel lacks, so the socket is masked and the classic setuid-root helper is used instead (`dpkg-statoverride`).
  That is the pre-126 configuration every Ubuntu release before 26.04 ran with.
- **Third-party repositories** added by the first-click installers use pinned signing keys: Google's
  `EB4C1BFD4F042F6DDDCCEC917721F63BD38B4796`, VSCodium's `1302DE60231889FE1EBACADC54678CF75A278D9C`. A key that
  does not match is refused and nothing is installed.
- **Hardware video** device nodes (`/dev/mpp_service`, `/dev/rga`) are opened to group `video` and to the user at
  the screen (`uaccess`), so the desktop user can decode in hardware; they are root-only on a stock Armbian image.
- **Hardware watchdog:** if the kernel or systemd itself hangs for 30 seconds, the board resets itself. A frozen
  desktop on an otherwise running system does not trigger it.
- **Logs are in RAM:** Armbian keeps `/var/log`, the system journal included, on a 50 MB RAM disk and writes it to
  the card at a clean shutdown (or when it fills up). The journal survives a normal reboot; after a crash or a
  watchdog reset, the logs since boot are normally lost.
- **Closed components we cannot audit:** Arm's Mali user-space driver, Rockchip's boot binaries, the radio and GPU
  firmware (see [LEGAL.md](LEGAL.md)).

## If you run Pi-Desktop 4.0

4.0 never installed updates by itself: Armbian's BSP package switches the daily apt job off, and 4.0 did not switch
it back on. That same Armbian file also lets automatic updates install Armbian's own packages, including its
rebranded browser copies. So, in this order:

1. Apply the SYS-3 fix in [KNOWN-ISSUES.md](KNOWN-ISSUES.md) (it takes Armbian's browser copies out of apt's view).
2. Install the pending updates and write the same file 5.0 ships, which turns the daily job on and limits automatic
   updates to Ubuntu's security updates and the browser vendors (`#clear` drops Armbian's patterns):

```
sudo apt update && sudo apt upgrade
sudo tee /etc/apt/apt.conf.d/52pidesktop-unattended >/dev/null <<'EOF'
APT::Periodic::Enable "1";
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
#clear Unattended-Upgrade::Origins-Pattern;
Unattended-Upgrade::Origins-Pattern {
	"origin=LP-PPA-xtradeb-apps";
	"origin=LP-PPA-mozillateam";
	"origin=Google LLC";
};
EOF
```

3. Check: `sudo unattended-upgrade --dry-run -d 2>&1 | grep "Allowed origins"` must not list `origin=Armbian`.

The 4.0 kernel (6.1.172) gets no further fixes: install the 5.0 image for the 6.1.189 kernel.

## Known hazards that are not vulnerabilities

The vendor Wi-Fi driver bug fixed by patch 0006 failed closed (the association was refused); it never joined a
network with weaker security. Nothing in Plaid Pi's documentation asks you to lower your router's security
to work around it.
