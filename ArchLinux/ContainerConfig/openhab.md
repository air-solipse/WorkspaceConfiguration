
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

