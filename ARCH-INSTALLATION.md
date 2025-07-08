# Arch Installation

## Part 1 - Prepare to install Arch

```sh
# load fonts
loadkeys pt-latin1
setfont ter-118b

# create 2 partitions
# 1º for efi with 512M
# 2º for btrfs with remaining spaces
fdisk /dev/nvme0n1

# formatting new partitions
mkfs.fat -F 32 /dev/nvme0n1p5
mkfs.btrfs /dev/nvme0n1p6
mount /dev/nvme0n1p6 /mnt

# Create BTRFS subvolume and mount it
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
umount /mnt
mount -o compress=zstd,subvol=@ /dev/nvme0n1p6 /mnt
mkdir -p /mnt/home
mount -o compress=zstd,subvol=@home /dev/nvme0n1p6 /mnt/home

# Create EFI folder and mount it
mkdir -p /mnt/efi
mount /dev/nvme0n1p5 /mnt/efi

# Update Packages Sources
reflector --country 'FR,PT,GB' -p "https" --ipv4 -a 24 --sort rate --latest 10 --save /etc/pacman.d/mirrorlist

# Install most critical softwares
pacstrap -K /mnt \
 base base-devel linux linux-firmware amd-ucode git btrfs-progs grub \
 efibootmgr grub-btrfs inotify-tools timeshift neovim networkmanager \
 pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber \
 reflector openssh man sudo os-prober fuse3 sbctl

# Generate init filesystem table and go into by
# chroot
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

## Part 2 - Configure arch

```sh
# Timezone
ln -sf /usr/share/zoneinfo/Europe/Lisbon /etc/localtime
hwclock --systohc

# Language
sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen
locale-gen
cat > /etc/locale.conf <<EOF
LANG=en_US.UTF-8
EOF

# Default console keymap
cat > /etc/vconsole.conf <<EOF
KEYMAP=pt-latin1
EOF

# Set hostname
cat > /etc/hostname <<EOF
ARCH-LAPTOP
EOF

# Set local dns
cat > /etc/hosts <<EOF
127.0.0.1 localhost
::1 localhost
127.0.1.1 ARCH-LAPTOP
127.0.1.1 archlaptop
EOF

# Setup user
# setup root pass
passwd
# set up personal user
useradd -m -s /bin/bash -c "User" user
passwd user
# grant full perm
cat > /etc/sudoers.d/50-user <<EOF
user ALL=(ALL) NOPASSWD:ALL
EOF

# Set up default editor
echo "EDITOR=nvim" >> /etc/environment

# Config bootloader
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --modules="tpm" --disable-shim-lock
grub-mkconfig -o /boot/grub/grub.cfg
```

## Part 3 - Enabled Secure Boot

Note:
- Its required that before this, enable setup mode

```sh
# enable ntp
timedatectl set-ntp true

# run as root
sudo -i
# create keys
sbctl create-keys
sbctl enroll-keys -m
# sign files
sbctl verify | sed -E 's|^.* (/.+) is not signed$|sbctl sign -s "\1"|e'
sbctl sign -s /boot/vmlinuz-linux

# enable windows os detection
nvim /etc/default/grub
..
GRUB_DISABLE_OS_PROBER=false
..
# update config
sudo os-prober
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

## Part 4 - Install Yay

```sh
sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si

# install snapshot manager
yay -S timeshift-autosnap

# enable snapshot boot entries
sudo systemctl edit --full grub-btrfsd
-- 
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
--
```