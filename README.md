# Arch-Linux-Installation
UEFI + systemd-boot + LVM + swap file + Xorg + awesomeWM + tablet mode  
Intel i7 12th Gen, Iris Xe Graphics, 512G SSD, touch screen

## Get Started:
Download the arch linux iso file (add link here) and use an empty USB stick.
> `cat path/to/archlinux-version-x86_64.iso > /dev/disk/by-id/usb-My_flash_drive`    

Boot into the USB stick.

## In the live environment:
Connect to wifi using `iwctl`
> `[iwd]# device list`        
> `[iwd]# station wlan0 scan`    
> `[iwd]# station wlan0 get-networks`    
> `[iwd]# station wlan0 connect wifi-name`    
> `[iwd]# exit`   

Test your connection using `ping archlinux.org`    

Next,
> `timedatectl set-ntp true`  

### Partitioning your disk (LVM):
> `fdisk -l`  
> `fdisk /dev/thenameofyourdisk`

(Fill this section)

Format the EFI partition.
> `mkfs.fat -F32 /dev/devicename1`

LVM:
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

Edit the `/etc/hosts` file:
> `vim /etc/hosts`  
> `# See hosts(5) for details.`  
> `127.0.0.1    localhost`  
> `::1          localhost`  
> `127.0.1.1    username.localdomain    username`

Edit the `/etc/mkinitcpio.conf` file:  
`HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystem fsck)`  

(enter into the root missing)
Install packages:  
> `pacman -Syu efibootmgr metworkmanager base-devel linux-headers iwd linux-firmware pipewire-audio xorg xorg-xinit xorg-server awesome picom mesa`

Next,
> `bootctl --path=/boot install`

Go into `/boot/loader`  
Edit `loader.conf`:
> `#console-mode keep` 
> `default arch-*`

Go into `/boot/loader/entries` (not sure if the path is right)  
Edit `arch.conf`:
> `title    Arch Linux`  
> `linux    /vmlinuz-linux`  
> `initrd   /initramfs-linux.img`  
> `options  root=/dev/GROUPNAME/rootname rw`

Enable NetworkManager: 
> `systemctl enable NetworkManager`

Add new user: 
> `useradd -mG wheel username`  
> `passwd username`  
> `EDITOR=vim visudo`

Edit `/etc/sudoers.tmp` by uncommenting `%wheel ALL=(ALL:ALL) ALL`  

`reboot` the system.  
## Outside your live environment (normal usage):    
Install a terminal and a web browser to get you started:
> `sudo pacman -Syu alacritty firefox`

Install graphic drivers:
> `sudo pacman -Syu vulkan-icd-loader vulkan-intel`

### Install for gaming:
Enable multilib first:
> `sudo pacman -Syu lutris wine wine-mono`
