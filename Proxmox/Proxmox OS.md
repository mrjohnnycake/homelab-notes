---
created: Tue 2023-05-02 @ 06:32 PM
modified: Thu 2023-12-28 @ 08:53 AM
tags:
  - Homelabbing
---
# To Do's

- [ ] #proxmox [[Upgrades Project]] #Homelabbing 
- [ ] #proxmox Backup Proxmox system files? #Homelabbing 
- [ ] #proxmox Update [[Proxmox Drive Switch]] now that the [[Base Proxmox Setup]] note is there #Homelabbing 
- [ ] #proxmox Proxy and Websites VMs don't work when using IdentityAgent #Homelabbing 
- [ ] #proxmox [Proxmox Automation Scripts](https://tteck.github.io/Proxmox/) #Homelabbing 
- [ ] #proxmox Setup notifications #Homelabbing 
	- Need to document this process
	- https://youtu.be/85ME8i4Ry6A
	- https://pve.proxmox.com/pve-docs/chapter-notifications.html


# Setup

For setup steps, see [[Base Proxmox Setup]]


# Upgrading the OS

#### For minor upgrades

If you're just going from say 7.3.1 to 7.4, make sure everything is backed up and then just run this:
```
apt update

apt upgrade or apt dist-upgrade or apt full-upgrade
```

#### For major upgrades

- If you're upgrading from say 7.4 to 8.0 you can follow the steps from [here](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8)

	- Run this for Proxmox to give you pointers on what might cause issues:
		```
		pve7to8
		```

- On my last major upgrade I decided to start fresh. Here are some pre-steps I took before starting over:
	- Upgrade to the most recent sub-version (ex. 7.3.1 to 7.4) and then run `apt update` and `apt upgrade` again
	- Backup all VMs to a different server
	- Copy `/etc` and home directories
	- Export the zpool
	- Get rid of or leave the cluster
	- Copy ZFS snapshot schedule
	- Disconnect Netdata


# Upgrading Hardware

If you need to change hardware and you put the install disk in another motherboard you can get it working again by using the command prompt when starting up with a monitor connected. You need to update `etc/network/interfaces` with the new interface names for this new motherboard.


# Updating

-   From the Proxmox profile in Windows Terminal

```
apt update

apt upgrade or apt dist-upgrade or apt full-upgrade
```

-   Reboot via the Proxmox GUI


# Networking #

To get the network device names:
```
apt install lshw

lshw -C network
```

