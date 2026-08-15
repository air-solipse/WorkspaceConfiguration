# Smart Home Server Configuration

First, [install Arch Linux](ArchLinuxInstallation.md) on your machine.
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

## OpenHAB in Docker

### Install and configure Docker

```
sudo pacman -S docker docker-compose
sudo systemctl start docker.service
sudo systemctl enable docker.service
sudo usermod -aG docker $USER
```

Then restart the computer so that the usermod permission change applies.

```
mkdir -p /opt/openhab/{conf,userdata,addons,.java}
cd /opt/openhab
vim docker-compose.yml
```

This will create the Docker Compose configuration file.
It can then be edited with `vim` to become

```
services:
    openhab:
        image: openhab/openhab:5.1.3-debian
        container_name: openhab
        restart: unless-stopped
        network_mode: host
        environment:
            USER_ID: "1000"
            GROUP_ID: "1000"
            CRYPTO_POLICY: "unlimited"
            OPENHAB_HTTP_PORT: "8080"
            OPENHAB_HTTPS_PORT: "8443"
            EXTRA_JAVA_OPTS: "-Xms256m -Xmx512m"
            LC_ALL: "fr_CA.UTF-8"
            LANG: "fr_CA.UTF-8"
            LANGUAGE: "fr_CA.UTF-8"
        volumes:
            - /etc/localtime:/etc/localtime:ro
            - /etc/timezone:/etc/timezone:ro
            - openhab_conf:/openhab/conf
            - openhab_userdata:/openhab/userdata
            - openhab_addons:/openhab/addons
            - openhab_java:/openhab/.java
volumes:
    openhab_conf:
    openhab_userdata:
    openhab_addons:
    openhab_java:
```

The stack is started by the command:

```
docker compose up -d
```

