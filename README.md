Notes for my new install guide:
* https://archlinux.org/packages/community/any/wqy-microhei/

For old install guide, go to https://github.com/EpicGamer007/Arch-Install-Guide/blob/main/KDE%20Plasma%2BWindows.md

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

# If I want to mount my windows partition
mkdir /mnt/windows
moutn /dev/nvme0n1p3 /mnt/windows

pacstrap /mnt linux linux-headers linux-firmware base base-devel git nano mesa intel-media-driver libinput pipewire wireplumber sof-firmware pacman-contrib reflector networkmanager curl

genfstab genfstab -U /mnt >> /mnt/etc/fstab
```

### `arch-chroot /mnt` To enter the system

### Swapfile
```bash
fallocate -l 4GB /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# For windows
pacman -S ntfs-3g

nano /etc/fstab

# ADD:
/swapfile none swap sw 0 0
# Replace the line which mounts windows with "ntfs" with "ntfs-3g"
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

# IT WONT BE LIKE THIS IN THE NEW UPDATE - Just add encrypt before filsystems

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

### Chaotic aur
```bash
pacman-key --recv-key FBA220DFC880C036 --keyserver keyserver.ubuntu.com
pacman-key --lsign-key FBA220DFC880C036
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst' 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'

# Add the following to the end of /etc/pacman.conf
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

### Web Browser
```bash
yay -S chaotic-aur/firedragon

# Create a shell script (like `~/browserstartup.sh`) and add this script in the kde plasma settings to Startup and Shutdown in the Autostart section

#!/bin/bash
if [[ "$XDG_SESSION_TYPE" == "wayland" ]]; then
	export MOZ_ENABLE_WAYLAND=1
	export MOZ_DBUS_REMOTE=1
fi

# Make sure to do chmod +x to this file
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

### fstrim (For SSDs)
```bash
sudo systemctl enable fstrim.timer
```

### Burn-My-Windows KWin effect: https://github.com/Schneegans/Burn-My-Windows
```
Go to Discover and search Burn-My-Windows or one of the supported effects in the above repo
Go to Settings then go to the settings section for KWin and add your desired effect (Incinerate is the coolest)
```

### Webcord (Discord client with support for wayland screen sharing): `yay -S webcord-bin` (or `-git` or just `webcord`) See https://github.com/SpacingBat3/WebCord

### Run `yay` to update everything

## Auto Clean Cache: https://herbort.me/posts/automatically-cleaning-pacman-and-yay-cache-in-arch-linux/

## xdg-desktop
```bash
sudo pacman -S xdg-desktop-portal xdg-desktop-portal-kde
```

### Dns-over-https
[WIP]: [https://wiki.archlinux.org/title/Dnscrypt-proxy](https://wiki.archlinux.org/title/Dnscrypt-proxy) + [https://www.privacyguides.org/dns/](https://www.privacyguides.org/dns/#recommended-providers)

### one.one.one.one DNS over WARP
```bash
yay -S cloudflare-warp-bin
warp-cli # register, accept tos, enable
```

