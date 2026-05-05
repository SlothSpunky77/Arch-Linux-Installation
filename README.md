# Arch Linux Installation (Lenovo Flex 5 Optimized)

This guide installs Arch Linux with a focus on:

- Battery life
- Thermal stability
- Low idle power
- Smooth performance

Target hardware:
- Intel 11th/12th gen (Iris Xe)
- SSD
- Touchscreen (Flex series)

---

# 0. Pre-install Setup

## Connect to WiFi
iwctl
station wlan0 connect <SSID>

## Update system clock
timedatectl set-ntp true

---

# 1. Optimize Mirrors (IMPORTANT)

pacman -Sy reflector

reflector --country India --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

### Why
- Faster downloads = less CPU/network usage → less energy waste

---

# 2. Disk Partitioning

## Recommended layout (NO LVM)

- EFI: 512M
- ROOT: rest of disk

### Why NOT LVM
- No performance gain
- Adds complexity
- Not needed for single SSD laptop

---

# 3. Format Partitions

mkfs.fat -F32 /dev/sdX1

## Use btrfs (recommended)
mkfs.btrfs /dev/sdX2

---

# 4. Mount with Optimizations

mount -o compress=zstd,noatime /dev/sdX2 /mnt

mkdir /mnt/boot
mount /dev/sdX1 /mnt/boot

### Why
- zstd compression → fewer disk reads
- noatime → avoids unnecessary writes

---

# 5. Install Base System

pacstrap /mnt base linux linux-firmware intel-ucode

### Why
- intel-ucode improves CPU stability + power states

---

# 6. Generate fstab

genfstab -U /mnt >> /mnt/etc/fstab

---

# 7. Chroot

arch-chroot /mnt

---

# 8. System Configuration

## Timezone
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc

## Locale
nano /etc/locale.gen
# Uncomment:
en_US.UTF-8 UTF-8

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

---

# 9. Hostname

echo "arch-flex" > /etc/hostname

---

# 10. Bootloader (systemd-boot)

bootctl install

nano /boot/loader/entries/arch.conf

title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=UUID=<your-uuid> rw quiet loglevel=3 nowatchdog intel_pstate=active i915.enable_psr=1 i915.enable_fbc=1

### Why these kernel params

- quiet/loglevel → fewer CPU wakeups
- intel_pstate → modern CPU power scaling
- i915 tweaks → lower GPU power usage

---

# 11. Networking

pacman -S networkmanager
systemctl enable NetworkManager

---

# 12. Create User

useradd -m -G wheel user
passwd user

EDITOR=nano visudo
# Uncomment:
%wheel ALL=(ALL:ALL) ALL

---

# 13. Install Essential Packages

pacman -S \
sudo vim git base-devel \
mesa \
xf86-input-libinput \
acpi acpid \
thermald \
tlp \
zram-generator

---

# 14. Enable Services

systemctl enable acpid
systemctl enable thermald
systemctl enable tlp

### Why

- thermald → prevents overheating using Intel hardware controls :contentReference[oaicite:1]{index=1}  
- TLP → applies aggressive power-saving automatically :contentReference[oaicite:2]{index=2}  
- acpid → proper laptop event handling  

---

# 15. Configure ZRAM (Replace Swap)

nano /etc/systemd/zram-generator.conf

[zram0]
zram-size = ram / 2
compression-algorithm = zstd

### Why
- Faster than disk swap
- Reduces SSD writes
- Better battery efficiency

---

# 16. SSD Optimization

systemctl enable fstrim.timer

---

# 17. WiFi Power Saving

mkdir -p /etc/NetworkManager/conf.d

nano /etc/NetworkManager/conf.d/wifi-powersave.conf

[connection]
wifi.powersave = 2

---

# 18. Touchscreen + Auto-Rotation

pacman -S iio-sensor-proxy
systemctl enable iio-sensor-proxy

---

# 19. Optional: Desktop / WM

## Lightweight (recommended)
pacman -S xorg-server xorg-xinit awesome

OR (better battery)
pacman -S sway

### Why
- Wayland often has lower idle power than Xorg

---

# 20. Final Steps

exit
umount -R /mnt
reboot

---

# Post-Install Power Verification

Install:
pacman -S powertop

Run:
powertop

Check:
- CPU idle states
- Wakeups per second
- Power usage

---

# Key Design Decisions (Summary)

## 1. TLP over manual tuning
TLP enables multiple power optimizations automatically and is still needed because kernel defaults don’t enable everything :contentReference[oaicite:3]{index=3}

## 2. thermald
Controls thermal behavior before hardware throttling → smoother performance

## 3. zram instead of swap
Improves responsiveness and reduces disk I/O

## 4. btrfs + compression
Less data read/write → lower energy usage

## 5. intel_pstate
Modern CPU scaling improves efficiency vs legacy governors

---

# Expected Results

Compared to a generic Arch install:

- Lower idle temps
- Reduced fan noise
- Better battery life (often +20–40%)
- Faster perceived responsiveness

---

# Notes

- Do NOT run multiple power tools together (e.g., TLP + power-profiles-daemon)
- Tune incrementally if stability issues occur
- Test GPU power flags individually if needed

---
