---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 04:03 PM
---
# Server Side

Install the NFS server
```
sudo apt install nfs-kernel-server
```

- The ZFS setup DOES NOT use `/etc/exports` for setting things up. Instead you use commands:

```
sudo zfs set sharenfs=on Deep-13
```
OR
```
sudo zfs set sharenfs="rw=@192.168.40.110,no_all_squash,no_subtree_check,async" Deep-13
```

To share with multiple IPs:
```
sudo zfs set sharenfs="rw=@192.168.40.110,rw=@192.168.70.170,no_all_squash,no_subtree_check,async" Deep-13
```

Check on share(s)
```
cat /etc/exports.d/zfs.exports
```
AND
```
sudo zfs get sharenfs Deep-13
```



# Client Side

```
sudo apt install nfs-common

sudo mkdir /mnt/zpool

sudo chmod 775 /mnt/zpool
```

To temporarily mount the share:
```
sudo mount -t nfs 192.168.10.10:/mnt/Deep-13 /mnt/zpool
```

To mount the share on boot:
```
sudo nano /etc/fstab
```

Paste this to the bottom:
```
# Mount zpool from the host (Crow)
192.168.10.10:/mnt/Deep-13  /mnt/zpool  nfs  auto  0  0
```

- Create all of the same users and groups that are part of the share so that everything works correctly and you don't mess up any permissions on the host.



# Updating Shares

Refresh the shares after editing `/etc/exports`
```
sudo exportfs -r
```

Verify shares
```
sudo exportfs -v
```


```
sudo systemctl status nfs-server.service
```



# NFSv4 Only
I not sure if I am using this or not. I remember editing the `nfs.conf` file but I can't remember if I ran the services script below or not.

`sudo nano /etc/nfs.conf`

```
[nfsd]
 vers2=n
 vers3=n
 vers4=y
 vers4.0=y
 vers4.1=y
 vers4.2=y
```

Disable services that aren't needed for NFSv4
```
sudo systemctl mask --now rpc-statd.service rpcbind.service rpcbind.socket
```
