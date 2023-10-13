---
layout: post
title:  "Authenticating with github via different accounts on the same device"
---

## Why?
I have been wanting to develop and push changes to code in my personal repository as well as my work repo but these use different accounts and thus I would have to setup auth for 2 accounts on the same device.

## How?

Github currently provides functionality to push authenticate via SSH tokens. My work account is currently setup on my device using PATs (Personal Access Tokens) at a system level so I don't have to think about authentication for that any more. To solve this issue I used SSH keys for my personal device and overrode the local git config for personal repositories.

## Steps:
1. generate a new SSH key
    * `ssh-keygen -t ed25519 -C "your_personal_email@example.com" -f ~/.ssh/<personal_key> `
    * add a passphrase on prompt
    * validate SSH key generation wil ls `-la ~/.ssh`
2. Tell the `ssh-agent`
    * `eval "$(ssh-agent -s)" && ssh-add -K ~/.ssh/<personal_key> `
3. Edit SSH config
    * edit the file `~/.ssh/config` I prefer vscode so running something like  `code ~/.ssh/config` will open that file in the editor. (You may have to generate this file if it doesn't already exist)
    * add the following to the config file: 
      ```
      Host *
        AddKeysToAgent yes
        UseKeychain yes
        IdentityFile ~/.ssh/<created_key>
      ```

4. Add the public key to you personal Github account
    * Currently this looks like: Github -> user icon -> settings -> SSH and GPG keys -> new SSH key
    * Give it a name thaats relevent and add in the public key {personal_key}.pub (you can copy this by running: `tr -d '\n' < ~/.ssh/<created_key>.pub | pbcopy`)

5. Update your local repo git config
    * Add the following: 
  ```bash
    git config --local user.name 'Your Name'
    git config --local user.email 'your.email.address@example.com'
    git config --local core.sshcommand 'ssh -i ~/.ssh/personal-key -F /dev/null'
    git remote add origin 'git@.....'
  ```

    * Within your personal repo open the `.git/config` file, it should look something like:
      ```conf
        [user]
          email = your_pers_email@example.com
          name = Your Name
        
        [core]
          sshcommand = ssh -i ~/.ssh/personal_key -F /dev/null

        [remote "origin"]
          url = git@github.com:{username}/{repo name}.git
          fetch = +refs/heads/*:refs/remotes/origin/*
      ```

6. You can validate your git config with: `git config --list`
    * local config will override system config
    * hopefully you should now be able to interact with your personal repo with separate authentication!

### Happy Shipping!

References:
* <a href="https://atornblad.se/use-a-different-user-account-for-each-git-repo">use-a-different-user-account-for-each-git-repo</a>
* <a href="https://blog.gitguardian.com/8-easy-steps-to-set-up-multiple-git-accounts/">8-easy-steps-to-set-up-multiple-git-accounts</a>
* <a href="https://therajanmaurya.medium.com/git-push-pull-with-two-different-account-and-two-different-user-on-same-machine-a85f9ee7ec61">git-push-pull-with-two-different-account</a>
* <a href="https://stackoverflow.com/questions/13103083/how-do-i-push-to-github-under-a-different-username">how-do-i-push-to-github-under-a-different-username``</a>
* <a href="https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration?ref=blog.gitguardian.com">git config</a>
