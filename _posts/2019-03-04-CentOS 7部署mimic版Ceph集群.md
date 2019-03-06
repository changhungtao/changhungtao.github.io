---
title: "CentOS 7部署mimic版Ceph集群"
uuid: 6758e3d1-75b6-40ba-80ad-6ca70b86c880
excerpt: "本文主要介绍在CentOS 7上部署mimic版本Ceph集群的步骤。"
last_modified_at: 2019-03-05T10:06:00
comments: true
categories:
  - 技术
tags:
  - Ceph
  - 云存储
---

### 一、简介

#### Ceph

**Ceph**: 开源的分布式存储系统。主要分为`对象存储`、`块设备存储`、`文件系统服务`。Ceph核心组件包括:`Ceph OSDs`、`Monitors`、`Managers`、`MDSs`。Ceph存储集群**至少需要**一个`Ceph Monitor`、`Ceph Manager`和`Ceph OSD`（对象存储守护进程）。运行Ceph Filesystem客户端时也需要Ceph MDS。

#### Ceph OSDs

**Ceph OSDs**: Ceph OSD 守护进程（ceph-osd）的功能是存储数据，处理数据的复制、恢复、回填、再均衡，并通过检查其他 OSD 守护进程的心跳来向 Ceph Monitors 提供一些监控信息。冗余和高可用性通常至少需要3个Ceph OSD。当 Ceph 存储集群设定为有2个副本时，至少需要2个 OSD 守护进程，集群才能达到active+clean 状态（ Ceph 默认有3个副本，但你可以调整副本数）。

#### Monitors

**Monitors**: Ceph Monitor（ceph-mon） 维护着展示集群状态的各种图表，包括监视器图、 OSD 图、归置组（ PG ）图、和 CRUSH 图。 Ceph 保存着发生在Monitors 、 OSD 和 PG上的每一次状态变更的历史信息（称为 epoch ）。监视器还负责管理守护进程和客户端之间的身份验证。冗余和高可用性通常至少需要三个监视器。

#### Managers

**Managers**: Ceph Manager守护进程（ceph-mgr）负责跟踪运行时指标和Ceph集群的当前状态，包括存储利用率，当前性能指标和系统负载。Ceph Manager守护进程还托管基于python的插件来管理和公开Ceph集群信息，包括基于Web的Ceph Manager Dashboard和 REST API。高可用性通常至少需要两个管理器。

#### MDSs

**MDSs**: Ceph 元数据服务器（ MDS ）为 Ceph 文件系统存储元数据（也就是说，Ceph 块设备和 Ceph 对象存储不使用MDS ）。元数据服务器使得 POSIX 文件系统的用户们，可以在不对 Ceph 存储集群造成负担的前提下，执行诸如 ls、find 等基本命令。

### 二、准备

本文主要介绍使用`ceph-deploy`来部署三节点的ceph集群，节点信息如下：

- 192.168.126.111 zht-ceph-1
- 192.168.126.112 zht-ceph-2
- 192.168.126.113 zht-ceph-3

硬件配置：

- zht-ceph-1，2核2G内存，2硬盘（50G /dev/sda，20G /dev/sdb）
- zht-ceph-2，2核2G内存，2硬盘（50G /dev/sda，20G /dev/sdb）
- zht-ceph-3，2核2G内存，2硬盘（50G /dev/sda，20G /dev/sdb）

**提示**：以下步骤中，`#`开头的命令是使用`root`用户执行，`$`开头的命令是使用`cephd`用户执行。
{: .notice--info}

#### A、创建Ceph部署用户

`ceph-deploy`工具必须以普通用户登录 Ceph 节点，且此用户拥有无密码使用 sudo 的权限，因为它需要在安装软件及配置文件的过程中，不必输入密码。官方建议所有 Ceph 节点上给`ceph-deploy`创建一个特定的用户，而且不要使用 ceph 这个名字。这里为了方便，我们使用 cephd 这个账户作为特定的用户，而且每个节点上（monitor、node）上都需要创建该账户，并且拥有 sudo 权限。

```shell
# useradd -d /home/cephd -m cephd
# passwd cephd
```

```shell
# echo "cephd ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephd
# sudo chmod 0440 /etc/sudoers.d/cephd
```

**提示**：创建用户的步骤需要在其他机器手动执行，后续的操作都只需要在`zht-ceph-1`机器上执行。
{: .notice--info}

