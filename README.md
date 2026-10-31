# T3 Code Flatpak Repository

Self-hosted Flatpak repository for [T3 Code](https://t3.codes) (`codes.t3.T3Code`) with automatic updates — secureblue-compliant (no AppImage/fuse2, no rpm-ostree layering).

Repackages the official `x86_64.AppImage` from [`pingdotgg/t3code` GitHub Releases](https://github.com/pingdotgg/t3code/releases) via `--appimage-extract` inside the Flatpak sandbox (`org.electronjs.Electron2.BaseApp` + `zypak`).

## Setup

### 1. Enable GitHub Pages
- Go to Settings → Pages
- Set Source to "GitHub Actions"

### 2. Add Repository to Your System
```bash
flatpak remote-add t3code https://julianrottenberg.github.io/t3code-flatpak/index.flatpakrepo
flatpak install t3code codes.t3.T3Code
```

### 3. Updates
Updates are automatic:
```bash
flatpak update
```
The build workflow polls upstream every 15 minutes and rebuilds automatically as soon as a new **stable** release appears (typically within ~15–30 minutes; GitHub's scheduler can lag under load).

## How It Works
1. A `check` job polls `releases/latest` for `pingdotgg/t3code` every 15 minutes. That endpoint only ever returns stable releases — never pre-releases or drafts — so nightly/preview builds can't trigger a rebuild.
2. Only when the tag differs from the version pinned in the manifest, `flatpak-external-data-checker` updates the manifest (`x-checker-data` in `codes.t3.T3Code.yml`) and commits it.
3. Flatter builds the Flatpak and publishes it to GitHub Pages.
4. Your system gets updates via `flatpak update`.

Pushes to `main` and manual `workflow_dispatch` runs always rebuild. One build runs at a time, so a publish is never cancelled halfway.

Cost: this repo is public, and GitHub Actions is free for standard runners on public repositories, so the polling runs cost nothing. The 15-minute poll is ~96 short jobs/day; the actual Flatpak build only runs when there is a new release.

Note: GitHub disables scheduled workflows after 60 days without repository activity. New releases (and the resulting manifest commits) keep it alive; if upstream goes quiet that long, re-enable the workflow from the Actions tab.
## Security Notes for secureblue
- Flatpak bubblewrap sandboxing isolates the app from system
- No `fuse2` SUID host binary (AppImage is extracted at build time only)
- Electron `chrome-sandbox 4755` stays inside the Flatpak, not on the host
- No rpm-ostree layering = no reboots required
- Updates are atomic with rollback capability
- This follows [secureblue.dev/faq](https://secureblue.dev/faq#appimage)'s recommended app installation method (Flatpak)
