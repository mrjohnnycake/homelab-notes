You've read it and heard it already but I cannot stress it enough - BACKUP YOUR SERVERS!

Now that's all well and good but what you should backup and how to do it can take a very long time to figure out. This guide is meant as a quick go-to to help you narrow down the scope of what you need to do.

This is a layout of how I backup everything on my servers. Your setup will most certainly be different than mine but hopefully this will help you start narrowing down what to do.


# What to Backup #

Let's put in basic English what needs to be backed up on one of my Proxmox servers.

Proxmox is Linux-based so it has the normal Linux file structure that you may already be used to. Then, I have LXC containers and virtual machines installed that run my various services (Plex, Pi-hole, etc.). I also run a NAS on the same machine and use ZFS as a storage system.

Let's start with Proxmox since It's the most top-level OS and then we'll work down from there.

#### Files ####
- Although some editing to the system is done through the Proxmox GUI, I want to make sure I backup all of those files that have been edited so that, in the case of a server failure, I can have that information ready to go instead of looking up how to do it all over again.
	- Examples:
		- A lot of files in `/etc` 
		- CT and VM backups (Proxmox calls that folder `dump`)
		- Data backup (see "Backup To Crow" below)
		- Fix permissions every once in a while
		- Zpool maintenance
			- Scrubs
			- Snapshots
			- Error and weekly report notifications

# Schedule and Explanation

I'm currently running my servers in UTC time (though I plan to change that in the future to my time zone). 4pm my time is the same as midnight on UTC

The schedules below are in this format:

- Time allowed for the backup to run
- Can it run while other jobs are running or does it need to happen first and then allow another backup to run?
- How is it ran?
- When will it run?
- Details followed by examples of the particular backup (if needed)


## Every Day except Sundays ##

#### VM to Host ####
- 15 min.
- Can run concurrently on all servers
- Cron jobs running shell scripts
- See schedules below
- Details
	- Media VM @ Tom-Servo: 1:55am - 2:10am (55 ? UTC)
	- Websites VM @ Crow: 1:55am - 2:10am (55 ? UTC)
	- Plex / GPU VM @ Tom-Servo: 2:10am - 2:25am (10 ? UTC)

#### Docker Watchtower ####
- 10 min.
- Can run concurrently on all Portainer instances
- Docker container
- 2:25am - 2:35am (25 ? UTC)
- Details
	- Media (2:25a / 25 ? UTC)
	- NAS (2:25a / 25 ? UTC)
	- Proxy (2:25a / 25 ? UTC)
	- Websites (2:25a / 25 ? UTC)

#### Tom-Servo to Crow ####
- 1 hr.
- CANNOT run concurrently
- Cron job running a shell script
- 2:35am - 3:35am (35 ? UTC)

#### Crow to Tom-Servo ####
- 15 min.
- Runs concurrently w/ PMM
- Cron job running a shell script
- 3:35am - 3:50am (35 ? UTC)

#### Plex Meta Manager ####
- 15 min.
- Runs concurrently w/ "Crow to Tom Servo"
- Docker automated container
- 3:35am - 3:50am (35 ? UTC)
- Comes programmed by default to run every day at 5am (override)

#### Permissions fix ####
- 5 min.
- Can run concurrently on both main servers and any VM
- Cron jobs running shell scripts
- See schedules below
- Detail
	- Tom Servo media files (3:50a / 50 ? UTC)
	- Crow NAS files (3:50a / 50 ? UTC)
	
#### ZFS Snapshots (5 min.) ####
- Instant
- Can run concurrently on all servers
- Runs via zfs-auto-snapshot
- 3:55am - 4:00am (55 ? UTC)
- See "ZFS" note for more details

