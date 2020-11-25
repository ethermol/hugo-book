---
title: Arch
weight: 1
---
# Arch workstation
Arch is rolling so arch is current. Arch can be setup and used as a minimalist distribution as such lowering the attack surface. The arch [archwiki](https://wiki.archlinux.org/) is top notch. The initial part of the installation is described at: [prepare_installation_medium](https://wiki.archlinux.org/index.php/installation_guide#Prepare_an_installation_medium).

## Prepare
Plan the desired layout of the disk. In this case: (HOW TO FIXed font ??)
```
+---------------+----------------+----------------+----------------+
|ESP partition: |Boot partition: |Volume 1:       |Volume 2:       |
|               |                |                |                |
|/boot/efi      |/boot           |root            |swap            |
|               |                |                |                |
|               |                |/dev/vg0/root   |/dev/vg0/swap   |
|/dev/sda1      |/dev/sda2       +----------------+----------------+
|unencrypted    |LUKS encrypted  |/dev/sda3 encrypted LVM on LUKS  |
+---------------+----------------+---------------------------------+
```
P.S. while deploying only two of the partitions were created within the system partition (sdb). Only the 512M sized boot partition is left unencrypted. The laptop contains 2 drives (512GB and 2TB). The 512GB disk will be used as the system drive, the 2 TB drive will be set as the data-drive.
```
NAME            MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda               8:0    0   1.8T  0 disk
└─sda1            8:1    0   1.8T  0 part
  └─user        254:4    0   1.8T  0 crypt
    └─vg1-store 254:5    0   1.8T  0 lvm   /store
sdb               8:16   0 476.9G  0 disk
├─sdb1            8:17   0   512M  0 part  /boot
└─sdb2            8:18   0 476.4G  0 part
  └─system      254:0    0 476.4G  0 crypt
    ├─vg0-swap  254:1    0    16G  0 lvm   [SWAP]
    ├─vg0-root  254:2    0   128G  0 lvm   /
    └─vg0-home  254:3    0 332.4G  0 lvm   /home
```
    
## Boot from USB-stick
- prepare medium
- aquire installation image
```bash
curl http://ftp.nluug.nl/os/Linux/distr/archlinux/iso/2020.10.01/archlinux-2020.10.01-x86_64.iso -o archlinux-2020.10.01-x86_64.iso
curl http://ftp.nluug.nl/os/Linux/distr/archlinux/iso/2020.10.01/archlinux-2020.10.01-x86_64.iso.sig -o archlinux-2020.10.01-x86_64.iso.sig
pacman-key -v archlinux-version-x86_64.iso.sig
sudo dd bs=16M if=path /to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```
- boot live
- set keyboard layout
- verify boot mode
- connect
- update clock
```bash
systemctl start gpm
iwctl
>station wlan0 connect mosGate
>> <<ssid credentials>>
# pacman -S terminus-font
setfont ter-v24b
```
## Prepare Disk for crypted install
```bash
gdisk /dev/sda
# 0,n +512MB,ef00,n,<defaults>
# ls /usr/share/kbd/keymaps/**/*.map.gz
# loadkeys us-latin1 # Default!
ls /sys/firmware/efi/efivars # should be efi
ip link
rfkill list / # rfkill unblock wifi
# wifi-menu
ping google.com
timedatectl set-ntp true
timedatectl status
```
- partition and format, desired layout: sdb1 = 512MB efi, sdb2 <rest> 
```bash
   # parted/fdisk/cfdisk
gdisk /dev/sdb
   # select o to cleanup
   # create 2 new partitions:
   # 1. type EF00 size 512M
   # 2. type 8E00 size <remaining>
 
   # # Use "o" to create GPT table
   # # "n" to create partitions:
   # # "w" to write changes
   # # "q" to quit
   #
   # # Make filesystem for EFI
mkfs.fat -F32 /dev/sda1
   #
   # # Create crypted /boot container <SKIPPED>
   # cryptsetup -y -v luksFormat /dev/sdb2
   # cryptsetup open /dev/sdb2 system
   # mkfs.ext2 /dev/mapper/system
   #
   # # Create crypted LVM with /root and swap
cryptsetup luksFormat /dev/sdb2
cryptsetup open /dev/sdb2 system
pvcreate /dev/mapper/system #<?? - created by luksOpen>
vgcreate vg0 /dev/mapper/system
lvcreate -L 16G vg0 -n swap
lvcreate -L 128G vg0 -n root
lvcreate -l 100%FREE vg0 -n home
mkfs.ext4 /dev/mapper/vg0-root
mkswap /dev/mapper/vg0-swap
swapon /dev/mapper/vg0-swap
   # # Mount
mount /dev/mapper/vg0-root /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
   #
   # # Check
   # lsblk
   # # Should look something like sample above.
```
## Perform Minimal Arch install
- select mirrors ==> /etc/pacman.d/mirrorlist
- select packages to install
```bash
pacman -S reflector
pacstrap /mnt base base-devel linux linux-firmware lvm2 neovim git efibootmgr iwd
```
## Configure:
- fstab
- chroot
- time zone
- localisation
- network
- initramfs
- root passwd
- boot loader
```bash
   # # Generate fstab
genfstab -pU /mnt >> /mnt/etc/fstab
   #
   # # Chroot into our newly installed system
arch-chroot /mnt
   #
   # # Set timezone, hostname...
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime
hwclock --systohc --utc
echo saga > /etc/hostname
   #
   # # Configure locales
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 >> /etc/locale.conf
   #
   # # Set root password
passwd
   #
   # # Open this file
vim /etc/mkinitcpio.conf
   # # and replace HOOKS="..." with HOOKS=(base udev autodetect modconf block keymap encrypt lvm2 resume filesystems keyboard fsck)
   # # use "i" key to edit (insert something), ESC and ":wq" to write changes and quit
   #
   # # Regenerate initrd image
mkinitcpio -p linux
   #
   # # If you got warnings about missing firmware for wd719x and aic94xx, you can ignore it, with high probability you don't even have this hardware
   # # But you can install it from AUR if you actually use it
   #
bootctl install
cd /boot/loader
   # vim /etc/mkinitcpio.conf
   # vim loader.conf
cd entries
vim arch.conf
   # title Arch Linux
   # linux /vmlinuz-linux
   # initrd /intel-ucode.img
   # initrd /initramfs-linux.img
   # options cryptdevice=UUID="32577506-8619-4061-a790-fbb89a2aaa4c":system root=/dev/mapper/vg0-root quiet rw

   # mkinitcpio -p linux
   #
   # # If you got errors "/run/lvm/lvmetad.socket: connect failed: No such file or directory", that's OK
   # # you can get rid of this errors with some workarounds, but this is not really necessary
   # # but in any case DO NOT disable lvmetad! This installation will not work without it
   #
   # # Need some additional stuff to get wifi working, intel micro-code update, console mouse support and man pages 
pacman -S wireless_tools dhcpcd gpm intel-ucode man xdg-utils
systemctl enable iwd dhcpcd gpm
   #
   # # Exit from chroot, unmount system, shutdown, extract flash stick. You made it! Now you have fully encrypted system.
exit
umount -R /mnt
swapoff -a
shutdown now
   #
```
### Reboot
### Post Install
```bash
   # setup networking
   # systemctl enable iwd
   # systemctl start iwd
iwctl station wlan0 connect elft
pacman -S git openssh terminus-font tmux htop
   # initial <promtp> for passwd
   # # Some additional security
   # chmod 700 /boot
   # chmod 700 /etc/iptables
   #
   # # Create non-root user, set password
useradd -m -g users -G wheel YOUR_USER_NAME
passwd YOUR_USER_NAME
   #
   # # Open file
vim /etc/sudoers
   # # and uncomment string %wheel ALL=(ALL) ALL
   # # For additional security, start PC, login in UEFI menu during boot (in most cases by pressing F2 or DEL)
   # # enable Secure boot option, and choose our EFI image as trusted. Path will be something like this:
   # # HDD0 -> EFI -> ArchLinux -> grubx64.efi
   #
   # # Of course, you must protect UEFI menu with password.
   # # Choose DIFFERENT password from that you used for encryption, because some lazy manufacturers store this password not securely enough.
   #
   # # Reboot again, login as user, use sudo for installation all other software you want: drivers, display server, desktop environment, etc...
```   
## Add another crypted volume [optional]
```bash
   #### --> adding a encrypted data disk.
   # # Going to encrypt sda1
   # shred -v -n 1 /dev/sda1 # Skipped this . . .
fdisk /dev/sda
   # # Answer g (gpt) and w (write)
cryptsetup -y -v luksFormat /dev/sda1
cryptsetup luksOpen /dev/sda1 user
pvcreate /dev/mapper/user #<?? - created by luksOpen>
vgcreate vg1 /dev/mapper/user
lvcreate -l 100%FREE vg1 -n store
mkfs.ext4 /dev/vg1/store
mount /dev/vg1/store /store
dd if=/dev/urandom of=/root/.keyfile bs=1024 count=4
chmod 0400 /root/.keyfile
cryptsetup luksAddKey /dev/sda1 /root/.keyfile
   # # --> for /etc/crypttab -->
blkid /dev/sda1
   # user       UUID="42376139-a479-4143-88f0-aa12962da458" /root/.keyfile luks,discard
vi /etc/fstab:
   # /dev/mapper/vg1-store    /store  ext4    defaults        0       2
```   

## Install tiling windows manager [spectrwm]
Stil have some minor issues with this wm, fi apps initially start on wrong WorkSpace.
```bash
pacman -S xf86-video-intel xf86-input-synaptics xorg xorg-init nitrogen picom lxappearance alacritty firefox spectrwm i3lock-color
cp /etc/X11/xinit/xinitrc ${HOME}/.xinitrc
cp /etc/spectrwm.conf ~/.config/spectrwm/spectrwm.conf

chmod u+s /usr/bin/xinit  # --> to prevent FAILED TO SET IOPL FOR IO

pacman -S pulseaudio pulseaudio-jack pulseaudio-bluetooth pavucontrol mpv bluez bluez-utils cups audacity entr eslint ethtool fzf httpie neofetch ncdu nmap putty socat tcpdump tig tldr ueberzug ufw vifm vpnc xarchiver yamllint zsh zsh-autosuggestions zsh-syntax-highlighting dunst scrot meld arandr materia-gtk-theme papirus-icon-theme acpi qt5ct keepassxc base-devel pidgin newsboat zathura zathura-ps zathura-pdf-poppler zathura-djvu xdg-utils xdg-user-dirs youtube-dl ffmpegthumbnailer sxiv p7zip tldr xclip openvpn btrfs-progs tldr cmus qutebrowser 

sudo pacman -S fuse fuseiso s3fs-fuse sshfs fuse-zip squashfuse rclone terragrunt terraform docker qemu virt-manager ebtables aws-cli flameshot ascii python-fonttools qjackctl gimp trayer redshift pass msmtp isync neomutt abook lynx yq ttf-joypixels  screenfetch

# Install yay:
cdr; cd arch
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

yay -S nerd-fonts-mononoki ttf-mononoki devour slack-libpurple-git-r223.2e9fa02-1 task-spooler

trayer issues: one workspace only, boxed (ugly, waste of space), not very useful.
```
## Configure stuff
## Manage dotfiles
Dotfiles store the configuration for the installed applications and can b found in the .config directory. To be able to rebuild the system quickly after disaster strikes the .config directory plis the .zenv directory will be replicated to github as a "bare repository". Password and secrets are stored in .ssh and within the .local/share directories which should be handled/saved separately. Care should be taken however, for instance .config/aws/credentials should NOT be replicated !! In stead of generically replicating .config it might be a good idea to specifically assign the dotfiles to be replicated.
