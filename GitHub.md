# GitHub CLI Setup

Install Github CLI (this is all one command)
```
type -p curl >/dev/null || sudo apt install curl -y
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

```
gh auth login
```

```
Select GitHub.com

Select SSH

Yes
```

- For setups that don't need cron or other automation that can't enter a passphrase, enter a passphrase

- For using cron or something that can't enter passphrases itself, hit enter to leave the passphrase blank

```
Name it or just hit enter

Login with a web browser
	- Hit enter
	- Click the link
	- Copy the one-time code over
```

```
rm ~/.ssh/id_ed25519.pub
```



# Alternate Method for Authentication #

```sh
ssh-keygen -t rsa -b 4096 -C "youremail@gmail.com"
```

```
/home/administrator/.ssh/machine-keys/tomservo-github
```

Start SSH Agent
```
eval "$(ssh-agent -s)"
```

```
ssh-add ~/.ssh/machine-keys/tomservo-github
```

Now add your public key
```
gh ssh-key add ~/.ssh/machine-keys/tomservo-github
```

```
rm ~/.ssh/machine-keys/playground_vm-github.pub
```


# Usage

#### Create a repo:
```
gh repo create
```

#### Clone a repo:
- To be able to clone a repo you'll need to have already need to have setup GitHub CLI Setup below
- You'll also need a token (see below)
- In SSH on the server where you want to clone the repo to, change directory to the parent directory of where you want your repo to live
	- If you want the repo to be `/docker/appdata/repo-directory` then you'd `cd /docker/appdata` before running the next command
- The command to run is structured like this 
```
git clone https://TOKEN@github.com/OWNER/REPO.git
```
- Paste in the specific info until it looks something like this:
```
git clone https://ghp_Vm11kPZuXXXXX324790VZMe4HMfd2k8D0D@github.com/yourusername/yourrepo.git
```

#### Create a token:
- In GitHub, go to `Settings -> Developer Settings -> Personal access tokens -> Tokens (classic)`
- Click on `Generate new token` and then click `Generate new token (classic)`
- Give it a name
- Set an expiration date (I did a year later)
- Under Select Scopes, click on `repo`
- Click Generate token
- Copy the token somewhere until you're done

 NOTE- *I don't save the token to 1P as a find it's not needed after the initial cloning. If you run into a problem just re-clone the repo*

#### Re-Cloning a Repo:
- For most repos, compare what's in the repo on GitHub to what is needed to run your app and copy the missing data that's NOT in the repo
	- Assume you have a working directory on your server called `/docker/appdata/docker-app` that contains your GitHub repo
	- And assume inside of that directory is folder 1, folder 2, folder 3, file 1 and file 2
	- But on the GitHub site you see that the only data that GitHub is watching is folder 1, folder 2, and file 1
	- In this case you would want to copy folder 3 and file 3 in your working server directory to `/docker/appdata/temp-directory` or `/docker/appdata/docker-app-bak`
- Next you'll delete `/docker/appdata/docker-app`
- Clone the repo following the instructions above
- After confirming everything is where you want it, copy folder 3 and file 3 back into the repo so that your app will work properly
