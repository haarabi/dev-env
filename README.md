# Dev Environment Setup (macOS)

## 🎯 Purpose & Strategy

This project defines a **portable, idempotent, and intuitive** macOS development environment that is:

- ✅ Fast to restore on a new laptop or after a system failure
- 🧱 Modular and maintainable using organized scripts and folders
- 🔐 Secure — sensitive files are encrypted and synced via Bitwarden
- 📦 Git-tracked (via `yadm`) for dotfile management and reproducibility

> ⚠️ This environment is tailored **only for macOS (Apple Silicon)**. Some tooling may work cross-platform, but scripts and paths are macOS-specific.

---

## 🚀 Bootstrap Strategy

The environment is broken into modular parts under `~/bin/`, making it easy to automate, debug, and customize:

### `~/bin/setup`
- **TUI launcher** built with [gum](https://github.com/charmbracelet/gum)
- Provides an interactive menu to run:
  - `restore` (initial setup)
  - `finalize` (plugin setup)
  - `backup.sh` / `restore.sh` (Bitwarden secrets)
  - `lint-dotfiles`
  - Exit

> 💡 This is the recommended entrypoint for a smooth restore experience:
> ```bash
> ~/bin/setup
> ```

### `~/bin/bootstrap/`
- `init-machine` – Installs Homebrew, sets up base folders
- `setup-core-tools`, `setup-kubernetes-tools`, `setup-terraform-tools` – Installs key CLI tools and symlinks runners

### `~/bin/dev-env/`
- `restore` – Core logic for restoring the dev environment (used by the TUI)
- `finalize` – Handles plugin setups (Oh My Zsh, Powerlevel10k, Tmux, Vim, kubectl Krew, etc.)
- `lint-dotfiles` – Validates shell scripts and formatting (can be run manually or via TUI)

### `~/bin/secrets/`
- `backup.sh` – Encrypts and uploads secrets to Bitwarden
- `restore.sh` – Downloads secrets and git identity
- `ensure-cron.sh` – Sets up a weekday cron job to auto-backup secrets at 9 AM

### `~/bin/runners/`
- Dockerized CLI wrappers (e.g., `kubectl-1.29`, `terraform-1.6`)
- Helpers under `~/bin/runners/helpers/` for launching version-specific tools

---

## 🔁 Restore Flow

### 🧭 One-Liner TUI Launcher (Recommended)

To launch the interactive setup menu:

```bash
~/bin/setup
```

You’ll get a clean menu UI powered by [`gum`](https://github.com/charmbracelet/gum) with options to:

```bash
Choose:
> 🛠  Run Full Restore
  🔐  Restore Secrets Only
  📤  Backup Secrets to Bitwarden
  🔁  Finalize Setup (plugins, completions)
  📦  Lint Dotfiles
  ❌  Exit

←↓↑→ navigate • enter submit
```

Install `gum` if not already:

```bash
brew install gum
```

---

### 🛠 Manual Restore (Advanced)

If you prefer running scripts directly (e.g., in CI or custom automation), use:

```bash
~/bin/dev-env/restore
```

This will:

1. Run `init-machine` to install Homebrew and core setup
2. Restore secrets (SSH, AWS, kube configs, git identity)
3. Finalize plugins: zsh, Oh My Zsh, Powerlevel10k, Vim, Tmux, kubectl krew, etc.

---

## 🔐 Secrets & Bitwarden Strategy

These paths are excluded from version control and backed up:

- `~/.ssh/`
- `~/.aws/`
- `~/.gnupg/`
- `~/.kube/`
- `~/.vpn-configs/`
- `~/.gitconfig-centerfield`
- `~/.saml2aws`
- `~/.mylogin.cnf`

Use:

```bash
~/bin/secrets/backup.sh   # to push updated secrets
~/bin/secrets/restore.sh  # to pull them from Bitwarden
```

> ✅ A cron job is configured to run `backup.sh` every weekday at 9 AM via `ensure-cron.sh`.  
> Confirm it’s installed: `crontab -l`

---

## 📦 Installed Tools & Plugin Ecosystem

This setup includes:

### 🐚 Shell
- `zsh` + `Oh My Zsh` + `Powerlevel10k`
- Plugins: `autojump`, `git`, `zsh-autosuggestions`, `zsh-syntax-highlighting`, `fzf`, `colorize`, `docker`, `aws`, `gcloud`, `krew`

### 📦 Package Management
- `Homebrew`, `brew bundle`, `coreutils`, `findutils`, `jq`, `thefuck`, `tldr`, `lsd`, `htop`, `asdf`, `direnv`

### 🛠️ Dev & Cloud Tools
- `kubectl`, `krew`, `k9s`, `terraform`, `awscli`, `session-manager-plugin`, `gcloud`, `tmux`, `vim`, `YouCompleteMe`, `goenv`, `pyenv`, `poetry`, `pipenv`, `bitwarden-cli`, `git-extras`, `gh`, `bb`

---

## 🖥️ Terminal Customization (Manual)

- Install Nerd Font:

  ```bash
  brew tap homebrew/cask-fonts
  brew install --cask font-hack-nerd-font
  ```

- In Terminal > Settings > Profiles:

  1. Import `gruvbox-dark.terminal`
  2. Set font: `Hack Nerd Font Mono`, style: `Regular`, size: `12`
  3. Spacing: `1`
  4. Window size: `125 x 200`
  5. Set profile as default

> Confirm mouse reporting works in Tmux/Vim.

---

## 🐳 Docker via Colima (Replaces Docker Desktop)

Docker is now powered by [`Colima`](https://github.com/abiosoft/colima) — a lightweight, fast, and scriptable alternative to Docker Desktop using native macOS virtualization.

### 🛠 Installation

Install Colima and the Docker CLI:

```bash
brew install colima docker
```

### 🚀 Start Colima VM

Start Colima with custom resource limits:

```bash
colima start --cpu 2 --memory 2 --disk 20
```

Optional: Start Colima with Kubernetes (via k3s):

```bash
colima start --cpu 2 --memory 4 --disk 20 --kubernetes
```

### 🐳 Usage

The Docker CLI (`docker`, `docker compose`) works out of the box:

```bash
docker ps
docker run hello-world
docker compose up
```

Colima VM management:

```bash
colima stop
colima restart
colima status
```

### 📦 Migration from Docker Desktop

If migrating from Docker Desktop:

- **Images**  
  Save from Docker Desktop:  
  ```bash
  docker save my-image:tag -o my-image.tar
  ```  
  Load into Colima:  
  ```bash
  docker load -i my-image.tar
  ```

- **Volumes**  
  Migration is manual; recreate or restore via backup.

- **Uninstall Docker Desktop**  
  Optional cleanup:  
  ```bash
  sudo rm -rf /Applications/Docker.app
  rm -rf ~/.docker ~/Library/Containers/com.docker.docker
  ```

### ✅ Notes

- Colima is scriptable and integrates well with dotfile-based setups.
- You can manage different environments using `colima start --profile <name> ...`.
- Works with your `~/bin/runners/`, aliases, and containerized workflows.

> 💡 This setup eliminates heavy resource usage from Docker Desktop and improves startup time and system performance.

---

## 📄 Tracked Dotfiles via `yadm`

All relevant files (non-sensitive) are version-controlled, including:

- `~/.bash_profile`, `~/.zshrc`, `.zshrc.local`, `.zshrc.plugins`
- `~/.vimrc`, `~/.tmux.conf`, `~/.p10k.zsh`
- `~/README.md`, `~/.gitignore_global`, `~/.editrc`, `~/.inputrc`

```bash
yadm status
yadm commit -am "Updated dotfiles"
yadm push
```

---

## 🧠 Philosophy

> Your dev environment should never be a mystery.  
> It should be **deterministic**, **modular**, **secure**, and **fast to restore**.

---

## 💬 Contributing

PRs welcome if you use a similar dotfile strategy and want to generalize any parts.

---

## 📄 License

MIT — use at your own risk, fork at your own will.
