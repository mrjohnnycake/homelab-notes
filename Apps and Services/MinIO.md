---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 02:10 PM
---
[Install Video followed](https://www.youtube.com/watch?v=2iVhfbrP_-o)


# Install MinIO (LXC)

### Install Go ###

In Windows Terminal

```
lxc-attach --name 120

wget -c https://dl.google.com/go/go1.18.linux-amd64.tar.gz

tar xvf go1.18.linux-amd64.tar.gz

chown -R root:root ./go

mv go /usr/local

echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile

source /etc/profile

go version
```

* As long as the version shows up:
```
rm go1.18.linux-amd64.tar.gz
```

### Install MinIO ###

```
wget https://dl.min.io/server/minio/release/linux-amd64/minio

useradd --system minio --shell /sbin/nologin

usermod -L minio

chage -E0 minio

mv minio /usr/local/bin

chmod +x /usr/local/bin/minio

chown minio:minio /usr/local/bin/minio

sudo touch /etc/default/minio

sudo echo 'MINIO_ROOT_USER="username"' >> /etc/default/minio

sudo echo 'MINIO_VOLUMES="/mnt/minio/backups"' >> /etc/default/minio

sudo echo 'MINIO_OPTS="-C /etc/minio --address :9000 --console-address :46699"' >> /etc/default/minio

sudo echo 'MINIO_ROOT_PASSWORD="password"' >> /etc/default/minio

mkdir /mnt/minio/backups

mkdir /etc/minio

chown minio:minio /mnt/minio/backups

chown minio:minio /etc/minio
```

### Install MinIO Service ###

```
wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service

sed -i 's/User=minio-user/User=minio/g' minio.service

sed -i 's/Group=minio-user/Group=minio/g' minio.service

mv minio.service /etc/systemd/system

systemctl daemon-reload

systemctl enable minio

systemctl start minio

systemctl status minio
```

* Confirm it's active and running

* Take note of the IP address


### Firewall Settings ###

```
ufw default deny incoming

ufw default allow outgoing

ufw allow ssh

ufw allow 9000

ufw allow 46699

ufw allow http

ufw allow https

ufw enable

ufw status verbose
```

Confirm everything looks good

* Make sure you forward ports 9000 and 46699 on your router to your server


### Console Setup ###

* Open it by going to http://ip-address:46699 that you wrote down earlier

Set Server Location
* Go to Configurations
* Set the Server Location to whatever you'd like (mine is "gizmonics-institute")
* Save
* Restart the instance by clicking on the popup or by rebooting the CT

* Identity → Users → Create User
    * Enter a User Name (essentially an Access Key)
    * Enter a Password (essentially a Secret Key)
    * Select "readwrite" under Assign Policies
    * Save

* Buckets → Create Bucket
    * Only needs a name
    * Click "Create Cucket"

That's it in MinIO. Now is a good time to restart your Proxmox server.

* Node → Reboot


### Certificate ###

```
apt install software-properties-common

add-apt-repository universe

apt update

apt install certbot

certbot certonly --standalone -d minio-server.your_domain

cp /etc/letsencrypt/live/minio-server.your_domain_name/privkey.pem /etc/minio/certs/private.key

cp /etc/letsencrypt/live/minio-server.your_domain_name/fullchain.pem /etc/minio/certs/public.crt

chown minio:minio /etc/minio/certs/private.key

chown minio:minio /etc/minio/certs/public.crt

systemctl restart minio
```

