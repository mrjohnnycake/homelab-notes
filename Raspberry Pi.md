---
created: Wed 2023-12-27 @ 08:09 PM
modified: Thu 2023-12-28 @ 08:44 AM
---
# Raspberry Pi OS Setup

It's Linux so the setup can be mostly copied from other notes but here are a few specifics (ex. Raspberry Pi OS doesn't create the SSH folder by default)
```
sudo apt update

sudo apt upgrade -y

sudo apt install vim -y

mkdir ~/.ssh

chmod 700 ~/.ssh

touch ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

sudo vim /etc/ssh/sshd_config
```
* Uncomment `PermitRootLogin` and change the answer to `No`

To customize the SSH login screen::
```
sudo vim /etc/motd
```
* Delete all of the text in this file and save

```
sudo vim /etc/update-motd.d/10-uname
```
* Add `printf "\n"` between `#!/bin/sh` and `uname -snrvm` to create some space between a couple lines during login



# Picture Frame Setup #

https://www.thedigitalpictureframe.com/how-to-set-up-your-raspberry-pi-for-your-digital-picture-frame/

```
sudo raspi-config



sudo python3 -m pip install -U pip

sudo python3 -m pip install picframe


sudo python3 -m pip install pi-heif
sudo pip3 install picframe
sudo reboot
picframe -i .


```
