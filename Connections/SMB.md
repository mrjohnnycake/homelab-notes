---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 04:05 PM
---
# Creating Samba Shares #

This is for creating a SMB share to connect to in Windows

* this section is incomplete. finish is next time you need to create a share.

```
sudo nano /etc/samba/smb.conf
```




# Mounting Samba Shares #

This is for attaching an existing external SMB share to the VM

- In the VM CLI

```
sudo apt install smbclient cifs-utils
```

Test connection
```
smbclient '\\192.168.10.5\Deep_13_Media_Audio' -U dave
```

As long as that was able to connect, do this to automount at startup
```
sudo nano /root/.smb
```

```
user=dave
password=enter-password-here
```
* Save and exit

```
sudo nano /etc/fstab
```

Add this to the bottom:
```
//192.168.10.5/Deep_13_NAS /mnt/NAS cifs credentials=/root/.smb,uid=1010,gid=1020 0 0
```

For OVMF (UEFI) VMs, if it's not working, add `noauto,x-systemd.automount` like this:
```
//192.168.10.10/Deep_13_Media /mnt/Media/ cifs credentials=/root/.smb,uid=1010,gid=1000,noauto,x-systemd.automount 0 0
```


# Unsharing

```
sudo zfs set sharesmb=off Deep-13/Media
```



# Listing Shares

```
sudo zfs get sharesmb Deep-13
```
