I have decided against having multilib support because 32bit functionality is not highly needed unless i plan on using wine.

Minimal 
```bash

pacstrap /mnt \
linux linux-headers linux-firmware \
base base-devel git nano \
mesa intel-media-driver libinput \
pipewire wireplumber \
pacman-contrib reflector \
networkmanager
```

Install grub later
```bash
pacman -S grub efibootmgr os-prober
# Run os-prober to test

# MAKE SURE TO UNCOMMENT the os-prober line in /etc/default/grub

# GRUB_CMDLINE_LINUX="cryptdevice=UUID={ROOT_UUID_FROM_blkid}:cryptroot root=/dev/mapper/cryptroot"

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```
