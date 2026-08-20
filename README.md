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
Or wait for the daily cron job to fetch new versions from GitHub releases.

## How It Works
1. GitHub Actions runs daily checking for new T3 Code releases
2. `flatpak-external-data-checker` auto-updates the manifest (`x-checker-data` in `codes.t3.T3Code.yml`)
3. Flatter builds the Flatpak and publishes to GitHub Pages
4. Your system gets updates via `flatpak update`

## Security Notes for secureblue
- Flatpak bubblewrap sandboxing isolates the app from system
- No `fuse2` SUID host binary (AppImage is extracted at build time only)
- Electron `chrome-sandbox 4755` stays inside the Flatpak, not on the host
- No rpm-ostree layering = no reboots required
- Updates are atomic with rollback capability
- This follows [secureblue.dev/faq](https://secureblue.dev/faq#appimage)'s recommended app installation method (Flatpak)
