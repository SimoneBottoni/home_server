# System Setup

## Install Debian and XFCE4
```shell
sudo apt install xfce4 xfce4-goodies
```

Add user to sudoers
```shell
su root
```

Modify `/etc/sudoers`
```shell
# Append
ron ALL=(ALL) ALL
```


## Prepare the external disk
```shell
# List available disks using:
lsblk

# Suppose your storage drive is /dev/sda, you need to partition and format it:
sudo parted /dev/sda -- mklabel gpt
sudo parted /dev/sda -- mkpart primary ext4 1MiB 100%

# Format the partition:
sudo mkfs.ext4 /dev/sda1

# Mount the partition:
sudo mkdir -p /mnt/nas
sudo mount /dev/sda1 /mnt/nas

# To make this mount permanent, edit /etc/fstab:
echo 'UUID=<UUID> /mnt/nas ext4 defaults,nofail,x-systemd.automount,x-systemd.mount-timeout=60,x-systemd.idle-timeout=60 0 2' | sudo tee -a /etc/fstab

# Edit permissions
sudo chown -R $USER:$USER /mnt/nas
```

[Source](https://www.siberoloji.com/how-to-set-up-network-attached-storage-nas-in-debian-12-bookworm-system/)

### Set the spindowntime for the disk
```shell
sudo hdparm -S 60 /dev/sda1
```

Make it permanent modifying `/etc/hdparm.conf`
```shell
# Append
/dev/sda1 {
    spindown_time = 60
}
```
[Source](https://guide.debianizzati.org/index.php/Hdparm)

## Install XRDP
```shell
sudo apt install xrdp
sudo systemctl enable xrdp
```

Modify `/etc/xrdp/startwm.sh`
```shell
# Comment
test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession

# Append
startxfce4
```

Restart the xrdp service:
```shell
sudo systemctl restart xrdp
```

Open firewall ports 3389/tcp


[Source](https://phoenixnap.com/kb/debian-remote-desktop)

# Firewall
```shell
sudo apt install firewalld
sudo systemctl enable firewalld
```

Commands
```shell
# Open a port
sudo firewall-cmd --permanent --add-port=7878/tcp

# Reload the firewall
sudo firewall-cmd --reload

# Check if ports are open
sudo firewall-cmd --list-ports
```

[Source](https://phoenixnap.com/kb/debian-remote-desktop)

## Install Samba
```shell
sudo apt install samba
```

Modify `/etc/samba/smb.conf`
```shell
# Append

[Shared]
   path = /mnt/nas
   browseable = yes
   writable = yes
   guest ok = no
   valid users = @smbusers
```

Add the user to @smbusers
```shell
sudo groupadd smbusers
sudo usermod -aG smbusers $USER
sudo smbpasswd -a $USER
```

Restart samba
```shell
sudo systemctl restart smbd
```

Open firewall ports 445/tcp and 139/tcp

[Source](https://www.siberoloji.com/how-to-set-up-network-attached-storage-nas-in-debian-12-bookworm-system/)


## Docker Compose

### Installation
Set up Docker's apt repository
```shell
# Add Docker's official GPG key:
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Install the latest version
```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

[Source](https://docs.docker.com/engine/install/debian/)

### Post installation
Create the docker group
```shell
sudo groupadd docker
```
Add your user to the docker group
```shell
sudo usermod -aG docker $USER
```
Activate the changes to groups
```shell
newgrp docker
```

[Source](https://docs.docker.com/engine/install/linux-postinstall)

---

# Applications
[General Source](https://trash-guide.info/)

| App | Purpose | Proxy target |
|---|---|---|
| [Pi-hole](https://github.com/pi-hole/docker-pi-hole) | Local DNS and ad blocking | `pihole:80` |
| [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager) | Reverse proxy for `*.ron.home` | admin on port 81 |
| [Homepage](https://gethomepage.dev/) | Dashboard | `homepage:3000` |
| [Portainer](https://docs.portainer.io/) | Container management | `portainer:9000` |
| [Home Assistant](https://www.home-assistant.io/) | Home automation | `<server IP>:8123` |
| [Jellyfin](https://docs.linuxserver.io/images/docker-jellyfin/) | Media server | `jellyfin:8096` |
| [Prowlarr](https://docs.linuxserver.io/images/docker-prowlarr) | Indexer manager | `prowlarr:9696` |
| [Sonarr](https://docs.linuxserver.io/images/docker-sonarr/) | TV shows | `sonarr:8989` |
| [Sonarr Anime Downloader](https://github.com/MainKronos/Sonarr-AnimeDownloader) | Anime downloads for Sonarr | `sonarr_anime:5000` |
| [Radarr](https://docs.linuxserver.io/images/docker-radarr) | Movies | `radarr:7878` |
| [Bazarr](https://docs.linuxserver.io/images/docker-bazarr/) | Subtitles | `bazarr:6767` |
| [qBittorrent](https://docs.linuxserver.io/images/docker-qbittorrent/) | Download client | `qbittorrent:8088` |
| [Tailscale](https://tailscale.com/docs/features/containers/docker) | Remote access | - |
| [Watchtower](https://watchtower.nickfedor.com/) | Monthly image updates | - |
| [Backup](https://offen.github.io/docker-volume-backup/) | Weekly config backup to the NAS | - |
