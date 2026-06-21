# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Role Overview

Ansible role that installs [Ghostty](https://ghostty.org/) and deploys a config file. Supports Arch Linux, Debian/Ubuntu, macOS, and Steam Deck (SteamOS).

## Key Variables (`defaults/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall and remove config |
| `ghostty.font.size` | `14` | Font size |
| `ghostty.font.family` | `JetBrainsMonoNerdFont` | Font family |
| `ghostty.font.thicken` | `true` | Thicken font rendering |
| `ghostty.theme` | `light:Catppuccin Latte,dark:Catppuccin Mocha` | Theme string |
| `ghostty.background.opacity` | `0.9` | Background transparency |
| `ghostty.background.blur_radius` | `20` | Background blur |
| `ghostty.keybinds` | List of keybind strings | Keybinding configuration |
| `ghostty_steamos_version` | `1.3.1` | Ghostty version pulled from `archive.archlinux.org` on Steam Deck |

## Task Flow

`tasks/main.yml` -> `install.yml` or `uninstall.yml` based on `install | bool`

**install.yml:**
1. Stat `/etc/steamos-release`
2. If it exists, include `steamos.yml` (SteamOS code path, takes priority over generic Arch path)
3. Else include `debian.yml` (Debian family), `archlinux.yml` (Arch family, non-SteamOS), or `darwin.yml` (Darwin family)
4. Create `~/.config/ghostty/` directory
5. Template `~/.config/ghostty/config` from `templates/config.j2`

**archlinux.yml:** `pacman` install of `ghostty`

**debian.yml:** Installs prerequisites, then dispatches to `ubuntu_ppa.yml` (Ubuntu) or `debian_repo.yml` (Debian) to add the appropriate third-party repository, then installs `ghostty` via `apt`

**ubuntu_ppa.yml:** Downloads the PPA signing key from `keyserver.ubuntu.com`, dearmors it, and adds the `ppa:mkasberg/ghostty-ubuntu` repository via deb822 format

**debian_repo.yml:** Downloads the signing key from `debian.griffo.io`, dearmors it, and adds the community apt repository via deb822 format

**darwin.yml:** `community.general.homebrew_cask` install of the `ghostty` cask, `become: false`

**steamos.yml:** No `become` anywhere (SteamOS root is read-only).
- Downloads `ghostty-{{ ghostty_steamos_version }}-1-x86_64.pkg.tar.zst` from `archive.archlinux.org`
- Extracts just `usr/bin/ghostty` into `~/.local/bin/ghostty` via `unarchive` (`remote_src: true`, `--strip-components=2`)
- Deploys `files/ghostty.png` to `~/.local/share/icons/` and `templates/ghostty.desktop.j2` to `~/.local/share/applications/`, then runs `kbuildsycoca6` (if present) to refresh KDE's launcher cache

**uninstall.yml:** removes the SteamOS binary/desktop entry/icon (when `/etc/steamos-release` exists), uninstalls the Homebrew cask (Darwin), or removes the package via `ansible.builtin.package` (everywhere else), then always removes `~/.config/ghostty`

## Config Structure

```
~/.config/ghostty/
  config          <- templates/config.j2
```

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
molecule test -s steamdeck
molecule converge
molecule destroy
```

Localhost scenario (runs directly against the local machine):

```bash
molecule converge -s localhost
molecule verify -s localhost
```

## CI

- **Lint**: yamllint + ansible-lint
- **Molecule**: Ubuntu + Arch Linux via Docker (`molecule/default`)
- **Steam Deck**: Arch container with `/etc/steamos-release` stubbed (`molecule/steamdeck`)
- **macOS**: `ansible-playbook` against the GHA runner directly (`macos-latest`), using `molecule/localhost`
- **Release**: publishes to Ansible Galaxy on merge to `main`
