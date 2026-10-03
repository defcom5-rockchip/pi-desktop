# Privacy

Pi-Desktop collects nothing. There is no account, no telemetry of its own, no crash uploader of its own, and
nothing is sent to the maintainer. The image's privacy-policy link (`PRIVACY_POLICY_URL` in `/etc/os-release`)
points at this page.

What follows is what the image *does* send, by whom, and why, so you can judge it. It was checked on the 4.0
release candidate (test9) on 2026-10-03 by reading the installed configuration and services, not by assuming.

## What contacts the network by default

| When | What | Where | Why |
|---|---|---|---|
| First login (once) | Armbian's setup wizard (`armbian-firstlogin`), Armbian code | `ipinfo.io` (learns your public IP), then `ipwhois.app` (looks that IP up) | to suggest your time zone and locale. 5-second timeout; skipped when there is no network. Pi-Desktop's own addition to the wizard — the computer-name question — is local. |
| Boot, then continuously | chrony (NTP) | `1.ntp.ubuntu.com` … `4.ntp.ubuntu.com` with NTS, `ntp-bootstrap.ubuntu.com` | clock sync. Ubuntu's defaults. |
| Daily | apt (`apt-daily` timers, `unattended-upgrades`) | `ports.ubuntu.com` (Ubuntu), `apt.armbian.com` (Armbian), `ppa.launchpadcontent.net` (the xtradeb/apps and mozillateam PPAs) — and, once you have installed them, `dl.google.com` (Chrome) and `download.vscodium.com` (VSCodium) | package lists; `unattended-upgrades` installs Ubuntu security updates on its own. The held kernel and boot packages are not touched. |
| In the background | GNOME Software (PackageKit) | the same apt sources | checks for and downloads updates (`download-updates` is on). Ratings (ODRS) are off. No Flatpak remotes are configured, so Flathub is not contacted. |
| Daily | `fwupd-refresh.timer` | `cdn.fwupd.org` (LVFS) | downloads the firmware catalogue. Nothing on this board is updated through it. |
| Daily | Armbian's `armbian-quotes` cron job | `github.armbian.com/quotes.txt` | fetches the quotes shown in the terminal message of the day. |
| On the local network | Avahi (mDNS) | your LAN only | `.local` names. |

## What does not phone home

- **Crash reports.** Ubuntu's crash collector `apport` is installed and running; it writes crash data to
  `/var/crash` on the device. Nothing is uploaded: the GNOME crash dialog (`apport-gtk`) is not installed,
  "Send error reports to Canonical" is off, automatic reporting is off, and the uploader (`whoopsie`) is
  switched off twice — its systemd units are masked and `report_crashes=false` is set. It cannot be removed
  outright because Settings (gnome-control-center) depends on its preferences helper.
- **Location.** `geoclue` is installed but only runs when an application asks. GNOME's Location Services and
  Automatic Time Zone are off by default; submission of Wi-Fi scan data is off.
- **Online accounts.** GNOME Online Accounts is installed and contacts a provider only when you add one.
- **Not installed:** `popularity-contest`, `ubuntu-report`, the Ubuntu Pro client, `snapd`, Armbian's
  `armbian-config`. Ubuntu's message-of-the-day news fetch (`motd-news`) is disabled. Armbian's daily
  `armbian-apt-updates` job only simulates an upgrade locally.
- **Pi-Desktop's own services** — the screensaver, the Bluetooth sentinel, the hostname prompt, the
  headphone-follow and OUT1 services, the login-screen service, the os-release keeper — do not use the network.

## Browsers and the apps you install

- **Chromium** (xtradeb's build) is upstream Chromium and has Chromium's own network behaviour: Safe Browsing,
  component updates, Google sign-in through the API keys Debian's packaging carries. Google's privacy policy
  governs the Google services it talks to. Pi-Desktop adds only the hardware-video flags; Debian's defaults
  include `--disable-pings`.
- **The first-click installers** each contact one place: Google Chrome → `dl.google.com` (the signing key, checked
  against a pinned fingerprint, and the package; Google's apt repository then stays configured so Chrome updates
  from Google); VSCodium → `gitlab.com` (the repository key, pinned fingerprint) and `download.vscodium.com`;
  Firefox and Thunderbird → the Mozilla Team PPA on `ppa.launchpadcontent.net`; LibreOffice, GIMP and Inkscape →
  `ports.ubuntu.com`. The installer logs to `/var/log/pidesktop-install-app.log` on the device.
- **Once installed, those apps have their own policies**: Google (Chrome), Mozilla (Firefox, Thunderbird), the
  VSCodium project (VSCodium is VS Code with Microsoft's telemetry removed), The Document Foundation, GIMP,
  Inkscape. Pi-Desktop ships a Firefox enterprise-policy file that turns off Firefox telemetry, studies, Pocket
  and the first-run pages.

## The website and the issue tracker

The website is GitHub Pages; the issue tracker is GitHub. GitHub's privacy statement applies to both. The website
itself has no analytics, no cookies, no scripts and no third-party fonts. The maintainer keeps no list of users
and no data about them.

## Changes

If a later release changes any of this, this page changes with it and the [CHANGELOG](CHANGELOG.md) says so.
