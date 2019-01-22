---
layout: post
title: Linux和ESXi中的抓包工具
tags: Linux ESXi tcpdump
---

本文主要介绍Linux和ESXi环境中的抓包工具
<!--more-->

## Linux系统中使用tcpdump命令抓取VLAN的tag

### 1、tcpdump的参数信息

>  -n    不转换主机地址到主机名，这样用于避免DNS解析 
>
>  -i    指定网络接口
>
>  -e    增加以太网帧头部信息输出
>
>  -v    输出更详细的信息
  
### 2、抓取vlan的信息
```shell
[root@zht-controller ~]# tcpdump -i ens192 -v -e host 192.168.149.254
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 262144 bytes
04:28:52.105444 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
04:28:53.106900 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
04:28:54.108819 fa:16:3e:e0:04:96 (oui Unknown) > Broadcast, ethertype 802.1Q (0x8100), length 46: vlan 623, p 0, ethertype ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.149.254 tell 192.168.149.63, length 28
```

## ESXi主机上使用pktcap-uw和tcpdump-uw命令抓包

在ESXi主机上可以使用tcpdump-uw和pktcap-uw命令进行抓包，pktcap-uw是ESXi 5.5及以后版本默认包含的工具，它比tcpdump-uw要强大和复杂。

![tcpdump-uw-vs-pktcap-uw]({{ site.url }}/assets/img/tcpdump-uw-vs-pktcap-uw.png)

本章节主要介绍在ESXi主机上使用pktcap-uw命令抓包，pktcap-uw抓包保存在文件中，将文件拷贝到Windows机器上使用WireShark分析。

### 抓取虚拟机端口流量

#### 1、查看虚拟机的PORT-ID

运行`esxtop`命令，然后输入`n`，可以查看虚拟机port列表

![esxi-determine-port-id]({{ site.url }}/assets/img/esxi-determine-port-id.png)

#### 2、抓取port流量
```shell
pktcap-uw --switchport 33554439 -o zht-vm.pcap
```

### 抓取VMkernel接口流量

#### 1、查看VMkernel接口列表
```shell
esxcfg-vmknic -l
```

#### 2、抓取流量
```shell
pktcap-uw --vmk vmk0 -o vmk0.pcap
```

### 抓取物理网卡流量
```shell
pktcap-uw --uplink vmnic0 -o vmnic0.pcap
```

### 常用过滤条件

#### 1、捕捉点（Capture Points）

tcpdump-uw没有流量的概念，当你用tcpdump-uw对vmk0进行抓包时，你可以看到VMkernel入向和出向的报文。pktcap-uw则引入了捕捉点的概念，来决定你在哪儿进行抓包。对于VMkernel流量，你有2个捕捉点：

* PortOutput: 从虚拟交换机到VMkernel的流量。
* PortInput: 从VMkernel 到虚拟交换机的流量。（默认）

例如：
```shell
pktcap-uw --switchport 33554439 --capture PortOutput
```

完整的捕捉点列表参考[这里](https://docs.vmware.com/cn/VMware-vSphere/5.5/com.vmware.vsphere.networking.doc/GUID-33B3FDD7-0555-4D54-B9A9-CDBC827504DA.html)

#### 2、协议

pktcap-uw不关心具体的IP协议，因为协议ID是IPv4头部信息的一部分，所以pktcap-uw使用协议ID来过滤协议。

例如，过滤ICMP协议包
```shell
pktcap-uw --vmk vmk0 --proto 0x01
```

完整IP协议列表和协议ID参考[这里](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)

#### 3、VLAN的tag号过滤
```shell
pktcap-uw --uplink vmnic0 --vlan 622 -o vmnic0.pcap
```

> **参考资料**  
> [官方中文文档](https://docs.vmware.com/cn/VMware-vSphere/6.5/com.vmware.vsphere.networking.doc/GUID-5CE50870-81A9-457E-BE56-C3FCEEF3D0D5.html)
>
> [参考资料](https://www.virten.net/2015/10/esxi-network-troubleshooting-with-tcpdump-uw-and-pktcap-uw/)
>
> [官方文档主页](https://docs.vmware.com/cn/VMware-vSphere/index.html)
