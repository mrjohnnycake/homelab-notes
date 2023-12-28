---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 04:20 PM
---
#todo 

- [ ] need to auto detect distro



# Fresh Install #

Install
```
sudo apt install zsh
```

Make zsh the default shell
```
chsh -s /usr/bin/zsh
```

Run the config script
```
zsh
```

-So far I've chosen to go with option 2 where the .zshrc file gets created with a standard configuration.

You can confirm the shell has been changed by running
```
echo $SHELL
```

When you've made edits to the `.zshrc` file, reload it:
```
source ~/.zshrc
```


## Starship ##

Install:
```
sudo curl -sS https://starship.rs/install.sh | sh
```


## ZSH Autocomplete ##

```
mkdir .config/zsh/plugins && cd "$_"

git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git
```

- If on Ubuntu:
```
compaudit | xargs chmod g-w
```

```
vim ~/.config/zsh/.zshrc
```

Add at or near the top of your `.zshrc` file (before any calls to `compdef`):
```
source ~/Git/zsh-autocomplete/zsh-autocomplete.plugin.zsh
```

- Remove any calls to `compinit` from your `.zshrc` file.

- If you're using Ubuntu, add to your `.zshenv` file:
```
skip_global_compinit=1
```

- Restart your shell.

To update, do:
```
git -C ~zsh-autocomplete pull
```



# Converting Existing VMs #


- In the Windows Terminal profile you're wanting to update, change the font to "FuraMono Nerd Font"

```
sudo apt update

sudo apt install curl git yadm zsh -y

chsh -s /usr/bin/zsh
```

If on Ubuntu:
```
sudo reboot
```

If on Debian:
```
exit
```

On logging back in, select option 2 in the ZSH setup

```
sudo curl -sS https://starship.rs/install.sh | sh
```

GitHub CLI (all one command)
```
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

Authenticate GitHub
```
gh auth login
```

```
Select GitHub.com

Select SSH

Yes

No passphrase

Media VM

Login with a web browser
	- Hit enter
	- Click the link
	- Copy the one-time code over
```

```
rm ~/.ssh/id_ed25519.pub
```

```
yadm clone https://github.com/myusername/myrepo.git
```

For Ubuntu:
```
yadm checkout "/home/administrator"
```

For Debian:
```
yadm fetch -f
```

Exit and reopen

```
cd .scripts/setup
```

For Ubuntu:
```
./ubuntu.sh
```

For Debian:
```
./debian.sh
```

