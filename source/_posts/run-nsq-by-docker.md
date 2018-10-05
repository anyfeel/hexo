---
title: run nsq by docker
date: 2018-09-23 18:27:58
tags: Linux
---

# components
## nsqd
nsqd is the daemon that receives, queues, and delivers messages to clients.
two tcp ports: one for clients and one for HTTP API

## nsqlookupd
the daemon that manages topology information.
Clients query nsqlookupd to discover nsqd producers for a specific topic and nsqd nodes broadcasts topic and channel information.
two interfaces: A TCP interface which is used by nsqd for broadcasts and an HTTP interface for clients to perform discovery and administrative actions.

## nsqadmin
Web UI to view aggregated cluster stats in realtime and perform various administrative tasks

<!-- more -->

# run nsq
## get nsq docker image
```
docker pull nsqio/nsq
```
## run nsqlookupd
```
docker run --name lookupd -d -p 4160:4160 -p 4161:4161 nsqio/nsq /nsqlookupd
```
## run nsqd
1. get docker host's ip
```
ifconfig | grep docker addr  -> 172.17.0.1
```
2. run nsqd container
```
docker run --name nsqd -d -p 4150:4150 -p 4151:4151  nsqio/nsq /nsqd
    --broadcast-address=<host> --lookupd-tcp-address=<host>:<port>
```
eg:
```
docker run --name nsqd -d -p 4150:4150 -p 4151:4151  nsqio/nsq /nsqd
    --broadcast-address=172.17.0.1 --lookupd-tcp-address=172.17.0.1:4160
```
note:
a. no care for 4150 and 4151
b. --broadcast-address set to the docker addr
c. --lookupd-tcp-address set to the nsqlookupd http listen port, here is 4161

## run nsqadmin
```
docker run -d --name nsqadmin -p 4171:4171 nsqio/nsq /nsqadmin  --lookupd-http-address=172.17.0.1:4161
```
