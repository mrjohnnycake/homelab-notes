---
title: Kiwix setup and documentation
created: Sat 2023-12-16 @ 12:45 PM
tags:
  - Homelabbing
type: 
modified: Wed 2024-07-24 @ 07:22 PM
---
# To Do's

- [ ] #kiwix Setup automatic updating of offline .zim files #Homelabbing 
	```
	wget https://dumps.wikimedia.org/kiwix/zim/wikipedia/wikipedia_en_all_maxi_2022-05.zim
	```


# Installation

```bash
sudo usermod -aG docker username

sudo usermod -aG docker administrator

sudo usermod -aG administrator username

sudo chown -R username:administrator /mnt/NAS/Offline/kiwix

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
      - /mnt/NAS/USERNAME/Offline/kiwix:/data
    command:
      - "*.zim"
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
```

