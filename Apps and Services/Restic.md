---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 03:46 PM
---
[This](https://github.com/cupcakearmy/autorestic/) might be worth trying out sometime

# Running Options

Restic can be run a couple of different ways: from the command line and via script and service with environment variables. The setup is different depending on how you run it. Pick "Running Via the Command" OR "Running as a Service" and follow the instructions there.

# Running Via the Command Line

#### Environment Setup

```
mkdir /home/administrator/.config/restic

mkdir /home/administrator/.config/restic/backups-server1
mkdir /home/administrator/.config/restic/backups-server2

mkdir /home/administrator/.config/restic/media-audio
mkdir /home/administrator/.config/restic/media-books
mkdir /home/administrator/.config/restic/media-games
mkdir /home/administrator/.config/restic/media-photos

mkdir /home/administrator/.config/restic/nas-anna
mkdir /home/administrator/.config/restic/nas-dave
mkdir /home/administrator/.config/restic/nas-emmett
```

```
sudo chmod 700 -R /home/administrator/.config/restic
```

Setup files containing passwords for each each files so you don't have to continually enter them each time a backup is run (this will become absolutely necessary when running via crontab)
```
sudo nano /home/administrator/.config/restic/backups-crow/.pw
sudo nano /home/administrator/.config/restic/backups-tomservo/.pw

sudo nano /home/administrator/.config/restic/media-audio/.pw
sudo nano /home/administrator/.config/restic/media-books/.pw
sudo nano /home/administrator/.config/restic/media-games/.pw
sudo nano /home/administrator/.config/restic/media-photos/.pw

sudo nano /home/administrator/.config/restic/nas-anna/.pw
sudo nano /home/administrator/.config/restic/nas-dave/.pw
sudo nano /home/administrator/.config/restic/nas-emmett/.pw
```
* For each file, type or paste in the password for that specific repo, save and exit

Set permissions for each password file
```
chmod 600 /home/administrator/.config/restic/backups-crow/.pw
chmod 600 /home/administrator/.config/restic/backups-tomservo/.pw

chmod 600 /home/administrator/.config/restic/media-audio/.pw
chmod 600 /home/administrator/.config/restic/media-books/.pw
chmod 600 /home/administrator/.config/restic/media-games/.pw
chmod 600 /home/administrator/.config/restic/media-photos/.pw

chmod 600 /home/administrator/.config/restic/nas-anna/.pw
chmod 600 /home/administrator/.config/restic/nas-dave/.pw
chmod 600 /home/administrator/.config/restic/nas-emmett/.pw
```


Create (initialize) all the needed repositories:
```
restic --password-file /home/administrator/.config/restic/backups-crow/.pw -r sftp:backups:/mnt/Backups/Everything-Else/backups-crow init

restic --password-file /home/administrator/.config/restic/backups-tomservo/.pw -r sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo init

restic --password-file /home/administrator/.config/restic/media-audio/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-audio init

restic --password-file /home/administrator/.config/restic/media-books/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-books init

restic --password-file /home/administrator/.config/restic/media-games/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-games init

restic --password-file /home/administrator/.config/restic/media-photos/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-photos init

restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna init

restic --password-file /home/administrator/.config/restic/nas-dave/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-dave init

restic --password-file /home/administrator/.config/restic/nas-emmett/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-emmett init
```
* Note that the `--password-file` option has been added so that you don't have to constantly add the password to access the repo

Backup to the repository:
```
# SERVER BACKUP FILES

restic --password-file /home/administrator/.config/restic/backups-crow/.pw -r sftp:backups:/mnt/Backups/Everything-Else/backups-crow backup /mnt/Deep-13/Backups/Crow/{appdata,files}

restic --password-file /home/administrator/.config/restic/backups-tomservo/.pw -r sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo backup /mnt/Deep-13/Backups/Tom-Servo/{appdata,files}

* Note that the Proxmox VM/CT dump directory is not included in these backups


# MEDIA FILES

restic --password-file /home/administrator/.config/restic/media-audio/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-audio backup /mnt/Deep-13/Media/Audio

restic --password-file /home/administrator/.config/restic/media-books/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-books backup /mnt/Deep-13/Media/Books

restic --password-file /home/administrator/.config/restic/media-games/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-games backup /mnt/Deep-13/Media/Games

restic --password-file /home/administrator/.config/restic/media-photos/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-photos backup /mnt/Deep-13/Media/Photos


# NAS FILES

restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna backup /mnt/Deep-13/NAS/Anna

restic --password-file /home/administrator/.config/restic/nas-dave/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-dave backup /mnt/Deep-13/NAS/Dave

restic --password-file /home/administrator/.config/restic/nas-emmett/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-emmett backup /mnt/Deep-13/NAS/Emmett
```

For automating these commands, you can use `cron`

- For running all backups, use this as an example:
```
0 1 * * * restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna backup --verbose /mnt/Deep-13/NAS/Anna
```

- For pruning all backups, use this as an example:
```
0 2 * * * restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget --tag '' --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --keep-yearly 1 --prune --verbose
```



***

# Running as a Service #

While running from the command line is easy and familiar it makes it difficult to run multiple options at once. To do that you'll need to use environment variables. This is a project called [resticprofile](https://creativeprojects.github.io/resticprofile/) that works well but the catch is that it has to be run as root. If you try to run it as `user` as opposed to `system` (see profiles.conf), the schedule won't run until you are logged in as `administrator`. That's too bad because it works well. Here is how you use it:

Install with this method (the all-in-one script on the website installed it in the wrong directory)
```
curl -LO https://raw.githubusercontent.com/creativeprojects/resticprofile/master/install.sh

chmod +x install.sh

sudo ./install.sh -b /usr/local/bin

rm install.sh
```

#### Environment Setup

```
sudo -i
```

```
mkdir -p /root/.config/resticprofile/.keys

chmod 700 -R /root/.config/resticprofile
```

Setup files containing passwords for each each files so you don't have to continually enter them each time a backup is run (this will become absolutely necessary when running via crontab)
```
nano /root/.config/resticprofile/.keys/.backups-crow
nano /root/.config/resticprofile/.keys/.backups-tomservo

nano /root/.config/resticprofile/.keys/.media-audio
nano /root/.config/resticprofile/.keys/.media-books
nano /root/.config/resticprofile/.keys/.media-games
nano /root/.config/resticprofile/.keys/.media-photos

nano /root/.config/resticprofile/.keys/.nas-anna
nano /root/.config/resticprofile/.keys/.nas-dave
nano /root/.config/resticprofile/.keys/.nas-emmett
```
* For each file, type or paste in the password for that specific repo, save and exit

Change the permissions for all of the password files:
```
chmod 600 /root/config/resticprofile/.keys/.*
```

Create (initialize) all the needed repositories:
```
restic --password-file /root/.config/resticprofile/.keys/.backups-crow -r sftp:backups:/mnt/Backups/Everything-Else/backups-crow init

restic --password-file /root/.config/resticprofile/.keys/.backups-tomservo -r sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo init

restic --password-file /root/.config/resticprofile/.keys/.media-audio -r sftp:backups:/mnt/Backups/Everything-Else/media-audio init

restic --password-file /root/.config/resticprofile/.keys/.media-books -r sftp:backups:/mnt/Backups/Everything-Else/media-books init

restic --password-file /root/.config/resticprofile/.keys/.media-games -r sftp:backups:/mnt/Backups/Everything-Else/media-games init

restic --password-file /root/.config/resticprofile/.keys/.media-photos -r sftp:backups:/mnt/Backups/Everything-Else/media-photos init

restic --password-file /root/.config/resticprofile/.keys/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna init

restic --password-file /root/.config/resticprofile/.keys/.nas-dave -r sftp:backups:/mnt/Backups/Everything-Else/nas-dave init

restic --password-file /root/.config/resticprofile/.keys/.nas-emmett -r sftp:backups:/mnt/Backups/Everything-Else/nas-emmett init
```
* Note that the `--password-file` option has been added so that you don't have to constantly add the password to access the repo

/root/.config/resticprofile/profiles.conf
```toml
[global]
  # Ask restic to unlock a stale lock when its age is more than 2 hours
  # and the option "force-inactive-lock" is enabled in the profile.
  restic-stale-lock-age = "2h"
  scheduler = "systemd"

##### Backups/Crow #####

[backups-crow]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/backups-crow"
  password-file = "/root/.config/resticprofile/.keys/.backups-crow"
  lock = "/tmp/resticprofile_profile_backups-crow.lock"
  force-inactive-lock = true

  [backups-crow.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [backups-crow.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Backups/Crow/appdata", "/mnt/Deep-13/Backups/Crow/files" ]
    schedule = "*-*-* 13:35:00" # this means every day at 5:35am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/backups-crow_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [backups-crow.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 13:45:00" # this means every day at 5:45am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/backups-crow_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### Backups/Tom-Servo #####

[backups-tomservo]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo"
  password-file = "/root/.config/resticprofile/.keys/.backups-tomservo"
  lock = "/tmp/resticprofile_profile_backups-tomservo.lock"
  force-inactive-lock = true

  [backups-tomservo.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [backups-tomservo.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Backups/Tom-Servo/appdata", "/mnt/Deep-13/Backups/Tom-Servo/files" ]
    schedule = "*-*-* 13:50:00" # this means every day at 5:50am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/backups-tomservo_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [backups-tomservo.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:00:00" # this means every day at 6:00am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/backups-tomservo_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### Media/Audio #####

[media-audio]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/media-audio"
  password-file = "/root/.config/resticprofile/.keys/.media-audio"
  lock = "/tmp/resticprofile_profile_media-audio.lock"
  force-inactive-lock = true

  [media-audio.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [media-audio.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Media/Audio" ]
    schedule = "*-*-* 14:05:00" # this means every day at 6:05am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-audio_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [media-audio.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:10:00" # this means every day at 6:10am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-audio_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### Media/Books #####

[media-books]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/media-books"
  password-file = "/root/.config/resticprofile/.keys/.media-books"
  lock = "/tmp/resticprofile_profile_media-books.lock"
  force-inactive-lock = true

  [media-books.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [media-books.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Media/Books" ]
    schedule = "*-*-* 14:15:00" # this means every day at 6:15am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-books_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [media-books.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:20:00" # this means every day at 6:20am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-books_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### Media/Games #####

[media-games]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/media-games"
  password-file = "/root/.config/resticprofile/.keys/.media-games"
  lock = "/tmp/resticprofile_profile_media-games.lock"
  force-inactive-lock = true

  [media-games.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [media-games.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Media/Games" ]
    schedule = "*-*-* 14:25:00" # this means every day at 6:25am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-games_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [media-games.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:30:00" # this means every day at 6:30am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-games_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### Media/Photos #####

[media-photos]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/media-photos"
  password-file = "/root/.config/resticprofile/.keys/.media-photos"
  lock = "/tmp/resticprofile_profile_media-photos.lock"
  force-inactive-lock = true

  [media-photos.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [media-photos.backup]
    verbose = true
    source = [ "/mnt/Deep-13/Media/Photos" ]
    schedule = "*-*-* 14:35:00" # this means every day at 6:35am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-photos_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [media-photos.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:40:00" # this means every day at 6:40am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/media-photos_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### NAS/Anna #####

[nas-anna]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/nas-anna"
  password-file = "/root/.config/resticprofile/.keys/.nas-anna"
  lock = "/tmp/resticprofile_profile_nas-anna.lock"
  force-inactive-lock = true

  [nas-anna.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [nas-anna.backup]
    verbose = true
    source = [ "/mnt/Deep-13/NAS/Anna" ]
    schedule = "*-*-* 14:45:00" # this means every day at 6:45am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-anna_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [nas-anna.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 14:50:00" # this means every day at 6:50am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-anna_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### NAS/Dave #####

[nas-dave]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/nas-dave"
  password-file = "/root/.config/resticprofile/.keys/.nas-dave"
  lock = "/tmp/resticprofile_profile_nas-dave.lock"
  force-inactive-lock = true

  [nas-dave.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [nas-dave.backup]
    verbose = true
    source = [ "/mnt/Deep-13/NAS/Dave" ]
    schedule = "*-*-* 14:55:00" # this means every day at 6:55am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-dave_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [nas-dave.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 15:00:00" # this means every day at 7:00am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-dave_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"

##### NAS/Emmett #####

[nas-emmett]
  repository = "sftp:backups:/mnt/Backups/Everything-Else/nas-emmett"
  password-file = "/root/.config/resticprofile/.keys/.nas-emmett"
  lock = "/tmp/resticprofile_profile_nas-emmett.lock"
  force-inactive-lock = true

  [nas-emmett.env]
    RESTIC_CACHE_DIR = "/root/.cache/resticprofile/"

  [nas-emmett.backup]
    verbose = true
    source = [ "/mnt/Deep-13/NAS/Emmett" ]
    schedule = "*-*-* 15:05:00" # this means every day at 7:05am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-emmett_backup.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
    check-after = true
    run-after-fail = '/root/.config/resticprofile/resticprofile-send-error.sh postfixitman@gmail.com'

  [nas-emmett.forget]
    keep-daily = 7
    keep-weekly = 4
    keep-monthly = 6
    keep-yearly = 1
    prune = true
    schedule = "*-*-* 15:10:00" # this means every day at 7:10am PST
    schedule-permission = "system"
    schedule-priority = "background"
    schedule-log = "/root/.config/resticprofile/logs/nas-emmett_forget.log"
    schedule-lock-mode = "default"
    schedule-lock-wait = "15m30s"
```

/root/.config/resticprofile/resticprofile-send-error.sh
```sh
#!/usr/bin/env bash
[[ -z "${PROFILE_NAME}" ]] || sendmail -t <<ERRMAIL
To: $1
From: "resticprofile $(hostname -f)" <$USER@$(hostname -f)>
Subject: restic failed: ${PROFILE_COMMAND} "${PROFILE_NAME}"
Content-Transfer-Encoding: 8bit
Content-Type: text/plain; charset=UTF-8

${ERROR}

---- 
COMMANDLINE:

${ERROR_COMMANDLINE}

----
STDERR:

${ERROR_STDERR}

----
CONFIG:

$(resticprofile --name "${PROFILE_NAME}" show)

ERRMAIL
exit 0
```

```
sudo chmod 700 /root/.config/resticprofile/resticprofile-send-error.sh
```


## Usage ##

```
sudo resticprofile --name media-audio schedule

sudo resticprofile --name media-audio unschedule

sudo resticprofile schedule --all

sudo resticprofile unschedule --all

sudo resticprofile status --all

sudo resticprofile -n media-games backup

sudo resticprofile -n media-games forget
```

Updating resticprofile:
```
sudo resticprofile self-update
```


## Tips ##

- For some reason I had a problem getting the Backups/Tom-Servo schedule with `sudo resticprofile schedule --all` so after I ran that I also ran `sudo resticprofile --name backups-tomservo schedule`
- When you run `sudo resticprofile status --all` make sure the terminal window is wide otherwise it doesn't show up correctly



# Maintenance #

For most maintenance, the command line works just fine


## Managing Repos ##

List stats for a backup:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Video/media-audio stats
```

Check the health of a repository:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/media-audio check
```

List snapshots for a backup:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```

List all of the files in a snapshot along with their path (replace ID with yours)
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna ls b74f32ba
```

You can check the directory size of the original directory on Crow and the repository directory on Backups VM, respectively, as needed:
```
du -sh /mnt/Deep-13/NAS/Anna

du -sh /mnt/Backups/nas-anna
```


## Managing Snapshots ##

Restic works like this:
	- Backs up the data you want to the other server --> makes a snapshot of it --> checks newer backups against what's listed in the snapshot --> only backs up new files instead of re-uploading every file again --> creates a new snapshot with all of the files including the new ones

Therefore it is a good idea to limit the number of snapshots Restic is allowed to make and/or to delete old snapshots as desired, from time to time.

To delete snapshots, first list all of the snapshots for a given repository:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```

This will show all of the snapshots and give you their names, which look like `736949a9`, for example

First, delete that snapshot:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget 736949a9
```

Next, remove the unneeded data that may exist from the snapshot
	- So for instance, maybe that snapshot has a file in it you don't need anymore and you only want to keep the files that are in your other snapshots
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna prune
```

Edit this and run for all necessary back repos.
	- This will delete the snapshot(s) AND delete outdated files, all in one command
	- `--tag ''` says to consider all untagged snapshots. Since I'm not tagging any of my snapshots, this looks at all of the repos snapshots when deciding what to delete and what to keep
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget --tag '' --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --keep-yearly 1 --prune
```


## Restoring ##

List all snapshots:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```
- Take a note of the ID of the snapshot you want to restore from

Restoring a specific file (change the ID accordingly):
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp --include /mnt/Deep-13/NAS/Anna/test.pdf
```

Restoring a specific directory:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp --include /mnt/Deep-13/NAS/Anna/Pictures
```

Restoring an entire repo
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp
```

- Note that in practice, files will likely be needed to be restored to their original datasets and/or directories rather than a temp directory


## Errors ##

If you get an error about the repo being locked, just run this:
```
restic --password-file /home/administrator/.config/restic/nas-anna/.pw -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna unlock
```
