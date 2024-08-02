---
created: Tue 2023-05-02 @ 06:31 PM
modified: Fri 2024-01-12 @ 06:23 PM
tags:
  - Homelabbing
sticker: emoji//1f4c3
---
https://github.com/xcad2k/cheat-sheets


## SSH


SSH Into Proxmox Containers

```
lxc-attach --name 100 (change based on container number)
```


## SMB / Samba ##

### ZFS Specific ##

Install Samba
```
apt update
apt install samba
systemctl enable smbd
```

^c32f3f

Create a SMB user
```
adduser dave-deepape
usermod -aG sambashare dave-deepape
smbpasswd -a dave-deepape
```

Create a new dataset to share
```
zfs create -o mountpoint=/mnt/shares/nas Deep13/nas
```

Turn SMB sharing on in ZFS


* Share specifics datasets one at a time:
```
zfs set sharesmb=on sharename=gypsy-backups Deep-Ape/Backups
```

Set permissions for the share
```
chown -R dave-deepape:dave-deepape /mnt/deep-ape/media
```

Restart samba to get everything working
```
service smbd restart
```

* Reboot Proxmox node via the GUI

Connect to share on Windows

* Open File Explorer and go to the Network tab
* In the location bar where it says Network, paste this:
```
\\192.168.2.11\nas
```


Troubleshooting

Get a list of which datasets are shared
```
zfs get sharesmb
```





nano /etc/samba/smb.conf




## ZFS ##

* See [[ZFS]] note as well as below

List all ZFS samba shares:
```
sudo zfs get sharesmb
```


Clean up below:


zpool create -o ashift=12 -m /mnt/deep-ape Deep-Ape raidz /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD4AL2Z /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7101N /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD6YG1K


zfs create -o mountpoint=/mnt/deep-ape/backups Deep-Ape/Backups



zfs set compression=lz4 xattr=sa dnodesize=auto Deep-Ape


zfs destroy Deep-Ape/Media/Downloaders

zpool destroy deep13




# Terminal #

=======================================

## Path


Change path
```
cd [directory]
```
Go out one folder
```
cd ..
```
Show the full path you are at
```
pwd
```
Show the folder structure of your path
```
tree
```

<br>

***

## Text Files ##

Select default editor
```
sudo update-alternatives --config editor
```

Show the contents of a file
```
cat file.txt
```
Create a file w/o editting it
```
touch file.txt
```
Edit a file
```
nano file.txt
```

<br>

***

## Copy, Move, Delete ##

Move file or folder
```
mv /origin /destination
```

For monitoring mv operations
```
apt install progress

start the operation, open another ssh window for the same server and enter:

progress -mw
```

Move a file to a different server
```
scp [file_name.txt] [server/tmp]
```
Delete a file
```
rm [FILE]
```
Delete a folder
```
rm -r [FOLDER]
```

<br>

***

## Power and restart ##

```
reboot
poweroff
shutdown now
```



## Special Characters

See examples of how to do all allowed special characters [here](https://www.tecmint.com/special-character-filenames-linux/)



***

## Networking ##

```
ip a
ip r
traceroute
```

See what DNS servers your system is configured to use
```
cat /etc/resolv.conf	
```

See system hostname:
```
hostname
```

Show the IP address of the system:
```
hostname -i 
```

See active (listening) ports with the netstat command:
```
netstat -pnltu
```

Show tcp and udp ports and their programs:
```
netstat -nutlp
```

Lookup hostname:
```
nslookup 192.168.2.11
```



## Users and Groups

```
useradd -M <username>
```

List current users

```
less /etc/passwd
```

Adding new user accounts
```
adduser [user_name]
```

Modify a user account
```
usermod
```

Add a user to a group:
```
usermod -aG [group_name] [user_name]
```

List current user's groups
```
groups
```

List all groups:
```
less /etc/group
```

List all users in a group
```
getent group [group]
```

Delete a user account
```
deluser
```

Delete a group
```
delgroup	
```

Add a new group
```
groupadd [group_name]
```

List logged in user w/ PID
```
who -u
```

Logs out the user
```
kill [PID number]
```



## Downloads ##

Download a file from a domain using the wget command:
```
wget [file_name]
```

Save a remote file to your system using the filename that corresponds to the filename on the server:
```
curl -O [file-url]
```



## Run Scripts ##

Run shell script (you can remove the .sh if Linux lists it without)

```
./scriptname.sh
```


## Copy ##

```
cp -R /etc /etc_backup
```

Backup one folder to another using rsync (same as copy but will mirror the folder)

```
rsync --info=progress2 -auvz /origin-directory /destination-directory/
```


## Permissions ##

Change file owner to user and group to group
```
chown USER:GROUP FILE
```

Change mode of file to 775
```
chmod 775 file
```

Recurs­ively chmod folder to 600
```
chmod -R 600 folder
```



## SSH ##

```
ssh dave@linserv
```


## Search ##

List total number of files and directories (doesn't differentiate)
```
ls | wc -l
```

Get the size of a directory
```
du -sh /var
```


=======================================


![[Cheatsheet#ZFS]]

I can also create a [Reference Link][ID1] by noting the ID and then put the links at the end of the page later like this...

[ID1]:http://url.com "Reference Definition"

Footnote^[This is a footnote that will be placed at the bottom of this note in Reading view]


Link to an Obsidian note:
[[Cheatsheet]]

Link to an Obsidian note but call it something different:
[[Homelab|stuff]]


Link to a header somewhere on the same page or a different note:
[[Cheatsheet#Path]]


[[Homelab#Next Up]]

Link to blocks in a page
[[Cheatsheet#^c32f3f |Install Samba]]






## Tags ##

Use a hash followed by the word you want (ie. `#tag`)




## Images ##

![Image](./image.jpg "Title")



## Tables ##

Dimensions | Megapixels
---|---
1,920 x 1,080 | 2.1MP
3,264 x 2,448 | 8MP
4,288 x 3,216 | 14MP

<br>
<br>

| foo | bar |
| --- | --- |
| baz | bim |




## Blockquotes ##

> Blockquote [foo].


> [foo]: /url

<br>

   > > 1.  one
>>
>>     two



## Invisible Text ##

<!-- Invisible text -->


# Ubuntu / Server #

## Networking ##

Host name files

```
/etc/hosts
/etc/hostname
```


# Docker #

List containers
```
docker container ls
```



## Container Management ##

Delete container
```
docker stop /container-name

docker rm /container-name
```



# Crontab #

First, start out by getting the system time as that will likely be different than your (the user's) time
```
timedatectl
```

Using this as an example:
```
0 4 * * * /home/administrator/scripts/backup_to_Crow.sh

MIN HOUR DAY-OF-MONTH MONTH DAY-OF-WEEK /home/administrator/scripts/backup_to_Crow.sh

So that script would run every day at 4a (relative to system time)
```


### Format ###

Minute: 0-59
Hour: 0-23
Day of the month: 1-31
Month: 1-12 (or JAN to DEC)
Day of the week: 0-6 (or SUN to SAT; or 7 for Sunday)


### Special characters

- A field can contain an asterisk (`*`), which always stands for "first-last".
- Ranges are two numbers separated with a hyphen (`-`) and the specified range is inclusive.
- Following a range with `/NUMBER` specifies skips of the number's value through the range. For example, both `0-23/2` and `*/2` can be used in the `Hour` field to specify execution every two hours.
- A list is a set of numbers (or ranges) separated by commas (`,`). For example, `1,2,5,6` in the `Day of the month` field specifies an execution on the first, second, fifth, and sixth days of the month.

