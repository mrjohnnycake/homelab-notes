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

#### settings.yaml

```
---
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/settings

title: New Tab
theme: dark # or light
color: stone
target: _self

layout:
  Proxmox:
    style: row
    columns: 3
  VMs:
    style: row
    columns: 5
  Portainer:
    style: row
    columns: 4

# target: _self (uncomment this after finished testing)
```


#### services.yaml

```
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/services

- Proxmox:
    - Tom-Servo:
        icon: proxmox.png
        href: https://192.168.10.5:8006/
        # description: Node 1
        widget:
          type: proxmox
          fields: ["cpu", "mem"]
          url: https://192.168.10.5:8006
          username: api@pam!homepage
          password: my-api-password

    - Crow:
        icon: proxmox.png
        href: https://192.168.10.10:8006/
        # description: Node 2
        widget:
          type: proxmox
          fields: ["cpu", "mem"]
          url: https://192.168.10.10:8006
          username: api@pam!homepage
          password: my-api-password

    - Gypsy:
        icon: proxmox.png
        href: https://192.168.10.8:8006/
        # description: Backup storage
        widget:
          type: proxmox
          fields: ["cpu", "mem"]
          url: https://192.168.10.8:8006
          username: api@pam!homepage
          password: my-api-password

- Network:
    - Unifi:
        icon: unifi.png
        href: https://192.168.1.1/network/default/dashboard
        # description:      
        widget:
          type: unifi
          url: https://192.168.1.1
          username: my-username
          password: my-password

    - NxFilter:
        icon: nxfilter.png
        href: http://192.168.10.11/dashboard.jsp

    - Speedtest:
        icon: speedtest-tracker.png
        href: http://192.168.70.170:8765/
        # description:
        widget:
          type: speedtest
          url: http://192.168.70.170:8765

    - Cloudflare:
        icon: cloudflare.png
        href: https://dash.cloudflare.com/

    - Cloudflare Zero Trust:
        icon: cloudflare-zero-trust.png
        href: https://one.dash.cloudflare.com/

    - Tailscale:
        icon: tailscale.png
        href: https://login.tailscale.com/admin/machines

    # - Gluetun:
          # icon: gluetun.png
          # widget:
            # type: gluetun
            # url: http://192.168.40.40:6881/

- Services:
    - Portainer:
        icon: portainer.png
        href: http://192.168.70.200:9000/
        # description: environment
        widget:
          type: portainer
          fields: ["running", "stopped"]
          url: http://192.168.70.200:9000
          env: 2
          key: my-portainer-key

    - PiKVM:
        icon: pikvm.png
        href: https://192.168.10.14/
        # description:

    - Home Assistant:
        icon: home-assistant.png
        href: http://home-assistant.local:8123/lovelace/0
        # description:

    - NZBGet:
        icon: nzbget.png
        href: http://192.168.40.45:6789/
        # description:
        widget:
          type: nzbget
          url: http://192.168.40.45:6789
          username: my-nzbget-username
          password: my-nzbget-password

    - GitHub:
        icon: github-light
        href: https://github.com/mrjohnnycake

- Media:
    - Plex:
        icon: plex.png
        href: http://192.168.70.70:32400/web/index.html
        # description: environment
        widget:
          type: plex
          url: http://192.168.70.70:32400
          key: my-plex-token # see https://www.plexopedia.com/plex-media-server/general/plex-token/

    - YouTube:
        icon: youtube.png
        href: https://www.youtube.com/
        # description: environment

    - YouTube Music:
        icon: youtube-music.png
        href: https://music.youtube.com/
        # description: environment

    - Pocket Casts:
        icon: /icons/pocket-casts.png
        href: https://play.pocketcasts.com/podcasts
        # description: environment

    - Audible:
        icon: /icons/audible.png
        href: https://www.audible.com/
        # description: environment

    - Kindle:
        icon: /icons/kindle.png
        href: https://read.amazon.com/kindle-library
        # description: environment

- Something:
    - Tautulli:
        icon: tautulli.png
        href: https://mysubdomain.mydomain.com/
        # description: environment
        widget:
          type: tautulli
          url: http://192.168.70.170:8181
          key: my-tautulli-api-key

    - Overseerr:
        icon: overseerr.png
        href: https://mysubdomain.mydomain.com/
        # description: environment
        widget:
          type: overseerr
          fields: ["pending", "available", "processing"]
          url: http://192.168.70.170:5055
          key: my-overseer-key

    - JustWatch:
        icon: /icons/justwatch.png
        href: https://www.justwatch.com/

    - IMDb:
        icon: /icons/imdb.png
        href: https://www.imdb.com/?ref_=nv_home

    - Goodreads:
        icon: /icons/goodreads.png
        href: https://www.goodreads.com/

- Media 2:
    - Radarr:
        icon: radarr.png
        href: http://192.168.40.45:7878/
        # description: movie manager
        widget:
          type: radarr
          fields: ["wanted", "missing", "queued"]
          url: http://192.168.40.45:7878
          key: my-radarr-key (I think its the API key?)

    - Sonarr:
        icon: sonarr.png
        href: http://192.168.40.45:8989/
        # description: show manager
        widget:
          type: sonarr
          fields: ["wanted", "queued"]
          url: http://192.168.40.45:8989
          key: my-sonarr-key (I think its the API key?)

    - Lidarr:
        icon: lidarr.png
        href: http://192.168.40.45:8686/
        # description: show manager
        widget:
          type: lidarr
          fields: ["wanted", "queued"]
          url: http://192.168.40.45:8686
          key: my-lidarr-key (I think its the API key?)

    - Readarr:
        icon: readarr.png
        href: http://192.168.40.45:8787/
        # description: show manager
        widget:
          type: readarr
          fields: ["wanted", "queued"]
          url: http://192.168.40.45:8787
          key: my-readarr-key (I think its the API key?)

    - Bazarr:
        icon: bazarr.png
        href: http://192.168.40.45:6767/
        widget:
          type: bazarr
          fields: ["missingEpisodes", "missingMovies"]
          url: http://192.168.40.45:6767
          key: my-bazarr-key (I think its the API key?)

- Websites:
    - Blog:
        icon: ghost.png
        href: https://mydomain.com/

    - Business Site:
        icon: /icons/business-name.png
        href: https://my-business-domain.com/
        # description:

    - Wiki.js:
        icon: wikijs.png
        href: https://my-optional-subdomain.mydomain.com/
        # description:

    - Paperless:
        icon: paperless.png
        href: https://my-optional-subdomain.mydomain.com/
        description: Receipt organizer
        server: websites-docker # The docker server that was configured
        container: paperless-ngx # The name of the container you'd like to connect

    - Kiwix:
        icon: kiwix.png
        href: http://192.168.40.105:8888/
        # description:

    - Foundry VTT:
        icon: foundry-vtt.png
        href: https://my-optional-subdomain.mydomain.com/join
        # description:

- Monitoring and Alerts:
    - Powerschool:
        icon: /icons/powerschool.png
        href: https://powerschool.my-kids-school-district.org/guardian/home.html

    - Netdata:
        icon: netdata.png
        href: https://app.netdata.cloud/spaces/myusername-space/rooms/all-nodes/home
        # description:

    - Gotify:
        icon: gotify.png
        href: https://my-optional-subdomain.mydomain.com/
        # description:
        widget:
          type: gotify
          url: https://my-optional-subdomain.mydomain.com
          key: my-key

    - Changedetection.io:
        icon: changedetection-io.png
        href: http://192.168.70.170:5000/
        # description:

    - UrBackup:
        icon: urbackup.png
        href: http://192.168.10.8:55414/
        # description:

    - Syncthing:
        icon: syncthing.png
        href: https://192.168.40.105:8384/
        # description:

    - APC UPS:
        icon: apc.png
        href: http://192.168.40.249
        # description:

    - Powerchute:
        icon: /icons/powerchute.png
        href: https://192.168.10.5:6547/index.html
        description: change last pair to 10 for Crow instance

    - Nest Doorbell:
        icon: /icons/nest.png
        href: https://home.nest.com/home/my-unique-number
```

