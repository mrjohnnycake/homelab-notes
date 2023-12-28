---
created: Mon 2023-05-08 @ 06:13 PM
modified: Wed 2023-12-27 @ 03:51 PM
---
- Go [here](http://urbackup.org/download.html) and copy the link address for the download you need

```
wget https://hndl.urbackup.org/Server/2.5.31/debian/bullseye/urbackup-server_2.5.31_amd64.deb

sudo apt update

sudo dpkg -i urbackup-server*.deb
```

- Hit enter at the install
- That should be it but the install instructions say to run `sudo apt-get -f install` now but it didn't do anything for me
- Go to http://192.168.10.8:55414/
- First thing, go to Settings->Users->Create User. The first time going here will prompt to create a password for `admin`




/media/BACKUP/urbackup




```
sudo zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Backups/Windows Backups/Windows
```


```
sudo mkdir /mnt/Backups/Windows/Strong-Bad
```


```
sudo chown -R urbackup:urbackup /mnt/Backups/Windows

sudo chmod -R 770 /mnt/Backups/Windows

sudo usermod -aG urbackup administrator
```





```
C:\Users\mrjoh\Desktop\*;C:\Users\mrjoh\Downloads\*;C:\Users\mrjoh\Dropbox\*;C:\Users\mrjoh\Pictures\*
```