# Arch-Linux-Installation
UEFI + systemd-boot + LVM + swap file
Intel i7 12th Gen, Iris Xe Graphics, 512G SSD, touch screen

## Get Started:
Use an empty USB stick.    

`cat path/to/archlinux-version-x86_64.iso > /dev/disk/by-id/usb-My_flash_drive`    

Boot into the USB stick.

## In the live environment:
Connect to wifi using `iwctl`    
`[iwd]# device list`        
`station wlan0 scan`    
`station wlan0 get-networks`    
`station wlan0 connect wifi-name`    
`exit`   

Test your connection using `ping archlinux.org`    

Next, `timedatectl set-ntp true`  

Partitioning your disk (LVM):  
`fdisk -l`  
`fdisk /dev/thenameofyourdisk`  
