# Snapper Configuration

First install the necessary packages
```
sudo pacman -S snapper snap-pac grub-btrfs
```
Then, remove the `/.snapshots` folder and have snapper recreate it.
```
sudo umount /.snapshots
sudo rm -rf /.snapshots
sudo snapper -c root create-config /
sudo mount -a
```
To verify this has been done properly, run `findmnt /.snapshots`.

Open the configuration file with `sudo vim /etc/snapper/configs/root` and set reasonable retention values. For example,
```
TIMELINE_CREATE="yes"

TIMELINE_LIMIT_HOURLY="8"
TIMELINE_LIMIT_DAILY="5"
TIMELINE_LIMIT_WEEKLY="4"
TIMELINE_LIMIT_MONTHLY="6"
TIMELINE_LIMIT_YEARLY="5"
```

Finally, enable automatics snapshots
```
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```
and configure GRUB snapshot booting.
```
sudo systemctl enable --now grub-btrfsd.service
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
