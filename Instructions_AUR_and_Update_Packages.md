# Instructions: AUR submission and package updates

Operational notes for this repository (`pichamber-bin`), the Arch Linux package
for [PiChamber](https://github.com/RyderAsKing/PiChamber).

- GitHub repo: <https://github.com/cycologist/pichamber-bin>
- Current release: <https://github.com/cycologist/pichamber-bin/releases/tag/v1.0.7>
- Upstream release used: `PiChamber-1.0.7-linux-amd64.deb` (SHA-256
  `af19f7f8315026b914219943afbb52123c2b2cf629b407ad1b614124dd020244`)

## Repository layout

| File | Purpose |
|---|---|
| `PKGBUILD` | Repackages the upstream `.deb`; `arch=('x86_64' 'aarch64')` |
| `.SRCINFO` | Generated with `makepkg --printsrcinfo`; must stay in sync |
| `pichamber-bin.install` | Sets `chrome-sandbox` permissions after install/upgrade |
| `README.md` | User-facing install/removal/troubleshooting documentation |
| `.github/workflows/build.yml` | Builds on `v*` tags and attaches the package to the release |
| `.gitignore` | Keeps `pkg/`, `src/`, `*.pkg.tar.*` and downloaded sources out of git |

Remotes:

```
origin  https://github.com/cycologist/pichamber-bin.git
aur     ssh://aur@aur.archlinux.org/pichamber-bin.git
```

## Release flow (GitHub)

Pushing a `v*` tag triggers `.github/workflows/build.yml`, which:

1. Starts an `archlinux:base-devel` container, refreshes the keyring and installs
   `git libarchive gtk3 nss alsa-lib libsecret libnotify at-spi2-core xdg-utils`.
2. Diffs `makepkg --printsrcinfo` against the committed `.SRCINFO` and fails if
   they differ — this is the check the AUR enforces on push.
3. Builds as an unprivileged `builder` user (`makepkg` refuses to run as root).
4. Uploads the tarball as a workflow artifact.
5. Attaches it to the release for that tag.

Asset overwriting is **disabled** (`overwrite_files: false`) so the published
SHA-256 of a release stays valid. Re-running a tag that already has an asset
fails on purpose — bump `pkgrel` and tag a new version instead.

The v1.0.7 release was produced by this workflow:

- Asset: `pichamber-bin-1.0.7-1-x86_64.pkg.tar.zst` (142.6 MB)
- SHA-256: `f6d00c1a2d3cc0cdd564aa48c3e52e6d8817a13664f99f26e05b4109df230fd9`
- 1114 files, 441.38 MiB installed

Direct install:

```bash
sudo pacman -U https://github.com/cycologist/pichamber-bin/releases/download/v1.0.7/pichamber-bin-1.0.7-1-x86_64.pkg.tar.zst
```

## Updating to a new upstream release

1. Find the new version and the `.deb` checksums:

   ```bash
   curl -s https://api.github.com/repos/RyderAsKing/PiChamber/releases/latest \
     | jq -r '.tag_name, (.assets[] | select(.name|endswith(".deb")) | "\(.name) \(.digest)")'
   ```

2. Edit `PKGBUILD`:

   - set `pkgver=<new version>` and reset `pkgrel=1`
   - update `sha256sums_x86_64` (amd64 `.deb`) and `sha256sums_aarch64` (arm64 `.deb`)
   - the `pichamber-license` checksum only changes if upstream edits `LICENSE`

   `updpkgsums` can do the checksum work, but it only downloads sources for the
   current architecture — verify the `aarch64` entry against the API digest
   above before committing.

3. Regenerate `.SRCINFO` with the same `makepkg` version used for the build:

   ```bash
   makepkg --printsrcinfo > .SRCINFO
   ```

4. Sanity-check locally:

   ```bash
   namcap PKGBUILD
   makepkg -f            # builds ./pichamber-bin-<ver>-<rel>-x86_64.pkg.tar.zst
   ```

   Expected remaining `namcap` output on the package is limited to the
   prebuilt-bundle issues: ELF files under `/opt/` and the baked-in RUNPATH in
   `sherpa-onnx.node`. These are inherent to repackaging upstream binaries.

5. Commit, push, tag:

   ```bash
   git commit -am "pichamber-bin <version>-1"
   git push origin master
   git tag -a v<version> -m "pichamber-bin <version>-1"
   git push origin v<version>
   ```

6. Watch the run and verify the asset:

   ```bash
   gh run list --limit 3
   gh run watch <run-id> --exit-status
   gh release view v<version> --json url,assets
   ```

## AUR submission (pending)

The PKGBUILD is AUR-ready — `.SRCINFO` generated, checksums verified against the
GitHub release API, `namcap PKGBUILD` clean, `provides`/`conflicts` set to
`pichamber`.

**Blocker:** new AUR registrations are suspended as part of the
[active AUR malicious packages incident](https://archlinux.org/news/active-aur-malicious-packages-incident/).
There is no manual queue, and staff have asked that the registration page not be
retried automatically. Monitor:

- Arch news feed: <https://archlinux.org/feeds/news/>
- aur-general archives: <https://lists.archlinux.org/archives/list/aur-general@lists.archlinux.org/>
- Arch status: <https://status.archlinux.org/>

Even after registering, creating new packages may still be unavailable while the
incident response continues.

### Already prepared

- SSH key: `~/.ssh/aur` (ed25519, no passphrase), public key
  `SHA256:P+zcX+YlS1HPrpZk0PJSEk5SnYCW2K5yxcykYHBlafo`
- `Maintainer:` line in `PKGBUILD` set to `Matthias von Davier <mvondavier@gmail.com>`
- Remote `aur` configured

### Steps once registration reopens

1. Register at <https://aur.archlinux.org/register> and confirm the email.
2. *My Account → SSH Public Key*: paste the single line from `~/.ssh/aur.pub`
   (the key itself, not the file path), then click **Update**.
3. Verify authentication:

   ```bash
   ssh -i ~/.ssh/aur aur@aur.archlinux.org
   ```

   Expect a banner ending in `Interactive git shell is not enabled.`
   If you get `Permission denied (publickey)`, the key is not stored on the
   account (or the account/email is not confirmed). Compare fingerprints with:

   ```bash
   ssh-keygen -lf <(cat ~/.ssh/aur.pub)
   ```

4. Push. The first push creates the package — there is nothing to create in the
   web UI beforehand:

   ```bash
   cd /home/matthias/python/piChamber_AUR
   git push aur master
   ```

5. Verify:

   - <https://aur.archlinux.org/pkgbase/pichamber-bin>
   - `yay -Ss pichamber` (the RPC index updates within a minute or so)

After the AUR package exists, prefer installing through an AUR helper
(`yay -S pichamber-bin`) so upgrades are tracked like any other package.

## Local installation and testing

Install the locally built package:

```bash
cd /home/matthias/python/piChamber_AUR
sudo pacman -U pichamber-bin-<version>-<rel>-x86_64.pkg.tar.zst
```

Or build and install in one step:

```bash
makepkg -si
```

Test without installing:

```bash
bsdtar -xf pichamber-bin-<version>-<rel>-x86_64.pkg.tar.zst -C /tmp/pichamber-test
/tmp/pichamber-test/opt/PiChamber/pichamber
```

Uninstall (user data stays in `$HOME`):

```bash
sudo pacman -Rns pichamber-bin
ls -d ~/.config/*hamber ~/.local/share/*hamber 2>/dev/null
```

## Notes and caveats

- `options=('!strip' '!debug' '!emptydirs')` is required: stripping the prebuilt
  Electron bundle breaks it.
- The package installs to `/opt/PiChamber` to match the upstream `.deb`, so
  `namcap` reports ELF files outside a standard path. This is expected for
  prebuilt Electron applications.
- `aarch64` is supported by the PKGBUILD (`source_aarch64`/`sha256sums_aarch64`)
  but is not built by CI: GitHub's hosted runners would need an Arch ARM rootfs
  or QEMU. Build it natively with `makepkg -si` on an ARM machine.
- The package `provides`/`conflicts` with `pichamber`, so it can later be
  swapped for a source-built package without file conflicts.
