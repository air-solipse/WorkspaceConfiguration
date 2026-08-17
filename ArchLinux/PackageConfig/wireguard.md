# WireGuard Configuration

First, install WireGuard's command-line tools and `qrencode` (the latter is used by `add-peer` to generate QR codes).

```
sudo pacman -S wireguard-tools qrencode
````

## 1. Initial Server Configuration

Create the required directories with restrictive permissions.
```
sudo install -d -m 700 /etc/wireguard
sudo install -d -m 700 /etc/wireguard/peers
```
Generate the server's private key.
```
sudo sh -c 'umask 077; wg genkey > /etc/wireguard/server.key'
```
Create the initial WireGuard configuration file for the server.
```
sudo vim /etc/wireguard/wg0.conf
```
At this stage, the configuration contains only the server interface. Peer sections will be added later by `add-peer`.
```
[Interface]
Address = 172.28.18.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
```
Replace `SERVER_PRIVATE_KEY` with the contents of `/etc/wireguard/server.key`. The private key can be displayed with:
```
sudo cat /etc/wireguard/server.key
```

Do not share the private key with anyone.

Then, set the permissions of the configuration file and bring up the WireGuard interface.
```
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable --now wg-quick@wg0
```

## 2. Configure the router

Forward incoming UDP traffic on port 51820 to the Arch server's LAN IPv4 address on UDP port 51820. For example,
```
UDP 51820 → 192.168.1.50:51820
```
Ensure that the Arch server has a stable LAN IP address, preferably through a DHCP reservation on the router.

This configuration is intended to allow WireGuard clients to access the home network through the Arch server.

## 3. Add Peers to the network

### Install the `add-peer` script

Copy the [add-peer](../scripts/add-peer) script onto a USB stick, then transfer it to the Arch computer.
Then, copy the script to the Arch computer and make it executable.

```
sudo mkdir -p /mnt/usb
sudo mount /dev/<usb-partition-name> /mnt/usb

sudo cp /mnt/usb/add-peer /usr/local/bin/
sudo chmod +x /usr/local/bin/add-peer

sync
sudo umount /mnt/usb
sudo rmdir /mnt/usb
```
Replace `/dev/<usb-partition-name>` with the USB partition, such as `/dev/sdb1`. Do not specify the entire disk, such as `/dev/sdb`.

### Run the `add-peer` script

For each peer to add, run `add-peer` with the peer's name and the [public hostname or IP address](https://datatracker.ietf.org/doc/html/rfc1035/) of the WireGuard server.
```
sudo add-peer <name> <server-address>
```
The peer name may contain only letters, numbers, _, and -.

To add a mobile device, use the WireGuard app on the device and the QR code generation option of `add-peer`.

To add a computer, copy the appropriate `.conf` file to a USB stick.
It can be mounted as before. Then, run
```
sudo cp /etc/wireguard/peers/<name>.conf /mnt/usb/
```
It can also be unmounted as before.
Once the peer has been configured, it is recommended to remove the `.conf` file from the server.
```
sudo rm /etc/wireguard/peers/<name>.conf
```
