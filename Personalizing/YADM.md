---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 04:17 PM
---
*Yet Another Dotfile Manager


https://yadm.io/docs/common_commands


#### Installation ####

```
sudo apt-get install yadm
```


#### First things first... ####

- Create the repo on Github and take note of the SSH link

git@github.com:myusername/myreponame.git


- Clone into the repo to download it to your machine
```
yadm clone https://github.com/myusername/myreponame.git

yadm fetch -f
```

Add user info to the repo for commits:
```
git config --global user.email "10XXXX689+myusername@users.noreply.github.com"

git config --global user.name "My Name"
```


#### Usage ####

Add a file:
```
yadm add .gitconfig
```

Check status of repo and anything that needs to get down:
```
yadm status
```

Commit changes:
```
yadm commit -m "note about what was changed"
```

Push changes to the GitHub repo:
```
yadm push
```

Pull changes down from the GitHub repo if changes were made there:
```
yadm pull
```

