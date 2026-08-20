# Full CLI Setup Guide

Everything below can be done from the terminal without touching the GitHub web UI.

## Prerequisites

Install `gh` (GitHub CLI):

```bash
# macOS / Linux (Homebrew)
brew install gh

# Fedora
sudo dnf install -y gh

# Ubuntu/Debian
sudo apt update && sudo apt install -y gh
```

Login to GitHub:

```bash
gh auth login --web
```

## Option 1: Automated Script (Recommended)

Run the provided script from inside the repo directory:

```bash
cd t3code-flatpak
./setup-repo.sh
```

This handles: repo creation → push → Pages enable → workflow trigger.

## Option 2: Manual CLI Commands

If you prefer to run each step yourself:

### 1. Create the repo

```bash
REPO_NAME="t3code-flatpak"
USERNAME=$(gh api user -q '.login')

cd /var/home/julian/t3code-flatpak

gh repo create "$REPO_NAME" \
    --public \
    --description "Self-hosted Flatpak repo for T3 Code with auto-updates" \
    --source=. \
    --remote=origin \
    --push
```

### 2. Enable GitHub Pages (GitHub Actions source)

```bash
gh api --method POST \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2022-11-28" \
    "/repos/$USERNAME/$REPO_NAME/pages" \
    -f build_type="workflow"
```

If that fails, the push already triggered a Pages build request. Just go to **Settings → Pages** and select "GitHub Actions" as the source.

### 3. Trigger the first build

```bash
gh workflow run build.yml --repo "$USERNAME/$REPO_NAME"
```

### 4. Monitor the build

```bash
gh run list --repo "$USERNAME/$REPO_NAME" --limit 5
gh run watch --repo "$USERNAME/$REPO_NAME"
```

### 5. Install on your machine

```bash
flatpak remote-add t3code "https://$USERNAME.github.io/$REPO_NAME/index.flatpakrepo"
flatpak install t3code codes.t3.T3Code
```

## Useful Commands

| Task | Command |
|------|---------|
| Check repo | `gh repo view "$USERNAME/$REPO_NAME" --web` |
| Watch Actions | `gh run watch --repo "$USERNAME/$REPO_NAME"` |
| Pull latest | `git pull origin main` |
| Push updates | `git push origin main` |
| Delete repo | `gh repo delete "$USERNAME/$REPO_NAME" --yes` |
| List remotes | `git remote -v` |

## Troubleshooting

**"Pages API returned 404"**
→ The repository may need a few seconds after creation before Pages can be enabled. Wait 10s and retry, or enable manually in Settings → Pages.

**"Workflow not found"**
→ The `.github/workflows/build.yml` must be on the default branch before `gh workflow run` works. Push first, then trigger.

**"flatpak remote-add fails"**
→ The first Pages deploy takes ~2-3 minutes. Check `gh run list` until the deploy job is green.
