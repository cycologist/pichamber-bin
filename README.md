# pichamber-bin

Arch Linux package for [PiChamber](https://github.com/RyderAsKing/PiChamber) — the
desktop app for running Pi Coding Agent sessions.

This repository repackages the official upstream Debian build
(`PiChamber-<version>-linux-amd64.deb` / `-linux-arm64.deb`) into a pacman
package, following the AUR convention of a `-bin` suffix for prebuilt software.
Nothing is compiled here: the upstream Electron bundle is installed as-is under
`/opt/PiChamber`.

**Status:** the AUR submission for `pichamber-bin` is pending. New AUR
registrations are currently suspended as part of the [active AUR malicious
packages incident](https://archlinux.org/news/active-aur-malicious-packages-incident/),
so this GitHub repository is the interim distribution channel. The PKGBUILD is
AUR-ready (`.SRCINFO` generated, checksums verified, `namcap` clean); publishing
becomes a single `git push` once registrations reopen.

## Install

### From the release package (no build step)

```bash
sudo pacman -U https://github.com/cycologist/pichamber-bin/releases/download/v1.0.7/pichamber-bin-1.0.7-1-x86_64.pkg.tar.zst
```

For later versions, replace `v1.0.7` / `1.0.7` with the tag you want, or list
the available assets:

```bash
gh release view --repo cycologist/pichamber-bin
```

### Build from source of this repo

```bash
sudo pacman -S --needed base-devel
git clone https://github.com/cycologist/pichamber-bin.git
cd pichamber-bin
makepkg -si
```

`makepkg` downloads the upstream `.deb` release, verifies its SHA-256, and `-si`
installs the result through pacman. The 450 MB payload is unpacked under `/opt`,
so the installed size is roughly 441 MiB.

## Requirements

Runtime dependencies are resolved by pacman:

| Package | Why |
|---|---|
| `gtk3` | Chromium UI toolkit |
| `nss` | Network security services |
| `alsa-lib` | audio backend |
| `libsecret` | credential storage |
| `libnotify` | desktop notifications |
| `at-spi2-core` | accessibility bridge |
| `xdg-utils` | opening links with the default browser |

Optional: `libdbusmenu-glib` for tray menu support (on GNOME you additionally
need an AppIndicator extension; KDE Plasma works natively).

## What gets installed

| Path | Contents |
|---|---|
| `/opt/PiChamber/` | Electron app, `pichamber` launcher, bundled libs, `resources/app.asar` |
| `/usr/bin/pichamber` | symlink to `/opt/PiChamber/pichamber` |
| `/usr/share/applications/pichamber.desktop` | desktop entry |
| `/usr/share/icons/hicolor/*/apps/pichamber.png` | icons (16–1024 px) |
| `/usr/share/licenses/pichamber-bin/LICENSE` | upstream MIT license |

The package `provides`/`conflicts` with `pichamber`, so it can be swapped for a
future source-built package without pacman complaining about file conflicts.

## Updating

Bump `pkgver` in `PKGBUILD`, refresh both checksums, and regenerate `.SRCINFO`:

```bash
updpkgsums
makepkg --printsrcinfo > .SRCINFO
```

Checksums for a new upstream release can be taken from the GitHub Releases API:

```bash
curl -s https://api.github.com/repos/RyderAsKing/PiChamber/releases/latest \
  | jq -r '.assets[] | select(.name|endswith(".deb")) | "\(.name) \(.digest)"'
```

Pushing a `v*` tag runs the GitHub Actions workflow, which builds the package in
a clean `archlinux:base-devel` container, checks that `.SRCINFO` is in sync with
`PKGBUILD`, and attaches the result to the release.

## Removing

```bash
sudo pacman -Rns pichamber-bin
```

Your sessions and settings stay in your home directory; find them with:

```bash
ls -d ~/.config/*hamber ~/.local/share/*hamber 2>/dev/null
```

## Troubleshooting

- **`The SUID sandbox helper binary was found, but is not configured correctly`**
  user namespaces are disabled. Either enable them
  (`sudo sysctl -w kernel.unprivileged_userns_clone=1`) or make the helper
  setuid: `sudo chmod 4755 /opt/PiChamber/chrome-sandbox`. The install script
  does this automatically when its user-namespace probe fails.
- **Blurry UI on Wayland** — Electron defaults to XWayland. Force native
  Wayland with `pichamber --ozone-platform-hint=auto`. To make it permanent,
  copy the desktop file to `~/.local/share/applications/` and append the flag to
  its `Exec=` line rather than editing the packaged copy.
- **No tray icon** — install `libdbusmenu-glib`, plus an AppIndicator extension
  on GNOME.
- **App exits silently** — run `/opt/PiChamber/pichamber` in a terminal to see
  Chromium's stderr output.

## Package details

- `pkgbase`: `pichamber-bin`
- `arch`: `x86_64`, `aarch64` (both upstream `.deb` builds are supported)
- `license`: MIT (see [upstream LICENSE](https://github.com/RyderAsKing/PiChamber/blob/main/LICENSE))
- `options`: `!strip !debug !emptydirs` — required, stripping a prebuilt
  Electron bundle breaks it
- Package layout follows the upstream `.deb` (`/opt/PiChamber`), which is why
  `namcap` reports ELF files outside a standard path; this is expected for
  prebuilt Electron applications.

This is an unofficial package. Report packaging issues here; report PiChamber
bugs to [RyderAsKing/PiChamber](https://github.com/RyderAsKing/PiChamber/issues).
