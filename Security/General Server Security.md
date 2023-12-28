---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 08:08 PM
---

* Turn off root user access

```
sudo nano /etc/ssh/sshd_config
```

* Change these lines

```
# Authentication:
PermitRootLogin no

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
```

Save and restart SSH

```
sudo systemctl restart ssh
```


## Don't expose things ##

```
ss -lptn
```

Go thru the list and see if I need that service/app exposed or not



## Firewall ##

```
ufw enable
```

Only allow ports I need


## AppArmor ##

```
sudo apparmor_status
```
