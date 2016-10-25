---
layout: post
title:  Configuring Centos installation from scratch
date:   2016-10-26 10:00:00
categories: []
tags: [ linux, centos, devops ]
image: "/images/2016/centos.jpg"
---

![centos](/images/2016/centos.jpg)

This articles describes the basic steps to setup a just installed Centos Linux. As my focus is on software development and I am not constantly practicing server configuration, these steps are my reference where I cant return to when I eventually forget it.

## Network setup

If your network was not configured on the installation, you'll need to do it manually. You can configure it as DHCP or fixed IP. On both cases, you'll have to change the files in
`/etc/sysconfig/network-scripts`

To test if your network is working (with the cable connected), ping to a well known dns server. Make sure the your network policy allow that.

`ping 8.8.8.8`
This ip should be unreachable.

Go to the directory and list the files

```
  cd /etc/sysconfig/network-scripts
  ls  
```

You will find there a file called `ifcfg-eno1` that you have to edit. If you are using a virtual machine, this name can change. In my case is was `ifcfg-enp0s3`. Edit this file with `vi ifcfg-eno1`.

### DHCP

To configure it to use DHCP, make sure the line BOOTPROTO is set to dhcp. Also change ONBOOT to yes.


Run `service network restart` to make the change work.

Now try pinging again.
`ping 8.8.8.8`

The result should be something like this:
```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=58 time=6.708 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=58 time=6.901 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=58 time=5.387 ms
```

Type `^+C` to stop pinging.
