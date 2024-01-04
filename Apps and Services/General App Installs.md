*For my personal needs, see the [[Homelab/App Specifics (unshared)]] note*

## Bazaar ##

```yaml
version: "2.1"
services:
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/bazarr:/config
      - /mnt/Media/Video/movies:/movies
      - /mnt/Media/Video/shows:/shows
    ports:
      - 6767:6767
    restart: unless-stopped
```


## changedetection.io ##

Install:
```d
version: '2'

services:
  changedetection:
    image: ghcr.io/dgtlmoon/changedetection.io
    container_name: changedetection
    hostname: changedetection
    volumes:
      - /docker/appdata/change-detection:/datastore
    ports:
        - 5000:5000
    restart: unless-stopped
```


```d
version: '2'

services:
    changedetection:
      image: ghcr.io/dgtlmoon/changedetection.io
      container_name: changedetection
      hostname: changedetection
      volumes:
        - /docker/appdata/change-detection:/datastore
# Configurable proxy list support, see github.com/dgtlmoon/changedetection.io/wiki/Proxy-configuration#proxy-list-support
#       - ./proxies.json:/datastore/proxies.json

      environment:
  #        Default listening port, can also be changed with the -p option
  #      - PORT=5000

  #      - PUID=1000
  #      - PGID=1000
  #
  #       Alternative WebDriver/selenium URL, do not use "'s or 's!
        - WEBDRIVER_URL=http://browser-chrome:4444/wd/hub
  #
  #       WebDriver proxy settings webdriver_proxyType, webdriver_ftpProxy, webdriver_noProxy,
  #                                webdriver_proxyAutoconfigUrl, webdriver_autodetect,
  #                                webdriver_socksProxy, webdriver_socksUsername, webdriver_socksVersion, webdriver_socksPassword
  #
  #             https://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.proxy
  #
  #       Alternative Playwright URL, do not use "'s or 's!
  #      - PLAYWRIGHT_DRIVER_URL=ws://playwright-chrome:3000/?stealth=1&--disable-web-security=true
  #
  #       Playwright proxy settings playwright_proxy_server, playwright_proxy_bypass, playwright_proxy_username, playwright_proxy_password
  #
  #             https://playwright.dev/python/docs/api/class-browsertype#browser-type-launch-option-proxy
  #
  #        Plain requests - proxy support example.
  #      - HTTP_PROXY=socks5h://10.10.1.10:1080
  #      - HTTPS_PROXY=socks5h://10.10.1.10:1080
  #
  #        An exclude list (useful for notification URLs above) can be specified by with
  #      - NO_PROXY="localhost,192.168.0.0/24"
  #
  #        Base URL of your changedetection.io install (Added to the notification alert)
  #      - BASE_URL=https://mysite.com

  #        Respect proxy_pass type settings, `proxy_set_header Host "localhost";` and `proxy_set_header X-Forwarded-Prefix /app;`
  #        More here https://github.com/dgtlmoon/changedetection.io/wiki/Running-changedetection.io-behind-a-reverse-proxy-sub-directory
  #      - USE_X_SETTINGS=1

      # Comment out ports: when using behind a reverse proxy , enable networks: etc.
      ports:
        - 5000:5000
      restart: unless-stopped

     # Used for fetching pages via WebDriver+Chrome where you need Javascript support.
     # Now working on arm64 (needs testing on rPi - tested on Oracle ARM instance)
     # replace image with seleniarm/standalone-chromium:4.0.0-20211213

    browser-chrome:
        hostname: browser-chrome
        image: selenium/standalone-chrome-debug:3.141.59
        environment:
            - VNC_NO_PASSWORD=1
            - SCREEN_WIDTH=1920
            - SCREEN_HEIGHT=1080
            - SCREEN_DEPTH=24
        volumes:
            # Workaround to avoid the browser crashing inside a docker container
            # See https://github.com/SeleniumHQ/docker-selenium#quick-start
            - /dev/shm:/dev/shm
        restart: unless-stopped

     # Used for fetching pages via Playwright+Chrome where you need Javascript support.

    playwright-chrome:
        hostname: playwright-chrome
        image: browserless/chrome
        restart: unless-stopped
        environment:
            - SCREEN_WIDTH=1920
            - SCREEN_HEIGHT=1024
            - SCREEN_DEPTH=16
            - ENABLE_DEBUGGER=false
            - PREBOOT_CHROME=true
            - CONNECTION_TIMEOUT=300000
            - MAX_CONCURRENT_SESSIONS=10
            - CHROME_REFRESH_TIME=600000
            - DEFAULT_BLOCK_ADS=true
            - DEFAULT_STEALTH=true
```

