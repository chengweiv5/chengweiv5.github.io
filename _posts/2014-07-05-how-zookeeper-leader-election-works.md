---
layout: post
title: "How Zookeeper Leader Election Works"
tags: [zookeeper]
---

As quote from [zookeeper homepage](http://zookeeper.apache.org/ "zookeeper") below.

>ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

Which can provides leader election for clients, in generally speaking, zookeeper clients are kind of service, for example, [mesos](http://mesos.apache.org "mesos") masters, [chronos](https://github.com/airbnb/chronos chronos) and etc.

With leader election of zookeeper, it's quite easy to elect a leader from multiple clients, this blog will describe the most simple way zookeeper uses to elect leader of clients.

The most simple way is every clients creates a znode in a chosen path, for example /candidates, with flag: _SEQUENCE|EPHEMERAL_.

We assume there are four clients started, e.g. four chronos instances started. See below diagram.

![diagram1](/assets/images/zookeeper/zookeeper-1.png "four clients")

<center>diagram 1. There are four clients connected with zookeeper service: c_i, c_i+1, c_i+2, c_i+3</center>

In above diagram, there are four clients, created four znodes in the /candidates path: c\_i, c\_i+1, c\_i+2, c\_i+3. We assume there sequence number assigned by zookeeper are i, i+1, i+2, i+3.

The above diagram describes the leader election rules:

- the client has smallest sequence number is the leader
- every client watch its previous client, e.g. c\_i+1 watches c\_i, c\_i+3 watches c\_i+2

###How about a non-leader client die?
If a follower client die, say c\_i+2 in below diagram, since c\_i+2 isn't the current leader client, so c\_i+3 will watch c\_i+1 now.

![diagram2](/assets/images/zookeeper/zookeeper-2.png "a non-leader client die")

<center>diagram 2. A non-leader client(c_i+2) die</center>

###How about the leader client die?
If the current leader die, that's mean c\_i die, see diagram below.

![diagram3](/assets/images/zookeeper/zookeeper-3.png "the leader client die")

<center>diagram 3. The leader client(c_i) die</center>

Since c\_i is the current leader client and it die, then c\_i+1 which has the smallest sequence number now will be the leader client.
