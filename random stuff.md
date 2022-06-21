##### pacstrap

```bash
pacstrap /mnt linux linux-headers linux-firmware intel-ucode base base-devel git nano efibootmgr libinput xf86-input-libinput mesa lib32-mesa vulkan-intel networkmanager pipewire lib32-pipewire pipewire-pulse wireplumber pipewire-alsa reflector pacman-contrib dialog
```

idk if `sof-firmware` is needed

^^ Make sure to enable `multilib` in iso first

##### Maybe mkinitcpio stuff?
* `MODULES=(nvme)` for nvme stuff?

##### Grub

```bash
pacman -S grub efibootmgr dosfstools
nano /etc/default/grub
```
for `GRUB_CMDLINE_LINUX`, make it equal to `cryptdevice=UUID={uuid}:cryptroot root=/dev/mapper/cryptroot`
Use `blkid` and get the root partition uuid

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

##### Fun stuffs
```bash
sudo pacman -S neofetch lolcat cmatrix
yay -S pipes.sh
```

##### Plasma

```bash
sudo pacman -S plasma konsole dolphin kwrite krunner firefox sddm fwupd packagekit-qt5 spectacle partitionmanager
```

##### yay
```
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

##### ClamAV (Virus scanner)
```bash
sudo pacman -S clamav
sudo freshclam
```

##### `systemctl`

```bash
sudo systemctl enable NetworkManager.service
sudo systemctl enable sddm.service
sudo systemctl enable clamav-freshclam.service
sudo systemctl enable clamav-daemon.service
sudo systemctl enable bluetooth.service # sudo pacman -S bluez bluez-utils if not done already
```

##### Install and enable zsh

```bash
sudo pacman -S zsh zsh-completions
chsh -s /usr/bin/zsh
# reboot after

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

yay -S nerd-fonts-fira-code # Set this as the monospace font in System Settings

yay -S zsh-theme-powerlevel10k-git
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >> ~/.zshrc

# p10k configure if not automatically done after install
```
Add configs and stuffs - `nano .zshrc`
```bash
# Add this at the end
if [[ "$XDG_SESSION_TYPE" == "wayland" ]]; then
	export MOZ_ENABLE_WAYLAND=1
	export MOZ_DBUS_REMOTE=1
fi

# alias neofetch='neofetch | lolcat' # Enabling this means neofetch --help returns lolcat help

```

##### paccache

`sudo paccache -rk` - manual clean

```bash
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
```
add the following
```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk
```

##### Firewall

```bash
sudo pacman -S ufw
sudo ufw enable
```

##### KDE Plasma Settings

Application/Plasma Theme:
* Set Global Theme to Plasma Dark
* https://github.com/daltonmenezes/aura-theme/tree/main/packages/plasma <-- Then choose aura theme obviously
* Go to Application Style and choose `Configure GNOME/GTK Application Style` and choose breeze (it will include the aura colors)
* Workspace Behaviour/Desktop Effects - Translucency & Wobbly Windows. Dim Screen for admin mode and slide back
Konsole theme
* Ctrl-Shift-m to hide/show menu bar
* Right click and hide Main Toolbar/Session toolbar. menu bar + right click to reenable them
* Settings > Window Color Scheme should be set to default (which should make the title bar in Aura theme)
* Settings > New Profile
  * Change name and set as default profile
  * Appearance > Color scheme and set it to Aura but click on edit and make the transparency 20%
Papirus Icons
* `sudo pacman -S papirus-icon-theme ` and go to System Settings > Appearance > Icons and select papirus-dark
* Set the application launcher to the arch icon and set "Show buttons for Power and Session"





