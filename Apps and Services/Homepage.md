---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 02:01 PM
---


# Installation

```yaml
version: "3.3"
services:
  homepage:
    image: ghcr.io/benphelps/homepage:latest
    container_name: homepage
    environment:
      - PUID=1000
      - PGID=1000
    ports:
      - 3005:3000
    volumes:
      - /docker/appdata/homepage/config:/app/config
      - /docker/appdata/homepage/public/images:/app/public/images
      - /mnt/Homepage/icons:/app/public/icons
      # - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integration
    restart: unless-stopped
```


## Custom Icons Folder
- Note the volume `/mnt/Homepage/icons:/app/public/icons` in the docker stack above
- The /mnt/homepage directory and subsequent icons directory should be `755` and `dave:administrator`
	- You may find that Homepage changes the ownership back to `administrator:administrator` but that's fine. You can change them back next time you need to upload icons (see below).
- The icon files themselves should be `644`

#### Uploading icons
- Before connecting to the SMB share we need to update the folder ownership as Homepage has a tendency to change it
	```
	sudo chown -R dave:administrator /mnt/Homepage/icons
	```
- Now connect to the Websites SMB share (should be in the left hand list in Windows Explorer)
- Upload the icons you've downloaded or created to the `/mnt/Homepage/icons` folder
- Change the permissions for the files you just uploaded
	```
	sudo chmod -R 644 /mnt/Homepage/icons/*
	```
- Now you need to recreate the container so that the icons show up
	- Go to Portainer and in the Websites container settings for Homepage click on "Recreate"


# Configuration #

#### widgets.yaml ####

```
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/widgets

# - resources:
#     cpu: true
#     memory: true
#     disk: /

- unifi_console:
    url: https://192.168.1.1
    username: [username]
    password: [password]

# - glances:
#   url: http://ip-address:61208
#     username: [username] # optional if auth enabled in Glances
#     password: [password] # optional if auth enabled in Glances
#     label: MyMachine # optional

- openmeteo:
  # label: Your-City # optional
    latitude: enter-coordinate
    longitude: enter-coordinate
    timezone: America/Los_Angeles # optional
    units: imperial
    cache: 5 # Time in minutes to cache API responses, to stay within limits

# https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat
- datetime:
    locale: en
    format:
      weekday: long
      month: long
      day: 2-digit
      year: numeric
```

