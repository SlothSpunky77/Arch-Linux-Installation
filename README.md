# Arch Linux Installation (Optimized Laptop Setup)

Target:

- Lenovo Flex 5
- Intel Iris Xe
- SSD
- Touchscreen
- X11 + AwesomeWM
- btrfs
- Battery + thermal optimized

---

# 1. Connect to Internet

## WiFi

```
iwctl
```

Inside iwctl:

```
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect"SSID"
exit
```

Enable time sync:

```
timedatectl set-ntp true
```

---

# 2. Update Mirrors

Install reflector:

```
pacman -S reflector
```

Generate fast mirrors:

```
reflector \
--country India \
--latest10 \
--protocol https \
--sort rate \
--save /etc/pacman.d/mirrorlist
```

---

# 3. Partition Disk

Launch:

```
cfdisk /dev/nvme0n1
```

Create:

| Partition | Size | Type |
| --- | --- | --- |
| EFI | 512M | EFI System |
| ROOT | Remaining | Linux filesystem |

No swap partition needed because zram will be used.

---

# 4. Format Partitions

```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -f /dev/nvme0n1p2
```

---

# 5. Mount Filesystem

Mount root with optimization flags:

```
mount-o compress=zstd,noatime /dev/nvme0n1p2 /mnt
```

Mount EFI:

```
mkdir-p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

Why:

- `compress=zstd`
    - reduces disk writes
    - improves SSD efficiency
- `noatime`
    - prevents unnecessary metadata writes

---

# 6. Install Base System

```
pacstrap /mnt \
base \
linux \
linux-firmware \
intel-ucode \
btrfs-progs \
networkmanager \
sudo \
vim \
git
```

---

# 7. Generate fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

---

# 8. Chroot

```
arch-chroot /mnt
```

---

# 9. Configure Time

```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
```

---

# 10. Configure Locale

Edit locale file:

```
vim /etc/locale.gen
```

Uncomment:

```
en_US.UTF-8 UTF-8
```

Generate locales:

```
locale-gen
```

Set language:

```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

# 11. Hostname

```
echo"arch-flex" > /etc/hostname
```

Optional hosts file:

```
vim /etc/hosts
```

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch-flex.localdomain arch-flex
```

---

# 12. Set Root Password

```
passwd
```

---

# 13. Create User

```
useradd -m -G wheel shashank
passwd shashank
```

Enable sudo:

```
EDITOR=vim visudo
```

Uncomment:

```
%wheel ALL=(ALL:ALL) ALL
```

---

# 14. Install Bootloader

Install systemd-boot:

```
bootctl install
```

Get root UUID:

```
blkid
```

Create loader entry:

```
vim /boot/loader/entries/arch.conf
```

```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=UUID=ROOT_UUID rw quiet loglevel=3 nowatchdog i915.enable_fbc=1
```

Avoid forcing PSR initially because some Iris Xe systems flicker with it.

---

# 15. Install Graphics + X11

```
pacman -S \
mesa \
xorg-server \
xorg-xinit \
xorg-apps \
xf86-input-libinput
```

---

# 16. Install Window Manager

Example:

```
pacman -S awesome picom
```

Create xinitrc:

```
echo "exec awesome" > ~/.xinitrc
```

Launch X11:

```
startx
```

---

# 17. Install Power Management

```
pacman -S \
tlp \
thermald \
acpi \
acpid \
powertop \
zram-generator
```

Enable services:

```
systemctl enable NetworkManager
systemctl enable tlp
systemctl enable thermald
systemctl enable acpid
systemctl enable fstrim.timer
```

Do NOT install:

- power-profiles-daemon
- auto-cpufreq

They overlap/conflict with TLP.

---

# 18. Configure ZRAM

```
vim /etc/systemd/zram-generator.conf
```

```
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```

This replaces traditional swap for most laptop workloads.

---

# 19. Configure WiFi Power Saving

```
mkdir -p /etc/NetworkManager/conf.d
```

```
vim /etc/NetworkManager/conf.d/wifi-powersave.conf
```

```
[connection]
wifi.powersave=3
```

---

# 20. Touchscreen + Sensors

```
pacman -S iio-sensor-proxy
```

No need to manually enable the service.

---

# 21. Reboot

Exit chroot:

```
exit
```

Unmount:

```
umount -R /mnt
```

Reboot:

```
reboot
```

---

# 22. Post-Install Verification

Check TLP:

```
sudo tlp-stat-s
```

Check thermals:

```
sudo thermald--no-daemon--loglevel=info
```

Check power usage:

```
sudo powertop
```

Important metrics:

- idle wattage
- wakeups/sec
- package C-states

---

# Design Decisions

## btrfs

- transparent compression
- reduced SSD writes
- snapshots possible later

## zram instead of swap partition

- lower SSD wear
- better responsiveness under memory pressure

## X11 instead of Wayland

- better AwesomeWM compatibility
- more predictable compositor behavior

## TLP + thermald

- TLP handles platform power management
- thermald handles Intel thermal behavior separately

---

# Expected Results

Compared to default Arch:

- lower idle power
- quieter fans
- reduced heat spikes
- smoother battery discharge
- faster perceived responsiveness
- better standby efficiency
