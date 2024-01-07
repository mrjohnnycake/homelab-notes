---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:52 PM
---
I don't really use LXC much anymore (just a personal thing) so that part of these notes might be outdated but I still use the Portainer part.
# LXC Install #

**[https://theorangeone.net/posts/docker-in-lxc/](https://theorangeone.net/posts/docker-in-lxc/)**

## Create CT ##
* Set host name, check unprivileged & nesting, set password
* local storage, ubuntu 20.04
* storage- local-zfs, disk size varies by project
* set cores
* set memory
* set IPv4 and 6 to DHCP
* Leave start after finished unchecked
* After it builds, go into container-->options-->features and check keyctl
* Start the container


## Setup Environment ##

Windows Terminal:

```
lxc-attach --name 100
```

```
sudo apt install docker.io docker-compose net-tools -y

sudo systemctl enable docker

sudo systemctl start docker
```

* Reboot the container via the GUI

* Give it a static IP in Unifi Controller-->Client Devices
