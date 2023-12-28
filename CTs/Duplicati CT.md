---
created: Tue 2023-05-02 @ 06:32 PM
modified: Thu 2023-12-28 @ 08:48 AM
---
Create a basic Ubuntu CT


# Setup Container #

```
lxc-attach --name 301

apt update

dpkg-reconfigure locales

apt install unattended-upgrades gnupg

dpkg-reconfigure --priority=low unattended-upgrades

apt upgrade
```

Add users
```
useradd -u 1000 -M administrator
useradd -u 1010 -M dave
useradd -u 1011 -M emmett
```

Add groups
```
groupadd -g 1020 nas
groupadd -g 1021 plex-movies
groupadd -g 1022 plex-music
groupadd -g 1023 plex-shows
groupadd -g 1024 books
groupadd -g 1025 photos
```



# Install Duplicati #

```
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF

echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list

apt update

sudo apt install mono-devel gtk-sharp2 libappindicator0.1-cil libmono-2.0-1 apt-transport-https git software-properties-common -y

wget https://updates.duplicati.com/beta/duplicati_2.0.6.3-1_all.deb

apt install ./duplicati_2.0.6.3-1_all.deb
```

```
nano /etc/systemd/system/duplicati.service
```

```
[Unit]
Description=Duplicati web-server
After=network.target

[Service]
Nice=19
IOSchedulingClass=idle
EnvironmentFile=-/etc/default/duplicati
ExecStart=/usr/bin/duplicati-server $DAEMON_OPTS
Restart=always

[Install]
WantedBy=multi-user.target
```

```
nano /etc/default/duplicati
```

```
# Defaults for duplicati initscript
# sourced by /etc/init.d/duplicati
# installed at /etc/default/duplicati by the maintainer scripts
#
# This is a POSIX shell fragment
#
# Additional options that are passed to the Daemon.
DAEMON_OPTS="--webservice-interface=any --webservice-port=8200 --portable-mode"
```

```
systemctl enable duplicati.service
systemctl daemon-reload
systemctl start duplicati.service
systemctl status duplicati.service
```

```
mkdir /mnt/Backups
mkdir /mnt/Games
mkdir /mnt/Media
mkdir /mnt/Websites
```

```
chown administrator:administrator /mnt/Backups
chown dave:administrator /mnt/Games
chown dave:administrator /mnt/Media
chown dave:administrator /mnt/Websites
```

Shutdown


# Create Mountpoint #

From Proxmox:
```
nano /etc/pve/lxc/301.conf
```

* Add this to the bottom:
```
mp0: /mnt/Deep-13/Backups/Crow,mp=/mnt/Backups/Crow
mp1: /mnt/Deep-13/Backups/Tom-Servo,mp=/mnt/Backups/Tom-Servo
mp2: /mnt/Deep-13/Emmett,mp=/mnt/Emmett
mp3: /mnt/Deep-13/Games/DnD,mp=/mnt/Games/DnD
mp4: /mnt/Deep-13/Games/Emulators,mp=/mnt/Games/Emulators
mp5: /mnt/Deep-13/Games/Minecraft,mp=/mnt/Games/Minecraft
mp6: /mnt/Deep-13/Media/Audio,mp=/mnt/Media/Audio
mp7: /mnt/Deep-13/Media/Books,mp=/mnt/Media/Books
mp8: /mnt/Deep-13/Media/Photos,mp=/mnt/Media/Photos
mp9: /mnt/Deep-13/Media/Video,mp=/mnt/Media/Video
mp10: /mnt/Deep-13/NAS,mp=/mnt/NAS
mp11: /mnt/Deep-13/Websites/Blog,mp=/mnt/Websites/Blog
mp12: /mnt/Deep-13/Websites/Daycare,mp=/mnt/Websites/Daycare
mp13: /mnt/Deep-13/Websites/Offline,mp=/mnt/Websites/Offline
```


# Duplicati Settings #

Go to http://192.168.40.115:8200/ or whatever the IP is

Upon first visit, you'll be prompted to set the password

After setting the password, log back in and go back to Settings

- change "Display and color theme" to the dark theme

Click OK


# Creating Backups #

```
Select "Configure a new backup"
Name: Websites/Blog or whatever the dataset name is
Enter and repeat the passphrase
Storage Type: S3 Compatible
Server: Custom server url ()
Custom S3 endpoint: 192.168.10.3:9000 (or whatever the IP is)
Bucket name: websites-blog (adjust based on Name of this backup)
Bucket create region: Custom region value ()
Custom region: gizmonics-institute
AWS Access ID: in MinIO entry in 1P
AWS Access Key: in MinIO entry in 1P
Client library: Minio SDK
Test connection and select No on renaming and Yes on folder creation
Source data: /mnt/Websites/Blog
Set schedule as needed
Backup retention: Keep a specific number of backups
Backup retention #: 1
```

Save


