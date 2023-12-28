---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:31 PM
---
## Netdata ##

- I didn't document the initial install on the first node but I remember it being pretty easy


#### Adding Additional Nodes ####

- Go to Netdata Cloud --> On the top horizontal menu click on Nodes --> Add Nodes
- Copy the Linux script and paste and run in the terminal of the server you want to add


#### Gotify notifications ####

From Server CLI:
```
cd /etc/netdata

sudo ./edit-config health_alarm_notify.conf
```

- Adjust to look like this
```
# gotify global notification options
SEND_GOTIFY="YES"

# App token and url
GOTIFY_APP_TOKEN="insert-your-token-here"
GOTIFY_APP_URL="https://subdomain-if-you-have-one.domain.com"

DEFAULT_RECIPIENT_GOTIFY="youremail@gmail.com"
```


#### Testing

To be able to test you need to do a couple of things first:
1) Set a password for the `netdata` user with `sudo passwd netdata` (use the password in 1P under Netdata --> server user password)
2) Change the shell for the `netdata` user in `/etc/passwd` to `/bin/bash`

```
su -s /bin/bash netdata

Enter the netdata user password

/usr/libexec/netdata/plugins.d/alarm-notify.sh test admin
```

Check your email and you should see it.

- After testing, change the `netdata` user shell back to `/usr/sbin/nologin`


#### Silencing Alarms ####

```
cd /etc/netdata

./edit-config health.d/cpu.conf
```

* cpu.conf is just one file. There are others like net.conf, etc. To find what file to edit, go to Netdata Cloud -> Alerts -> Alert Configurations and then find the alert you want to silence and then which node it is on.

- In the conf file, under the specific template that you're wanting to silence, change `to: sysadmin` to `to: silent`