#### widgets.yaml ####

```
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/widgets

# - resources:
#     cpu: true
#     memory: true
#     disk: /

# - unifi_console:
#     url: https://192.168.1.1
#     username:
#     password:

# - glances:
#     url: http://192.168.70.170:61208
#     username: [username] # optional if auth enabled in Glances
#     password: [password] # optional if auth enabled in Glances
#     label: MyMachine # optional

- openmeteo:
  # label: My-City-Name # optional
    latitude: my coordinate number
    longitude: my coordinate number
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


#### bookmarks.yaml

```
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/bookmarks

- News:
    - LoCO:
        - icon: /icons/loco.png
          href: https://loco.com/
    - KK:
        - icon: /icons/kK.png
          href: https://news-blog-from-southern-part-of-my-county.com/
    - NCJ:
        - icon: /icons/ncj.png
          href: https://www.ncj.com/
    - Google News:
        - icon: google.png
          href: https://news.google.com/home?hl=en-US&gl=US&ceid=US:en
    - Drudge Report:
        - icon: /icons/drudge-report.png
          href: https://www.drudgereport.com/
    - ESPN:
        - icon: /icons/espn.png
          href: https://www.espn.com/
    - CNBC:
        - icon: /icons/cnbc.png
          href: https://www.cnbc.com/
    - KRON4:
        - icon: /icons/kron4.png
          href: https://www.kron4.com/
    - Al Jazeera:
        - icon: /icons/al-jazeera.png
          href: https://www.aljazeera.com/

