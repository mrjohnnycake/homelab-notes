---
title: Installing Proxmox on Bare Metal
created: Tue 2023-12-19 @ 9:12 AM
aliases:
  - Default Note
  - Note Layout
tags:
  - "#template"
type: 
modified: Thu 2023-12-28 @ 08:53 AM
---
> [!abstract] In This Install
> - 

*This note is for brand new installs OR starting over completely. For upgrading pre-existing installs see* my [[Proxmox]] note

# Pre-Steps

- Do any hardware changes, if necessary
- Connect networking cable(s)
- Connect a monitor and keyboard OR a KVM
- Update USB stick with new ISO (use Rufus with the DD function)

# Initial Install

*These steps were written for version 8.1-1 but should be good for other versions*

- Start the server and load your motherboards boot screen
	- Take note of any boot issues as those may need to be fixed later
- Select the install disk.
- Select Install Proxmox VE (Terminal UI)
	- With PiKVM, I find that the graphical installer has parts of the screen cut off which inhibits the whole process
- Select "I Agree"
- Choose your Target Harddisk and hit Next
- Choose your Country, Timezone and Keyboard Layout and hit Next
- For Root Password, use something you know off the top of your head for now and we'll change it later
- You can enter a real email you'll be using for the server notifications or just change the last part of the fake email address to `.com`
- Hit Next
- Enter the info specific to your machine
	- I used this:
	```
	Management interface: ens4f0
	Hostname (FQDN): Crow.local
	IP address (CIDR): 192.168.10.10   /28
	Gateway address: 192.168.10.1
	DNS server address: 192.168.10.1
	```
- Hit Next
- Confirm that your info is correct and hit Install
- During reboot:
	- Remove or detach / disconnect the install disk
	- Take note of any errors on the screen in case they need to be troubleshooted later

- Open `https://192.168.10.10:8006/` (use your specific IP address)
	- If the page fails to load then `/etc/network/interfaces` needs to be edited. Use the direct connected keyboard and monitor OR your KVM to login and make the changes
	- Ignore the subscription pop-up that shows upon logging in

- There's no need to set a static IP on your router for the server since you've already told Proxmox to use a static IP


# Setup

These steps can be done as root in the Proxmox console

```
apt update

apt upgrade
```
*You'll get some repo errors but just ignore them for now*

- Reboot

```
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```
- Proxmox will be fully updated during this process so no need to run `apt update` and `apt upgrade` afterwards


Log in to the GUI for this next step:

GUI --> Node -> System -> DNS
```
local
192.168.10.1
1.1.1.1
1.0.0.1
```


# Storage 

- Create or Import a zpool by following the [[Deep-13 ZFS Zpool]] note

GUI --> Datacenter --> Storage
- Delete the default LVM-Thin storage
- Change `local` content to ISO image, Snippets, Container template
- Go to Add --> LVM-Thin
	- ID: see your specific machine's note
	- Volume group: pve
	- Thin pool: data
	- Content: Disk image, Container
	- Nodes: select this machine
	- Click Add


GUI --> Datacenter → Storage → Add → Directory
* ID: Backups-MachineName
* Directory: `/mnt/Deep-13/Backups/[MACHINE-NAME]`
* Content: VZDump backup file
* Node: Choose your machine
* Click Add


GUI --> Datacenter → Backup → Add…
* Node: Select the machine
* Storage: Select the storage you just created
* Schedule: Select "Every day 21:00" but then change the time to 01:00
* Selection Mode = All
* Send email: On failure only
* Send email to: enter your email address
* Leave the rest on this tab as default
* On the Retention tab set "Keep Last" to 3
* Click OK


# Environment

```
sudo apt update

sudo apt install fail2ban git vim yadm zfs-auto-snapshot zsh -y
```

- Reboot (don't skip this reboot)

- Log back in to the CLI

- Follow the Permissions section in the note for your specific machine

Setup the host to allow virtualization
```
sudo nano /etc/default/grub
```

* Comment out:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet”
```

* Add beneath that:

	* If Xeon machine:
	```
	GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
	```

	* If Threadripper machine:
	```
	GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
	```

```
sudo update-grub

sudo nano /etc/modules
```

* Add these lines:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

* Reboot



# Change Login User

Now we're going to add `administrator` user to avoid logging in as `root` going forward

* Create users WITH home directories (lowercase m)
```
useradd -u 1000 -m -s /bin/bash administrator

passwd administrator
```
- Choose a secure password for this

```
apt install sudo

usermod -aG sudo administrator
```

Change the `root` password to something more secure
```
passwd
```

```
sudo vim /etc/ssh/sshd_config
```

* Change this line to look like this (need to uncomment)
```
# Authentication:
PermitRootLogin no
```

Going forward we'll login via Windows Terminal using `administrator`



# Personalize

- Setup SSH with [[1Password with SSH]]

Make zsh the default shell
```
chsh -s /usr/bin/zsh
```

Run the config script
```
zsh
```
-Use option 2

Install Starship and ZSH-Autocomplete:
```
sudo curl -sS https://starship.rs/install.sh | sh

y

mkdir -p .config/zsh/plugins && cd "$_"

git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git
```


Add the GitHub CLI ( *you need to do this to be able to use YADM* )

All one command:
```
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

Authenticate GitHub
```
gh auth login
```

```
Select GitHub.com

Select SSH

Yes

No passphrase

Name: VM-Name-GH_CLI

Login with a web browser
	- Hit enter
	- Click the link
	- Copy the one-time code over
```

```
rm ~/.ssh/id_ed25519.pub
```

Clone into and pull down dotfiles repo:
```
yadm clone https://github.com/mrjohnnycake/dotfiles.git
```

- Reboot the server and then log back in to the CLI


Run the `debian.sh` script to clean out the home folder:
```
cd .scripts/setup

./debian.sh
```



# Additional Installs

Turn on Fail2Ban:
```
sudo systemctl enable fail2ban --now
```

- [[Email Setup]]
	- Update the documentation as needed next time

- Install PowerChute [[UPS]] connection
		- Update the documentation as needed next time

- [[Netdata]]

- Work on new Datacenter --> Notifications documentation

# Snapshots

- In the [[Backup and Maintenance]] note follow the "ZFS Snapshots w/ cron" section and consult your specific machine's note of what snapshots to turn on or not
