---
layout: post
title:  Configuring Centos from scratch
date:   2016-10-25 10:00:00
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

Here is my configuration:
![centos7-ifcgf](/images/2016/Centos7_ifcfg-eno1.png)

Run `service network restart` to make the change take effect.

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


### Fixed IP

Similarly you have to edit the same file for the DHCP configuration. But now you'll set BOOTPROTO to static and that line will become `BOOTPROTO=static`

Also add the keys IPADDR, NETMASK and GATEWAY and the respective desired IP address, the network mask and the gateway IP address. Make sure your IP will not conflict with the DHCP range, that in my network is reserved to the values between .2 and .200

The values that I used were:

```
IPADDR=192.168.1.201
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
```

Additionally, you have to define the DNS server, since in a static configuration you have to define it.

The keys are DNS1 and DNS2 if a secondary dns is needed. I used the google dns servers and the values are:

```
DNS1=8.8.8.8
DNS2=8.8.4.4
```

Use the same procedure to restart the network and test the reachability as on the DNS example.

To see the current configuration, type `ip address` or `ip a` as a shorten form.

### Hostname
By default, the hostname is `localhost.localdomain` after installation. You can change it for whatever make sense for you. I'll change it to ship1.

And it is located in the file `/etc/hostname`

I have also to add the line `127.0.0.1 ship1` to the hosts file. To edit this file, type `vi /etc/hosts`

This change requires reboot. To reboot the system in the command line, type `shutdown -r now`