#### Restic (54 min.) ####
- 1 hr. (actually 54 min total)
- 5 min per backup, 1 min per forget
- All "backups" run consecutively followed by all "forgets"
- See schedules below
- Details
	- "backup" jobs
		- backups-crow.backup (4:00a / 0 ? UTC)
		- backups-tomservo.backup (4:05a / 5 ? UTC)
		- media-audio.backup (4:10a / 10 ? UTC)
		- media-books.backup (4:15a / 15 ? UTC)
		- media-games.backup (4:20a / 20 ? UTC)
		- media-photos.backup (4:25a / 25 ? UTC)
		- nas-anna.backup (4:30a / 30 ? UTC)
		- nas-dave.backup (4:35a / 35 ? UTC)
		- nas-emmett.backup (4:40a / 40 ? UTC)
	- "forget" jobs
		- backups-crow.forget (4:50a / 50 ? UTC)
		- backups-tomservo.forget (4:51a / 51 ? UTC)
		- media-audio.forget (4:52a / 52 ? UTC)
		- media-books.forget (4:53a / 53 ? UTC)
		- media-games.forget (4:54a / 54 ? UTC)
		- media-photos.forget (4:55a / 55 ? UTC)
		- nas-anna.forget (4:56a / 56 ? UTC)
		- nas-dave.forget (4:57a / 57 ? UTC)
		- nas-emmett.forget (4:58a / 58 ? UTC)

#### Rsync (movies & shows) ####
- Open ended (no time frame)
- Runs movies until finished followed by show until finished
- Cron job running a shell script
- 5:00am (0 ? UTC)



## Sundays Only #

#### VM Updates ####
- 15 mins
- Can run concurrently ON ALL VMs
- Cron job running shell scripts
- Every Sunday @ 11am (7am UTC)

#### Server Updates ####
- 15 mins
- Can run concurrently ON ALL SERVERS
- Cron job running shell scripts
- Every Sunday @ 11:15am (7:15am UTC)

#### Proxmox dumps ####
- 30 min.
- Can run concurrently ON ALL SERVERS
- Proxmox internal backup schedule via GUI
- Every Sunday @ 11:30am (7:30am UTC)

#### Zpool Scrubs ####
- 12+ hours
- Can run concurrently ON ALL SERVERS
- Ran thru ZFS
- Every 2nd Sunday of every month @ ~ midnight (8 UTC)
- Check `cat /etc/cron.d/zfsutils-linux` for specifics


## Automatic Pool Notifications ##
- Summary notification (once a week)
- Notifications for problems in the metrics
	- Long term: use metrics from InfluxDB to create a notification
- Run in the daytime



# Instructions

Make sure `cron` and `rsync` are installed

```
sudo apt install cron rsync
```

1) Create SSH key pairs

	- See my [[SSH Setup]] note on how to create key pairs

	- Assuming you want to connect to Tom-Servo FROM Media-VM, you need to create the pairs on Tom-Servo

2) Create script on VM:
```
mkdir ~/scripts

chmod 775 ~/scripts

touch scripts/backup-to-Deep13.sh

chmod 770 scripts/backup-to-Deep13.sh

vim scripts/backup-to-Deep13.sh
```
- Insert your script code (found in the specific VM section)
* Whatever your backup destination is owned by, that's the user you need to ssh into with in your script. See the note on Backups to Crow for more of an explanation.

3) Create cron job
```
sudo crontab -e
```
- needs to use `sudo`

- see examples below

* Don't run backups on Sundays to allow for pool scrubs

- Restart cron if necessary
	```
	sudo service cron restart
	```


## ZFS Snapshots w/ cron ##

To get automatic snapshots working, you need to install `zfs-auto-snapshot`. That, along with cron, will allow you to create snapshots based on timeframe AND control how many to keep.

Installation- this is what ended up working without errors where the normal install caused errors
```bash
wget [https://github.com/zfsonlinux/zfs-auto-snapshot/archive/upstream/1.2.4.tar.gz](https://github.com/zfsonlinux/zfs-auto-snapshot/archive/upstream/1.2.4.tar.gz)

tar -xzf 1.2.4.tar.gz

cd zfs-auto-snapshot-upstream-1.2.4

make install
```

