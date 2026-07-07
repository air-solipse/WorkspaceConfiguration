# Server Configuration

First, [install Arch Linux](ArchLinuxInstallation.md) on your machine.

## Step 1: Preliminaries

### Admin User

Create an administrator user and set its password
```
useradd -m -G wheel admin
passwd admin
```
Add the `wheel` group to the sudoers by running
```
EDITOR=vim visudo
```
and uncommenting the line `%wheel ALL=(ALL:ALL) ALL`

Once sudo access is given to the `admin` user, lock the password of the `root` user.
```
passwd -l root
```

### Packages

Then, install the following packages:
```
pacman -S openssh nftables wireguard-tools smartmontools reflector borg git curl wget rsync unbound
pacman -S docker docker-compose
pacman -S samba ffmpeg beets picard lm_sensors
```

There might arise the need to reconnect to Wi-Fi. Use
```
nmcli device wifi list
nmcli device wifi connect name password your_password
```
where `name` is given by the first command.

### Directories

Create relevant subdirectories
```
mkdir /media/incoming
mkdir /media/staging
mkdir /media/library/music
mkdir /media/library/movies
mkdir /media/library/tv
mkdir /docker/jellyfin
mkdir /docker/caddy
```

## Step 2: Secure Shell

Configure SSH and WireGuard, then restrict SSH to only WireGuard.
\[...\]

## Step 3: 

Configure nftables, local DNS.

## Step 4: 

Configure docker, Jellyfin and Caddy (+ website)

## Step 5:

SMB and SFTP, samba config

## Step 6:

Configure Borg, SMART, BTRFS scrub timer, reflector timer, and other maintenance-type things.