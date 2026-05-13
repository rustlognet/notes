## Summary

Sway installation on ArchLinux.

## Update the system

```bash
sudo pacman -Syu
```
## Install Sway and basic tools

```bash
sudo pacman -S sway swaybg swaylock swayidle waybar foot wofi xdg-desktop-portal-wlr
```

- `sway` - the compositor/window manager
- `swaybg` - wallpaper utility
- `swaylock` - screen locker
- `swayidle` - idle management
- `waybar` - top bar/panel
- `foot` - lightweight Wayland terminal
- `wofi`- app launcher
- `xdg-desktop-portal-wlr` - screen sharing / portals

## Install graphics + audio utilities

Graphics:

```bash
sudo pacman -S mesa vulkan-intel
```

Audio:

```bash
sudo pacman -S pipewire wireplumber pipewire-pulse pavucontrol
```
Enable audio session:

```bahs
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

## Install a login manager (optional)

```bash
sudo pacman -S greetd tuigreet
```

Create config:

```bash
sudo mkdir -p /etc/greetd
sudo nano /etc/greetd/config.toml
```

```text
[terminal]
vt = 1

[default_session]
command = "tuigreet --cmd sway"
user = "greeter"
```

Enable it:

```bash
sudo systemctl enable greetd
```

```bash
reboot
```


## No login manager - start manually

Login.

```bash
sway
```

You can automate this in `.bash_profile`:

```bash
nano ~/.bash_profile
```

```bash
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
  exec sway
fi
```

## First launch - basics

Useful shortcuts:

`Super + Enter` - open terminal
`Super + D` - app launcher
`Super + Shift + E` - exit sway
`Super + Shift + Q` - close window
`Super + 1..9` - switch workspace

## Configuration

Copy default config:

```bash
mkdir -p ~/.config/sway
cp /etc/sway/config ~/.config/sway/config
```

Then edit:

```bash
nano ~/.config/sway/config
```

## Nice extra packages

```bash
sudo pacman -S firefox thunar grim slurp wl-clipboard
```

- `thunar` - file manager
- `grim + slurp` - screenshots
- `wl-clipboard` - clipboard tools