Turn on/off snapshots for datasets:
```bash
sudo zfs set com.sun:auto-snapshot=true Deep-13
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Crow
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Tom-Servo
sudo zfs set com.sun:auto-snapshot=false Deep-13/Media
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Audio
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Books
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Games
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Photos
sudo zfs set com.sun:auto-snapshot=false Deep-13/Media/Video
sudo zfs set com.sun:auto-snapshot=false Deep-13/NAS
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Anna
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Dave
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Emmett
```

sudo vim /etc/cron.d/zfs-auto-snapshot
```bash
PATH="/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"

*/5 * * * * root which zfs-auto-snapshot > /dev/null || exit 0 ; zfs-auto-snapshot --quiet --syslog --label=frequent --keep=24 //
```

sudo vim /etc/cron.hourly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=hourly --keep=24 //
```

sudo vim /etc/cron.daily/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=daily --keep=14 //
```

sudo vim /etc/cron.weekly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=weekly --keep=4 //
```

sudo vim /etc/cron.monthly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=monthly --keep=12 //
```

### Testing

Show all snapshots:
```
zfs list -t snapshot
```

Show system logs for cron jobs (to verify that jobs went thru)
```
grep CRON /var/log/syslog
```



# Local Machines#

## Tom-Servo ##
- Proxmox backs up VMs and CTs to the specified backup folder
- Everything gets backed up to the destination folder(s) on Crow and deleted files on Tom-Servo that were previously synced remain there.
- Every once in a while, I need go to the destination folder(s) and delete things that I know I no longer need.
- Everything runs through cron jobs


### Crontab ###

sudo crontab -e
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# All cron times are UTC

# My scripts
0 12 * * * /home/administrator/scripts/backup_to_Crow.sh > /dev/null
15 13 * * * /home/administrator/scripts/permissions.sh > /dev/null

# Updates- every Sunday night at 11pm
0 7 * * 0 /home/administrator/scripts/update.sh > /dev/null

# Snapshots- every five minutes
# */5 * * * * root zfs-auto-snapshot -q -g --syslog --label=frequent --keep=24 //
# Snapshots- every hour, on the hour
# 0 * * * * root zfs-auto-snapshot -q -g --syslog --label=hourly --keep=24 //
# Snapshots- every day @ 4:00am
# 0 4 * * * root zfs-auto-snapshot -q -g --syslog --label=daily --keep=14 //
# Snapshots- every Sunday @ 4:00am
# 0 4 * * 0 root zfs-auto-snapshot -q -g --syslog --label=weekly --keep=4 //
# Snapshots- every first day of the month @ 4:00am
# 0 4 1 * * root zfs-auto-snapshot -q -g --syslog --label=monthly --keep=12 //
```
* Don't run backups on Sundays to allow for scrubs


### Scripts ###

#### Backup to Crow ####

vim scripts/backup_to_Crow.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/appdata/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/appdata/

sudo rsync -av --delete -e "ssh -i /root/.ssh/tom_servo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/dump/ root@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/dump/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/files/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/files/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Audio/ dave@192.168.10.10:/mnt/Deep-13/Media/Audio/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Books/ dave@192.168.10.10:/mnt/Deep-13/Media/Books/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Games/ dave@192.168.10.10:/mnt/Deep-13/Media/Games/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Photos/ dave@192.168.10.10:/mnt/Deep-13/Media/Photos/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Video/ dave@192.168.10.10:/mnt/Deep-13/Media/Video/
```
* Note that this script calls two different key pairs. The `administrator` user is for the Backups folder as that is owned by `administrator` but the Media files are owned by `dave` so that required a separate login via `dave` which required a second key pair. Trying to do everything thru `administrator` resulted in permissions issues whether `administrator` was added to all applicable groups, on both servers, or not.


#### Permissions Fixes ####