```
http://websitetotrack.org/
Website Name

mailto://youremail:[2fa-app-password]@gmail.com/?to=youremail@gmail.com,secondaryemail@gmail.com
gotifys://subdomain.yourdomainname.com/gotifytoken

What You Want Your Emails To Be Titled

There has been a change to the Website Name website!
---------------
{diff}
```



## Duplicacy ##

https://www.youtube.com/watch?v=yEr15Pms-IA&t=1111s&ab_channel=IBRACORP


```
docker run -d --name='Duplicacy' -p '3875:3875/tcp' -v '/docker/appdata/duplicacy':'/config':'rw' -v '/docker/appdata/duplicacy/cache':'/cache':'rw' -v '/docker/appdata/duplicacy/logs':'/logs':'rw' -v '/mnt/zpool/Duplicacy':'/restored':'rw' -e PUID=1000 -e PGID=1000 --hostname=duplicacy 'cr.hotio.dev/hotio/duplicacy'
```



```
version: "3.7"
services:
  duplicacy:
    image: cr.hotio.dev/hotio/duplicacy
    container_name: duplicacy
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/duplicacy:/config
      - /docker/appdata/duplicacy/cache:/cache
      - /docker/appdata/duplicacy/logs:/logs
    ports:
      - 3875:3875
    restart: unless-stopped
```


Example options:
```
-v '/mnt/user/appdata/duplicacy':'/config':'rw' 
-v '/mnt/user/appdata/duplicacy/cache':'/cache':'rw' 
-v '/mnt/user/appdata/duplicacy/logs':'/logs':'rw' 


-v '/mnt/user/backups/':'/backups/server':'rw' 
-v '/mnt/user/NAS/':'/backups/nas':'rw' 
-v '/mnt/user/Bucket/':'/backups/bucket':'rw' 
-v '/mnt/user/data/':'/backups/data':'rw' 
-v '/mnt/user/websites/':'/backups/websites':'rw'
```



### Usage ###

Add the path to the shares that you want to back up

* Duplicacy → Storage → plus sign
	* S3 tab → Service Provider
	* If you have a domain name, select "minio (over HTTPS)
	* If you choose this option, follow the MinIO Certificate Setup below
	* If you do not have a domain name, select "minio (over HTTP)
	* Endpoint- enter subdomain.domain.tld (eg. minio.yourdomain.com) or enter your IP address with the MinIO API port (eg. 192.168.1.2:9000)
	* Region- enter the Server Location you setup in MinIO
	* Access ID- enter the Access Key from your MinIO user
	* Access Key- enter the Secret Key from your MinIO user
	* Bucket- select the bucket you created in MinIO
	* Directory- enter the name of one of your unRAID shares (eg. appdata)
	* For this first time setup, choose the share with the least amount of data. This will speed up any needed troubleshooting later
	* Click Continue
	* Name your storage whatever you want. It'll show up in the Duplicacy/Storage page
	* Click Create

* Duplicacy → Backup → Click the plus sign
	* Directory- select the Duplicacy container folder you mapped earlier of the share you want to back (eg. appdata)
	* Storage- select the Duplicacy storage you just created
	* Backup ID- name it whatever you want as it'll show up on this page
		* Consider other backups you'll want to make including to different servers or services such as Backblaze B2
    * Click Save

* Click the "play" button underneath the backup you just created. It should bring up a progress bar and hopefully complete pretty quickly. If your share has very small data in it you may not even see a progress bar.

* Go back over to the MinIO Console and browse your bucket. You should see a folder called [whatever share name you chose] and if you click on that you should see a config file and two folders named chunks and snapshots. If that's true for you then congratulations for creating a new backup server.


### Optional Install on the Host###

```
apt install apt-transport-https git-core software-properties-common -y
```

```
wget https://acrosync.com/duplicacy-web/duplicacy_web_linux_x64_1.6.3
```

```
apt install ./duplicati_2.0.6.3-1_all.deb -y
```



## Duplicati ##

```
version: "2.1"
services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
      - CLI_ARGS= #optional
    volumes:
      - /docker/appdata/duplicacy/config:/config
      - /docker/appdata/duplicacy/backups:/backups
      - /docker/appdata/duplicacy/source:/source
      - /mnt/zpool:/mnt/zpool
    ports:
      - 8200:8200
    restart: unless-stopped
```

OR THE CLI VERSION

```
docker run -d \
  --name=duplicati \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Los_Angeles \
  -e CLI_ARGS= `#optional` \
  -p 8200:8200 \
  -v /docker/appdata/duplicati/config:/config \
  -v /docker/appdata/duplicati/backups:/backups \
  -v /docker/appdata/duplicati/source:/source \
  -v /mnt/zpool:/mnt/zpool \
  --restart unless-stopped \
  lscr.io/linuxserver/duplicati:latest
