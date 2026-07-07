# Arch Linux Installation

Begin by booting your machine with a [ISO drive](ArchLinuxISO.md).

Then, follow the [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide).
The rest of this document is dedicated to elaborating on the details of this.

## Step 1: Pre-Install Verifications

### Interface Settings

My keyboard follows the Canadian Multilingual Standard Layout.
Hence I configure
```
loadkeys ca
```
Other options include Canadian French Layout `cf`.

### Boot Mode

The command
```
cat /sys/firmware/efi/fw_platform_size
```
will return the UEFI bitness if the system is booted in UEFI mode and `No such file or directory` if the system is booted in BIOS mode.

The expected value is thus `64`. 

### Internet

First get a list of recognized network interfaces with the following command.
```
ip link
```

If the Wi-Fi card is not `UP`, run `iwctl`.
Then, the commands
```
device list
device name set-property Powered on
station name scan
station name get-networks
station name connect network-name
```
where `name` is obtained with the first command, will allow the machine to connect to the Wi-Fi. To exit, press `Ctrl+d`.

### System Clock

Finally, display the status of the internal clock to check if it is correct.
```
timedatectl
```

## Step 2: Disk Partition and Formatting

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
btrfs subvolume create /mnt/@docker
btrfs subvolume create /mnt/@media
btrfs subvolume create /mnt/@backups
umount /mnt
```
On my personal computer, `home`, `var` and `snapshots` are sufficient. The directories `docker`, `media` and `backups` are specific to my server.

### Mount the Volumes

The subvolumes created before can now be properly mounted.
```
mount -o noatime,compress=zstd,subvol=@ /dev/mapper/cryptroot /mnt

mkdir -p /mnt/{boot,home,var,.snapshots,docker,media,backups}

mount -o noatime,compress=zstd,subvol=@home /dev/mapper/cryptroot /mnt/home
mount -o noatime,compress=zstd,subvol=@var /dev/mapper/cryptroot /mnt/var
mount -o noatime,compress=zstd,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
mount -o noatime,compress=zstd,subvol=@docker /dev/mapper/cryptroot /mnt/docker
mount -o noatime,compress=zstd,subvol=@media /dev/mapper/cryptroot /mnt/media
mount -o noatime,compress=zstd,subvol=@backups /dev/mapper/cryptroot /mnt/backups
```
As before, the subset of directories realized depends on wheither this is for my server or not.

Then, the ESP can be mounted.
```
mount /dev/sda1 /mnt/boot
```

## Step 3: Install and Configure Core Packages

### Installation

Through `pacstrap`, install the Linux Kernel and essential packages.
```
pacstrap -K /mnt base linux linux-firmware btrfs-progs sudo vim grub efibootmgr networkmanager
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
vim /etc/locale.gen
```
and using the [vim](https://wiki.archlinux.org/title/Vim) text editor to uncomment the desired UTF-8 locales.
(For example, `en_CA.UTF-8 UTF-8`, `fr_CA.UTF-8 UTF-8` and `nb_NO.UTF-8 UTF-8`.)
Then, exit (`:wq`) and run
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
vim /etc/mkinitcpio.conf
```
and edit the `HOOKS=` line to `HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt btrfs filesystems fsck)`.
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
vim /etc/default/grub
```
and enable cryptodisk support in GRUB by uncommenting the line `GRUB_ENABLE_CRYPTODISK=y`. (Apparently, this is superfluous. Need to check.)

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

## Conclusion

Now that the main constituents of the system are installed and configured,
close everything properly and reboot into the freshly installed system.
```
exit
umount -R /mnt
reboot
```
Note: The username to use when loging in is `root`.

The configuration of the system is continued in [the next file](ArchLinuxConfiguration.md).