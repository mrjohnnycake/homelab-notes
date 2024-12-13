---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:47 PM
---

# Cloudflare Tunnel with App Access via Docker

This tutorial will allow you to expose an app running on your server to the internet in a secure manner.

This was setup with my own docker Homepage instance that can be found at `start.mrjohnnycake.com`. I think it will be easier to show the simplicity of tunnel creation by using this site as an example.

This tutorial assumes you have a Linux server running Docker with Docker Compose v2 (for lower versions please adjust your Docker commands)

There will be another step you'll want to do after your app is exposed (assuming you want to control who can access your app) and that is adding an authentication method so that only those with proper permission can see your app. This tutorial doesn't cover that but it is easily done thru Cloudflare as well. In the meantime, when you're done with this tutorial, anyone online will be able to see your app / website.

Note- *There is no need to set any port forwarding rules on your router with this setup*

## Steps

- Login to Cloudflare and click on the domain name you want to work with
- First go to DNS and delete the DNS entries for your domain
	- As of my functioning test, I started with zero DNS entries. The 
- Next click Access on the left side navigation and click Launch Zero Trust
- Select your account
- Go to Networks --> Tunnels and click Add a tunnel
- Choose "Select Cloudflared"
- Name your tunnel
- Under "Choose your environment" go to the Docker tab and copy the command under "Install and run a connector". Paste it in a notepad or something for now.
- Click Next
- Under Public Hostnames, enter the info for your app you are trying to expose.
	- For my Homepage instance, I'll enter `start` under Subdomain and select `mrjohnnycake.com` under the dropdown under Domain
	- Under Type I'm selecting HTTP. This doesn't make sense since we want to access the site using HTTPS but choosing that here will cause everything to not work so just go with this for now and you can confirm for yourself later that your app is being exposed with HTTPS.
	- Under URL, I entered `192.168.70.160:3005` which is the IP and port my app is at on my server.
	- Click Save Tunnel

Your tunnel is currently inactive because you haven't set up your server yet so let's do that now.

- Connect to your server via SSH
- We're going to setup the Docker container that will run the tunnel connector on our end. My Docker apps are in `/opt` so that's what I'll be using here but you're free to put your directory wherever you see fit.
- Run these commands (changing `cloudflared-mrjc` to whatever you want to call your directory)

```
sudo mkdir -p /opt/cloudflared-mrjc

sudo chmod -R 755 /opt/cloudflared-mrjc

cd /opt/cloudflared-mrjc
```

Alternately, you can combine all of this into one command if you like:
```
sudo mkdir -p /opt/cloudflared-mrjc && sudo chmod -R 755 /opt/cloudflared-mrjc && cd /opt/cloudflared-mrjc
```

Create the `compose.yaml` (if you're running Docker Compose v1 you'll want to name it `docker-compose.yaml`)

```
sudo nano compose.yaml
```

Paste this into the file:
```
services:
  cloudflared-mrjc:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared-mrjc
    command: ["tunnel", "--no-autoupdate", "run", "--token", "${TOKEN}", "--hello-world"]
    env_file:
      - .env
```

- I've named my tunnel `cloudflared-mrjc` because I have multiple domains I want to use tunnels for so I need to keep track of which is which. So change those two references to whatever you want.

- Save the file and exit

Next you need to create the `.env` file to "hide" your token a bit from prying eyes:
```
sudo nano .env
```

For this part you'll need that Docker command you copied from Cloudflare. The last part of it is your token so you'll need that now. Paste this in the `.env` file and replace "your-token" with the token found in that Cloudflare command:
```
TOKEN="your-token"
```

My .env file looks like this (using a changed token here for my privacy):
```
TOKEN=eyJhIjoiMjdmABCDEFGmM2M2YTYxMDU0NTM1ZmY4Nzc2NWEwNmYiLCJ0IjoiNDBlNTU4OWQtMWRmYi00MzVhLWI1234567DEzNDEwMmQwIiwicyI6Ik1qSTVZek5oWkdVdE56WTJOQzAwTURKakxUbGhObVXYZqUmpaakJrWVRVMSJ9
```

