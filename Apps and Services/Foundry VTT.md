---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:57 PM
---
# Install

```
sudo useradd -u 421 -M -s /usr/sbin/nologin foundry

sudo usermod -aG docker foundry

sudo usermod -aG foundry dave
```


```yml
version: "3.8"

services:
  foundry:
    image: felddy/foundryvtt:release
    container_name: foundry-vtt
    hostname: foundryvtt
    volumes:
      - /docker/appdata/foundry-vtt:/data
      - /mnt/FoundryVTT:/data/Data
    environment:
      - FOUNDRY_PASSWORD=your-password
      - FOUNDRY_USERNAME=your-username
      - FOUNDRY_ADMIN_KEY=admin-password
      - CONTAINER_PRESERVE_CONFIG=true
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    ports:
      - 30000:30000/tcp
    restart: unless-stopped
```
