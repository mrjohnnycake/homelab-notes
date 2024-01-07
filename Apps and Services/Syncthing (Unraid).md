---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 03:49 PM
---
*I switched to using Proxmox instead of Unraid but I wrote most of this while using it.*

# Syncthing / SyncTrayzor Setup Walkthrough

This is my walkthrough for setting up Syncthing on unRAID and its complementing app SyncTrayzor on Windows. I felt that the instructions were lacking so hopefully this will help a fellow homelab-er.


## Install Syncthing on unRAID ##

```
docker create --name='Syncthing' --net='bridge' -e TZ="America/Los_Angeles" -e HOST_OS="Unraid" -e 'PUID'='99' -e 'PGID'='100' -e 'UMASK'='022' -p '8384:8384/tcp' -p '22000:22000/tcp' -p '22000:22000/udp' -p '21027:21027/udp' -v '/mnt/user/NAS/Downloads':'/data1':'rw' -v '/mnt/user/NAS/Pictures/':'/data2':'rw' -v '/mnt/user/NAS/Desktop/':'/data3':'rw' -v '/mnt/user/NAS/Notes/':'/data4':'rw' -v '/mnt/user/appdata/syncthing':'/config':'rw' 'lscr.io/linuxserver/syncthing'
```


* Install Syncthing from the Community Apps repository / page / store / whatever (I used the "linuxserver" version)
* The default install settings should be fine except you need to app the paths to the server folders you want to sync
* In my case, I want to sync my Windows Downloads folders between my laptop and my office computer. The extra benefit will be that those files will also exist on unRAID so I can then back them up per my server backup scheme. I also had an issue getting this to work with Android so until I get that working I can just connect to my NAS share on unRAID through an Android file browser and get whatever I need out of the Downloads folder and copy it to my phone.
* Click on Add Another Path
	- For the Name, call it whatever makes sense to you as this is what will show in the left hand side of the Syncthing docker config in unRAID
	- For the Container Path, I'm using /data/downloads because when I want to sync another folder I can just use /data/folder-name and it'll keep everything nice and tidy and my OCD will be happy.
	- For Host Path, I have a share called NAS that is my general / go to storage and I created a Downloads folder in there. So my path is /mnt/user/NAS/Downloads/
	- Click Save.


## Setup Syncthing

Disclaimer- I already had my server instance of Syncthing setup so there may or may not be an order of operations that Syncthing wants you to do that I've forgotten. I don't think so though so just let me know if my order of operations is off.

Also, in case you were wondering, Syncthing doesn't require you to forward any ports on your router so no need to worry about that.

* Open up the Web GUI of the Syncthing by clicking on its icon
* Syncthing will ask you to set up a user and password for the GUI so do that first.
* Let's set up the basic config found under Under Actions-->Settings
	- General tab:
		+ Device Name: call this whatever you want. I just named mine ```unRAID```
		+ I can't recall if you need the API Key so go ahead and generate one just in case. You can always delete it later if you'd like.
	- GUI tab
		+ Everything can stay as is on this tab but you can change the GUI Theme if you'd like.
	- Connections
		+ Set the Sync Protocol Listen Address to ```default```
		+ My Incoming and Outgoing Rate Limits are both set to ```0```
		+ Check Enable NAT traversal, Global Discovery, Local Discovery, and Enable Relaying
		+ Set Global Discovery Servers to ```default```
	- Click Save
	- That's it here for now. We'll be jumping back and forth between unRAID and your computer going forward so keep the window open and pay attention to the terms unRAID and Windows because both GUIs look the same. The unRAID Web GUI will be in your browser and the Windows GUI will be inside of SyncTrayzor's window on your computer.


## Install and Setup SyncTrayzor on Windows ##

* First of all, you don't install Syncthing on it’s own in Windows. Instead, install SyncTrayzor, which runs Syncthing as a part of it, and that's it. You can download it here.
* On first launch, SyncTrayzor will likely fail to startup Syncthing but that's okay.
		+ Still on Windows, go into SyncTrayzor→Settings→Syncthing→Advanced and add this to the “Syncthing Command-line Flags”
		+ ```-allow-newer-config```
    	+ It should start up fine now
* After Syncthing starts up, set the GUI username and password
* Click on Add Remote Device (still in SyncTrayzor)
    * After the window pops up, minimize SyncTrayzor and go back to your browser tab with the unRAID Syncthing Web GUI
    	+ Copy the Device ID with Actions→Show ID
    * Now go back to the SyncTrayzor window and enter that Device ID into the Trayzor “Add Device” box under Device ID
    * Leave the Device Name blank because the server will populate that on it's own
    * On the Sharing tab, click the Introducer and Auto Accept boxes
    * Click Save
* Now go back to the browser window for the server side of things and accept the new connection by allowing it in the pop-up box
* Back in SyncTrayzor, delete the Default folder that came pre-setup for you by the app
* Now click Add Folder
    * Folder Label: Downloads (or whichever folder you are wanting to sync)
    * Folder ID: ```downloads``` (lowercase) or whatever folder you entered above
    * Folder Path is the Downloads folder on Windows (or your desired folder)
    * On the Sharing tab, select unRAID (or whatever you called your unRAID Syncthing server)
    * Click Save
* Back in your browser window for the server side, accept the folder connection by allowing in in the pop-up box
* That’s it! Let it do it’s first time sync before adding any new files
* Back in Windows, you'll likely want to go to SyncTrayzor→Settings→SyncTrayzor and click the Minimize to tray box so that it will all run in the background.

Enjoy!