This is a good [video](https://www.youtube.com/watch?v=zx5LFqyMPMU) for Proxmox networking

To see details of bonds you have setup
```
cat /proc/net/bonding/bond1
```

You can run `iperf3` to see network speeds (but it never seemed to work right for me so look it up next time)
- You have to set it up on two different servers
```
iperf3 -c 192.168.10.10
```

***

Bonds should primarily be used for failover and/or link aggregation

Don't forget that you can bridge two or more VMs together so that they can only talk to each other
- You could also use an unused VLAN ID to accomplish the same thing


## Bonded Interfaces w/ VLANs ##

First, remove the static IP in Unifi for the server interface

For VLAN networking, set up a bonded interface:

* Node-->System-->Network-->Create-->Linux Bond

```
Name: bond0

Slaves: eno1 enp4s0 (or whichever ports you need to be in it)

Mode: LACP (802.3ad)

Hash policy: layer2+3

Autostart checked
```

* Everything else can be left empty

* For all of the physical interfaces, they can stay empty other than autostart for the used ones. I also like to comment on which one is which (ex. "built in" for eno1)

Now we need to create a bridge pointing to the bond:

```
Name: vmbr0

Autostart checked

VLAN aware checked

Bridge ports: bond0
```

Next create a Linux VLAN for one of the VLANs you'll be using. This one will double as the static IP of the Proxmox interface:

```
Name: vmbr0.10 (where 10 is the VLAN ID/Tag)

IPv4/CIDR: 192.168.10.10/28 (change as needed)

Gateway: 192.168.10.1

Autostart checked

VLAN raw device: vmbr0

VLAN Tag: 10

Comment: SERVERS VLAN (or whatever)
```

* Click OK

* Apply Configuration

* Reboot Proxmox

You probably won't be able to get back into the interface. That's because you need to tell your switch to Aggregate these two separate connections into one

* Go to Unifi Controller-->Devices-->24 Port Switch-Ports
* The ports used need to be sequential (ie. 5-6, 20-21, etc.)
* Go to the lower number port to be used
* Set the Port Profile to All since you'll need access to more than one VLAN with this connection
* Click on Port Profile Override and change the Operation to Aggregate
* For the Aggregate Ports, select the next port up from this one
* Apply Changes

Now log back into your Proxmox interface. If it comes up, it worked.

Now create another Linux VLAN for any other VLANs you need on the server. These 2nd+ Linux VLANs will follow the same format and are slightly different than the first VLAN you set up

```
Name: vmbr0.40 (where 40 is the VLAN ID/Tag)

IPv4/CIDR: 192.168.40.10/28 (change as needed) - this IP is necessary for switching though you can't access anything with it yourself - change it to .20 for Gypsy

Gateway: leave empty (Proxmox is supposed to only use one default gateway)

Autostart checked

VLAN raw device: vmbr0

VLAN Tag: 40

Comment: INTERNAL VLAN (or whatever)
```

* Click OK

* Apply Configuration

## 10Gbe Direct Connection ##

### Initial Setup ###

In Node-->Network, find the 10Gbe card ports
* They were called ens2f0 and ens2f1 before

Edit port 1

```
Autostart checked
Comment: 10Gbe port 1
MTU:9000
```

* Do the same for port 2 and just adjust the comment

Create Linux Bridge

```
Name: vmbr1 is fine
IPv4/CIDR: 10.10.10.10/24 (or whatever unused network address you can think of)
Autostart checked
Bridge ports: ens2f0 ens2f1 (you actually only need the one you're using, if you want)
Comment: 10Gbe direct connection
MTU: 9000
```




## Working Setup Examples

`nano /etc/network/interfaces`

#### Option 1 - Separate interfaces

*`bridge-vids 70` has to be edited via the CLI*

```
# network interface settings; autogenerated
# Please do NOT modify this file directly, unless you know what
# you're doing.
#
# If you want to manage parts of the network configuration manually,
# please utilize the 'source' or 'source-directory' directives to do
# so.
# PVE will preserve these directives, but will NOT read its network
# configuration from sourced files, so do not attempt to move any of
# the PVE managed interfaces into external files!

auto lo
iface lo inet loopback

auto eno1
iface eno1 inet manual
        mtu 9000
#Built-in: Lower

auto enp16s0
iface enp16s0 inet manual
        mtu 9000
#Built-in: Upper

auto enp5s0f0
iface enp5s0f0 inet manual
        mtu 9000
#10G- top

auto enp5s0f1
iface enp5s0f1 inet manual
        mtu 9000
#10G- bottom

iface enp11s0f0 inet manual
        mtu 9000
#1G- top

iface enp11s0f1 inet manual
        mtu 9000
#1G- second from top

iface enp11s0f2 inet manual
        mtu 9000
#1G- second from bottom

auto enp11s0f3
iface enp11s0f3 inet manual
        mtu 9000
#1G- bottom

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.10/28
        gateway 192.168.10.1
        bridge-ports enp5s0f1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 10
        mtu 9000
#Management bridge

auto vmbr70
iface vmbr70 inet manual
        bridge-ports enp11s0f3
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 70
        mtu 9000
#External bridge

auto vmbr40
iface vmbr40 inet manual
        bridge-ports enp11s0f2
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 40
        mtu 9000
#Internal bridge

auto vmbr20
iface vmbr20 inet manual
        bridge-ports enp11s0f1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 20
        mtu 9000
#IoT bridge
```



#### Option 2 - Bonded interfaces w/ active backup

```
auto lo
iface lo inet loopback

iface eno1 inet manual
        mtu 9000
#Built-in - lower

iface enp16s0 inet manual
        mtu 9000
#Built-in - upper

auto ens4f0
iface ens4f0 inet manual
        mtu 9000
#10GbE - top

auto ens4f1
iface ens4f1 inet manual
        mtu 9000
#10GbE - bottom

iface ens6f0 inet manual
        mtu 9000
#1G - top

iface ens6f1 inet manual
        mtu 9000
#1G - second from top

auto ens6f2
iface ens6f2 inet manual
        mtu 9000
#1G - second from bottom

auto ens6f3
iface ens6f3 inet manual
        mtu 9000
#1G - bottom

auto bond0
iface bond0 inet manual
        bond-slaves ens4f0 ens4f1
        bond-miimon 100
        bond-mode 802.3ad
        bond-xmit-hash-policy layer3+4
        mtu 9000
#Dual 10G LACP

auto bond1
iface bond1 inet manual
        bond-slaves ens6f2 ens6f3
        bond-miimon 100
        bond-mode 802.3ad
        bond-xmit-hash-policy layer3+4
        mtu 9000
#Dual 1G LACP

auto bond2
iface bond2 inet manual
        bond-slaves bond0 bond1
        bond-miimon 100
        bond-mode active-backup
        bond-primary bond0
        mtu 9000
#10G LACP failover to 1G LACP

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.10/28
        gateway 192.168.10.1
        bridge-ports bond2
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094
        mtu 9000
```



## Container Networking ##

For setting up containers to use the bonded, VLAN aware interface correctly:

* Container-->Network-->Edit the default setup (or Add if it doesn't exist)

```
Name: eth0

MAC address: this should already be populated

Bridge: vmbr0

VLAN Tag: 10

IPv4: DHCP

IPv6: DHCP
```

* You can copy this same setup for other containers. Just change the VLAN Tag as needed.


### CT Config ###

For any CTs that need to use the 10Gbe connection, just change CT-->Network settings to:

```
Bridge: vmbr1
IPv4: Static
IPv4/CIDR: 10.10.10.30/24 (or whatever IP that in the range of the vmbr1 settings)
```

* Leave any fields not referred to above as whatever their default is


# Clustering #

## Initial Cluster Creation ##

The easiest way to create a cluster is thru the GUI

* You need to stop all running CTs and VMs. If you get an error you may need to delete the second node's CTs and VMs entirely

Go to the main server -> Datacenter -> Cluster -> Create Cluster
* Name it
* Leave the IP the same
* Create

Copy the Join Information

If you get a known_hosts issue when trying to use the shell, you have to run the command it tells you to on both of the servers BUT you have to run the command in the server from that server's IP. So to run the command on server 1 you'd run it from the Shell in the GUI @ 192.168.10.5 and to run the command on server 2 you'd run it from the Shell in the GUI @ 192.168.10.10


- To add a third or more node to the cluster, you have to delete any VMs off of that third node and then copy the join information from the main node and go to the third node and click Join Cluster and paste that in.




## RPi QDevice ##

### Create the OS ###

* Insert the MicroSD card
* Open Raspberry Pi Imager
* Choose OS -> Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32-bit)
* Choose the Storage (MicroSD)
* Gear Icon ->
	* Set hostname to "pveqdevice"
	* Enable SSH
		* Use password authentication
	* Set username and password
		* pi
		* whatever password you want (make secure)
	* Set locale settings
	* Save

Take out the card, install it into the RPi and start it up


### Set the IP ###

Go into Unifi Controller and set a static IP for the RPi and name it something you like Proxmox QDevice
* Make sure that the IP is in the same VLAN as the Proxmox servers


### Setup the OS ###
Log in to Raspberry Pi
* First, confirm that the IP assigned in Unifi Controller is correct and then continue:
```
ip a
```

```
sudo passwd root

sudo systemctl restart sshd

sudo apt update && sudo apt upgrade -y

sudo apt install corosync-qnetd corosync-qdevice -y

sudo systemctl start corosync-qnetd.service

sudo systemctl enable corosync-qnetd.service
```

Before being able to add the QDevice to the cluster, you need to enable the Pi to accept root logins
```
sudo nano /etc/ssh/sshd_config
```

- Uncomment the line `PermitRootLogin` and change the "prohibited-password" to "yes"
- Save and exit

```
/etc/init.d/ssh restart
```

My understanding at the time of this writing is that you should leave the login as root enabled after adding it to the cluster.


### Add to Cluster ###
From Tom-Servo:
```
pvecm qdevice setup [qdevice IP]

example:
pvecm qdevice setup 192.168.10.6
```

Run `pvecm status` on both nodes to confirm there are now 3 votes



## Removing a Cluster Node

#### From the node you want to remove

Go into `root` to run all of these:
```
systemctl stop pve-cluster corosync

pmxcfs -l

rm -rf /etc/corosync/*

rm -rf /etc/pve/corosync.conf

killall pmxcfs

systemctl start pve-cluster

rm -rf /etc/pve/nodes/[NODE-NAME]
```

Still in root, remove anything from the node you removed from these files:
```
nano /etc/pve/storage.cfg

nano /etc/pve/jobs.cfg
```

And finally, run these
```
systemctl disable corosync-qdevice.service

systemctl disable pve-cluster corosync

reboot
```


#### From the Remaining Nodes

Remove the node's folder that is no longer in the cluster
```
rm -rf /etc/pve/nodes/[NODE-NAME]
```

Still in root, remove anything from the node you removed from these files:
```
nano /etc/pve/storage.cfg

nano /etc/pve/jobs.cfg
```

Remove the node from this AND change the remaining nodeid's:
```
sudo nano /etc/pve/corosync.conf
```

- Now restart the main node and let it come all the way back up

- Finally, reboot any other remaining nodes


## Deleting the Cluster

I don't know if there's an official way to do this because I couldn't find it anywhere. What worked for me was just running the "Removing a Cluster Node" steps on the last remaining node.


# Resizing VM Drives #

First- Resize the VM Disk in the Proxmox GUI

	- Node --> VM --> Hardware
	- Select Hard Disk (scsi0) and then click Disk Action --> Resize
	- Adjust size by adding how many more GBs you need

Second- You need to resize the VM Disk partition to use that exist disk space.  The steps below all take place in the VM CLI

The resizing process is different dependent on whether or not you are using an LVM setup inside the VM or not (consult how you installed the OS on your VM to find out). Follow one of the sections below, according to your needs, to finish the resizing process.

* Note- I found that I had to update at least one docker image (NZBGet) and then update the stack for that container before the container would recognize the change in disk size.


## LVM Disks ##

```
df -h
```
* take note of the `/dev/mapper/...` path as you'll need it later

```
sudo fdisk /dev/sda
```

These are the fdisk options you'll need to enter exactly in the order they're given:
```
p

d

2

n

e

2

Enter

Enter

n

Enter

Enter

No

t

5

8e

p

w (as long as the print looked good)
```

```
sudo reboot
```

- Log back in

```
sudo pvresize /dev/sda5

sudo lvextend -r -l +100%FREE /dev/mapper/debian--template--vg-root

df -h

sudo reboot
```



## Non-LVM Disks ##

```
df -h -T
```
* Take note of the size of `/dev/sda1`

```
sudo apt install parted -y

sudo parted /dev/sda

print

Fix (if needed)

resizepart 3 100%

print

quit
```

Third- Resize the VM LV
```
sudo su - root

pvdisplay

pvresize /dev/sda3

pvdisplay

lvresize -t -v -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

- If the last command worked in test mode, remove the test option and run again:
```
lvresize -v -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

resize2fs -p /dev/mapper/ubuntu--vg-ubuntu--lv

df -h -T | grep vg
```

Compare the last command results with what they originally were. That should be it.


# Containers

## Mount Points Setup

### Mount Points

```
nano /etc/pve/lxc/100.conf
```

* Add this to the bottom:

```
mp0: /mnt/Deep-13/websites,mp=/mnt/websites
```


### Permissions

```
nano /etc/pve/lxc/100.conf
```

* Add this to the bottom:

```
# uid map: from uid 0 map 1005 uids (in the ct) to the range starting 100000 (on the host), so 0..1004 (ct) → 100000..101004 (host)

lxc.idmap = u 0 100000 1005

lxc.idmap = g 0 100000 1005

# we map 1 uid starting from uid 1005 onto 1005, so 1005 → 1005

lxc.idmap = u 1005 1005 1

lxc.idmap = g 1005 1005 1

# we map the rest of 65535 from 1006 upto 101006, so 1006..65535 → 101006..165535

lxc.idmap = u 1006 101006 64530

lxc.idmap = g 1006 101006 64530
```

Open this:

```
nano /etc/subuid
```

* Add this to the bottom:

	```
	root:1005:1
	```

Open this:

```
nano /etc/subgid
```

* Add this to the bottom:

	```
	root:1005:1
	```

Change the directory permissions for the mounted folder

```
chown -R 1005:1005 /mnt/Deep13
```


More in depth explanation here https://www.itsembedded.com/sysadmin/proxmox_bind_unprivileged_lxc/
