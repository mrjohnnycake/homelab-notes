## Install ##

- In Portainer, go to the node you want to install NPM on and click Networks and "Add network". Name it "npm-main" and then click "Create the network"

Create a stack and add this BUT do not deploy it until you've followed the other instructions below:
```yaml
version: '3'
services:
  nginx-proxy-manager:
    container_name: nginx-proxy-manager
    image: 'jc21/nginx-proxy-manager:latest'
    restart: always
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    environment:
      - DB_MYSQL_HOST=nginx-proxy-manager-db
      - DB_MYSQL_PORT=3306
      - DB_MYSQL_USER=npm
      - DB_MYSQL_PASSWORD=${DB_MYSQL_PASSWORD}
      - DB_MYSQL_NAME=npm
    volumes:
      - /docker/appdata/nginx-proxy-manager/container-data/data:/data:Z
      - /docker/appdata/nginx-proxy-manager/container-data/letsencrypt:/etc/letsencrypt:Z
    networks:
      - npm-db
      - npm-main

  nginx-proxy-manager-db:
    container_name: nginx-proxy-manager-db
    image: 'jc21/mariadb-aria:latest'
    restart: always
    environment:
      - MYSQL_DATABASE=npm
      - MYSQL_USER=npm
      - MYSQL_PASSWORD=${DB_MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    volumes:
      - /docker/appdata/nginx-proxy-manager/container-data/mysql:/var/lib/mysql:Z
    networks:
      - npm-db

networks:
  npm-db:
  npm-main:
    external: true
```

- Before deploying, go down the page and click on "Add an environment variable" and click on it a second time to create two
	- For the first one, put "DB_MYSQL_PASSWORD" (without the quotes) in the name box and then generate a password in 1P and put it in the value box
	- For the second one, put "MYSQL_ROOT_PASSWORD" (without the quotes) in the name box and then generate a password in 1P and put it in the value box
	* You won't need to remember either of these passwords as they'll always be accessible in Portainer

- Now click "Deploy the stack"

- Open up the interface to confirm it's working:
```
http://your-ip-address:81/login
```


## Setup ##

Use the default login credentials
```
Email address: admin@example.com
Password: changeme
```

- Change the email address to yours (though it probably doesn't matter as it's more of a username)
- Change the password

None of this will work without port forwarding in the router. Consult the networking document to see how to do that.


## Usage ##

The first step should be to upload an SSL certificate. Follow the directions on the Cloudflare document.
	- Click on SSL Certificates -> Add SSL Certificate -> Custom
	- Give it a name and select the certificate files on your computer and click Save

To add a domain, go to Hosts (or Dashboard)-> Proxy Hosts -> Add Proxy Host
```
Details pane
Domain Names: subdomain.websites.com
Forward Hostname / IP: 192.168.___.___
Forward Port: 5005
To begin, turn on Websockets Support only

SSL pane
SSL Certificate: select the certificate for this domain
Turn on Force SSL at minimum

Save
```

* After it is set up and working you can go back and turn on Cache Assets and Block Common Exploits, as needed

If this is a new domain you are setting up you'll likely need to consult the troubleshooting section below.


## Troubleshooting ##

After spending a whole day, multiple times, trying to get NPM with a domain name working I found out a major annoying problem with getting it setup. Luckily it is an easy fix.

- In Cloudflare, under the DNS section for your domain, turn off the orange "Proxied" setting on your A and CNAME records until after you have initially setup a NPM proxy host. After you have it working properly (a domain is pointed at your public IP and that domain is setup inside of NPM and is successfully connecting to a service) ONLY THEN can you turn the orange Proxied status back on in Cloudflares DNS records. Whew! Pain in the ass to figure that one out!
- I would think that if you ever purge your cache or add a new domain in Cloudflare that you'd have to do the same thing to get it working again.

