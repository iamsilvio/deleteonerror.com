---
title: "Upgrade from Debian 10 (buster) to 11 (bullseye)"
subtitle: "it's time for an upgrade"
date: 2021-07-05T09:01:56+02:00
draft: false
tags: ["security", "selfhosted", "oss", "debian","bullseye", "buster"]
categories: [itstuff, HowTo]

hide_reading_time: true
implementation_time: 15

twitter_creator_id: 94538574
---

Having an up-to-date system is a big challenge with increasing security problems, systems should be updated as early as possible. Debian 11 will be released this fall, since 12-03-2021 the next version is in [Hard freeze](https://release.debian.org/bullseye/freeze_policy.html#hard) state, which means that after the freeze only minor changes will take place.  

> so: it's time for an upgrade

## Prepare Debian 10 for the upgrade

Update and upgrade all installed packages to the latest version by running:

``` bash
sudo apt update && sudo apt upgrade
```

For the upgrade we need to install the latest version of **GNU Compiler Collection**

``` bash
sudo apt install gcc-8-base
```

## Update the package source for bullseye

We need to edit the package sourcelist to tell the system where the new sources for debian bullseye are located.

``` bash
sudo nano /etc/apt/sources.list
```

``` bash
# base list
deb http://deb.debian.org/debian bullseye main contrib non-free
deb-src http://deb.debian.org/debian bullseye main contrib non-free

#  official repository for changes that cannot wait for the next point release
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian bullseye-updates main contrib non-free

# frequent security updates 
deb http://security.debian.org/debian-security bullseye-security main
deb-src http://security.debian.org/debian-security bullseye-security main

# backports - more recent versions of some packages
deb http://deb.debian.org/debian bullseye-backports main contrib non-free
```

## ready for the upgrade

Run `sudo apt update` again to upgrade all bullseye packages. If no error is shown on the command line, you can start the upgrade with `sudo apt full-upgrade`, this will take a while and you may be asked if you want to overwrite configuration files and restart services. 

For some configurations, you will be asked if you want to keep the current configuration. You can view the changes by typing `D`, to exit the diff view press `Q`.


After the upgrade is complete, you need to reboot the system with `sudo reboot`

You can check your current version with the command `cat /etc/os-release`, the output will show something like:

``` bash
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
...
```
