---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:49 PM
---
I gave up on torrenting because I had a hard time getting Deluge and Gluetun to work together. Consider this note a starting off point only.

## Install ##

```
version: "2.1"
services:
  deluge:
    image: lscr.io/linuxserver/deluge:latest
    container_name: deluge
    network_mode: "container:gluetun"
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
      - DELUGE_LOGLEVEL=error #optional
    volumes:
      - /docker/appdata/deluge:/config
      - /mnt/Downloads/torrents:/torrents/:rw
    restart: unless-stopped
```

- ports are handled through Gluetun
- Torrents folder doesn't have any subfolders


## App Settings ##

- IMPORTANT!!! You need to select the default connection and click connect for this all to work properly. If not, the settings won't be saved and plugins won't show up and it just won't work.

Connection Manager
```
Host: 127.0.0.1
Port: 58846
Username: localclient
Password: [blank]
```

Preferences

* Downloads
	```
	Download to: /torrents
	Pre-allocate disk space checked
	```

* Network
	```
	Incoming Address: 10.5.112.7
	Incoming Port: 6881 (random port unchecked)
	Outgoing Interface: tun0
	Outgoing Ports: 58946 (random port unchecked)
	Network Extras: all checked
	Peer TOS Byte: 0x00
	```

* Encryption
	```
	Incoming and Outgoing: Enabled
	Level: Either
	```

* Bandwidth
	```
	Global
	Max Connections: 1200
	Max Upload Slots: 14
	Max DL Speed: 30000
	Max Upload Speed: 5001
	Max Half Open: 50
	Max Attempts p/s: 20
	
	Ignore limits and Rate limit both checked
	
	Per Torrent
	Max Connections: 96
	Max Upload Slots: -1
	Max DL Speed: -1
	Max Upload Speed: -1
	```
* Interface
	```
	Allow the use... checked
	Set the password if haven't already
	Session Timeout: 3600
	Port: 8112
	```
* Daemon
	```
	Port: 58846
	Allow Remote Connections: checked
	```
* Queue
	```
	Active
	Total: 5
	Downloading: 5
	Seeding: 3
	
	Ignore and Prefer checked
	
	Seeding
	Share Ratio: 1
	Time Ratio: 6.3
	Time: 180
	
	Share Ratio: checked / 0.1
	Remove torrent checked
	```
* Plug-ins
	```
	Label checked
	```

* Side Filters Panel -> Labels -> Right click and create labels
	* need books, complete, movies, music, shows
	* The right click Options menu has all empty and unchecked options

