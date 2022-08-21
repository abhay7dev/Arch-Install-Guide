I have decided against having multilib support because 32bit functionality is not highly needed unless i plan on using wine. I also decided to use grub because grub has better looking themes lmao

### Minimal pacstrap
```bash
pacstrap /mnt \
linux linux-headers linux-firmware \
base base-devel git nano \
mesa intel-media-driver libinput \
pipewire wireplumber \
pacman-contrib reflector \
networkmanager
```

### Swapfile
```bash
# AFTER MAKING /etc/fstab
# (in arch-chroot)
fallocate -l 4GB /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

nano /etc/fstab
# ADD:
/swapfile none swap sw 0 0
# ctrl-o ctrl-x to save n' exit
```

### Install grub
```bash
pacman -S grub efibootmgr os-prober
# Run os-prober to test

# MAKE SURE TO UNCOMMENT the os-prober line in /etc/default/grub

# GRUB_CMDLINE_LINUX="cryptdevice=UUID={ROOT_UUID_FROM_blkid}:cryptroot root=/dev/mapper/cryptroot"

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```
