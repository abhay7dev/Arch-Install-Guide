# ***THIS IS A WORK IN PROGRESS***

DISCLAIMER: THIS IS A PERSONAL GUIDE FOR ME AND FOR MY OWN COMPUTER.

# Dual Boot Arch Linux and Windows 10
## Following Features
* Windows 10
* Veracrypt encrypted windows partition
* Arch Linux encrypted partition with LUKS (seperate unecrypted boot partition)
* rEFInd boot manager and en ext4 filesystem for linux
* KDE Plasma

# THIS GUIDE WILL BE WITH UEFI, PLEASE VERIFY YOU HAVE THAT BEFORE PROCEEDING.

## Step 1: Install Windows
I am not going to go through this as I am going to be installing arch on a machine with windows already installed but I will have a quick quide below.
1. Install the Windows 10 Media Creation Tool: https://www.microsoft.com/en-us/software-download/windows10
2. Create an ISO file for windows on your hard drive
3. Burn the ISO file to a disc or usb. I suggest using [Balena Etcher](https://github.com/balena-io/etcher/releases). [rufus](https://rufus.ie/en/) also works

## Setup Windows
1. Go to the control panel, go to power options, go to `Change what the power buttons do` and **turn off fast startup**
2. Configure your windows partition by searching `diskmg` and clicking on the `Create and format hard disk partitions`.
3. Configure your Windows partition to how much you want it to be. **DO NOT DELETE THE RECOVERY OR EFI SYSTEM PARTITION**. Right click on the windows partition and click shrink partition and shrink it to the amount you want for your arch linux partition.
![image](https://user-images.githubusercontent.com/80220229/166091402-28567ccc-b809-4da3-bff1-67061202d30c.png)

## Setup Veracrypt
1. Download and install veracrypt from https://www.veracrypt.fr/en/Downloads.html
2. Click on `Encrypt a system partition`
![image](https://octetz.s3.us-east-2.amazonaws.com/linux-windows-install/veracrypt-encrypt-system.png)
3. Choose normal, encrypt windows partition, and **click single-boot for number of operating systems**. Then choose the encryption and hash algorithm you prefer, I will go with standard AES and SHA-512.
4. Create a password and confirm it
5. Generate random data with your mouse
6. Create a rescue disk for Veracrypt. 
7. Choose your wipe mode (I will choose none).
8. Allow Veracrypt to restart your computer and boot up again and log back in, then the veracrypt encrypt process will begin.
9. And you are done with Windows! Power off the system

## Get into arch linux
1. You need to have a bootable usb of arch linux which you can get from the downloads page. Once again, you need to create a bootable usb or disk for the arch linux iso. I suggest using rufus, which I linked above. https://archlinux.org/download/
2. If you have an ethernet cable, now is the time to plug it in. Also plug in your arch usb drive.
3. Boot your machine and from the boot menu, select the arch linux iso to open up to a menu which looks like the following.
![image](https://user-images.githubusercontent.com/80220229/166091811-a7d48315-722c-42a1-a8f7-87db3dce177f.png)
4. Boot into the arch linux instlal medium and enter the arch iso!

## Basic setup
1. If you would like to change the keyboard layout, do it now.
2. Connect to the internet, if you have an ethernet cable plugged in, you should already be connected. If not, then use `iwctl` to connect to the internet
3. `ping archlinux.org` to verify you are connected
4. `timedatectl set-ntp true` to start network time synchronization.

## OPTIONAL STEP. ENABLE SSH FOR EASIER INSTALLATION PROCESS
If you want to install arch from a seperate machine through ssh, you can do so by following these steps.
1. run `passwd` and set a password for the root user of the arch iso
2. Enable `sshd` by running `systemctl start sshd`
3. determine the machine's local ip address by running `ip a`
![image](https://user-images.githubusercontent.com/80220229/166091995-860af5e2-8f74-4ad3-901a-b25f340e51e0.png)
4. The above image means that my local ip address is `192.168.20.159`
5. You can now ssh in by doing `ssh root@ip` replacing `ip` with the address you just got
![image](https://user-images.githubusercontent.com/80220229/166092049-de8c77be-ba4d-4b5b-9681-082e37ce8578.png)


## Partitioning the disk
1. run `lsblk` just to see what you have right now. Here is an example output from vmware
![image](https://user-images.githubusercontent.com/80220229/166091945-c17d5bc2-8b4d-4d9e-956c-106584e08806.png)
2. I will be using `cgdisk` to partition my drive. Type `cgdisk /dev/nvme0n1` to bring up the utility
![image](https://user-images.githubusercontent.com/80220229/166092163-7629f7db-1281-4968-a4ff-6fb530d296d1.png)
3. Above is an example of how the output may look. **Do not touch the EFI System partition or microsoft/windows partitions**. The only partition we are worried about is the free space partition we created earlier, the 48.8 GiB is the largest free space partition and the one I chose to have arch linux.
4. Go to the free space and use the arrow keys to go to `[ New ]` and click enter
5. Use the default `first sector` value by just pressing enter
6. Enter 512Mib for the `size in sectors`
7. Leave the default partition type of Linux Filesystem and create a partition with the name of `boot`. We will be seperating the boot and root as I do not want to encrypt the immediate boot of the computer.
8. With the free space, choose the default first sector, choose teh default size (which takes up the rest of the disk) and the default filesystem. Name this partition `root`.
9. Use the arrow keys to navigate to `[ Write ]` and type yes to confirm
10. Take note of your current partitions.
![image](https://user-images.githubusercontent.com/80220229/166092393-abac0a56-8b10-471b-b72d-702fc6bedad7.png)
12. Specifically, the partitions which matter are (**These will likely be different for you**)
```
EFI System Partition: /dev/nvme0n1p1
Boot Partition: /dev/nvme0n1p5
Root Partition: /dev/nvme0n1p6
```
13. navigate to `[ Quit ]` and exit cgdisk
14. Verify your partitions with `lsblk`
![image](https://user-images.githubusercontent.com/80220229/166092470-3da80d2e-6be2-4e3e-9354-9c6edd0b7678.png)

## Encrypt root and format boot and root

1. Run `cryptsetup -y --use-random luksFormat /dev/nvme0n1p6` - Change `/dev/nvme0n1p6` according to your needs. We are encrypting the *root* partition.
  * `-y`: interactively requests the passphrase twice.
  * `--use-random`: uses /dev/random to produce keys.
  * `luksFormat`: initializes a LUKS partition.
2. type `YES` and enter the password you want for the encrypted linux root partition
3. Now the partition is encrypted and we cannot access it! Open it up using the command `cryptsetup luksOpen /dev/nvme0n1p6 cryptroot` to create a mapping in `/dev/mapper/cryptroot`
4. The output of `lsblk` should now look similar to the following with cryptroot attached to the root partition
![image](https://user-images.githubusercontent.com/80220229/166092626-370b5824-2f24-4241-ac3d-0df2ffbe93be.png)
5. Format the boot and root partitions by running: `mkfs.ext4 /dev/nvme0n1p5` for the boot partition (again, change the device to the name of yours) and `mkfs.ext4 /dev/mapper/cryptroot` for the root partition. We need to access the root partition through the cryptroot mapper we created.

## Mounting the arch linux partition, and installing linux
1. `mount /dev/mapper/cryptroot /mnt` mount cryptroot at /mnt
2. `mkdir /mnt/boot` create a boot directory
3. `mount /dev/nvme0n1p5 /mnt/boot` mount the boot partition to this directory (change the name as required)
4. `mkdir /mnt/boot/efi` make efi directory
5. `mount /dev/nvme0n1p1 /mnt/boot/efi` to mount the existing efi directory.
6. Now comes the fun part. Running `pacstrap` to install everything you need for a basic arch linux install. There are a multitude of packages you need to install depending on your needs. The following is a command to install everything I need.
7. `pacstrap /mnt linux linux-headers linux-firmware intel-ucode base base-devel git nano efibootmgr xf86-input-vmmouse xf86-video-vmware mesa networkmanager sof-firmware`
  * `linux`, `linux-headers`, and `linux-firmware` are all packages for linux itself
  * `intel-ucode` is for my processor microcode which is an intel one. Use `amd-ucode` for AMD processors.
  * `base` and `base-devel` are some essential packages
  * `git` is git
  * `nano` is a text editor for the console
  * `efibootmgr`
  * `xf86-input-vmmouse` for the touchpad in vmware. I would use **`xf86-input-synaptics` or `libinput`** for my actual computer
  * `xf86-video-vmware` as a video driver. I would use `xf86-video-intel` for my computer. https://wiki.archlinux.org/title/Intel_graphics
  * `mesa` for opengl and other graphics things
  * `networkmanager` a network and internet manager for the command line
  * `sof-firmware` Sound Open Firmware
8. MAKE SURE TO DO THIS: `genfstab -U /mnt >> /mnt/etc/fstab` to generate the file system tables. We use `-U` to use uuids rather than the names.

## Enter the system and set it up.
1. Time to enter the system! Run `arch-chroot /mnt` and you are now in your linux installation!
2. Set the timezone by running `ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime` but with your timezone
3. set the hardware clock with `hwclock --systohc`
4. Run `nano /etc/locale.gen` (or vim) and navigate to the commented out line for your locale. In my case, it would be `en_US.UTF-8 UTF-8` then ctrl-o and ctrl-x to save and exit in nano
5. `locale-gen` to generate the locale
6. `echo "LANG=en_US.UTF-8" >> /etc/locale.conf` Just setting it up. Make sure to change the lang to the line you uncommented
7. `echo "arkde" >> /etc/hostname` Set your hostname with whatever you would like to name your pc :)
8. Set hosts. Run `nano /etc/hosts` and add the following lines.
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arkde.localdomain arkde
```
Rename "arkde" with your hostname

## Ramdisk Config
Ramdisk is essentially what to do when you boot up your machine. 
1. Use `nano` or any texteditor to change the `HOOKS` in `/etc/mkinitcpio.conf` with `HOOKS=(base udev autodetect keyboard modconf block encrypt filesystems fsck)`. You will need to add `encrypt` and move `keyboard` earlier.
![image](https://user-images.githubusercontent.com/80220229/166093808-be5dcdd2-b739-4d96-aebf-71f5ff064c7a.png)
2. Run `mkinitcpio -p linux`

## User configuration
1. Set the root password with `passwd`
2. add a user with `useradd -m -G wheel abhay` to create a user in the wheel group and one which has a home directory.
3. set the password of that user with `passwd abhay` or whatever you named your user.
4. Use `EDITOR=nano visudo` or whatever editor you choose to allow users of the wheel group to use sudo by uncommenting the line `%wheel ALL=(ALL:ALL) ALL`
![image](https://user-images.githubusercontent.com/80220229/166093897-0f8069dd-d4ff-42a4-80d8-64a85b168711.png)

## Boot manager and networkmanager
1. Enable network manager by running `systemctl enable NetworkManager.service`
2. Download rEFInd by running `pacman -S refind`
3. Install it by running `refind-install`
4. Now, we need to make sure refind actually works and boots up linux because right now, it is configured based on the arch iso and the flash drive. This is a simple task to do. I will be doing the following in nano because that is easiest for me but you can do it in vim as well.
5. Run `cp /boot/refind_linux.conf /boot/refind_linux.conf.bak` to copy the current configuration (optional, you likely will not need it)
6. Run `blkid >> /tmp/id.txt` to get the block ids and ouput it to `/tmp/id.txt`.
7. Run `nano /boot/refind_linux.conf` and erase everything except the first line
8. Erase the text in the second string so that your file now looks like `"Boot with standard options"  ""`
9. Now navigate below that and use `Ctrl-R` and type `/tmp/id.txt` to paste the contents into the file.
10. Erase everything but the line with your root filesystem, which would be `/dev/nvme0n1p6` for us.
11. Erase everything in that line **except the PARTUUID**
12. On the first line in the second set of quotations, add `cryptdevice=PARTUUID={YOUR PART UUID HERE}:root root=/dev/mapper/root` replacing the text inside the {} and the {} themselves with the Partuuid you saved on step 11. You can `ctrl-k` and `ctrl-u` to cut and paste.
13. Save and quit with `ctrl-o` and `ctrl-x`
![image](https://user-images.githubusercontent.com/80220229/166124557-a6ecdf56-98c1-4e6e-a3a1-1340ef2799fd.png)

## Exit the environment and reboot
1. run `exit` to get back to the arch iso
2. unmount the partitions with `umount -R /mnt`
3. Reboot with `reboot`!

## If you used grub and want refind, login and run `refind-install`, you should not have to do any editing and you can safely remove grub

## Result
![image](https://user-images.githubusercontent.com/80220229/166094211-303a10bc-c268-4341-8cd7-da50d04f5036.png)
You should now have a boot menu which looks like this!
You can choose the windows icon or the second icon to boot windows
You can choose linux from the third option and the archiso flashdrive is the fourth option

## Help! I Screwed up
Here is how to get back into your system through the arch iso
1. Boot up into the arch flash drive/iso.
2. Mount everything by running:
```bash
cryptsetup luksOpen /dev/nvme0n1p6 cryptroot # To open the encrypted partition
mount /dev/mapper/cryptroot /mnt
mount /dev/nvme0n1p5 /mnt/boot 
mount /dev/nvme0n1p1 /mnt/boot/efi
arch-chroot /mnt
```
from which you can fix anything you need inside the system.
3. To exit, just follow the same instructions listed up above for exiting the environment and rebooting.

## References
* https://octetz.com/docs/2020/2020-2-16-arch-windows-install/ ([Video version too](https://www.youtube.com/watch?v=ybvwikNlx9I))
* https://wiki.archlinux.org/title/REFInd
* https://bbs.archlinux.org/viewtopic.php?id=252853
* https://wiki.archlinux.org/title/Installation_guide


# Configuring Arch Linux

## Allow 32bit apps
Edit `/etc/pacman.conf` with nano or any texteditor uncomment the below two lines.
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```
You can also uncomment the line for installing packages concurrently
I like to run `sudo pacman -Syyu` after that just to sync multilib as well as doing any updates that I need to do.

## Install `yay` for the AUR
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
Run the above commands

## Install plasma

1. Run `sudo pacman -S xorg plasma plasma-wayland-session sddm pipewire packagekit-qt5`
2. After installation run `systemctl enable sddm.service`
3. Run `sudo nano /usr/lib/sddm/sddm.conf.d/default.conf` (or vim) and change the theme to "Breeze" so that the result is as follows
```
[Theme]
Current=breeze
```
4. `sudo pacman -S konsole dolphin`
5. Run `sudo pacman -S packagekit-qt5 fwupd` for getting app backends for kde discover
6. `pacman -S ufw` for a firewall which can be managed from KDE's system settings
7. `yay -S librewolf-bin` to install [librewolf](https://librewolf.net)
8. `sudo pacman -S neofetch`
9. `reboot` into plasma!

## Some settings
* Dark theme. Set screen resolution to 1920x1080.
* Wobbly windows and translucency
