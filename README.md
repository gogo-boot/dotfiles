# Dotfiles

Managed with [chezmoi](https://www.chezmoi.io/).
If you have installed git and brew. It will install the rest of applications and setup the Dotfiles on your machine

## Setup on a New Machine

```bash
# Install chezmoi and apply dotfiles in one command
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply gogo-boot
```

Or if chezmoi is already installed:

```bash
chezmoi init --apply gogo-boot
```

## Usage

### Edit a managed file

```bash
# Edit .zshrc (opens in $EDITOR, then apply changes)
chezmoi edit ~/.zshrc
chezmoi apply
```

### Pull latest changes from remote

```bash
# Fetch and apply updates from the repo
chezmoi update
```

### Add a new file to chezmoi

```bash
chezmoi add ~/.some-config
```

### See what would change before applying

```bash
chezmoi diff
```

## What's Managed

| File | Purpose |
|------|---------|
| `.zshrc` | Zsh configuration |
| `.zprofile` | Zsh profile (login shell) |
| `run_onchange_darwin-install-packages.sh.tmpl` | Auto-installs Homebrew packages |

### Packages (macOS)

Defined in `.chezmoidata/packages.yaml` — automatically installed via Homebrew on apply:

**Brews:** yq, jq, kubernetes-cli, kustomize, helm, age, sops, goenv, pyenv, nodenv

**Casks:** OpenLens, Gureumkim, Podman Desktop, Ice, DBeaver, KeePassXC
