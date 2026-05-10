# Installing Linux

Good day. If you're not me, why are you here?

There are way better resources out there...
For instance, the [Installation guide](https://wiki.archlinux.org/title/Installation_guide) I am poorly following here.

In any case, these are the instructions and details and notes and such that I am collecting to help with my Linux setup.

## Writing the ISO file to a flash drive

Booting from an external drive makes more sense to me than the alternatives, so here we are.

First, download the ISO file from a magnet link available on [the official Arch Linux website](https://archlinux.org/download/).

Then, provided a USB drive is inserted into the current computer, it is converted into a flash drive thus.

### On MacOS

In the terminal, list the drives to get the number of the external one.
```
diskutil list
```
Then, unmount the disk, write the ISO to it and eject it.
```
diskutil unmountDisk /dev/disk2
sudo dd if=filename.iso of=/dev/rdisk2 bs=4M
diskutil eject /dev/disk2
```

### On Arch Linux

I have not yet had to perform this task, but this section will eventually contain information.

## Pre-Install Configuration

Once succefully booted into the Arch Linux ISO, there will appear a virtual console.

### Interface Settings

My keyboard follows the Canadian Multilingual Standard Layout.
Hence I configure
```
loadkeys ca
```

### Boot mode

The command
```
cat /sys/firmware/efi/fw_platform_size
```
will return the UEFI bitness if the system is booted in UEFI mode and `No such file or directory` if the system is booted in BIOS mode.

UEFI is the desirable one unless applying this process to an old machine.
If BIOS is the only option, it is easier to install Linux through a more beginner-friendly distro. 

### Internet

First get a list of recognized network interfaces with the following command.
```
ip link
```

Make sure the Wi-Fi card shows up.
I have not yet had to troubleshoot this part, so it is assumed for now everything is hunky dory.

Finally, update the system clock to ensure it is synchronized
```
timedatectl
```

### Disk Partition

First, list the disks to identify the target.
```
lsblk
```
I will henceforth assume it is `/dev/sda`, as it usually is.

Then, launch `gdisk` to begin partitioning the disk.
```
gdisk /dev/sda
```
The procedure is then summarized as follows:
- Run `o` to create a new partition table.
- Run `n` to add a new partition (EFI System Partition)
  - Non-default parameters: Last sector is `+512M` and Hex code is `ef00`
- Run `n` to add a new partition (Linux Filesystem)
  - Non-default parameter: Hex code is `8300`
- Run `w` to write the table to disk and exit

### Partition Formatting

First, format the ESP to correctly store bootloader files.
```
mkfs.fat -F32 /dev/sda1
```

Next, set up LUKS encryption on the other Linux Filesystem partition.
```
cryptsetup luksFormat /dev/sda2
```
and enter the desired system passphrase.

Open the encrypted container
```
cryptsetup open /dev/sda2 cryptroot
```
thus creating the temporary path `/dev/mapper/cryptroot`.

Format (something) into a Btrfs Filesystem.
```
mkfs.btrfs /dev/mapper/cryptroot
```

Then, create desired subvolumes through a temporary mount
```
mount /dev/mapper/cryptroot /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
btrfs subvolume create /mnt/@snapshots
umount /mnt
```

### Mount the Volumes

The subvolumes created before can now be properly mounted.
```
mount -o noatime,compress=zstd,subvol=@ /dev/mapper/cryptroot /mnt

mkdir -p /mnt/{boot,home,var,.snapshots}

mount -o noatime,compress=zstd,subvol=@home /dev/mapper/cryptroot /mnt/home
mount -o noatime,compress=zstd,subvol=@var /dev/mapper/cryptroot /mnt/var
mount -o noatime,compress=zstd,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
```

Then, the ESP can be mounted.
```
mount /dev/sda1 /mnt/boot
```

## Installation and Configuration

### Install

Through `pacstrap`, install the Linux Kernel and essential packages.
```
pacstrap -K /mnt base linux linux-firmware btrfs-progs sudo nano
pacstrap /mnt grub efibootmgr networkmanager snapper snap-pac
```
Depending on the CPU, install one of the following
```
pacstrap /mnt intel-ucode
pacstrap /mnt amd-ucode
```

### Basic Configurations

Run
```
genfstab -U /mnt >> /mnt/etc/fstab
```
lest words mean nothing at all. ([Persistent block device naming](https://wiki.archlinux.org/title/Persistent_block_device_naming))

Change the root to now interact directly with the new system.
```
arch-chroot /mnt
```

Set the time
```
ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime
```
where `Area/Location` can be (for example) `America/Toronto` or `Europe/London` and synchronize the hardware clock
```
hwclock --systohc
```
Also, activate the default [SNTP client](https://wiki.archlinux.org/title/Systemd-timesyncd).
```
timedatectl set-ntp true
```

Edit the `/etc/locale.gen` file by running
```
nano /etc/locale.gen
```
and using the [nano](https://wiki.archlinux.org/title/Nano) text editor to uncomment the desired UTF-8 locales.
(For example, `en_CA.UTF-8 UTF-8`, `fr_CA.UTF-8 UTF-8` and `nb_NO.UTF-8 UTF-8`.)
Then, exit (`ESC+X`) and run
```
locale-gen
```
Define the system language and keyboard layout with
```
echo "LANG=en_CA.UTF-8" > /etc/locale.conf
echo "KEYMAP=ca" > /etc/vconsole.conf
```

Set a [hostname](https://datatracker.ietf.org/doc/html/rfc1178) for the network
```
echo "chosenhostname" > /etc/hostname
```

(Network Configuration and Network Manager)
```
systemctl enable NetworkManager
```

Open the initramfs file
```
nano /etc/mkinitcpio.conf
```
and edit the `HOOKS=` line to `HOOKS=(base systemd autodetect microcode modeconf kms keyboard sd-vconsole block sd-encrypt btrfs filesystems fsck)`.
Hence adding `sd-encrypt` and `btrfs` at the correct locations.

Create a root password
```
passwd
```

### Boot Loader

Get the UUID of `/dev/sda2` through
```
blkid /dev/sda2
```
(Perhaps write it down.)

Then, open
```
nano /etc/default/grub
```
and enable cryptodisk support in GRUB by uncommenting the line `GRUB_ENABLE_CRYPTODISK=y`.
Still in the same file, modify the `GRUB_CMDLINE_LINUX=` line to
```
GRUB_CMDLINE_LINUX="rd.luks.name=the-uuid=cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@ rw"
```
where `the-uuid` is the UUID obtained earlier.

Recreate the [initramfs](https://wiki.archlinux.org/title/Arch_boot_process#initramfs).
```
mkinitcpio -P
```

Finally, install GRUB to the ESP and generate the config file.
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### End of Installation

Close everything properly and reboot into the freshly installed system.
```
exit
umount -R /mnt
reboot
```
Note: The username to use when loging in is `root`.