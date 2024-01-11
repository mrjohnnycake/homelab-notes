---
title: Installing and managing Portainer
created: Sat 2024-01-06 @ 09:42 AM
tags:
  - containers
  - stacks
  - docker
  - compose
type: tutorial
modified: Sat 2024-01-06 @ 09:42 AM
---
# Install Portainer

```
sudo docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/appdata/portainer:/data:rw portainer/portainer-ce
```

Go to your IP address with port 9000 (http://192.168.40.110:9000) and set it up


# Setting Up Edge Agents

You only need to install Portainer once. For the other CTs or VMs, you can add them as Portainer environments.

* First, go to Environments-->Groups-->Add Group
	* I named it Gypsy for the server it needs to connect to
	* Put a description in there if you want
	* Select the associated environment
	* Click 'Create the group"

* Environments-->Add Environment
	* Select Docker Standalone and click Start Wizard
	* Select Edge Agent
	* For the name, you are naming the remote environment you are setting up. So lets say this Portainer server you are currently working on is 192.168.100.1 and you want to connect to 192.168.100.2, you'll want to name it after the service you're connecting to on 192.168.100.2
		* I use "VM-Name @ Server" as my naming scheme
	* Leave the Portainer server URL the same. This should be the IP of the Portainer instance you are currently working in
	* Click on the "More settings" dropdown and set the group to the group you just made
	* Click Create
	* The page will now give you a command to run but just ignore that and click Close
	* Select the environment you just created.
	* Go to the Docker Standalone tab
	* Copy and paste the command given into the terminal for your server (remote environment) and remember to add "sudo" to the front of it
	* After that's done running, go back to the Portainer page and refresh it. It should now have some "Edge Information" on the top which means that it was set up properly.
	* While still on that page, enter the IP of your server in the Public IP box and click Update Environment

The environment is now added and you will see it if you head to the Home page in Portainer. But to be able to access it you'll first need to set up some port forwarding rules in your local network router.



# Uninstalling
#### Removing Edge Environments

```
sudo docker stop portainer_edge_agent

sudo docker rm portainer_edge_agent

sudo docker volume rm portainer_agent_data
```


#### Remove Portainer

```
sudo docker stop portainer

sudo docker rm portainer

sudo rm -rf /docker/appdata/portainer
```
