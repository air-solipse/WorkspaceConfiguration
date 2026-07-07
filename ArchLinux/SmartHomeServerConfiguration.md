# Smart Home Server Configuration

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
sudo pacman -S openssh wireguard-tools nftables unbound reflector smartmontools
sudo pacman -S borg rsync curl wget git
sudo pacman -S htop tmux jq jre-openjdk-headless bind traceroute tcpdump nmap
sudo pacman -S docker docker-compose
```

Enable and disable services according to needs.
```
sudo systemctl enable sshd
sudo systemctl enable nftables
sudo systemctl enable smartd
sudo systemctl enable systemd-timesyncd
sudo systemctl disable docker
```

