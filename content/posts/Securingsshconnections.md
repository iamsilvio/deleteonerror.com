---
title: "Securing ssh connections with ed25519 keys"
subtitle: "I have a secure password, why isn't it enough?"
date: 2019-06-20T19:11:27+01:00
draft: false
tags: ["security", "devEnv", "ssh", "ed25519","Cryptography"]
categories: [itstuff]

aliases:
    - /posts/Securingsshconnections/
---


The problem is not that's to easy for you to log in, the problem is that everyone else can try it too. If there are no additional mechanisms in place, such as *fail2ban*, an attacker will have endless time to try to guess your password or even worse the root password.

## Step 1: Create a ssh key

First lets start with the key generation for your client

``` sh
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/example.com_id_ed25519
```
<!--more-->
The used options are

- `-a` When saving a private key this option specifies the number of KDF (key derivation function) rounds used.  Higher numbers result in slower passphrase verification and increased resistance to brute-force password cracking (should the keys be stolen).
- `-t` Specifies the type of key to create. The possible values are `dsa`, `ecdsa`, `ed25519`, or `rsa`.
- `-f` Specifies the filename of the key file.

I chose a password for all of my keys and you should also consider to do so.

## Step 2: copy the key to your Server

``` bash
ssh-copy-id -i ~/.ssh/example.com_id_ed25519.pub [AwesomeServer]
```

## Step 3: configure the ssh key

To configure your ssh keys you have to edit `~/.ssh/config`

I prefer to have a general config section and just specify the host specific stuff per host

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

## Step 4: Add the key to your ssh-agent

The last optional step for the Client is to add your ssh key to your ssh agent

``` bash
ssh-add -K ~/.ssh/example.com_id_ed25519
```

## Step 5: Configure the ssh daemon on the Server

open the sshd config `/etc/ssh/sshd_config` and set the following values

``` config
# Disables the Password Authentication acording to RFC-4256 ('keyboard-interactive' authentication scheme)
ChallengeResponseAuthentication no
# Disables the Password Authentication acording to RFC-4252 ('password' authentication scheme)
PasswordAuthentication no

# Pam (Pluggable Authentication Modules) we dont want anything other than ssh key to work so we disable this setting
UsePAM no

# Deny root to logon via ssh
PermitRootLogin no
```

And restart the SSH Daemon `sudo systemctl reload sshd`.

## References

[ssh config documentation](https://www.ssh.com/ssh/sshd_config/)
[RFC-4256- Generic Message Exchange Authentication for the Secure Shell Protocol (SSH)](https://www.rfc-editor.org/rfc/rfc4251.html)
[RFC-4252 -  The Secure Shell (SSH) Authentication Protocol](https://www.rfc-editor.org/rfc/rfc4252.html)

## Appendix

### SSH JumpHost configuration

To add the ability to use one of your Servers as Jump host to the others you can add the following line to the clients `~/.ssh/config`.

``` config

#----------------Servers
Host [YourJumpHost]
  ...
  ...
Host [AwesomeServer]
  ProxyJump [YourJumpHost]
```
