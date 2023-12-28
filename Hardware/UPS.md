---
created: Tue 2023-05-02 @ 06:31 PM
modified: Wed 2023-12-27 @ 03:54 PM
---
# To Do #

- Get some useful notifications working
	- Document how to do it



# Emergency Usage #

In a power failure, under the current settings, after 2 minutes, PowerChute on each server will initiate a graceful shutdown. They will not come back on until they are physically restarted.



# Initial Setup #

1. Dedicate a switch port for the unit and setup that port on the correct network. Assign it a dedicated IP and take note of it.
2. Put the battery in, connect the ethernet cable, and plug in the unit to the wall. It should fire up but might not be "On", which means that the unit is running but isn't powering anything.
3. On the Network Management Card on the back, stick a paperclip in the reset button and hold it there. One of the ethernet port lights should start blinking Green. Keep holding it down until that light starts blinking Orange and then release the reset button.
4. After the lights are done doing their thing and the card is completely reset and is functioning again, go to the IP address to open the management portal.
5. Log in with both the user and password as "apc".

The first thing to check is the firmware. If the interface looks dated it's probably out of date. This next step is fine to run either way.

To update the firmware
- go [here](https://www.apc.com/shop/us/en/tools/software-firmware/)
- Type in model of the Network Management Card 2 (ap9631) in the model number box (not the UPS model number) and select the correct one from the options that appear
- Check "Firmware Upgrades" in the filters box
- You'll need the UPS Network Management Card Firmware for Smart-UPS (if its called something else in the future just get the one that makes the most sense). Download it.
- Extract the zip file and open the Windows application inside of it. As of this writing, it returned a WinZip Self-Extractor window. Make sure that "When done unzipping open..." is checked and click Unzip.
- That should open a program called NMC Firmware Update Utility. Enter the IP of the UPS, username and password and click Start Update.
- Let it run and that should be all there is to it.
- Log on to the interface again to make sure it works.

The second thing to change is the password. Go to Configuration -> Security -> Local Users -> Management
- Click on the "apc" Super User
- Enter the current password
- Enter the new password twice
- Change the User Description to something useful (I wrote "Default User")
- Change Session Timeout to 5 minutes
- Change Temperature Scale to US Customary
- Click on next
- Apply the changes

Now you're ready to change the UPS settings as needed.


# Settings #

* If it's not on here then I didn't change anything

Configuration -> UPS
	- Change the UPS Name to whatever you want
	- Set the Last Battery Replacement to the correct time period
	- Apply

Configuration -> Security -> Local Users -> Default Settings
	- Access enabled
	- User Type: Device
	- Temperature Scale: US Customary
	- Strong Passwords enabled
	- Apply

Configuration -> Network -> TCP/IP -> IPv6 Settings
	- Uncheck Enable
	- Apply

Configuration -> Network -> DNS -> Configuration
	- Primary DNS Server: 1.1.1.1
	- Secondary DNS Server: 1.0.0.1
	- Enable System Name Synchronization
	- Change the Host Name to APC-UPS or whatever you want
	- Apply

Configuration -> Network -> Web -> Access
	- Just a note to not enable HTTPS because that will break the abilty to log in

Configuration -> General -> Identification
	- Change as desired
	- Apply

Configuration -> General -> Date/Time -> Mode
	- Set the Time Zone to Pacific Time
	- Select "Synchronize with NTP Server"
	- Leave "Override Manual NTP Settings" unchecked
	- Set Primary and Secondary NTP Server IPs
		- Google "NTP servers" and pick one from a list
	- Check "Update using NTP now"
	- Apply

Configuration -> General -> Date/Time -> Daylight Savings
	- Set Daylight Saving Time to "Traditional US DST"
	- Apply


# PowerChute #

#### Install ####

- I'm pretty sure JRE gets installed with the PCNS package so skip this next time and only come back to it if you need to
```
apt install openjdk-11-jre
```

- Download the 64 bit version of the PowerChute file
	- At the time of writing it was "pcns441Linux-x86-64.tar.gz"
	- I had the damnedest time downloading it directly with wget because of the way APC has their download links set up. I ended up downloading in Windows into the /mnt/Deep-13/Media folder and then moving it into the root folder

```
tar -xvf pcns441Linux-x86-64.tar.gz

cd Linux_x64

./install.sh
```

- It's a pretty straightforward install


#### Setup ####

The only confusing part of the setup is the user, password, and authentication phrase.
	- The user name must match the NMC name. Set the user to "apc".
	- The password is just the normal one you'll need to log in to this PowerChute instance. Generate it in 1P.
	- The Authentication Phrase can be first set in the regular APC UPS Network Management Card interface under Configuration -> Shutdown
		- Set the User Name to APC (not sure where this is used but best to keep it the same)
		- Generate a new password greater than 15 characters in 1P and put it in the Authentication Phrase box.
		- While you're on that inferface, check the HTTP box and set the On-Battery Shutdown Behavior to whatever you want