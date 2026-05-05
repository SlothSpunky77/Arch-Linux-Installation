# Arch Linux Installation (Lenovo Flex 5 • btrfs • X11 Optimized)

This guide installs Arch Linux with a focus on:

- Battery efficiency
- Thermal stability
- Low idle power
- Smooth responsiveness

Target system:
- Lenovo Flex 5 (Intel iGPU – Iris Xe)
- SSD
- Touchscreen
- X11 (AwesomeWM or similar)

---

# 0. Pre-install Setup

## Connect to WiFi
iwctl
station wlan0 connect <SSID>

## Enable time sync
timedatectl set-ntp true

---

# 1. Optimize Mirrors (IMPORTANT)

pacman -Sy reflector

reflector --country India --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

---

# 2. Partitioning (No LVM)

## Layout
- EFI: 512M
- ROOT: rest of disk

### Why
- LVM adds complexity without power/performance benefits
- btrfs gives flexibility + compression natively

---

# 3. Format Partitions

mkfs.fat -F32 /dev/sdX1
mkfs.btrfs /dev/sdX2

---

# 4. Mount with Performance Flags

mount -o compress=zstd,noatime /dev/sdX2 /mnt

mkdir /mnt/boot
mount /dev/sdX1 /mnt/boot

### Why
- zstd → fewer disk reads → lower power usage
- noatime → prevents constant disk writes

---

# 5. Install Base System

pacstrap /mnt base linux linux-firmware intel-ucode

### Why
- intel-ucode improves CPU behavior and stability

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

# 13. Install Core System Packages

pacman -S \
sudo vim git base-devel \
mesa \
xf86-input-libinput \
xorg-server xorg-xinit \
acpi acpid \
thermald \
tlp \
zram-generator

---

# 14. Enable Services

systemctl enable NetworkManager
systemctl enable acpid
systemctl enable thermald
systemctl enable tlp

---

# 15. Power Management (CRITICAL)

### TLP
- Applies CPU scaling, PCIe, USB, WiFi power optimizations automatically  
- Works without manual tuning :contentReference[oaicite:1]{index=1}  

### thermald
- Prevents overheating before hardware throttling kicks in  
- Uses Intel thermal controls for smoother performance :contentReference[oaicite:2]{index=2}  

### Important Rule
DO NOT install:
- power-profiles-daemon

These tools conflict with each other.

---

# 16. Configure ZRAM (Replace Swap)

nano /etc/systemd/zram-generator.conf

[zram0]
zram-size = ram / 2
compression-algorithm = zstd

---

# 17. SSD Maintenance

systemctl enable fstrim.timer

---

# 18. WiFi Power Saving

mkdir -p /etc/NetworkManager/conf.d

nano /etc/NetworkManager/conf.d/wifi-powersave.conf

[connection]
wifi.powersave = 2

---

# 19. Touchscreen + Rotation

pacman -S iio-sensor-proxy
systemctl enable iio-sensor-proxy

---

# 20. X11 Setup (No Wayland)

## Install Xorg

pacman -S xorg-server xorg-xinit xorg-apps

## Install Window Manager (example)

pacman -S awesome

## Start X

echo "exec awesome" > ~/.xinitrc
startx

---

# 21. Optional: Compositor (Recommended for X11)

pacman -S picom

### Why
- Enables vsync → smoother rendering
- Reduces tearing
- Minimal overhead if configured correctly

---

# 22. Final Steps

exit
umount -R /mnt
reboot

---

# Post-Install Verification

Install powertop:

pacman -S powertop

Run:

powertop

Check:
- Idle power usage
- Wakeups/sec
- CPU states

---

# Architecture Summary

Arch power management works in layers :contentReference[oaicite:3]{index=3}:

1. Kernel level
   - intel_pstate
   - i915 GPU parameters

2. Userspace tools
   - TLP (primary controller)
   - thermald (thermal control)

This guide optimizes both layers.

---

# Key Design Decisions

## btrfs over ext4
- Compression reduces disk I/O
- Better long-term efficiency

## X11 over Wayland
- Better compatibility with AwesomeWM
- Stable input handling for touchscreen devices

## TLP + thermald
- Proven combination for laptops
- Covers CPU, GPU, PCIe, USB, and thermals

---

# Expected Results

Compared to a generic Arch install:

- Lower idle power draw
- Reduced fan usage
- Better battery life (~20–40% improvement typical)
- Stable thermals under load
- Faster perceived responsiveness

---

# Notes

- Avoid installing multiple power managers
- Tune TLP only if necessary (defaults are already optimized)
- Kernel parameters can be adjusted per device behavior

---