```


## Fail2Ban ##

* I personally choose to run mine directly on the host (Proxmox or a VM) but the docker method is here if you're interested in that route (I would think as long as you passed the necessary networking to the container it would work perfectly fine)
* Install Fail2Ban from scratch based on [this](https://www.youtube.com/watch?v=Ha8NIAOsNvo) video

	* For NPM, the log file can be reached thru a mount point in the container
	* Not sure about the /var/log/auth.log

	```
	version: "3.7"
	services:
     fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban_docker
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - "/docker/appdata/fail2ban/data:/data"
      - "/var/log/auth.log:/var/log/auth.log:ro"
      - "/docker/appdata/nginx-proxy-manager/logs/:/log/npm/:ro"
    restart: always
    environment:
      - TZ=America/Los_Angeles
      - F2B_LOG_TARGET=STDOUT
      - F2B_LOG_LEVEL=INFO
      - F2B_DB_PURGE_AGE=365d
	```



## Flame ##

There's unfortunately no import/export feature you can use to restore easily.  What I did last time was open the db.sqlite file from the Deep-13/backups/appdata folder and export the bookmarks, apps, and categories tables as CSV and then open them in Sheets and then copy and paste each entry cell by cell into new entries in the new Flame install. 

I ended up using Homepage instead but Flame is a nice app

```
sudo useradd -u 1065 -M -s /usr/sbin/nologin flame

sudo usermod -aG docker flame
```

```yaml
version: '3.6'

services:
  flame:
    image: pawelmalak/flame
    container_name: flame
    volumes:
      - /docker/appdata/flame:/app/data
    ports:
      - 5005:5005
    environment:
      - PUID=1065
      - PGID=1065
      - TZ=America/Los_Angeles
      - PASSWORD=your-password-for-this-app
    restart: unless-stopped
```


* Once the container is running and you logout and log in again, the /docker/appdata changes it's ownership from flame:flame to administrator (or 1000 actually):flame
* This has to do with the way that the image was built and as long as the administrator 1000:1000 user is created it should work just fine  
* I wonder if replacing the PUID and PGID variables the compose file to this would work, if I wanted to be that anal?
	```
	- USERMAP_UID=1065
    - USERMAP_GID=1065
	```


## Gluetun ##

I never quite got this working right so play around with it

[Setup Video](https://youtu.be/0F6I03LQcI4 "Gluetun Setup Video")

Run this on the Media VM terminal:
```
sudo modprobe tun
```
* this may already come setup in the kernal but do it just in case

```yml
version: "3"
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    volumes:
      - /docker/appdata/gluetun:/gluetun
    environment:
      - VPN_SERVICE_PROVIDER=private internet access
      - OPENVPN_USER=your-pia-username
      - OPENVPN_PASSWORD=your-pia-password
      - SERVER_REGIONS=CA Toronto
    ports:
      - 8112:8112 # Deluge interface
      - 6881:6881 # Deluge incoming
      - 6881:6881/udp # Deluge incoming
      - 58846:58846/tcp # Deluge daemon
      - 58946:58946 # Deluge outgoing
    restart: unless-stopped
```

To test, go to the gluetun container in Portainer and checks the logs. At the bottom should be the IP address. Make note of it.

Now open a terminal window for Media VM. The next command will create a temporary container.

```
sudo docker run -it --rm --network="container:gluetun" alpine:3.14
```

- This will run and you'll now be in the CLI for that container.
- Now run this to check the IP of the container

```
wget -qO- https://ipinfo.io
```

As long as the IP matches the IP from the gluetun container, you're good to go.

### Container Setup ###

For any container that needs to use gluetun to connect, like Deluge, add this to the Portainer Stack (or Docker Compose) script

```
network_mode: "container:gluetun"
```


## Ghost ##

```
version: '3.1'

services:

  ghost:
    image: ghost:latest
    restart: always
    ports:
      - 8888:2368
    environment:
      # see https://ghost.org/docs/config/#configuration-options
      database__client: mysql
      database__connection__host: db
      database__connection__user: root
      database__connection__password: [insert password here]
      database__connection__database: ghost
      # this url value is just an example, and is likely wrong for your environment!
      url: http://192.168.70.170:8888

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: [insert password here]
```



## Gotify ##

```
sudo useradd -u 1066 -M -s /usr/sbin/nologin gotify

sudo usermod -aG docker gotify
```

```yaml
version: "3"

services:
  gotify:
    image: gotify/server:latest
    container_name: gotify
    ports:
      - 8070:80
    environment:
      - PUID=1066
      - PGID=1066
      - TZ=America/Los_Angeles
    volumes:
      - "/docker/appdata/gotify:/app/data"
    restart: unless-stopped
```


## Kiwix ##

```bash
sudo usermod -aG docker dave

sudo usermod -aG docker administrator

sudo usermod -aG administrator dave

