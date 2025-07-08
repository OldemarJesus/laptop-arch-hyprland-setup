# Hyprland Installation

## Critical Packages

```sh
yay -S hyprland kitty sddm swaync xdg-desktop-portal-hyprland qt6-wayland \
 qt5-wayland hyprpolkitagent hyprpaper hypridle hyprlock hyprcursor \
 hyprutils hyprland-qtutils waybar xorg-xwayland rofi copyq clipse \
 nemo stow
```

## Enable Display Manager

```sh
sudo systemctl enable sddm
reboot
```