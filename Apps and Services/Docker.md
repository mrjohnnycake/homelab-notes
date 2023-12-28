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


## Portainer ##

### Install ###

```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/appdata/portainer:/data:rw portainer/portainer-ce
```

### Edge Agent ###

You only need to install Portainer once. For the other CTs or VMs, you can add them as Portainer environments.

* First, go to Environments-->Groups-->Add Group
	* I named it Gypsy for the server it needs to connect to
	* Put a description in there if you want
	* Select the associated environment
	* Click 'Create the group"

* Environments-->Add Environment
	* Select Docker Standalone and click Start Wizard
	* Select Edge Agent
	* For the name, you are naming the remote environment you are setting up. So lets say this Portainer server you are currently working on is 192.168.100.1 and you want to connect to 192.168.100.2, you'll want to name it after the service you're connecting to on 192.168.100.2
	* Leave the Portainer server URL the same. This should be the IP of the Portainer instance you are currently working in
	* Click on the "More settings" dropdown and set the group to the group you just made
	* Click Create
	* The page will now give you a command to run but just ignore that and click Close
	* Select the environment you just created.
	* Go to the Docker Standalone tab
	* Copy and paste the command given into the terminal for your server (remote environment) and remember to add "sudo" to the front of it
	* After that's done running, go back to the Portainer page and refresh it. It should now have some "Edge Information" on the top which means that it was set up properly.
	* While still on that page, enter the IP of your server in the Public IP box and click Update Environment

The environment is now added and you will see it if you head to the Home page in Portainer. But to be able to access it you'll first need to set up some port forwarding rules in your local network router.


### Port Forwarding ###

Unifi Controller-->Firewall & Security-->Create New Port Forwarding

```
Name: something that make sense

Enable checked

Interface: WAN

From: Limited

Source: the IP that has the agent you just installed

Port: needs 8000 & 9000. Set one of them here and make another port forwarding rule for the other

Foward IP: the IP of the Portainer server instance

Forward Port: whichever one you chose above

Protocal: for 8000, set it as TCP. For 9000, set it as both
```

* Apply changes

The new environment should be accessible in Portainer-->Home now. Just click on it and give it a sec for the first time you access it. If it times out just try again before troubleshooting.