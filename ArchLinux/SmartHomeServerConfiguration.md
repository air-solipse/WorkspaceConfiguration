# Smart Home Server Configuration

First, [install Arch Linux](./OSInstall/ArchLinuxInstallation.md) on your machine.
Create the `/home`, `/var`, and `/.snapshots` directories

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

### Snapshots

Install and [configure snapper](./PackageConfig/snapper.md).

## Step 2: Packages

### Remote access through a VPN

Install and [configure WireGuard](./PackageConfig/wireguard.md).

### Remote terminal access

Install and [configure openSSH](./PackageConfig/openssh.md).

### Firewall

Install and [configure nftables](./PackageConfig/nftables.md).

### OpenHAB container

Install Docker and configure an [OpenHAB container](./ContainerConfig/openhab.md).

### Automated Backups

Install and [configure Borg](./PackageConfig/borg.md).

#
#
#
### Packages

Then, install the following packages:
```
sudo pacman -S openssh wireguard-tools nftables unbound reflector smartmontools
sudo pacman -S borg rsync curl wget git
sudo pacman -S htop tmux jq jre-openjdk-headless bind traceroute tcpdump nmap
```

Enable and disable services according to needs.
```
sudo systemctl enable sshd
sudo systemctl enable nftables
sudo systemctl enable smartd
sudo systemctl enable systemd-timesyncd
```
