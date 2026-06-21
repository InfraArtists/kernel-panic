# Scenario 01 — Deleted vmlinuz Kernel Image


**How it was broken:**
```bash
# Find the running kernel, then delete its image (the destructive action)
uname -r
sudo rm /boot/vmlinuz-$(uname -r)
sudo reboot
```

## Root Cause
GRUB's menu entries (generated in `/boot/grub/grub.cfg`) reference the exact
kernel image path. When the `vmlinuz` file is removed, GRUB cannot locate the
kernel binary to hand control to, so boot halts before the kernel executes.

## Recovery Steps
1. At the GRUB menu, pick an **older kernel** under *Advanced options for Ubuntu*
   if one exists, and boot it. If no other kernel exists, boot from a Live USB.
2. Reinstall the affected kernel package to regenerate the image.
3. Update GRUB and reboot.

### Option A — An older kernel still boots
```bash
# After booting an older kernel from the GRUB "Advanced options" submenu:
# Reinstall the missing kernel (adjust version to the one deleted)
sudo apt-get update
sudo apt-get install --reinstall linux-image-6.8.0-31-generic
sudo update-grub
sudo reboot
```

### Option B — No bootable kernel, use a Live USB
```bash
# Boot the Ubuntu 24.04 Live USB, open a terminal, then chroot into the install.
# Identify your root partition (e.g. /dev/sda2) with: lsblk -f
sudo mount /dev/sda2 /mnt
sudo mount /dev/sda1 /mnt/boot/efi      # EFI system partition, if separate
for d in /dev /dev/pts /proc /sys /run; do sudo mount --bind $d /mnt$d; done
sudo chroot /mnt

# Inside chroot:
apt-get update
apt-get install --reinstall linux-image-generic
update-grub
exit

# Back in the Live session:
for d in /run /sys /proc /dev/pts /dev; do sudo umount /mnt$d; done
sudo umount /mnt/boot/efi
sudo umount /mnt
sudo reboot
```

## Commands
```bash
# Verify the kernel image is back after recovery
ls -l /boot/vmlinuz-*
uname -r
```
