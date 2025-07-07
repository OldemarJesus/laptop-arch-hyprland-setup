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
```