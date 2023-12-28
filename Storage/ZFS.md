https://wiki.debian.org/ZFS

## Pool and Datasets ##
* In the Proxmox GUI, wipe the disks that are to be used

* Create the pool:

```
zpool create -o ashift=12 -m /mnt/Deep-13 Deep-13 raidz /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD4AL2Z /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7101N /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD6YG1K
```

* Set pool options

```
zfs set compression=lz4 xattr=sa dnodesize=auto Deep-13
```

* Create Media datasets and all associated datasets

```
zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media Deep-13/Media

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/audio Deep-13/Media/Audio

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/books Deep-13/Media/Books

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/downloaders Deep-13/Media/Downloaders

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/video Deep-13/Media/Video
```

* Create associated folders

```
mkdir /mnt/Deep-13/media/audio/music
mkdir /mnt/Deep-13/media/audio/other
mkdir /mnt/Deep-13/media/books/audiobooks
mkdir /mnt/Deep-13/media/books/ebooks
mkdir /mnt/Deep-13/media/downloaders/metatube
mkdir /mnt/Deep-13/media/downloaders/torrents
mkdir /mnt/Deep-13/media/downloaders/usenet
mkdir /mnt/Deep-13/media/video/movies
mkdir /mnt/Deep-13/media/video/tv-shows
```


# Sharing
## NFS

- See the [[NFS]] note


## SMB/CIFS ##

```
apt install samba
systemctl enable smbd
```

```
zfs set sharesmb=on Deep-13/Media
zfs set sharesmb=on Deep-13/Media/Audio
zfs set sharesmb=on Deep-13/Media/Books
zfs set sharesmb=on Deep-13/Media/Downloaders
zfs set sharesmb=on Deep-13/Media/Video
```

```
zfs share Deep-13/Media
zfs share Deep-13/Media/Audio
zfs share Deep-13/Media/Books
zfs share Deep-13/Media/Downloaders
zfs share Deep-13/Media/Video
```


Create a SMB user
```
adduser dave
usermod -aG sambashare dave
smbpasswd -a dave
```

# Permissions

Set permissions for the share

```
chown -R dave:media /mnt/Deep-13/media
```


# Windows Connection

Restart samba to get everything working

```
service smbd restart
```

* Open File Explorer and go to the Network tab
* In the location bar where it says Network, paste this:
	* This is for sharing my Zpool/Dataset/Dataset of Deep-13/NAS/Dave

```
\\192.168.10.10\Deep_13_NAS_Dave
```



# Exporting / Importing

This is for moving physical drives to a new server or if you just need to unplug the drives for a while for whatever reason AND you don't want to destroy the pool.

- First, turn off "Start on boot" for all VMs and containers in Proxmox
- Second, reboot the server
- After reboot run this command:
```
sudo zpool export -f Deep-13
```

```
zpool import -R /mnt/Deep-13 Deep-13
```
*You will likely receive an error about the dataset not sharing correctly. Fix that later by setting the smb shares again*


If you don't use the mountpoint option in the command above it messes up the mountpoints pretty good and you have to run these commands for all of datasets

```
zfs set mountpoint=/mnt/Deep-13 Deep-13

zfs set mountpoint=/mnt/Deep-13/Backups Deep-13/Backups

zfs set mountpoint=/mnt/Deep-13/Backups/Crow Deep-13/Backups/Crow
```

If you have trouble setting the mountpoint you may need to unmount the dataset first:
```
zfs unmount -f Deep-13/Backups

zfs unmount -f Deep-13/Backups/Crow

zfs unmount -f Deep-13/Backups/Tom-Servo
```



# Maintenance #

## Replacing Failed Drives ##

- Check which disk is unavailable
```
zpool status -v
```

- In this last instance, the faulted disk was WX52D32D5L5D

- Take the faulted disk offine
```
zpool offline Deep-13 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX52D32D5L5D-part1
```

- Shutdown the server and replace the faulted drive with the new one
	- Take note of the new disk's serial number. In this case, the new disk is WX72D32NCU7L

- Start the server and log back into the CLI

- Get the new disk location
```
ls /dev/disk/by-id
```

- Run this to replace the failed drive (the first disk listed in the command) with the new disk (the second disk listed in the command)
```
zpool replace Deep-13 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX52D32D5L5D-part1 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX72D32NCU7L
```

- Let it go thru the resilvering process (about the same amount of time as a scrub)


## Managing Snapshots ##

- List all snapshots
```
zfs list -t snapshot
```

Delete a snapshot (edit end of command with exact snapshot name)
```
zfs destroy Deep-13/Backups@zfs...
```

Destroy all snapshots from a particular dataset
```
sudo -i

zfs list -H -o name -t snapshot Deep-13/Media/Video | xargs -n1 zfs destroy
```


Destroy all snapshots from entire pool
```
sudo -i

zfs list -H -o name -t snapshot | xargs -n1 zfs destroy
```