---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 03:32 PM
---
# Restoring #

I was able to successfully move an installation to another server by:
	- Putting the receipt files in the new location and setting the permissions to dave:administrator
	- Copying the /docker/appdata/paperless directory to the new location and setting the permissions to dave:administrator

So make sure to backup those directories accordingly


# Install #

On the VM:
- Make sure fstab entry for the NAS SMB share looks like this:
```
//192.168.10.5/zpool_NAS /mnt/NAS cifs credentials=/root/.smb,uid=1010,gid=1020,file_mode=0664,dir_mode=0775 0 0
```

The Paperless folder relies on three subfolders
- consume (for adding files to that Paperless will then process)
- export (for exporting purposes down the line)
- media (the actual files that have already been processed)

* The docker stack will create all of the necessary folders and mount points


```yaml
version: "2.1"
services:
  paperless-ngx:
    image: lscr.io/linuxserver/paperless-ngx:latest
    container_name: paperless-ngx
    environment:
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
      - REDIS_URL= #optional
      - PAPERLESS_URL=https://subdomain.domain.com
    volumes:
      - /docker/appdata/paperless/config:/config
      - /mnt/Paperless/consume:/data/consume
      - /mnt/Paperless/export:/data/export
      - /mnt/Paperless/media:/data/media
    ports:
      - 8010:8000
    restart: unless-stopped
```


## Setup ##

If you are importing from a former install, the only setup that needs to happen is creating the storage paths

- Storage Paths-->Create
```
Name: 2020
Path: {created_year}/{document_type}/{created_month}-{created_day}-{created_year} - {tag_list}
Pattern: 2020
```
* Also add one for 2021 and 2022 and any other years needed


- The default Settings upong install look good

* For most settings below, use "Case insensitive"

Correspondents
```
Name: My-Name
Algorithm: Any
Pattern: my-name
```

Tags example
```
Name: Home Depot
Algorithm: Exact
Pattern: Home Depot
```

Document Types example
```
Name: IRS Forms
Algorithm: Any
Pattern: 1099-MISC 1095-B
```

Saved Views
```
Go to Documents
Choose what documents you want to see
Views-->Save As...
Enter a name
Click the boxes as needed
```

- On first saved view creation, you can create an Inbox tag


All documents should be tagged with (at a minimum)
- document type (1099-MISC, debit, etc)
- source (Home Depot, California, etc.)



# Post install Scripts #

## Document Renamer ##

```
python3 /app/paperless/src/manage.py document_renamer
```


## Exporter ##

In the console for the container:
```
python3 /app/paperless/src/manage.py document_exporter /data/export
```


## Importer

To be able to import without errors, you have to add the Storage Paths before importing

Run this to import
```
python3 /app/paperless/src/manage.py document_importer /data/export
```


## Document Retagger ##

* I couldn't get this to work. The log says that the learning model hasn't been created yet
```
python3 /app/paperless/src/manage.py document_retagger
```

