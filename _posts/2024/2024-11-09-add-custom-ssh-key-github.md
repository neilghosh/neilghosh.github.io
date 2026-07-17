---
layout: post
title: "Add custom ssh key for github"
description: "In this post, we will generate generate a custom ssh key and set it for githib.com account."
---

The recommended way of accessing (cloning/fetching/pushing code) github repos via command line is to generate ssh key pair (private key and public key) and [setting the public key content in the Github's settings](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).


Generating key pair
```
ssh-keygen -t ed25519 -C "Your EMail"
```
Provide the file name this time.
```
Enter file in which to save the key (/Users/sabyasachi.ghosh/.ssh/id_ed25519): github
```

Once can use the default key generated (without any file name specified) globally for all git hosts, however there is a chance you are using two different git servers and want to keep the keys separate. This becomes really important in cases you are using the same machine for work and personal git repos.

```
laptop % ls -ltr ~/.ssh/
total 56
-rw-------@ 1 sabyasachi.ghosh  staff   432 Oct 27 13:20 id_ed25519
-rw-r--r--@ 1 sabyasachi.ghosh  staff   113 Oct 27 13:20 id_ed25519.pub
-rw-------@ 1 sabyasachi.ghosh  staff   411 Nov  9 10:16 github
-rw-r--r--@ 1 sabyasachi.ghosh  staff   102 Nov  9 10:16 github.pub
-rw-r--r--@ 1 sabyasachi.ghosh  staff    84 Nov  9 10:38 config
-rw-------@ 1 sabyasachi.ghosh  staff  1523 Nov  9 10:39 known_hosts
```


In this case you have to explicitly provide the key generated specifically for a git server in `~/.ssh/config` file. For example

```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/github
```

Now you can test the connection (even before attempting to clone)

```
ssh -T git@github.com
```
```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi neilghosh! You've successfully authenticated, but GitHub does not provide shell access.
```

Now we can go ahead and go all git operation.