#### B、配置hosts

```shell
# cat /etc/hosts
192.168.126.111 zht-ceph-1
192.168.126.112 zht-ceph-2
192.168.126.113 zht-ceph-3
```

#### C、配置ssh互信

配置互信需要切换到cephd用户，生成 SSH 密钥并把其公钥分发到各 Ceph 节点上，提示输入密码时，直接回车。 

```shell
$ ssh-keygen
$ ssh-copy-id zht-ceph-1
$ ssh-copy-id zht-ceph-2
$ ssh-copy-id zht-ceph-3
```

#### D、安装ansible

```shell
$ sudo yum -y install ansible
$ sudo cat /etc/ansible/hosts | grep -v ^# | grep -v ^$
[node]
192.168.126.112
192.168.126.113
```

**提示**：安装`ansible`只是为了减少多个机器重复执行命令，例如需要配置`zht-ceph-2`和`zht-ceph-3`的`/etc/hosts`文件的时候，只需要使用`ansible`命令`ansible node -m copy -a 'src=/etc/hosts dest=/etc/'`来将`zht-ceph-1`机器上的`/etc/hosts`文件拷贝过去。
{: .notice--info}

#### E、关闭SeLinux和Firewall

拷贝`/etc/hosts`文件：

```shell
# ansible node -m copy -a 'src=/etc/hosts dest=/etc/'
```

关闭SeLinux：

```shell
# sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
# ansible node -m copy -a 'src=/etc/selinux/config dest=/etc/selinux/'
```

关闭防火墙：

```shell
# systemctl stop firewalld
# systemctl disable firewalld
# ansible node -a 'systemctl stop firewalld'
# ansible node -a 'systemctl disable firewalld'
```

#### F、安装NTP

官方建议在所有 Ceph 节点上安装 NTP 服务（特别是 Ceph Monitor 节点），以免因时钟漂移导致故障。

```shell
# yum -y install ntp ntpdate ntp-doc
# systemctl start ntpdate
# systemctl start ntpd
# systemctl enable ntpd ntpdate
```

使用`ansible`命令在112和113上安装NTP服务。（以下命令在111节点上执行）

```shell
# ansible node -a 'yum -y install ntp ntpdate ntp-doc'
# ansible node -a 'systemctl start ntpdate'
# ansible node -a 'systemctl start ntpd'
# ansible node -a 'systemctl enable ntpdate'
# ansible node -a 'systemctl enable ntpd'
```

#### G、安装相应源

```shell
# cat /etc/yum.repos.d/mimic-ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-mimic/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
# yum clean all && yum makecache && yum install ceph-deploy
```

**备注**：源文件中的`baseurl`结构为：`baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/noarch`
{: .notice--info}

### 三、部署Ceph集群

#### A、安装`ceph-deploy`

```shell
$ sudo yum -y install ceph-deploy
```

#### B、创建执行目录

```shell
$ mkdir ~/ceph-cluster && cd ~/ceph-cluster
```

#### C、创建集群

```shell
$ ceph-deploy new zht-ceph-{1,2,3}
```

此时，我们会发现`ceph-deploy`会在`/etc/ceph`目录下生产3个文件，`ceph.conf`为ceph配置文件，`ceph-deploy-ceph.log`为`ceph-deploy`日志文件，`ceph.mon.keyring`为`ceph monitor`的密钥环:

```shell
$ ls
ceph.conf  ceph-deploy-ceph.log  ceph.log  ceph.mon.keyring
$ ceph-deploy new --cluster-network 192.168.126.0/24 --public-network 192.168.126.0/24 zht-ceph-{1,2,3}
```

#### D、安装Ceph软件包

通过`ceph-deploy`在各个节点安装`ceph`：

```shell
$ ceph-deploy install zht-ceph-{1,2,3}
```

此过程需要等待一段时间，因为`ceph-deploy`会 SSH 登录到各节点上去，依次执行安装 ceph 依赖的组件包。

