# Packaging

This project ships a zero-npm-dependency Node.js GUI that calls host `warp-cli`.
Packages do **not** bundle Cloudflare WARP — install the official WARP client first:
https://developers.cloudflare.com/warp-client/get-started/linux/

## Formats

| Artifact | Depends on | Notes |
|----------|------------|-------|
| `.deb` / `.rpm` / Arch `.pkg.tar.zst` | System `nodejs >= 20` | Built with [nfpm](https://nfpm.goreleaser.com/) |
| `.AppImage` | Host `warp-cli` | Bundles Node 20; x86_64 |
| `.flatpak` | Host `warp-cli` | Calls `flatpak-spawn --host warp-cli` (sandbox escape by design) |
| `.snap` (classic) | Host `warp-cli` | Classic confinement required |
| `*-src.tar.gz` + `PKGBUILD` | — | For AUR / manual builds |
| `SHA256SUMS` | — | Always published with releases |

Store publishing (Flathub, Snap Store, AUR) is **manual** — CI only attaches artifacts to GitHub Releases.

## Local commands

```bash
npm run package:stage      # FHS tree under dist/payload
npm run package:deb
npm run package:rpm
npm run package:arch
npm run package:appimage
npm run package:flatpak    # needs flatpak-builder + Freedesktop 24.08
npm run package:snap       # needs snapcraft
npm run package:source
npm run package:checksums
```

## Install layout (deb/rpm/arch)

```
/usr/bin/cloudflare-one-gui
/usr/lib/cloudflare-one-gui/   # server.js, public/, bin/, scripts/, assets/
/usr/share/applications/cloudflare-one-gui.desktop
/usr/share/icons/hicolor/scalable/apps/cloudflare-one-gui.svg
/usr/lib/systemd/user/cloudflare-one-gui.service
```

Enable the optional user service after install:

```bash
systemctl --user enable --now cloudflare-one-gui.service
```

## Signing

CI publishes `SHA256SUMS` for every release. Optional GPG signing of `.deb`/`.rpm` can be
enabled later by setting repository secrets `GPG_PRIVATE_KEY` and `GPG_PASSPHRASE` and
extending [`packaging/scripts/checksums.sh`](../packaging/scripts/checksums.sh).

## Architecture

v1 packages target **x86_64** (AppImage / Flatpak / Snap). Deb/rpm/arch use `all`/`any`
because the payload is architecture-independent JavaScript (system Node provides the arch).