- Save the file and exit

Now we need to spin up the Docker container:
```
sudo docker compose up --build -d
```

- If you get a "mapping" error it means something is wrong with the compose file formatting. It's almost always a space or tabbing issue. Play around with it and try again.

Now we just want to check the Docker log to see if everything looks good:
```
sudo docker logs cloudflared-mrjc
```

- As long as you don't see any errors you should be good

Now go back to the Cloudflare Tunnels page and hit refresh. You should see that your tunnel is up and healthy.

As one more thing to check, go back to the DNS records for your domain. You'll see that Cloudflare has created a new DNS record that points to the tunnel you just setup. Clever.

That's it. If you have any problems I suggest emptying Cloudflare's cache of your domain by going to Cloudflare --> your.domain --> Overview --> Configure Caching --> Purge Everything. 


## Adding Additional Apps

So you've setup your first app with Cloudflare Tunnel and you want to add another one. No problem.

Just go back to the tunnel you just created and selecting Configure from the "three dots" menu and click Add a public hostname. Do what you did before, changing the subdomain and URL as needed.

- Note- You have to use the same domain for this tunnel. I you want to use a different domain you'll have to create another tunnel for that purpose. Just follow the same steps and name it something different




# DNS Settings

Should like something like this without a tunnel:

```
-A       domain.com    public IP (using What's My IP)

-CNAME       www               public IP (What's My IP)

-CNAME   subdomain             domain.com
```

* All of the orange proxied sliders can be on

* Until you get it working, you can put the domain into developer mode, if you'd like.
	* At the time of this writing, it's under domain.name --> Overview


After setting up the tunnel it should look like this:

```
-CNAME    domain.com    2XX2cn0-a2fd-XXXX-XXXX-2b76XXXde6.cfargotunnel.com

-CNAME    www               domain.com

-CNAME    othersubdomain             domain.com
```

* the main tunnel cname will say something about flattening and that's fine




# App Authentication

For my needs, I'm going to create a policy where whenever I or anyone else first connects to my Homepage instance at `start.mrjohnnycake.com` they will be presented with a Cloudflare form to enter their email address. If their email address has been approved by me they'll get an email where they can click a link and be granted access for one month before they'll have to do it over.

It's easy to do.

* Login to Cloudflare
* Select your domain
* Click Access on the sidebar then Launch Zero Trust
* Select your account
* Click Access --> Applications --> Add an application
* Select Self Hosted

Under Application Configuration:
- Enter the name of your app under Application name, the amount of time you want the access to last under Session Duration, your subdomain (if any), and your domain name.

I everything else as default on this page and click Next 

On the Add Policies page:
- I'm going to name my Policy "Email"
- I select Allow under Action because I want approved emails to be allowed to see the site
- I set the Session Duration to 1 month because I don't want to be doing this all the time and it's a pretty low security risk app as it stands already
- Under Configure Rules, I'm going to select Emails under Selector and then in Value I'm going to enter my email because that's me and I want to access my own page.
	- If there's anyone else you want to be able to access you app you can enter their emails here and just hit Enter after each one
- Click Next on the bottom of the page

On the Setup page:
- I don't think I need any of this stuff so I just click Add Application on the bottom of the page

Now go to your app / website. You'll be greeted by a login code page and you'll need to enter your email and then click Send me a code

Now go to your email and go the email you just received from Cloudflare. You have two choices here:
- You can copy the code and go back and paste it into the Cloudflare page
- Or you can be lazy like me and just click the link because that's what I usually do. Just close that Cloudflare page because you won't need it for another month

Either choice you choose you'll be allowed to see your app / website.

That's it.


Note- *I leave this policy off (or just not setup) for apps that already have their own built-in or third party authentication methods (Google auth, Plex auth, etc.)*


