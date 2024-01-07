---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 03:49 PM
---
# Install

```bash
sudo useradd -u 1010 -M -s /usr/sbin/nologin dave
```

```yaml
version: "2.1"
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing-USER1
    hostname: syncthing #optional
    environment:
      - PUID=1010
      - PGID=1000
      - UMASK=002
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/syncthing-USER1:/config
      - "/mnt/NAS/USER1/Computer/Syncthing/:/USER1"
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
```
- I'm hosting two different containers for two different people in my house. I'll replace USER1 with their actual names.
- For this to work your NAS files need to be owned by 1010:1000 (or in my case USER1:administrator)

# Setup Syncthing

Before beginning...
- BACKUP YOUR FILES!
- You can do one of three things with your existing files (after you've already backed them up of course) on either the server or Windows:
	- Starting out, the directories can either be the exact same (the exact same files in both locations), you can have files in place on the Server side and empty on Windows side, or both directories can be empty. So to begin, chose how the directories are going to look and make those changes BEFORE proceeding.
- After the initial sync you can count on Syncthing to not screw up your files IF YOU FOLLOW MY ADVICE (at least that has been my experience). Just take caution now and you'll be fine. And if I'm lying or a complete fool at least you'll have a backup.
- Syncthing doesn't require you to forward any ports on your router so no need to worry about that.

Let's begin.

* Open up the Web GUI by going to your-server-IP-address:8384 (ex. 192.168.40.110:8384)
* Syncthing will ask you to set up a user and password for the GUI so do that first. It'll log you off after you hit Save so log back in with the new credentials.
* Let's set up the basic config found under Actions-->Settings
	- General tab:
		+ Device Name: call this whatever you want (I named mine `Admin VM @ Crow` so I didn't forget where it was installed)
		+ I can't recall if you need the API Key so go ahead and generate one just in case. You can always delete it later if you'd like.
	- GUI tab
		+ Everything can stay as is on this tab but you can change the GUI Theme if you'd like.
	- Connections
		+ Keep the defaults as they are
			+ Sync Protocol Listen Address to ```default```
			+ Incoming and Outgoing Rate Limits are both set to ```0```
			+ Enable NAT traversal, Global Discovery, Local Discovery, and Enable Relaying are all checked
			+ Global Discovery Servers to ```default```
	- Click Save
	- Now I'm going to go back in and edit Settings -> General -> Default Configuration -> Folder Defaults
		- This is optional but makes adding things easier going forward. Below are the changes I've made but you can do what you want
		- Changes
			- Folder Path: /USER1/
			- Advanced --> Ignore Permissions. Check this only if you have any issues with permissions later. I'll leave it unchecked for now.
		- I chose not to make any changes in Device Defaults
	- Click Save and then click Save again
	- Now delete the Default folder under Folders on the main screen by clicking on it, hitting Edit and then Remove

That's it here for now. We'll be jumping back and forth between the server and your computer going forward so keep the window open and pay attention to the terms Server and Windows because both GUIs look the same. The Server Web GUI will be in your browser and the Windows GUI will be inside of SyncTrayzor's window on your computer (although you may choose to also open that in a browser as well).


# Install and Setup SyncTrayzor on Windows

Note- *As of this writing, SyncTrayzor is two and a half years without an updated release. It still works fine but be aware that it is currently not being developed. Seeing as there is currently no other package for Windows that provides ease of use (such as a tray icon and log readout) I'm going to stick with it for the time being.*

* First of all, you don't install Syncthing on it’s own in Windows. Instead, install SyncTrayzor, which is a wrapper for Syncthing, and that's it. You can download it [here](https://github.com/canton7/SyncTrayzor). For my installation of Windows 11 on a decent computer I'm downloading SyncTrayzorSetup-x64.exe
* On first launch, SyncTrayzor will likely fail to startup Syncthing but that's okay.
	* On Windows, go into SyncTrayzor → File -> Settings → Syncthing → Advanced and add this to the “Syncthing Command-line Flags”
		+ `-allow-newer-config`
	 - While you have the Settings window open (these are optional but I found it helps the server / Windows communication go faster):
		 - On the same tab you were just on change "Syncthing Process Priority" to Above Normal
		 - On the SyncTrayzor tab, uncheck "Pause devices which connect over a metered network" under Metered Networks
		 - Click Save
- Now you can click on Start Syncthing in the main window and it should start up just fine.
* After start up, set the GUI username and password on the Actions --> Settings --> GUI tab
* Next, choose a Device Name on the General Tab (mine is `Strong-Bad`)
* Click on Add Remote Device (still in SyncTrayzor)
    * After the window pops up, minimize SyncTrayzor and go back to your browser tab with the Server Syncthing Web GUI
    	+ Copy the Device ID with Actions→Show ID
    * Now go back to the SyncTrayzor window and enter that Device ID into the Trayzor “Add Device” box under Device ID
    * Leave the Device Name blank because the server will populate that on it's own
    * Click Save
* Now go back to the browser window for the server side of things and accept the new connection by allowing it in the pop-up box. You may need to wait a minute for it to pop up first.
* Allow a minute for Windows and the Server to communicate fully. Remote Devices, on each side, should say "Connected (Unused)" when you're ready to move forward.
* On SyncTrayzor, delete the Default folder that came pre-setup for you by the app
* Back on the Server side, click Add Folder on the main screen
    * Folder Label: Desktop (or whichever folder you are wanting to sync)
    * Folder ID: `windows-desktop` (lowercase) or whatever folder you entered above
    * Folder Path is the Desktop folder location on your server (or your desired folder) which in my case is `/USER1/Desktop`
    * On the Sharing tab, select your Windows instance (whatever you called your SyncTrayzor / Syncthing)
    * Click Save
* Back in your SyncTrayzor window, accept the folder connection by clicking Add in the pop-up
	* You need to pick your Folder Path on the General tab when it pops up. Since I'm sharing my Desktop I'll Browse to that folder on my computer.

That’s it! Let it do it’s first time sync before adding any new files but you should be good to go! Follow the Add Folder instructions again to add any new folders to share.



# Tips

- Currently I'm only using Syncthing to backup my computer files to my server. Where Syncthing excels though is adding a third or more Remote Devices. So if you want to keep your desktop computer, your laptop, and maybe your work computer in sync then just repeat the steps as necessary on those devices. It works great.

* In Windows, you'll likely want to go to SyncTrayzor→Settings→SyncTrayzor and click the Minimize to tray box so that it will all run in the background. Starting on Windows start-up is default.


# Errors

- If you get any errors with out of sync files, make sure everything is EXACTLY the same in each directory (the folder on your computer and the directory on the server). Also double check that your permissions are correct.


- Permissions issues between different types of devices:
	- Settings to go with:
		- On Windows
			- Turn on "Ignore Permissions"
		- On Server
			- Turn off "Ignore Permissions"
	- All of this could have just been an issue I had with version 1.22.1 so ignore this unless you have an issue.