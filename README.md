# ChopAStem releases

Public download host for **ChopAStem Desktop**, a free, private stem splitter for Apple Silicon Macs (macOS 14+).

**Download the latest build:** <https://chopastem.com/download/> · or see [Releases](../../releases) here.
Every release lists the SHA-256 of its `.dmg` so you can verify it: `shasum -a 256 ChopAStem-X.Y.Z.dmg`.

This repository contains **no source code**. It exists only so release files have a stable, public, free home.

## How the three repos fit together

```
 chopastem-Desktop (private)          chopastem-releases (public)          chopastem-site (private)
 Swift app source + tests             no source: just GitHub Releases      PHP site → https://chopastem.com
 ───────────────────────              ───────────────────────────          ───────────────────────────
 build → sign → notarize → .dmg ──▶   v0.4.0 + ChopAStem-0.4.0.dmg  ◀──── Download page links to it,
        (gh release create)           (public, so anyone can download)     /download/latest redirects to it
                                                                           appcast.xml (Sparkle, planned)
```

| Repo | What it holds | Who reads it |
|---|---|---|
| [`chopastem-Desktop`](https://github.com/TheMusicDev/chopastem-Desktop) | The macOS app: source, tests, build/notarize scripts, changelog, roadmap | developers |
| [`chopastem-releases`](https://github.com/TheMusicDev/chopastem-releases) | **Only** the built `.dmg` files, as GitHub Releases. No source. Public on purpose: a private repo's release files can't be fetched by browsers, Sparkle or Homebrew without a login | everyone downloading |
| [`chopastem-site`](https://github.com/TheMusicDev/chopastem-site) | The website at chopastem.com (plain PHP on Apache shared hosting, behind Cloudflare) | visitors, developers |

**Why the dmgs aren't in the site repo or on the web host:** GitHub hosts them free with no bandwidth worries, keeps every old version, and keeps the site repo small. The one URL that must stay on our own domain is the Sparkle feed (`https://chopastem.com/appcast.xml`), because it is baked into every installed app and can never change. The dmg links inside it can move freely.

**Shipping a release touches all three, in this order:**
1. **Desktop** (`chopastem-Desktop`): bump the version in `project.yml`, run `xcodegen generate`, write the `CHANGELOG.md` entry, commit, merge to `main`, then run **`script/release.sh`**. It builds, notarizes, makes the dmg, computes the SHA-256, runs steps 2 and 3 for you, and finally tags the commit `vX.Y.Z`. Details: the Desktop README.
2. **Releases** (`chopastem-releases`): done by the script. It creates GitHub Release `vX.Y.Z` with `ChopAStem-X.Y.Z.dmg` attached and the changelog entry as release notes.
3. **Site** (`chopastem-site`): done by the script **in your working tree only**. It sets `version`, `size_mb`, `sha256` in `public/inc/config.php` and copies `CHANGELOG.md` to `public/inc/changelog.md`. You review the diff, commit, merge, and sync `public/` to the host. The script never commits or pushes the site.

## Maintainers: rules for this repo

- Releases are created by `script/release.sh` in the Desktop repo (it runs `gh release create` against this repo). Nothing is committed here by hand.
- Tag format `vX.Y.Z`, asset name `ChopAStem-X.Y.Z.dmg`. The site builds its download URL from exactly that pattern (`public/inc/config.php` + `public/latest.php`), so don't rename assets.
- Never delete an old release: Sparkle's appcast and the Homebrew cask may still point at it.
