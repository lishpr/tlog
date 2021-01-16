---
layout: post
title:  "Adding Network Support to a Container"
author: "Shori"
comments: false
tags: Docs
---

Last time, I looked into the Linux foundation of containers. But I left out one important part, networking.

Docker provided four options for networking when starting a container. We may select ```--net=none``` to not set up network support for our container.

And the goal would be to add network support to the netless container so that it may visit the internet or other containers.

First, let's take a look at the topology of a container network: 


![](../assets/network.png)
The general idea is, each container runs in a network namespace (presumably represented by those colored container boxes). And those network namespaces are connected to the Linux bridge (the purple docker0 in the picture). The two-way arrows represent virtual ethernets.

With that in the mind, we are going to reconstruct the network step by step.

<br />

## Create network namespace

We need to obtain the PID of the container process to get it a network namespace (netns).

```
# docker ps
CONTAINER ID        IMAGE            COMMAND     ...        
6310cddb3d73        ubuntu           "/bin/bash" ...
# docker top 6
UID                 PID                 PPID ...
root                40995               40967 ...
# pid=40995
```
Therefore, we could add a new netns by running
```
# ln -s /proc/$pid/ns/net /var/run/netns/$pid
```
*Only by creating a symbolic link could we make the new netns work (by assuring that the container is of the created namespace). It's tested that solely by running the* ```ip netns add``` *command does not work.*

Examine by the following command, we could see that the netns has been successfully created.
```
# ip netns show
40995
```

<br />

## Create virtual ethernet pair
In a rough analogy, a virtual ethernet (veth) pair is like a network cable. We should create a veth pair and hook one in the pair to the container netns.

***
### Setup veth pair

```
# ip link add name veth1 type veth peer name veth2
# ip a
/* ... irrelevant info ... */
6: veth2@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether c2:15:f3:d5:21:a6 brd ff:ff:ff:ff:ff:ff
7: veth1@veth2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 7a:51:6e:b8:77:ac brd ff:ff:ff:ff:ff:ff
```

***
### Set ```veth1``` to ```$pid``` namepace
```
# ip link set veth1 netns $pid
# ip a
/* ... irrelevant info ... */
6: veth2@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether c2:15:f3:d5:21:a6 brd ff:ff:ff:ff:ff:ff link-netns 40995
```

From the ```ip a``` command, we could see that ```veth1``` is no longer showing as it is now being assigned into the ```$pid``` namespace. Nonetheless, we could find ```veth1``` through running ```ip a``` in the ```$pid``` netns.
```
# ip netns exec $pid ip a
7: veth1@if6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 7a:51:6e:b8:77:ac brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

Then, we want to set up the ip for ```veth1```. And, of course it could successfully ping itself.
``` 
# ip netns exec $pid ifconfig veth1 10.0.0.2/24
# ip netns exec $pid ping 10.0.0.2
// Success
# ping 10.0.0.2
// Doesn't work outside the netns
```

The other end of the venv pair, ```veth2``` should be connected to the Linux bridge. And I'll talk about it in the next section.

***
### Create a Linux bridge

Now, in the pipeline is to connect ```veth2``` to a bridge to be created.

```
# brctl addbr br0           // create bridge
# brctl addif br0 veth2     // hook up with veth2
# ifconfig br0 10.0.0.1/24  // set IP
# ip link set veth2 up      // bring it up
```
And now, test it again by running ```ping 10.0.0.2``` from outside the netns, we now could do it successfully. And we could also ```ping 10.0.0.1``` from inside the container.

<br />

## Link the bridge to the internet

Now, we could have the container reach any device in the local network. 

In this step, we'll connect to the internet.

<br />

## References
[Docker -- 从入门到实践](https://yeasy.gitbook.io/docker_practice/)\
[docker网络原理](https://github.com/int32bit/notes/blob/master/cloud/docker网络原理.md)\
[How Docker Container Networking Works - Mimic It Using Linux Network Namespaces](https://dev.to/polarbit/how-docker-container-networking-works-mimic-it-using-linux-network-namespaces-9mj)\
[Introduction to Container Networking](https://rancher.com/learning-paths/introduction-to-container-networking/)