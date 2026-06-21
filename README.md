# ansible-ghostty

[![CICD](https://github.com/jahrik/ansible-ghostty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-ghostty/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.ghostty-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/ghostty/)

Ansible role that installs [Ghostty](https://ghostty.org/) terminal emulator and deploys a configuration file. Supports Arch Linux, Debian/Ubuntu, macOS, and Steam Deck (SteamOS).

## Supported Platforms

| Platform | Install Method |
|---|---|
| Arch Linux | `pacman` |
| Ubuntu | `apt` via [mkasberg/ghostty-ubuntu PPA](https://launchpad.net/~mkasberg/+archive/ubuntu/ghostty-ubuntu) |
| Debian | `apt` via [debian.griffo.io](https://debian.griffo.io/) community repo |
| macOS | Homebrew Cask |
| Steam Deck (SteamOS) | Binary from `archive.archlinux.org` |

## Configuration

Deployed to `~/.config/ghostty/config`. Includes font settings, theming (Catppuccin with automatic light/dark switching), quick terminal (quake mode), split pane navigation with vim-style keybindings, and paste protection.

### Config Structure

```
~/.config/ghostty/
  config
```

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall and remove config |
| `ghostty.font.size` | `14` | Font size |
| `ghostty.font.family` | `JetBrainsMonoNerdFont` | Font family |
| `ghostty.font.thicken` | `true` | Thicken font rendering |
| `ghostty.theme` | `light:Catppuccin Latte,dark:Catppuccin Mocha` | Theme with light/dark auto-switching |
| `ghostty.background.opacity` | `0.9` | Background transparency |
| `ghostty.background.blur_radius` | `20` | Background blur |
| `ghostty.quick_terminal.position` | `top` | Quick terminal drop-down position |
| `ghostty.keybinds` | See `defaults/main.yml` | List of keybind strings |
| `ghostty_steamos_version` | `1.3.1` | Ghostty version pulled from `archive.archlinux.org` on Steam Deck |

## Testing

```bash
yamllint .
ansible-lint
molecule test
molecule test -s steamdeck
molecule converge -s localhost
molecule verify -s localhost
```

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: jahrik.ghostty
      vars:
        ghostty:
          font:
            size: 16
            family: DejaVuSansMono Nerd Font Mono
            thicken: true
          theme: "dark:Catppuccin Mocha"
```

## License

GPLv2

## Author Information

jahrik@gmail.com
