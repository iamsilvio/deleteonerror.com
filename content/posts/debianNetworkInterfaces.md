---
title: "DebianNetworkInterfaces"
date: 2020-02-14T01:48:48+01:00
draft: true
---

#a static ip for your debian server 


sudo nano /etc/network/interfaces

find
iface eth0 inet dhcp

replace with 

iface eth0 inet static
  address 10.10.10.10
  netmask 255.255.255.0
  gateway 10.10.10.1
  dns-nameserver 10.10.10.253 10.10.10.252

sudo systemctl restart networking

sudo ifup eth0

ip a

your can add multiple network interfaces to this interface,
just add address entries if you need addresses from another subnet, you have to enter the full entry again


iface enp0s3 inet static
  address 10.0.0.105/24

iface enp0s3 inet static
  address 10.0.1.105/24

sudo ifdown ens3 && sudo ifup ens3