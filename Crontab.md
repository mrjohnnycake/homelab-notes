On minimized install VMs, crontab needs to be installed
```
sudo apt install cron
```

```
MAILTO="myemail@gmail.com"
SHELL=/bin/bash
HOME=/

# My scripts
0 5 * * * /administrator/scripts/backup_to_Crow.sh

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup_to_Tom-Servo.sh
# * * * * * /root/scripts/permissions.sh
```

Add this to the end of a cron job line to disable notifications for that particular job
```
>/dev/null 2>&1
```
