---
created: Thu 2023-11-30 @ 07:51 PM
modified: Thu 2023-12-28 @ 08:48 AM
---
# Prepare Proxmox

Get the current kernel version on your machine:
```
uname -r
```

In my case, it was
```
5.15.35-2-pve
```

Now search for available kernel updates:
```
apt-cache search pve-header
```

* From the results, find your kernel version and paste it into this command:
```
apt install pve-headers-*.*.*-*-pve

paste it in so it looks like this:

apt install pve-headers-5.15.35-2-pve
```


```
nano /etc/modprobe.d/pve-blacklist.conf
```

* Put this in there
```
blacklist nouveau
```

```
 update-initramfs -u
```

Reboot

```
apt install build-essential
```


# Install GPU Card Driver #

Go [here](https://www.nvidia.com/Download/index.aspx "NVIDIA Driver Downloads")

Enter your card details and click Search and then Download on the next page

On the next page, instead of clicking this second Download button, right click on it and copy the link address

In the Proxmox command line, enter "wget" followed by the link (example below):
```
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/515.48.07/NVIDIA-Linux-x86_64-515.48.07.run
```

```
chmod +x NVIDIA-Linux-x86_64-515.48.07.run

./NVIDIA-Linux-x86_64-515.48.07.run

rm NVIDIA-Linux-x86_64-515.48.07.run
```

* For the installer, just go with the defaults and click OK on any information prompts

Confirm it's installed
```
nvidia-smi
```

```
nano /etc/modules-load.d/modules.conf
```

Add these lines:
```
# Nvidia modules
nvidia
nvidia-modeset
nvidia_uvm
```

```
update-initramfs -u
```

```
nano /etc/udev/rules.d/70-nvidia.rules
```

Add lines 
```
KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L && /bin/chmod 666 /dev/nvidia*'"
KERNEL=="nvidia_modeset", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -m && /bin/chmod 666 /dev/nvidia-modeset*'"
KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u && /bin/chmod 666 /dev/nvidia-uvm*'"
```

Reboot

```
nvidia-smi
```



# Create the Container #

* Create the container as per usual except:
	* Uncheck "Unprivileged"
	* Size- 40G
	* Networking- Plex must be in the same VLAN as the computer used to set it up (laptop or office computer) to begin with in order to start the setup wizard. If you're not in the same subnet it won't work.
	* Do not start the container yet


* Find and note some GPU numbers:
	```
	ls -l /dev/nv*
	```

Edit the conf container you want to passthrough to
```
nano /etc/pve/lxc/161.conf
```

Compare the numbers from the previous command results and change the numbers accordingly in this text after pasting it into the 161.conf
```
# Allow cgroup access
lxc.cgroup2.devices.allow = c 195:0 rw
lxc.cgroup2.devices.allow = c 195:255 rw
lxc.cgroup2.devices.allow = c 195:254 rw
lxc.cgroup2.devices.allow = c 505:0 rw
lxc.cgroup2.devices.allow = c 505:1 rw
lxc.cgroup2.devices.allow = c 10:144 rw

# Pass through device files
lxc.mount.entry = /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry = /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry = /dev/nvram dev/nvram none bind,optional,create=file
```

* Note that the numbers on the top relate to the files in the "Pass through device files" section
* Note that we're not adding the media folder mount points just yet and also, because this is a privileged container, we don't need to do the normal permissions passthrough lines

Start the container


# Add the GPU to the CT

```
lxc-attach --name 161

apt update && apt upgrade -y

wget https://us.download.nvidia.com/XFree86/Linux-x86_64/515.48.07/NVIDIA-Linux-x86_64-515.48.07.run

chmod +x NVIDIA-Linux-x86_64-515.48.07.run

./NVIDIA-Linux-x86_64-515.48.07.run --no-kernel-module

rm NVIDIA-Linux-x86_64-515.48.07.run
```

Reboot

Confirm that the card was passed thru and is now working:
```
lxc-attach --name 161

nvidia-smi
```


# Install Plex #

```
lxc-attach --name 161 (if you're not already in there)

wget https://downloads.plex.tv/plex-media-server-new/1.27.1.5891-c29537f86/debian/plexmediaserver_1.27.1.5891-c29537f86_amd64.deb

dpkg -i plexmediaserver_1.27.1.5891-c29537f86_amd64.deb

rm plexmediaserver_1.27.1.5891-c29537f86_amd64.deb
```

Note that we haven't started the Plex service so it will not be functional yet. We need to set up folders and permissions first.


# Permissions #

Create the base "media" folder that will be owned by the CT root user
```
lxc-attach --name 161

mkdir /mnt/media
```

Still in the Plex CT, list the users and groups
```
less /etc/passwd

less /etc/group
```

* Make a note of the results for the users and groups relating to Plex's needs

Back in the Proxmox CLI, get the "plex" user's numbers
```
less /etc/passwd

less /etc/group
```

Now go back into the Plex CT CLI and change the "plex" user's numbers to match the host numbers
```
usermod -u 1002 plex

groupmod -g 1014 plex
```

I ran this command command in Proxmox as well but I'm not sure it matters (I'll note it here just in case)
```
usermod -aG plex dave-smb
```

