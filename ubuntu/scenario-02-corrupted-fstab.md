# Scenario 02 — Corrupted /etc/fstab Entry

**How it was broken:**
```bash
# Add a bogus mount line pointing at a non-existent UUID
echo "UUID=00000000-dead-beef-0000-000000000000 /data ext4 defaults 0 2" | sudo tee -a /etc/fstab
sudo reboot
```

## Root Cause
`systemd` generates mount units from `/etc/fstab`. A device that cannot be found
or an unparseable line makes the `local-fs.target` fail. Because file system
targets are a dependency of normal boot, systemd aborts into emergency mode.

## Recovery Steps
1. At the maintenance prompt, log in as root (enter the root password) — or if
   root login is disabled, boot a Live USB.
2. Remount the root filesystem read-write.
3. Edit `/etc/fstab` to fix or comment out the bad line.
4. Reboot.

### Emergency shell (root password available)
```bash
# The emergency shell often mounts root read-only; remount it writable
mount -o remount,rw /
nano /etc/fstab        # comment out or correct the bad line (prefix with #)
# Validate that all fstab entries mount cleanly
mount -a
systemctl daemon-reload
reboot
```

### Live USB fallback
```bash
sudo mount /dev/sda2 /mnt          # your root partition (check with lsblk -f)
sudo nano /mnt/etc/fstab           # fix/comment the bad line
sudo umount /mnt
sudo reboot
```

## Commands
```bash
# Safely verify fstab before rebooting: this is the command that catches typos
sudo findmnt --verify --verbose
```
