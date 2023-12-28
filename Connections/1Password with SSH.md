---
created: Sat 2023-12-02 @ 01:19 PM
modified: Wed 2023-12-27 @ 04:02 PM
---
> [!abstract] Summary
> If you use 1Password you can use it to login to your server using Windows Terminal


Some of this is gleaned from [here](https://developer.1password.com/docs/ssh/get-started/)


==IF YOU'VE ALREADY DONE THE INITIAL WINDOWS AND 1P SETUP, SKIP TO [[1Password with SSH#Key Pair Creation|KEY PAIR CREATION]] CREATION BELOW==


# Requirements for Windows

- 1P subscription / paid account
- 1P for Windows
- Microsoft OpenSSH
- Windows Hello must be configured to unlock 1P


# Initial Windows Setup

- Check if the OpenSSH Authentication Agent service is installed and running

	Before you can turn on the SSH agent in 1Password, you'll need to check if the OpenSSH Authentication Agent service is installed:

1. Go to **Services** > **OpenSSH Authentication Agent** (press Win + R on your keyboard to open the Run window, type `services.msc` and press Enter or click OK).
    
2. Look for the **OpenSSH Authentication Agent** in the list of services.
    
	*If you don't see the OpenSSH Authentication Agent in the list, skip ahead to the steps to [turn on the SSH agent in 1Password](https://developer.1password.com/docs/ssh/get-started/#turn-on-the-ssh-agent-in-1password).
        
3. Select **OpenSSH Authentication Agent** in the list of services and press Enter.
    
4. In the "Startup type" menu, choose **Disabled**. If the service status shows the agent is running, click **Stop**.
    
5. Click **Apply**, then click **OK**.

	The 1Password SSH agent can then take over and listen on the system-wide pipe at `\\.\pipe\openssh-ssh-agent`.


- Turn on the SSH agent in 1Password

1. Open the 1Password app, then click your account or collection at the top of the sidebar and choose **Settings** > **Developer**.

2. Select the checkbox to **Use the SSH agent**.

3. Select the checkbox to **Display key names when authorizing connections** from the security section.
	- When you turn on "Display key names when authorizing connections", it's easier for you to identify which key 1Password is requesting access to when authenticating a request from an SSH client.

4. [Adjust the authorization options](https://developer.1password.com/docs/ssh/get-started/#adjust-your-authorization-options) for when and how often the SSH agent will ask you to approve SSH requests.


- Make sure the SSH agent keeps running, even when the 1Password app is closed:

1. Open 1Password, then click your account or collection at the top of the sidebar and choose **Settings** > **General**.

2. Select the checkbox to **Keep 1Password in the notification area**.


- Paste this into Command Prompt to configure your SSH or Git client

```
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```



# Initial Server Setup

From Command Prompt:
```
ssh -o IdentitiesOnly=yes administrator@192.168.10.10
```

From the users directory
```
mkdir .ssh

touch .ssh/authorized_keys

chmod 700 -R .ssh
```


# Key Pair Creation

On the server you'll want to connect to:
```
ssh-keygen -t rsa
```

At the prompt, set the filename and location
```
/home/administrator/.ssh/admin-crow-vm-windows
```
* Set a passphrase that is known to you if you want. 1P will remember that for you as well

```
cat /home/administrator/.ssh/admin-crow-vm-windows.pub >> /home/administrator/.ssh/authorized_keys
```
* This will add it to the end of the file and not overwrite the exiting key(s)

```
rm /home/administrator/.ssh/admin-crow-vm-windows.pub

exit

cls
```


# Windows Terminal Setup

In Windows Terminal on the machine you want to add server access to:

- Download the key from the server
```
scp -o IdentitiesOnly=yes administrator@192.168.40.110:~/.ssh/admin-crow-vm-windows C:\Users\mrjoh/.ssh/
```
* Enter the administrator's server password

In File Explorer, open the C: / Users / mrjoh / .ssh / config file (create the file if you haven't already) and add this:
```
Host          admin-crow-vm-windows
    HostName      192.168.40.110
    User          administrator
    Port          22
    IdentityAgent \\.\pipe\openssh-ssh-agent
    IdentityFile  ~/.ssh/admin-crow-vm-windows
```
* Save

*It should be noted that if you decide to rename the key that works fine but just make sure you rename it on the server AND on Windows*

In the Windows Terminal settings, duplicate another profile (or create a new one)

* Rename it to whatever you're connecting to (Server-Name, etc.)

* Put this into the command line:
```
ssh plex_vm-windows
```


# Adding Key to 1P

The key has been downloaded to Windows but now you need to get it into 1P as well

- In 1P, click on New Item and then select SSH Key

	- Name it whatever you want (ie. Server-Name)
	- Click on Add Private Key -> Import a Key File and pick the key you just downloaded from the server
	- Click Save


# Finishing Up

Now in Command Prompt, you need to run this for Windows Terminal to recognize there's a new key to work with:
```
ssh-add -l
```

Now open up the new profile in Windows Terminal and the 1P prompt should pop-up and ask you to authenticate with your fingerprint or password.