vim scripts/permissions.sh
```
#!/bin/bash

# Media/Audio/music
sudo chown -R dave:plex-music /mnt/Deep-13/Media/Audio/music
sudo find /mnt/Deep-13/Media/Audio/music -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Audio/music -type f -exec chmod 664 {} \;

# Media/Video/movies
sudo chown -R dave:plex-movies /mnt/Deep-13/Media/Video/movies
sudo find /mnt/Deep-13/Media/Video/movies -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Video/movies -type f -exec chmod 664 {} \;

# Media/Video/shows
sudo chown -R dave:plex-shows /mnt/Deep-13/Media/Video/shows
sudo find /mnt/Deep-13/Media/Video/shows -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Video/shows -type f -exec chmod 664 {} \;
```
* TIME = ~2 min.
* This script does not create output and therefore doesn't report anything


#### OS Updates ####

vim scripts/update.sh
```
#!/bin/bash

sudo apt-get update

sudo apt-get full-upgrade -y
```
* make sure to set permissions to 770 afterwards


### VMs

#### Media VM

sudo crontab -e
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# My Scripts
5 11 * * * /home/administrator/.scripts/backup-to-Tom_Servo.sh > /dev/null

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup-to-Deep13.sh
```


crontab -e (*do not use `sudo` for this job as it needs to be run by administrator*)
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# My Scripts
0 0 * * * cd /docker/appdata/plex-meta-manager && git pull %3E/dev/null 2>&1
```
- add, commit and push are far too complex to leave to cron so they must be run manually from time to time

- If you get an error emailed to you that says the repo is not trusted, run this:
	```
	git config --global --add safe.directory '*'

	sudo git config --global --add safe.directory '*'
	```
- Note- I ran it with `sudo` and without because I didn't know which one it needed. Next time you set it up just run one of them to see if it worked and then update the note above 


vim scripts/backup-to-Tom_Servo.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/bazarr/backup/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Bazarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/lidarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Lidarr/

sudo rsync -av --exclude 'nzbget.log' -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/nzbget/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/NZBGet/

sudo rsync -av --exclude '.git' -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/plex-meta-manager/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Plex-Meta-Manager/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/prowlarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Prowlarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/radarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Radarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/readarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Readarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/sonarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Sonarr/
```



## Crow ##
  - All necessary internal datasets as well as backups from Tom-Servo will be backed up to Gypsy with `rsync`, `restic` and `resticprofile`
	  - Everything gets backed up to the destination folder(s) on Gypsy and deleted files on Crow that were previously synced remain there.
	  - Every once in a while, I need go to the destination folder(s) and delete things that I know I no longer need.
  - Everything runs through cron jobs
  - Restic should have full access to the pool
  - Wait until after rsync tasks complete


### Crontab ###

sudo crontab -e
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# All cron times are UTC

# My scripts
0 13 * * * /home/administrator/.scripts/backup_to_Tom-Servo.sh > /dev/null
15 13 * * * /home/administrator/.scripts/permissions.sh > /dev/null
30 14 * * * /home/administrator/.scripts/rsync-to-gypsy.sh > /dev/null

# Updates- every Sunday night at 11pm
0 7 * * 0 /home/administrator/.scripts/update.sh > /dev/null

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup_to_Tom-Servo.sh
# * * * * * /root/scripts/permissions.sh
```
* Don't run backups on Sundays to allow for scrubs


### Scripts ###

#### Backup to Tom-Servo ####

vim scripts/backup_to_Tom-Servo.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo" /mnt/Deep-13/Backups/Crow/appdata/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/appdata/

sudo rsync -av --delete -e "ssh -i /root/.ssh/crow-to-tom_servo" /mnt/Deep-13/Backups/Crow/dump/ root@192.168.10.5:/mnt/Deep-13/Backups/Crow/dump/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo" /mnt/Deep-13/Backups/Crow/files/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/files/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-a" /mnt/Deep-13/NAS/Anna/ anna@192.168.10.5:/mnt/Deep-13/NAS/Anna/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-d" /mnt/Deep-13/NAS/Dave/ dave@192.168.10.5:/mnt/Deep-13/NAS/Dave/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-e" /mnt/Deep-13/NAS/Emmett/ emmett@192.168.10.5:/mnt/Deep-13/NAS/Emmett/
```
* Note that this requires dave, anna, and emmett to all create their own key pairs


#### Permissions Fixes

vim scripts/permissions.sh
```
#!/bin/bash

