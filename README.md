# Arch-Linux-Installation-Guide
What you get: UEFI + systemd-boot + LVM + swap file + Xorg + awesomeWM + tablet mode  
Tested (2024) on: Intel i7 12th Gen, Iris Xe Graphics, 512G SSD, touch screen foldable

## Get Started:
Download the arch linux iso file (https://archlinux.org/download/#download-mirrors) and use an empty USB stick.
> `cat path/to/archlinux-version-x86_64.iso > /dev/disk/by-id/usb-My_flash_drive`    

Boot into the USB stick.

## In the live environment:
Connect to wifi using `iwctl`
```
[iwd]# device list          
[iwd]# station wlan0 scan    
[iwd]# station wlan0 get-networks    
[iwd]# station wlan0 connect wifi-name    
[iwd]# exit
```

Test your connection using `ping archlinux.org`    

Next,
> `timedatectl set-ntp true`  

### Partitioning your disk (LVM):
> `fdisk -l`    
> `fdisk /dev/thenameofyourdisk`

I will be creating two partitions, one for EFI and the other for LVM.    
Create a GPT:
> `Command (m for help): g`

First partition:
> `Command (m for help): n`

Accept the defaults (hit enter on a blank entry) but for the last sector, use `+500M` 

Second partition:
> `Command (m for help): n`

Accept all the defaults. For the system to know the type,
> `Command (m for help): t`

Accept the defaults if it matches and use `L` when prompted to show the list of types. Use the number against LVM in the list:
> `Partition type (type L to list all types): xx`

Write the changes:
> `Command (m for help): w`

Format the EFI partition.
> `mkfs.fat -F32 /dev/devicename1`

LVM:    
We're allocating 60GB for root, tools like Android Studio and docker take up space if you're going to use them.    
> `pvcreate /dev/devicename2`  
> `vgcreate GROUPNAME /dev/devicename2`  
> `lvcreate -L 60GB GROUPNAME -n rootname`  
> `lvcreate -l 100%FREE GROUPNAME -n homename`

Verify with `lvdisplay`

Few more commands before we move to the next section.
> `modprobe dm_mod`  
> `vgscan`  
> `vgchange -ay`  

Format and mount the root partition:
> `mkfs.ext4 /dev/GROUPNAME/rootname`  
> `mount /dev/GROUPNAME/rootname /mnt`  

Mount the EFI partition:
> `mkdir /mnt/boot`  
> `mount /dev/devicename1 /mnt/boot`

Verify with `lsblk`  

Format and mount the home partition:
> `mkfs.ext4 /dev/GROUPNAME/homename`  
> `mkdir /mnt/home`  
> `mount /dev/GROUPNAME/homename /mnt/home`

Get essential packages:
> `pacstrap -K /mnt base linux linux-firmware vim lvm2`

Next,
> `genfstab -U /mnt >> /mnt/etc/fstab`

Check with `cat /mnt/etc/fstab`  

### Enter into '/':    
> `arch-chroot /mnt`

Make a link for your localtime (figure out your own timezone though):
> `ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime`

More commands:
> `hwclock --systohc`

Edit `/etc/locale.gen` to uncomment `en_US.UTF-8 UTF-8` and then generate the locale:
> `locale-gen`    

Edit `/etc/locale.conf` to insert `LANG=en_US.UTF-8`    
Edit `/etc/hostname` to insert a custom `hostname`    
Edit the `/etc/hosts` file:
```
vim /etc/hosts  
# See hosts(5) for details.  
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    username.localdomain    username
```

Edit the `/etc/mkinitcpio.conf` file:  
> `HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block lvm2 filesystem fsck)`

Run:
> `mkinitcpio -p linux`    

### Swapfile:    
I would recommend that you allocate RAMsize + 2GB to the swapfile.
> `fallocate -l 18G /swapfile`    
> `mkswap /swapfile`
> `chmod 600 /swapfile`
> `swapon /swapfile`

Edit `/etc/fstab` to include the swap file in it:

```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/mapper/LVM00-lvmroot
UUID=2dfee3da-c567-4509-81bd-46b6220f27a3	/         	ext4      	rw,relatime	0 1

# /dev/nvme0n1p1
UUID=55AC-0C30      	/boot     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	0 2

# /dev/mapper/LVM00-lvmhome
UUID=728f1042-3331-4427-a651-4bb539e83a9e	/home     	ext4      	rw,relatime	0 2

/swapfile none swap defaults 0 0
```

Install packages:  
> `pacman -Syu efibootmgr networkmanager base-devel linux-headers iwd linux-firmware pipewire-audio xorg xorg-xinit xorg-server awesome picom mesa`

Next,
> `bootctl --path=/boot install`

Go into `/boot/loader`  
Edit `loader.conf`:
> `#console-mode keep` 
> `default arch-*`

Go into `/boot/loader/entries`    
Edit `arch.conf`:
```
title    Arch Linux  
linux    /vmlinuz-linux  
initrd   /initramfs-linux.img  
options  root=/dev/GROUPNAME/rootname rw
```

Enable NetworkManager: 
> `systemctl enable NetworkManager`

Before you add a new user, set the root password with `passwd`
Add new user: 
> `useradd -mG wheel username`  
> `passwd username`  
> `EDITOR=vim visudo`

Edit `/etc/sudoers.tmp` by uncommenting `%wheel ALL=(ALL:ALL) ALL`  

Exit the root and unmount all (should tell you everything's busy):
> `exit`    
> `umount -a`

`reboot` the system.  
## Outside your live environment (normal usage):    
Install a terminal and a web browser to get you started:
> `sudo pacman -Syu alacritty firefox nautilus gvfs-mtp`

Install graphic drivers:
> `sudo pacman -Syu vulkan-icd-loader vulkan-intel intel-ucode intel-media-driver`

Edit `arch.conf` in `/boot/loader/entries`:
```
title	Arch Linux    
linux	/vmlinuz-linux    
initrd	/initramfs-linux.img    
initrd  /intel-ucode.img    
options	root=/dev/LVM00/lvmroot rw
```

### ClamAV:
> `sudo pacman -S clamav`    
> `sudo freshclam`    
> `sudo systemctl enable clamav-freshclam-once.timer`

### Install for gaming:
Enable multilib first:    
Edit `/etc/pacman.conf`:    
Look for the following lines and uncomment them:    
```
#[multilib]    
#Include = /etc/pacman.d/mirrorlist
```
Then, install the following:    
> `sudo pacman -Syu discord steam lutris wine wine-mono`    
> `paru protonup-qt`


### Gestures for trackpad:
> `sudo pacman -S xf86-input-libinput xorg-xinput wmctrl xdotool`    
> `paru libinput-gestures`    
> `libinput-gestures-setup autostart start`


## Swapfile for hibernation:
Find your swapfile offset:    
> `filefrag -v swap_file`

From the output, take the first number from the physical offset.    
Add the `resume=` and `resume-offset=` flags to your `arch.conf`:
```
title	Arch Linux    
linux	/vmlinuz-linux    
initrd	/initramfs-linux.img    
initrd	/intel-ucode.img    
options	root=/dev/LVM00/lvmroot resume=/dev/LVM00/lvmroot resume_offset=3887104 rw
```
  
Regenerate using `sudo mkinitcpio -p linux` and you're good to go after a `reboot`.
 
## Custom Configuration:    
> `sudo cp /etc/libinput-gestures.conf ~/.config/`

Download my configuration from the files if you need it, else make your own by editing the file in the `.config` directory.
