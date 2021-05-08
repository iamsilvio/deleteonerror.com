---
title: "nftables firewall introduction (Debian 10 Buster)"
subtitle: "Do not let tables confuse you!"
date: 2020-02-21T20:57:00+01:00
draft: false
tags: ["security", "nftable", "ssh", "firewall","webserver"]
categories: [itstuff]

twitter_creator_id: 94538574
aliases:
    - /posts/nftables-intro/
---

The first thing everyone should do before connecting to the Internet is to set up a proper firewall.
Since Debian 10 (Buster), the iptables framework is replaced by the nftables framework.

To make it easier to get started, here is a short guide on how to get your firewall up and running in no time.
<!--more-->
## Step 1: install some tooling

I mentioned that Debian uses the nftable framework by default, but it is not enabled!

To enable the default firewall, you have to do the following:

``` shell

$ sudo apt install nftables

# You should start nftables at system start, so you should enable it.
$ sudo systemctl enable nftables.service

```

## Step 2: Configuration

So, we have a firewall, we're done!

The default configuration file is in `/etc/nftables.conf`, so we dive into the configuration.

Here is a basic configuration file for a web server with everything you need to get started.

``` shell

#!/usr/sbin/nft -f

flush ruleset

table inet filter {
  chain input {
    type filter hook input priority 0;

      # accept any localhost traffic
      iif lo accept

      # accept trafic originated from us
      ct state established,related accept

      # drop invalid packets
      ct state invalid counter drop

      # accept http, https
      tcp dport { 80,443 } accept

      # ssh for ip range
      ip saddr 1.1.1.0/28 tcp dport 22 accept
      ip saddr 2.2.2.2 tcp dport 22 accept
      tcp dport 22 drop

      # mosh for ip range
      ip saddr 1.1.1.0/28 udp dport 60000-60005 accept
      ip saddr 2.2.2.2 udp dport 60000-60005 accept
      udp dport 60000-60005 drop

      # accept icmp
      ip protocol icmp accept

      # accept icmpv6
      ip6 nexthdr icmpv6 accept

      #  meta nfproto ipv6
      icmpv6 type { nd-neighbor-advert, nd-neighbor-solicit, nd-router-advert} ip6 hoplimit 1 accept
      icmpv6 type { nd-neighbor-advert, nd-neighbor-solicit, nd-router-advert} ip6 hoplimit 255 counter accept

      # count and reject everithing else
      counter reject with icmpx type admin-prohibited

  }
  chain forward {
    type filter hook forward priority 0;
  }
  chain output {
    type filter hook output priority 0;
  }
}

```

The configuration should be relatively self-explanatory. I prefer to configure an IP range that is allowed to connect via ssh (mosh).
If you have a dynamic IP, you have to find another solution to reduce the attack surface on your server.

## Step 3: Starting the firewall

If you check the status of your firewall with `sudo service nftables status`, you will see that it is not yet running.  
So let's reload the configuration by running `sudo nft -f /etc/nftables.conf` and check if the configuration is correctly loaded by running `sudo nft list table inet filter -n -a`.
  
If there are no errors, we start the firewall by running `sudo service nftables start`!
  
If you want to secure your ssh connection even more you can continue with [Securing ssh connections with ed25519 keys]({{< ref "Securingsshconnections.md" >}})
