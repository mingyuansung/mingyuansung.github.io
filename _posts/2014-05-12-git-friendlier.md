---
layout: post
title: Making Git Friendlier
---


* Example ~/.gitconfig:  

    ```
    [user]  
        name = Your Name  
        email = yourname@sourceintelligence.com  
    [core]  
        quotepath = false  
        excludesfile = ~/.gitexcludes  
        autocrlf = input  
    [color]  
        ui = true  
    [format]  
        pretty = oneline  
    [push]  
        default = nothing  
    [mergetool]  
        keepBackup = true  
    [alias]  
        latest = for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) \n%(committerdate:relative) by %(authorname)\n------------------------------------------'  
        lol = log --graph --decorate --pretty=oneline --abbrev-commit  
        lola = log --graph --decorate --pretty=oneline --abbrev-commit --all  
```  
### Enable auto-completion and a custom bash prompt

* Download [git-completion.bash](https://raw.github.com/git/git/master/contrib/completion/git-completion.bash) to your home directory
* Download [git-prompt.bash](https://raw.github.com/git/git/master/contrib/completion/git-prompt.sh) to your home directory
* Add the following to your bash config (`~/.profile` on OSX)  

    ```
source ~/.git-completion.bash
source ~/.git-prompt.sh
GIT_PS1_SHOWDIRTYSTATE=true
GIT_PS1_SHOWSTASHSTATE=  
```
* Add the Git prompt function to your PS1 bash prompt (in `~/.profile` on OSX)

* Example: `PS1='[\u@\h \W$(__git_ps1 " (%s)")]\$ '`  
* Open a new terminal or `source ~/.profile` to see if the changes worked