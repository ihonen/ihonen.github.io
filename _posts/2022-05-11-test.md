---
layout: post
title:  "WIP: Linux: detecting new network interfaces in C/C++"
author: "ihonen"
tags: ["c", "c++", "linux"]
---

## The problem

At work we are developing a Linux-based embedded system that makes use of a
USB-to-Ethernet network adapter. There's a problem, though: while the adapter is
correctly detected by the kernel as both a USB device and an Ethernet interface
when plugged in during runtime, our software isn't notified and therefore the
interface is not properly configured.

So, time to write a bunch of code.

## The solution: netlink

[Netlink](https://en.wikipedia.org/wiki/Netlink) is an inter-process
communication protocol in the Linux kernel that can be used in a userspace
process to receive various messages from the kernel. It can also be used for IPC
between userspace processes, but that's a story for another time.

First, you have to include the necessary headers:

```c++
#include <sys/types.h>
#include <linux/netlink.h>
#include <linux/rtnetlink.h>
#include <sys/socket.h>
```

To open a netlink socket to listen to messages about routing changes, we'll call
`socket` like so:

```c++
int netlinkSocket = socket(AF_NETLINK, SOCK_RAW, NETLINK_ROUTE);
```

Our socket also needs an address:

```c++
struct sockaddr_nl netlinkAddress = {
    .nl_family = AF_NETLINK,
    .nl_pad    = 0,
    .nl_pid    = getpid(),
    .nl_groups = RTMGRP_LINK,
};
```

## References

* Linux man pages: [man 7 netlink](https://linux.die.net/man/7/netlink), [man 7 rtnetlink](https://linux.die.net/man/7/rtnetlink), [man 3 netlink](https://linux.die.net/man/3/netlink), [man 3 rtnetlink](https://linux.die.net/man/3/rtnetlink)
* [Oleg Kutkov's excellent blog post about netlink](https://olegkutkov.me/2018/02/14/monitoring-linux-networking-state-using-netlink/)