sudo chown -R dave:administrator /mnt/NAS/Offline/kiwix

sudo find /mnt/NAS/Offline/kiwix -type d -exec chmod 775 {} \;
```

```yaml
version: '3.3'

services:
  kiwix-serve:
    ports:
      - 8888:8080
    image: ghcr.io/kiwix/kiwix-serve:latest
    container_name: kiwix
    volumes:
      - /mnt/NAS/Dave/Offline/kiwix:/data
    command:
      - "*.zim"
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
```


## Lidarr ##

```yaml
version: "2.1"
services:
  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/lidarr:/config
      - /mnt/Media/Audio/music:/music
      - /mnt/SSD/Downloads:/downloads
    ports:
      - 8686:8686
    restart: unless-stopped
```


## Muximux ##

```yaml
version: "2.1"
services:
  muximux:
    image: lscr.io/linuxserver/muximux:latest
    container_name: muximux
    environment:
      - PUID=1069
      - PGID=1069
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/muximux:/config
    ports:
      - 8080:80
    restart: unless-stopped
```



## Nginx (Web Server) ##


```yaml
version: "3"

services:
  client:
    image: nginx
    container_name: eurekadaycare-nginx
    ports:
      - 8000:80
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /mnt/Daycare/html:/usr/share/nginx/html
    restart: unless-stopped
```



## NZBGet ##

[Usenet block account](https://members.usenetexpress.com/member)

[https://nzbgeek.info/dashboard.php](https://nzbgeek.info/dashboard.php)

```yaml
version: "2.1"
services:
  nzbget:
    image: lscr.io/linuxserver/nzbget:latest
    container_name: nzbget
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
      - NZBGET_USER=your-nzbget-username
      - NZBGET_PASS=your-nzbget-password
    volumes:
      - /docker/appdata/nzbget:/config
      - /mnt/SSD/Downloads/usenet:/downloads
    ports:
      - 6789:6789
    restart: unless-stopped
```



## Overseerr ##

```
sudo useradd -u 1070 -M -s /usr/sbin/nologin requests

sudo usermod -aG docker requests


sudo mkdir /docker/appdata/overseerr

sudo chown -R requests:requests /docker/appdata/overseerr




```


```yaml
version: "2.1"
services:
  overseerr:
    image: lscr.io/linuxserver/overseerr:latest
    container_name: overseerr
    environment:
      - PUID=1070
      - PGID=1070
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/overseerr:/config
    ports:
      - 5055:5055
    restart: unless-stopped
```



## Prowlarr ##

```
version: "3.7"

services:
  prowlarr:
    container_name: prowlarr
    image: cr.hotio.dev/hotio/prowlarr:testing
    ports:
      - 9696:9696
    environment:
      - PUID=1067
      - PGID=1067
      - UMASK=002
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/prowlarr:/config
    restart: unless-stopped
```

* See screenshots


## Radarr ##

```yml
version: "2.1"
services:
  radarr:
    image: lscr.io/linuxserver/radarr:develop
    container_name: radarr
    environment:
      - PUID=1010
      - PGID=1000
      - UMASK=002
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/radarr:/config
      - /mnt/Media/Video/movies:/movies
      - /mnt/SSD/Downloads:/downloads
    ports:
      - 7878:7878
    restart: unless-stopped
```

Restore from backup



## Readarr ##

```yaml
version: "2.1"
services:
  readarr:
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/readarr:/config
      - /mnt/Media/Books:/books
      - /mnt/SSD/Downloads:/downloads
    ports:
      - 8787:8787
    restart: unless-stopped
```


## Sonarr ##

```yaml
version: "2.1"
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:develop
    container_name: sonarr
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/sonarr:/config
      - /mnt/Media/Video/shows:/shows
      - /mnt/SSD/Downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped
```

Restore from backup


## Tautulli ##

```
sudo useradd -u 1062 -M -s /usr/sbin/nologin tautulli

sudo usermod -aG docker tautulli
```

```
version: "2.1"
services:
  tautulli:
    image: lscr.io/linuxserver/tautulli:latest
    container_name: tautulli
    environment:
      - PUID=1062
      - PGID=1062
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/tautulli:/config
    ports:
      - 8181:8181
    restart: unless-stopped
```



## Uptime Kuma ##

Docker install is basic

Settings are intuitive



## Watchtower ##

```yaml
version: '3'

services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    environment:
      - TZ=America/Los_Angeles
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command:
      --schedule "0 15 3 * * *"
    restart: unless-stopped
```
* this example runs at 3:15am every day. It uses six places instead of the normal five of cron.



## Wiki.js ##

```yaml
version: "3"
services:

  db:
    image: postgres:11-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: your-password
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: your-db-password
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "3000:3000"

volumes:
  db-data:
```