# Privacy

Plaid Pi collects nothing. There is no account, no telemetry of its own, no crash uploader of its own, and
nothing is sent to the maintainer. The image's privacy-policy link (`PRIVACY_POLICY_URL` in `/etc/os-release`)
points at this page.

What follows is what the image *does* send, by whom, and why, so you can judge it. It was checked on the 4.0
release image on 2026-10-03 by reading the installed configuration and services, not by assuming.
Rows marked 5.0 come from the 5.0 recipe; the 5.0 image's first-boot check (133 of 133 on 2026-10-19) confirms the
ones it covers (https apt sources, browsers from their vendors only, GNOME Software absent, Pro Updates present, the
dock's music pill off, nothing listening on port 111).

## What contacts the network by default

| When | What | Where | Why |
|---|---|---|---|
| First login (once) | Armbian's setup wizard (`armbian-firstlogin`), Armbian code | `ipinfo.io` over HTTPS (learns your public IP), then `ipwhois.app` over **plain HTTP** (looks that IP up, so the address crosses the network unencrypted) | to suggest your time zone and locale. 5-second timeout; skipped when there is no network. Plaid Pi's own addition to the wizard — the computer-name question — is local. |
| Boot, then continuously | chrony (NTP) | `1.ntp.ubuntu.com` … `4.ntp.ubuntu.com` with NTS, `ntp-bootstrap.ubuntu.com` | clock sync. Ubuntu's defaults. |
| Daily (5.0; on 4.0 only when you run apt, see SECURITY.md) | apt (`apt-daily` timers, `unattended-upgrades`) | over HTTPS in 5.0 (4.0: plain HTTP for the first two): `ports.ubuntu.com` (Ubuntu), `apt.armbian.com` (Armbian, which redirects to a mirror near you), `ppa.launchpadcontent.net` (the xtradeb/apps and mozillateam PPAs) — and, once you have installed them, `dl.google.com` (Chrome) and `download.vscodium.com` (VSCodium) | package lists; `unattended-upgrades` installs Ubuntu security updates on its own. The held kernel and boot packages are not touched. |
| In the background (4.0 only) | GNOME Software (PackageKit) | the same apt sources | checks for and downloads updates (`download-updates` is on). Ratings (ODRS) are off. **5.0 does not install GNOME Software**: updates come from the daily apt job and from Pro Updates when you open it. |
| Never by itself | Flatpak (installed, no remotes) | — | no Flatpak remote is configured, so Flathub is not contacted, until you add it: Pro Updates asks first and adds it for your user only. |
| When you open it (5.0) | Pro Updates (the update window) | DNS lookups of the update hosts it lists; your Flatpak remotes, if any (none by default); when you apply, the apt sources above; "Switch these to https" sends one HTTPS request to each mirror it is about to switch; `flathub.org` over HTTPS only if you agree to a Flatpak install, which adds Flathub for your user | shows and applies updates on request. No timer or autostart; nothing is sent to the maintainer. It keeps a log of its root actions in `/var/log/pro-updates.log`, backups of changed apt files in `/var/backups/pro-updates`, and your choices in `~/.config/pro-updates`. |
| Only if you switch it on | the Dhruva dock's music pill (off by default in 5.0) | `lrclib.net` over HTTPS; album-art addresses the playing app provides | when on, it looks up lyrics for every track you play (artist, title and length are sent to lrclib.net) and fetches album art. Plaid Pi ships it switched off; the rest of the dock contacts nothing. |
| Daily | `fwupd-refresh.timer` | `cdn.fwupd.org` (LVFS) | downloads the firmware catalogue. Nothing on this board is updated through it. |
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
- **Plaid Pi's own tools (5.0).** Killswitch and the Plaid Panel top bar contact nothing. Killswitch reads the
  local mount table and probes each mount with a local `stat`; the dock (Dhruva) contacts nothing unless you switch
  its music pill on (see the table).
- **Not installed:** `popularity-contest`, `ubuntu-report`, the Ubuntu Pro client, `snapd`, Armbian's
  `armbian-config`. Ubuntu's message-of-the-day news fetch (`motd-news`) is disabled. Armbian's daily
  `armbian-apt-updates` job only simulates an upgrade locally, and Armbian's `armbian-quotes` cron job (a quote of the
  day from `github.armbian.com`) is removed.
- **The login banner.** Armbian's message-of-the-day scripts include one that fetches your public address from
  `ipv4.whatismyip.akamai.com` at every console or SSH login. Plaid Pi switches that part off (`MOTD_DISABLE`),
  together with the rest of Armbian's banner; only a one-line Plaid Pi header and the pending-updates notice remain.
- **Plaid Pi's own services** — the screensaver, the Bluetooth sentinel, the hostname prompt, the
  headphone-follow and OUT1 services, the login-screen service, the os-release keeper — do not use the network.

## Browsers and the apps you install

- **Chromium** (xtradeb's build) is upstream Chromium and has Chromium's own network behaviour: Safe Browsing,
  component updates, Google sign-in through the API keys Debian's packaging carries. Google's privacy policy
  governs the Google services it talks to. Plaid Pi adds only the hardware-video flags; Debian's defaults
  include `--disable-pings`.
- **The first-click installers** each contact one place: Google Chrome → `dl.google.com` (the signing key, checked
  against a pinned fingerprint, and the package; Google's apt repository then stays configured so Chrome updates
  from Google); VSCodium → `gitlab.com` (the repository key, pinned fingerprint) and `download.vscodium.com`;
  Firefox and Thunderbird → the Mozilla Team PPA on `ppa.launchpadcontent.net`; LibreOffice, GIMP and Inkscape →
  `ports.ubuntu.com`. The installer logs to `/var/log/pidesktop-install-app.log` on the device.
- **Once installed, those apps have their own policies**: Google (Chrome), Mozilla (Firefox, Thunderbird), the
  VSCodium project (VSCodium is VS Code with Microsoft's telemetry removed), The Document Foundation, GIMP,
  Inkscape. Plaid Pi ships a Firefox enterprise-policy file that turns off Firefox telemetry, studies, Pocket
  and the first-run pages.

## The website and the issue tracker

The website is GitHub Pages; the issue tracker is GitHub. GitHub's privacy statement applies to both. The website
itself has no analytics, no cookies, no scripts and no third-party fonts. The maintainer keeps no list of users
and no data about them.

## Changes

If a later release changes any of this, this page changes with it and the [CHANGELOG](CHANGELOG.md) says so.
