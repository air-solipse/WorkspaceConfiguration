# Configuring Linux

Good day. If you're not me, why are you (still) here?

In any case, these are the instructions and details and notes and such that I am collecting to help with my Linux setup.

This document assumes every part of the setup is in accordance with my [ArchLinux Installation](ArchLinuxInstallation-ISO.md) notes.

## Step 1: Preliminairies

Create a username and password for yourself

```
useradd -m -G wheel -s /bin/bash username
passwd username
```

To give sudo power to this new user, open the appropriate configuration file with
```
vim /etc/sudoers
```
and uncomment the line `%wheel ALL=(ALL:ALL) ALL`.
This actually gives sudo power to all users in `wheel`.
A warning about this being a read-only file may appear. Simply use `:wq!` to close the file and all will be well.

Also, before configuring further, install the necessary packages
```
sudo pacman -S snapper snap-pac grub-btrfs inotify-tools borg openssh rsync reflector
```

## Step 2: Snapper

Remove the `/.snapshots` folder and have snapper recreate it.
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
sudo grub-mkconfig -o /boot/grub/grub.cfgs
```

## Step 2: Borg

This section will appear eventually.

To continue and configure a desktop environment, go to [the next file](ArchLinuxConfiguration-DE.md).