- Reading:
    - Gizmodo:
        - icon: /icons/gizmodo.png
          href: https://gizmodo.com/
    - The Next Web:
        - icon: /icons/the-next-web.png
          href: https://thenextweb.com/
    - Electrek:
        - icon: /icons/electrek.png
          href: https://electrek.co/
    - The Onion:
        - icon: /icons/the-onion.png
          href: https://www.theonion.com/
    - Untapped Brilliance:
        - icon: /icons/untapped-brilliance.png
          href: https://untappedbrilliance.com/
    - Energy Storage News:
        - icon: /icons/energy-storage-news.png
          href: https://www.energy-storage.news/

- Social:
    - Reddit:
        - icon: reddit.png
          href: https://reddit.com/
    - Facebook:
        - icon: facebook.png
          href: https://www.facebook.com/
    - Instagram:
        - icon: instagram.png
          href: https://www.instagram.com/
    - Threads:
        - icon: /icons/threads.png
          href: https://www.threads.net/
    - Nextdoor:
        - icon: /icons/nextdoor.png
          href: https://nextdoor.com/news_feed/
    - Mastodon:
        - icon: mastodon.png
          href: https://link.com

- Emergency:
    - Recent Quakes:
        - icon: /icons/usgs.png
          href: https://earthquake.usgs.gov/earthquakes
    - Zonehaven:
        - icon: /icons/zonehaven-aware.png
          href: https://aware.zonehaven.com/search?localnumbers
    - Hospital Capacity:
        - icon: /icons/hospital.png
          href: https://data.thecalifornian.com/covid-19-hospital-capacity/facility/my-hosptial
    - PG&E Outages:
        - icon: /icons/pge.png
          href: https://pgealerts.alerts.pge.com/outagecenter/
    - County OES:
        - icon: /icons/county-oes.png
          href: https://countywebsiteOES-site.org
    - FireMappers:
        - icon: /icons/firemappers.png
          href: https://napsg.maps.arcgis.com
    - CalFire Incidents:
        - icon: /icons/calfire.png
          href: https://www.fire.ca.gov/incidents
    - USFS Incidents:
        - icon: /icons/usfs.png
          href: https://inciweb.nwcg.gov/

- Weather:
    - Wunderground:
        - icon: /icons/wunderground.png
          href: https://www.wunderground.com/weather/us/ca/my-city
    - Radar:
        - icon: /icons/noaa.png
          href: https://www.nhc.noaa.gov/satellite.php
    - Wind Finder:
        - icon: /icons/windfinder.png
          href: https://www.windfinder.com/#9/coordinates
    - Weather Station:
        - icon: /icons/ambient-weather.png
          href: https://ambientweather.net/dashboard/my-token
    - Air Quality:
        - icon: /icons/purpleair.png
          href: https://map.purpleair.com/
    - Smoke Forecast:
        - icon: /icons/smoke.png
          href: https://www.weather.gov/city-code/AreaSmokeForecast

- Coding:
    - MongoDB:
        - icon: mongodb.png
          href: https://cloud.mongodb.com/v2/my-key/clusters
    - Oh My Git:
        - icon: gitea.png
          href: https://ohmygit.org/

- Finances:
    - My Bank:
        - icon: /icons/mybank.png
          href: https://www.mybank.com/
    - Quickbooks:
        - icon: /icons/quickbooks.png
          href: https://app.qbo.intuit.com/app/homepage
    - Finances:
        - icon: google-sheets.png
          href: https://docs.google.com/spreadsheets/d/
    - Charles Schwab:
        - icon: /icons/schwab.png
          href: https://www.schwab.com/

- Shopping:
    - Amazon:
        - icon: amazon.png
          href: https://www.amazon.com/
    - Ebay:
        - icon: ebay.png
          href: https://www.ebay.com/
    - Newegg:
        - icon: /icons/newegg.png
          href: https://www.newegg.com/
    - Craigslist:
        - icon: /icons/craigslist.png
          href: https://mycity.craigslist.org/

- Health:
    - MyFitnessPal:
        - icon: /icons/myfitnesspal.png
          href: https://www.myfitnesspal.com/
    - Cronometer:
        - icon: /icons/cronometer.png
          href: https://cronometer.com/
    - Garmin Connect:
        - icon: /icons/garmin-connect.png
          href: https://connect.garmin.com/
    - Komoot:
        - icon: /icons/komoot.png
          href: https://www.komoot.com/
    - MyChart:
        - icon: /icons/mychart.png
          href: https://mychart.ochin.org/mychart/Home/

- Google:
    - Drive:
        - icon: google-drive.png
          href: https://drive.google.com/drive/u/0/recent
    - Gmail:
        - icon: gmail.png
          href: https://mail.google.com/mail/u/0/#inbox
    - Photos:
        - icon: google-photos.png
          href: https://photos.google.com/
```