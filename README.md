# Arch Linux Install Guide

This is an install guide for my laptop. If you are going to try this guide, make sure to check your hardware, and make any changes necesary.
**READ THE COMMENTS IN THE CODE BLOCKS**. The contain important information.
This guide is seperated into two sections, `Inital Install`, and `Post Install`.
This guide has the following:
* Dual boot with Windows (tested on 10, should work with 11)
* UEFI Mode
* Remote ssh for install if required
* 2 partitions - Boot, Encrypted Root (LUKS)
* Optional Swapfile
* Grub Boot Manager (Check [the Old Guide](https://github.com/EpicGamer007/Arch-Install-Guide/blob/main/Old%20Guide.md#boot-manager-and-networkmanager) for refind boot manager)
* KDE Plasma and web browsers
* ClamAV (Antivirus)
* ZSH + ohmyzsh + powerlevel10k
* And More

## Initial Install

### Windows setup
* Disable Secure Boot
* Go to `regedit.msc` and go to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`. Add a new QWORD called `RealTimeIsUniversal` and give it a value of `1`. You do not have to touch the Settings App for time and date
* [Disable Fast Startup](https://www.lifewire.com/disable-fast-startup-in-windows-10-5094422)
* [Disable Hibernation](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/disable-and-re-enable-hibernation) (`powercfg.exe /hibernate` as admin in Command Prompt)
* Decrease size of C: Drive in `Create and Format Hard Disk Partitions` to the amount of empty space you want for your arch installation

### Other Windows Things
* Refer to [this ArchWiki guide for bluetooth after installation](https://wiki.archlinux.org/title/Bluetooth#Dual_boot_pairing)
* If you would like Windows Hard Disk Encryption refer to [this guide](https://octetz.com/docs/2020/2020-2-16-arch-windows-install/) which will help setup Veracrypt and Grub
* Intel Rapid Storage Technology (IRST) can be disabled from Intel Optane Memory and Storage Management App (if it cannot be disabled in bios). if IRST was included when the machine was purchased, the app should be preinstalled as well. You should have the option to disable it.

### Connect to internet, network time synchronization, setup ssh
```bash
iwctl
[iwd]# device list
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station device connect Network_Name
ping archlinux.org

timedatectl set-ntp true

passwd
systemctl start sshd
ip a # Use local ip (192.168.[num].[num])
# From other computer: ssh root@ip
```

### Partition disk, encrypt root, format boot and cryptroot
```bash
lsblk # fdisk -l
cgdisk /dev/nvme0n1
# Go to free space sector and select [ New ]
#   * Default first sector
#   * 512MiB Size
#   * Default partition type (linux filesystem)
#   * Name of "boot"
# Remaining space, select [ New ]
#   * Default options for 1 through 3
#   * Name of "root"
# Click on [ Write ] and type yes to confirm
lsblk # fdisk -l Note the EFI (likely 1), boot (likely 5), and root (likely 6)

cryptsetup -y --use-random luksFormat /dev/nvme0n1p6 # (Root partition)
cryptsetup luksOpen /dev/nvme0n1p6 cryptroot

mkfs.ext4 /dev/nvme0n1p5 # (Boot Partition)
mkfs.ext4 /dev/mapper/cryptroot
```

### Mount partitions, pacstrap, fstab
```bash
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p5 /mnt/boot # Mount boot partition
mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi # Mount EFI partition

pacstrap /mnt linux linux-headers linux-firmware base base-devel git nano mesa intel-media-driver libinput pipewire wireplumber pacman-contrib reflector networkmanager curl

genfstab genfstab -U /mnt >> /mnt/etc/fstab
```

### `arch-chroot /mnt` To enter the system

### Swapfile
```bash
fallocate -l 4GB /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

nano /etc/fstab
# ADD:
/swapfile none swap sw 0 0
```

### Initial setup
```bash
ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
hwclock --systohc
nano /etc/locale.gen # Uncomment en_US.UTF-8 UTF-8 
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
echo "Installation00" >> /etc/hostname
nano /etc/hosts
# 127.0.0.1    localhost
# ::1          localhost
# 127.0.1.1    Installation00.localdomain Installation00
```

### Ramdisk
```bash
nano /etc/mkinitcpio.conf
# Make the line with HOOKS as below
HOOKS=(base udev autodetect keyboard modconf block encrypt filesystems fsck)
# Save and exit nano
mkinitcpio -p linux
```

### Initial user administration
```bash
passwd # Set root password
useradd -m -G wheel abhay
passwd abhay
EDITOR=nano visudo
# Uncomment %wheel ALL=(ALL:ALL) ALL
```

### `systemctl enable NetworkManager.service` Enable NetworkManager

### Grub
```bash
pacman -S grub efibootmgr

nano /etc/default/grub
# Find the below line in the file and write as follows
# GRUB_CMDLINE_LINUX="cryptdevice=UUID={ROOT_UUID_FROM_blkid}:cryptroot root=/dev/mapper/cryptroot"

nano /etc/grub.d/40_custom
# Make the file as follows below (Not including the # lines at the top and bottom)
# Get your efi partition's uuid from blkid and replace $EFI_PARTITION_UUID. It is typically in the form of XXXX-XXXX
##########################
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
if [ "${grub_platform}" == "efi" ]; then
	menuentry "Windows 10" {
		insmod part_gpt
		insmod fat
		insmod search_fs_uuid
		insmod chain
		
		search --fs-uuid --set=root $EFI_PARTITION_UUID
		chainloader /EFI/Microsoft/Boot/bootmgfw.efi
	}
fi
##########################

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```

### Exit and reboot
```bash
exit
umount -R /mnt
reboot
```

## Post Install

### Connect to internet with NetworkManager
```bash
nmcli device wifi list
nmcli device wifi connect <SSID> password <password>
nmcli connection show # connection info
```

### Install yay
```bash
git clone https://aur.archlinux.org/yay-bin.git # Change yay-bin to yay if you want to compile yourself
cd yay-bin
makepkg -si
```

### Fun Stuff
```bash
sudo pacman -S neofetch cmatrix btop
yay -S pipes.sh
```

### KDE Plasma
```bash
sudo pacman -S plasma plasma-wayland-session konsole dolphin ark kwrite kcalc krunner spectacle partitionmanager gwenview sddm
sudo pacman -S fwupd packagekit-qt5
sudo systemctl enable sddm.service
```

### Web Browsers
```bash
yay -S firefox-kde-opensuse brave-bin # Use https://ffprofile.com/ to harden firefox as needed
# Don't forget to isntall kde integration for brave as well
```

### Bluetooth
```bash
sudo pacman -S bluez bluez-utils bluedevil
sudo systemctl enable bluetooth.service
```

### ClamAntiVirus
```bash
sudo pacman -S clamav
sudo freshclam
sudo systemctl enable clamav-freshclam.service
sudo systemctl enable clamav-daemon.service
```

### zsh
```bash
sudo pacman -S zsh zsh-completions
chsh -s /usr/bin/zsh
# sudo reboot

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" # Install ohmyzsh

yay -S nerd-fonts-fira-code # Set this as the monospace font in System Settings

yay -S zsh-theme-powerlevel10k-git
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >> ~/.zshrc

# p10k configure (should be done automatically)
```

### Config for browsers
```bash
# .zshrc
if [[ "$XDG_SESSION_TYPE" == "wayland" ]]; then
	export MOZ_ENABLE_WAYLAND=1
	export MOZ_DBUS_REMOTE=1
fi
# .config/chromium-flags.conf
--ozone-platform-hint=auto # Or chrome://flags and edit it there
```

### Pacman hook for paccache (pacman-contrib)
```bash
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
# Add following
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -r
```

### UFW (Firewall)
```bash
sudo pacman -S ufw

sudo ufw default allow outgoing
sudo ufw default deny incoming

sudo ufw enable
```

## fstrim (For SSDs)
```bash
sudo systemctl enable fstrim.timer
```

### Dns-over-https
[WIP]: [https://wiki.archlinux.org/title/Dnscrypt-proxy](https://wiki.archlinux.org/title/Dnscrypt-proxy) + [https://www.privacyguides.org/dns/](https://www.privacyguides.org/dns/#recommended-providers)