```shell
$ ceph -v
ceph version 13.2.4 (b10be4d44915a4d78a8e06aa31919e74927b142e) mimic (stable)
$ cat ceph.conf
[global]
fsid = 06979e05-2cf6-48a4-9c5f-c4ede9c363fb
public_network = 192.168.126.0/24
cluster_network = 192.168.126.0/24
mon_initial_members = zht-ceph-1, zht-ceph-2, zht-ceph-3
mon_host = 192.168.126.111,192.168.126.112,192.168.126.113
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

#### E、初始化monitor节点

初始化`monitor`节点并收集所有密钥：

```shell
$ ceph-deploy mon create-initial
```

执行完毕后，会在当前目录下生成一系列的密钥环。

```shell
$ ls
ceph.bootstrap-mds.keyring  ceph.bootstrap-osd.keyring  ceph.client.admin.keyring  ceph-deploy-ceph.log  ceph.mon.keyring
ceph.bootstrap-mgr.keyring  ceph.bootstrap-rgw.keyring  ceph.conf       ceph.log  rbdmap
```

#### F、查看健康状况

```shell
$ ceph health
HEALTH_OK
$ ceph -s
  cluster:
    id:     06979e05-2cf6-48a4-9c5f-c4ede9c363fb
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum zht-ceph-1,zht-ceph-2,zht-ceph-3
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
```

#### G、将管理密钥拷贝到各个节点上

通过`ceph-deploy admin`将配置文件和`admin`密钥同步到各个节点，以便在各个节点上使用 ceph 命令时，无需指定 monitor 地址和 ceph.client.admin.keyring 密钥。

```shell
$ ceph-deploy admin zht-ceph-{1,2,3}
```

查看是否拷贝成功：

```shell
$ ansible node -a 'ls /etc/ceph/'
```

为了确保对`ceph.client.admin.keyring`有正确的操作权限，需要增加权限设置：

```shell
$ sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

#### H、创建Ceph管理进程服务

```shell
$ ceph-deploy mgr create ceph{01,02,03}
$ ceph -s
  cluster:
    id:     06979e05-2cf6-48a4-9c5f-c4ede9c363fb
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum zht-ceph-1,zht-ceph-2,zht-ceph-3
    mgr: zht-ceph-1(active), standbys: zht-ceph-2, zht-ceph-3
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
```

#### I、启动 osd 创建数据

```shell
$ ceph-deploy osd create --data /dev/sdb zht-ceph-1
$ ceph-deploy osd create --data /dev/sdb zht-ceph-2
$ ceph-deploy osd create --data /dev/sdb zht-ceph-3
$ ceph -s
  cluster:
    id:     06979e05-2cf6-48a4-9c5f-c4ede9c363fb
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum zht-ceph-1,zht-ceph-2,zht-ceph-3
    mgr: zht-ceph-1(active), standbys: zht-ceph-2, zht-ceph-3
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   3.0 GiB used, 57 GiB / 60 GiB avail
    pgs:     
```

### 三、查看相关信息

#### A、查看运行状况

```shell
# systemctl status ceph-mon.target
# systemctl status ceph/*.service
# systemctl status | grep ceph
● zht-ceph-1
           │ │   ├─4456 sshd: cephd [priv
           │ │   ├─4458 sshd: cephd@pts/1
           │ │   └─9223 grep --color=auto ceph
             ├─system-ceph\x2dosd.slice
             │ └─ceph-osd@0.service
             │   └─9009 /usr/bin/ceph-osd -f --cluster ceph --id 0 --setuser ceph --setgroup ceph
             ├─system-ceph\x2dmgr.slice
             │ └─ceph-mgr@zht-ceph-1.service
             │   └─8488 /usr/bin/ceph-mgr -f --cluster ceph --id zht-ceph-1 --setuser ceph --setgroup ceph
             ├─system-ceph\x2dmon.slice
             │ └─ceph-mon@zht-ceph-1.service
             │   └─7310 /usr/bin/ceph-mon -f --cluster ceph --id zht-ceph-1 --setuser ceph --setgroup ceph
             │ ├─794 avahi-daemon: running [zht-ceph-1.local
```

#### B、查看 ceph 存储空间

```shell
# ceph df
```

#### C、查看 mon 相关信息

```shell
# ceph mon stat         ##查看 mon 状态信息
e1: 3 mons at {zht-ceph-1=192.168.126.111:6789/0,zht-ceph-2=192.168.126.112:6789/0,zht-ceph-3=192.168.126.113:6789/0}, election epoch 6, leader 0 zht-ceph-1, quorum 0,1,2 zht-ceph-1,zht-ceph-2,zht-ceph-3
# ceph quorum_status         ##查看 mon 的选举状态
# ceph mon dump                ##查看 mon 映射信息
dumped monmap epoch 1
epoch 1
fsid 06979e05-2cf6-48a4-9c5f-c4ede9c363fb
last_changed 2019-03-04 15:37:12.889445
created 2019-03-04 15:37:12.889445
0: 192.168.126.111:6789/0 mon.zht-ceph-1
1: 192.168.126.112:6789/0 mon.zht-ceph-2
2: 192.168.126.113:6789/0 mon.zht-ceph-3
# ceph daemon mon.zht-ceph-1 mon_status        ##查看 mon 详细状态
```

