---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:47 PM
---

All of my personal tokens and whatnot have been changed in the examples below


- Go to Cloudflare -> yourdomain.com -> Access -> Launch Zero Trust -> Access -> Tunnels and click on "Create a Tunnel"

```
Tunnel name: whatever-you-want-to-call-your-tunnel

- click "Save tunnel"

- click Next (you'll do this part in a minute)

Subdomain: enter if needed
Domain: domain.com
Type: HTTP
URL: 192.168.70.170:8000 (local path)

- save [NAME] tunnel
```

- The DNS records should be auto-updated




## DNS Settings ##

Should like something like this to begin with:

```
-A       domain.com    public IP (using What's My IP)

-CNAME       www               public IP (What's My IP)

-CNAME   subdomain             domain.com
```

* All of the orange proxied sliders can be on

* Until you get it working, put the domain into developer mode.
	* At the time of this writing, it's under thecanpart.com-->Overview


After setting up the tunnel it should look like this:

```
-CNAME    domain.com    2XX2cn0-a2fd-XXXX-XXXX-2b76XXXde6.cfargotunnel.com

-CNAME    www               domain.com

-CNAME    othersubdomain             domain.com
```

* the main tunnel cname will say something about flattening and that's fine



## Certificates ##

You need to create a certificate under SSL/TLS-->Origin Server-->Create Certificate. Just follow the instructions and save the files to your computer.


-Upload .pem and .key certificates in the SSL tab of Nginx Proxy Manager




## Authentication ##

Done thru Cloudflare Access

* Login to Cloudflare
* Select domain
* Click Access on the sidebar
* Click “Create Access Policy” for any new apps I need access to from other locations
* I left it off for Overseerr because that app uses Plex as an auth and adding another would deter users
* No need to set up with Plex as there is a plex.tv address with auth already supplied



## [[Cloudflare Tunnel]] ##

My use of `cloudflared-1` is because I had a couple of different tunnels for two different domains

```
sudo mkdir -p /docker/appdata/cloudflared-1

sudo chmod -R 777 /docker/appdata/cloudflared-1

sudo docker run -it --rm -v /docker/appdata/cloudflared-1:/home/nonroot/.cloudflared/ cloudflare/cloudflared:latest tunnel login

sudo docker run -it --rm -v /docker/appdata/cloudflared-1:/home/nonroot/.cloudflared/ cloudflare/cloudflared:latest tunnel create crow-1
```

The tunnel key will look something like this:

	8bXXXXa4-1e35-XXXX-a8ba-86aXXXXb8187

Create the config file:

```
sudo vim /docker/appdata/cloudflared-1/config.yaml
```

Add this and change as necessary:
	* The IP should point to the NPM installation
	* Change the tunnel UUID to your new one
	* Change the domain name as needed

```
tunnel: 8bcXXXX-1e35-4325-XXXX-86a8379b8187
credentials-file: /home/nonroot/.cloudflared/8bcXXXX-1e35-4325-XXXX-86a8379b8187.json

# NOTE: You should only have one ingress tag, so if you uncomment one block comment out the others

# forward all traffic to Reverse Proxy w/ SSL
ingress:
  - service: https://192.168.40.101:4443
    originRequest:
      originServerName: domain.com

#forward all traffic to Reverse Proxy w/ SSL and no TLS Verify
#ingress:
#  - service: https://REVERSEPROXYIP:PORT
#    originRequest:
#      noTLSVerify: true
```

Install the Docker container
	* Change the tunnel number at the end to match yours

```
sudo docker create --name='cloudflared-1' --net='bridge' -e TZ="America/Los_Angeles" -v '/docker/appdata/cloudflared-1':'/home/nonroot/.cloudflared/':'rw' --restart unless-stopped 'cloudflare/cloudflared:latest' tunnel run e8589644-e62b-44da-XXXX-eb4cd4a6246
```


```
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token eyJhIjoiMjdmNmYyNDFmXXXXXXXXXXXXXXX0NTM1ZmY4Nzc2NWEwNmYiLCJ0IjoiYjk4OWMxOGEtNmI3Zi00MDkwLTk2ZmEtMTczOTRmNmExM2E4IiwicyI6IlptUTVZV0ZsTnpjdE5qYzBPUzAwTkRBekxXSmhaR0XXXXXXXXXXXXXXXXOaiJ9
```


```
sudo docker create --name='cloudflared-1' --net='bridge' -e TZ="America/Los_Angeles" -v '/docker/appdata/cloudflared-1':'/home/nonroot/.cloudflared/':'rw' --restart unless-stopped 'cloudflare/cloudflared:latest' tunnel run e8589644-e62b-44da-XXXX-eb4cd4a6246
```


Start the container in Portainer