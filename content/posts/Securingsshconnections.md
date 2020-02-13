---
title: "Securing ssh connections with ed25519 keys"
date: 2019-06-20T19:11:27+01:00
draft: true
tags: ["security", "devEnv", "ssh", "ed25519","Cryptography"]
categories: [itstuff]
---

TODO: Introduction

## Step 1: Create a ssh key

First lets start with the key generation for your client

``` sh
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/example.com_id_ed25519
```

The used options are

- `-a` When saving a private key this option specifies the number of KDF (key derivation function) rounds used.  Higher numbers result in slower passphrase verification and increased resistance to brute-force password cracking (should the keys be stolen).
- `-t` Specifies the type of key to create. The possible values are `dsa`, `ecdsa`, `ed25519`, or `rsa`.
- `-f` Specifies the filename of the key file.

I chose a password for all of my keys and you should also consider to do so.

## Step 2: configure the ssh key

To configure your ssh keys you have to edit `~/.ssh/config`

I prefere to have a general config section and just specify the host specific stuff per host

``` config
Host *
     ForwardAgent no
     ForwardX11 no
     ForwardX11Trusted yes
     Protocol 2
     ServerAliveInterval 60
     ServerAliveCountMax 30
     Port 22
     AddKeysToAgent yes # this only applies on macOS
     UseKeychain yes    # this only applies on macOS

#----------------Servers
Host [AwesomeServer] # give it a name
  HostName [1.1.1.9] # ip of your server
  User [INSERT_YOU_USER]
  IdentityFile ~/.ssh/example.com_id_ed25519

#----------------Services

Host github.com # Has to be the domain name you want to use the key for
PubkeyAuthentication yes
IdentityFile ~/.ssh/github_id_ed25519
```

## Step 3: Add the key to your ssh-agent

The last optional step is to add your ssh key to your ssh agent

``` bash
ssh-add -K ~/.ssh/example.com_id_ed25519
```
