---
title: Readable Note Name that describes the note
created: Sat 2024-06-29 @ 08:39 PM
tags: []
type: 
modified: Sat 2024-06-29 @ 08:40 PM
---
# Layout

Starting Out
	- Where to start?
		- Start with what you'd like to do
	- What Can You Do?
		- Getting rid of the cloud
	- Home Environment
		- Space
		- Racks

Hardware
	- File Systems
	- Power Backup

Software
	- Hypervisors vs. bare metal
	- VMs vs CTs
	- Services

Networking
	- 

Maintenance & Backups

Resources
	- YouTube
	- Blogs
	- Websites
	- Social Media

# Writing

https://perfectmediaserver.com/

https://www.reddit.com/r/homelab/

https://www.reddit.com/r/homelab/wiki/index/

- Storage needs
- Get comfortable with the command line
- Linux
- Don't be afraid to explore and make mistakes (labs get messy)
- Everything has been done before (Google and Reddit are your friends!)
- Buying hardware
	- Upfront costs vs operating costs
- Jack of all trades, master of some (the learning curve)
- Smart Home


# Installation

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