Create groups in the Plex CT
```
lxc-attach --name 161

groupadd plex-movies

groupadd plex-music

groupadd plex-shows
```

Now change the group numbers for those new groups
```
groupmod -g 1006 plex-movies

groupmod -g 1007 plex-music

groupmod -g 1008 plex-shows
```

Now add the "plex" user to those groups
```
usermod -aG plex-movies plex

usermod -aG plex-music plex

usermod -aG plex-shows plex
```

Shutdown the Plex CT

```
nano /etc/pve/lxc/161.conf
```

Insert these lines above the "net0" entry
```
mp0: /mnt/deep-13/media/video/movies,mp=/mnt/media/movies
mp1: /mnt/deep-13/media/video/shows,mp=/mnt/media/shows
mp2: /mnt/deep-13/media/audio/music,mp=/mnt/media/music
```

Set the media folder ownership (if you haven't already)
```
chown dave-smb:pve-admin /mnt/deep-13/media

chown -R dave-smb:plex-music /mnt/deep-13/media/audio/music

chown -R dave-smb:pve-admin /mnt/deep-13/media/audio/other

chown dave-smb:pve-admin /mnt/deep-13/media/video

chown -R dave-smb:plex-movies /mnt/deep-13/media/video/movies

chown -R dave-smb:plex-shows /mnt/deep-13/media/video/shows
```

Set the media folder directory permissions (if you haven't already)
```
find /mnt/deep-13/media -type d -exec chmod 775 {} \;

find /mnt/deep-13/media/audio/music -type d -exec chmod 775 {} \;

find /mnt/deep-13/media/audio/other -type d -exec chmod 775 {} \;

find /mnt/deep-13/media/video -type d -exec chmod 775 {} \;

find /mnt/deep-13/media/video/movies -type d -exec chmod 775 {} \;

find /mnt/deep-13/media/video/shows -type d -exec chmod 775 {} \;
```

Set the media folder file permissions (if you haven't already)
```
find /mnt/deep-13/media -type f -exec chmod 664 {} \;

find /mnt/deep-13/media/audio/music -type f -exec chmod 664 {} \;

find /mnt/deep-13/media/audio/other -type f -exec chmod 664 {} \;

find /mnt/deep-13/media/video -type f -exec chmod 664 {} \;

find /mnt/deep-13/media/video/movies -type f -exec chmod 664 {} \;

find /mnt/deep-13/media/video/shows -type f -exec chmod 664 {} \;
```

Fix Plex install ownership issues (still shows up as plex:998)
```
chown -R plex:plex /var/lib/plexmediaserver
```


# Enable Plex Service

Startup the Plex CT 

Enable and start the Plex service
```
lxc-attach --name 161

systemctl enable plexmediaserver.service

systemctl start plexmediaserver.service
```

Open Plex at ip-address:32400/web and setup the server

Once the server is setup:
	* Stop the container
	* Change the container's VLAN to the Exposed VLAN
	* Start the container
	* Go into Unifi and set a static IP
	* Reboot the container
	* Go to the new IP and confirm everything works
	* Change the Port Forwarding Plex rule to the new IP


In Plex -> Settings -> Transcoder, check "Use hardware acceleration when available"


# Additional Permissions Notes

THE TEXT BELOW IS FROM [HERE](https://gist.github.com/pjobson/3811b73740a3a09597511c18be845a6c)


Maintenance

When copying files and directories around be sure to keep your permissions up to date.

Directories should be `775` or `rwxrwxrx-` and files should be `664` or `rw-rw-r--`.

I have my process automated, but what it would look like if it were manual would be:


Change the permissions of the directory of the movie to rwxrwxrw-

```
chmod 775 "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)"
```

Change the permissions of the files in the movie's path to rw-rw-r--

```
chmod 664 "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)/*"
```

Change the ownership of the movie and all contents using recursively to plex and the group to plex.

```
sudo chown -R plex.plex "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)"
```

I automate this process by creating a cron job which runs every fifteen minutes and checks the permissions and modifies them as needed.