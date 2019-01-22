---
layout: post
title: tcpdump抓取VLAN的tag
tags: Linux tcpdump
---

## 1、tcpdump的参数信息

  -n    不转换主机地址到主机名，这样用于避免DNS解析

  -i    指定网络接口

  -e    增加以太网帧头部信息输出

  -v    输出更详细的信息
  
## 2、抓取vlan的信息
```shell
[root@zht-controller ~]# tcpdump -i ens192 -v -e host 192.168.149.254
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 262144 bytes
04:28:52.105444 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
04:28:53.106900 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
04:28:54.108819 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
```