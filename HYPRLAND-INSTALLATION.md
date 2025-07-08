# Hyprland Installation

## Critical Packages

```sh
yay -S hyprland kitty sddm swaync xdg-desktop-portal-hyprland qt6-wayland \
 qt5-wayland hyprpolkitagent hyprpaper hypridle hyprlock hyprcursor \
 hyprutils hyprland-qtutils waybar xorg-xwayland rofi copyq clipse \
 nemo stow zsh brave-bin gnome-keyring
```

## Enable Display Manager

```sh
sudo systemctl enable sddm
reboot
```

## Default terminal

```sh
chsh -s /bin/zsh
# install theme
curl -sS https://starship.rs/install.sh | sh
echo '# Starshiprs\neval "$(starship init zsh)"' >> .zshrc
cat >> ~/.zshrc <<EOF
# key bindings
bindkey  "^[[H"   beginning-of-line
bindkey  "^[[F"   end-of-line
bindkey  "^[[3~"  delete-char
EOF
```