# ANNA NAS
chown -R anna:anna /mnt/Deep-13/NAS/Anna
find /mnt/Deep-13/NAS/Anna -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Anna -type f -exec chmod 664 {} \;

# DAVE NAS
chown -R dave:nas /mnt/Deep-13/NAS/Dave
find /mnt/Deep-13/NAS/Dave -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Dave -type f -exec chmod 664 {} \;

# EMMETT NAS
chown -R emmett:emmett /mnt/Deep-13/NAS/Emmett
find /mnt/Deep-13/NAS/Emmett -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Emmett -type f -exec chmod 664 {} \;
```
* TIME = ~2 min.
* This script does not create output and therefore doesn't report anything


#### Backing up video files to Gypsy

rsync-to-gypsy.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/gypsy-to-crow" /mnt/Deep-13/Media/Video/movies/ administrator@192.168.10.8:/mnt/Backups/Video/movies

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/gypsy-to-crow" /mnt/Deep-13/Media/Video/shows/ administrator@192.168.10.8:/mnt/Backups/Video/shows
```


#### OS Updates ####

vim scripts/update.sh
```
#!/bin/bash

sudo apt-get update

sudo apt-get full-upgrade -y
```
* make sure to set permissions to 770 afterwards



### VMs ###

#### Admin VM

sudo crontab -e
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# All cron times are UTC

# My scripts
0 13 * * * /home/administrator/.scripts/backup_to_Tom-Servo.sh > /dev/null
# 15 13 * * * /home/administrator/.scripts/permissions.sh > /dev/null
# 30 14 * * * /home/administrator/.scripts/rsync-to-gypsy.sh > /dev/null

# Updates- every Sunday night at 11pm
# 0 7 * * 0 /home/administrator/.scripts/update.sh > /dev/null

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup_to_Tom-Servo.sh
# * * * * * /root/scripts/permissions.sh
```

vim .scripts/backup_to_Tom-Servo.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/admin-crow-to-tomservo" /mnt/zpool/Backups/Crow/appdata/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/appdata/

sudo rsync -av --delete -e "ssh -i /home/administrator/.ssh/admin-crow-to-tomservo" /mnt/zpool/Backups/Crow/dump/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/dump/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/admin-crow-to-tomservo" /mnt/zpool/Backups/Crow/files/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/files/
```



#### Websites VM

sudo crontab -e
```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# My scripts
5 11 * * * /home/administrator/scripts/backup_to_Crow.sh > /dev/null

# For testing purposes (runs pretty soon after saving)
# * * * * * /home/administrator/scripts/backup_to_Crow.sh
# * * * * * /root/scripts/permissions.sh
```

backup-to-Crow.sh
```
#!/bin/bash

# ChangeDetection.io
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/change-detection/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Change-Detection/

# Daycare Website
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/Daycare/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/Daycare/

# FoundryVTT
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/FoundryVTT/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/FoundryVTT/

# Ghost

# Gotify
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/gotify/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Gotify/

# Homepage
sudo rsync -av --exclude '.git' -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/homepage/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Homepage/

# Paperless
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/Paperless/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/Paperless/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/paperless/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Paperless/

# Wiki.js
```

prune-db-backups.sh
```
#!/bin/bash

find /home/administrator/WPdbBackups/*.sql -mtime +14 -exec rm -rf {} \;
```


# Offsite #

## Gypsy ##

- Offsite backups will go from Crow to Gypsy only as everything local will be on Crow
- Video files are backed up using `rsync` and everything else is backed up using `restic` and `resticprofile`.
- See Crow scripts above for the `rsync` setup
- See the [[Restic]] note under "Running as a Service" for how `restic` is running.
