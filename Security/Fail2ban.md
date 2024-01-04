# Install

Install these if you haven't already:
```
sudo apt update

sudo apt install fail2ban iptables
```


# Configuring

When `fail2ban` gets updated it clears the config file so make a copy and it'll persist across upgrades:
```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit the config file:
```
sudo vim /etc/fail2ban/jail.local
```

At around line 92, add a line underneath the commented out example and make it look like this:
```
ignoreip = 127.0.0.1/8 ::1 192.168.50.22 192.168.50.23
```
- *the 192.168.50.__ addresses are for my local computers*

At around line 144, change it from "warn" to "no"
```
usedns = no
```

At around line 237, uncomment this:
```
action = %(action_)s
         apprise
```

At around line 264, comment out:
```
#action = %(action_)s
```

At around line 282, uncomment and make it look like this:
```
port     = ssh
logpath  = %(sshd_log)s
#backend = %(sshd_backend)s
backend  = systemd
enabled  = true
maxretry = 3
```

Get it working:
```
sudo systemctl enable fail2ban

sudo systemctl start fail2ban
```


# Notifications

I set up Apprise as the notification system since it's already built in and really easy to do.

Create the file that holds where you want the notification to go to:
```
sudo vim /etc/fail2ban/apprise
```

```
mailto://mygoogleusername:2FA-PASSWORD@gmail.com
```
- Replace `2FA-PASSWORD` with the password found in 1P

- Since I didn't end up editing `/etc/fail2ban/action.d/apprise.conf` there's no reason to change it to .local or anything like that

Reload fail2ban
```
sudo systemctl reload fail2ban.service
```



# Testing

```
sudo systemctl status fail2ban.service
```

```
sudo fail2ban-client status
```

If you're wondering if anyone has been banned you should check these:
```
sudo fail2ban-client status sshd

sudo iptables -L
```


# Troubleshooting

To get an IP out of jail, run this:
```
sudo fail2ban-client set sshd unbanip 192.168.50.22
```