#### D、查看 osd 相关信息

```shell
# ceph osd stat                  ##查看 osd 运行状态
3 osds: 3 up, 3 in; epoch: e13
# ceph osd dump                  ##查看 osd 映射信息
# ceph osd perf                  ##查看数据延迟
# ceph osd df                    ##详细列出集群每块磁盘的使用情况  
ID CLASS WEIGHT  REWEIGHT SIZE   USE     AVAIL  %USE VAR  PGS 
 0   hdd 0.01949  1.00000 20 GiB 1.0 GiB 19 GiB 5.01 1.00   0 
 1   hdd 0.01949  1.00000 20 GiB 1.0 GiB 19 GiB 5.01 1.00   0 
 2   hdd 0.01949  1.00000 20 GiB 1.0 GiB 19 GiB 5.01 1.00   0 
                    TOTAL 60 GiB 3.0 GiB 57 GiB 5.01          
MIN/MAX VAR: 1.00/1.00  STDDEV: 0
# ceph osd tree                  ##查看 osd 目录树
# ceph osd getmaxosd             ##查看最大 osd 的个数 
max_osd = 3 in epoch 13
```

#### E、查看 PG 信息

```shell
# ceph pg dump       ##查看 PG 组的映射信息
# ceph pg stat       ##查看 PG 状态
0 pgs: ; 0 B data, 3.0 GiB used, 57 GiB / 60 GiB avail
# ceph pg dump --format plain  ##显示集群中的所有的 PG 统计,可用格式有纯文本plain(默认)和json
dumped all
version 158
stamp 2019-03-04 15:52:23.250793
last_osdmap_epoch 0
last_pg_scan 0
PG_STAT OBJECTS MISSING_ON_PRIMARY DEGRADED MISPLACED UNFOUND BYTES LOG DISK_LOG STATE STATE_STAMP VERSION REPORTED UP UP_PRIMARY ACTING ACTING_PRIMARY LAST_SCRUB SCRUB_STAMP LAST_DEEP_SCRUB DEEP_SCRUB_STAMP SNAPTRIMQ_LEN 
         
                    
sum 0 0 0 0 0 0 0 0 
OSD_STAT USED    AVAIL  TOTAL  HB_PEERS PG_SUM PRIMARY_PG_SUM 
2        1.0 GiB 19 GiB 20 GiB    [0,1]      0              0 
1        1.0 GiB 19 GiB 20 GiB    [0,2]      0              0 
0        1.0 GiB 19 GiB 20 GiB    [1,2]      0              0 
sum      3.0 GiB 57 GiB 60 GiB                                
```

### 四、启用dashboard

使用如下命令即可启用dashboard模块：

```shell
# ceph mgr module enable dashboard
```

默认情况下，仪表板的所有HTTP连接均使用SSL/TLS进行保护。要快速启动并运行仪表板，可以使用以下内置命令生成并安装自签名证书:

```shell
# ceph dashboard create-self-signed-cert
Self-signed certificate created
```

创建具有管理员角色的用户：

```shell
# ceph dashboard set-login-credentials admin zht
Username and password updated
```

查看ceph-mgr服务:

```shell
# ceph mgr services
{
    "dashboard": "https://zht-ceph-1:8443/"
}
```

浏览器输入`https://192.168.126.111:8443`即可登录Ceph的dashboard，用户名`admin`，密码`zht`。

![ceph-dashboard-1.png]({{site.url}}/assets/img/ceph-dashboard-1.png){: .align-center}

![ceph-dashboard-2.png]({{site.url}}/assets/img/ceph-dashboard-2.png){: .align-center}

### 五、参考资料
[http://www.mamicode.com/info-detail-2404519.html](http://www.mamicode.com/info-detail-2404519.html)
[https://blog.csdn.net/liukuan73/article/details/79220230](https://blog.csdn.net/liukuan73/article/details/79220230)
