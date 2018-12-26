---
layout: post
title: DevStack安装OpenStack
tags: OpenStack Linux 
---

本文主要介绍使用DevStack在CentOS 7上搭建OpenStack

<!--more-->

## 1.安装epel源和git

```shell
yum -y install epel-release git
yum -y install net-tools
```

## 2.关闭防火墙

### 2.1 关闭firewall

```shell
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态(关闭后显示notrunning，开启后显示running)
```

### 2.2 关闭iptables，如果有也关一下，不然可能装完之后访问不了：

```shell
systemctl stop iptables
systemctl disable iptables
```

## 3.设置SELINUX为disabled

```shell
vim /etc/selinux/config #将SELINUX=enforcing改为SELINUX=disabled
```

关闭selinux防火墙:

```shell
setenforce 0
```

## 4.更新yum源

### 4.1 下载repo文件

```shell
wget http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 4.2 备份并替换系统的repo文件

```shell
cp Centos-7.repo /etc/yum.repos.d/ 
cd /etc/yum.repos.d/ 
mv CentOS-Base.repo CentOS-Base.repo.bak 
mv Centos-7.repo CentOS-Base.repo
```

### 4.3 执行yum源更新命令 

```shell
yum clean all 
yum makecache 
yum update -y
```

## 5.安装pip并修改pip源：

```shell
apt-get install python-pip
mkdir ~/.pip
vi ~/.pip/pip.conf
```

修改成以下内容(豆瓣源)

```shell
#cat ~/.pip/pip.conf
[global]
index-url = http://pypi.douban.com/simple/
trusted-host = pypi.douban.com
```


## 6.准备Devstack

```shell
cd /home
git clone https://github.com/openstack-dev/devstack.git -b stable/rocky
```

## 7.需要创建stack用户运行

```shell
cd /home/devstack/tools/
bash ./create-stack-user.sh
```

## 8.在root下修改devstack目录权限，让stack用户可以运行

```shell
chown -R stack:stack devstack
chmod 777 /opt/stack -R
```

## 9.切换到stack用户下

```shell
su stack
cd /home/devstack
```

## 10.创建local.conf文件

创建[local.conf]({{ site.url }}/assets/file/local.conf)文件，并填入一下内容：

```shell
[[local|localrc]]
DATA_DIR=/opt/stack/data
DEST=/opt/stack
LOGFILE=/opt/stack/logs/stack.sh.log


TIMEOUT=300
ACTIVE_TIMEOUT=900
BOOT_TIMEOUT=900
ASSOCIATE_TIMEOUT=600
TERMINATE_TIMEOUT=600
SERVICE_TIMEOUT=300


HOST_IP=192.168.126.111
USE_SCREEN=True
OS_CACERT=
STACK_USER=stack
TLS_IP=
HOST_IPV6=::1
SERVICE_IP_VERSION=4
ADMIN_PASSWORD=zht
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
ADMIN_PASSWORD=$ADMIN_PASSWORD

VIRT_DRIVER=libvirt
SWIFT_REPLICAS=1
SWIFT_START_ALL_SERVICES=False
LOG_COLOR=False
UNDO_REQUIREMENTS=False
CINDER_PERIODIC_INTERVAL=10
export OS_NO_CACHE=True
#CEILOMETER_BACKEND=mysql
EBTABLES_RACE_FIX=True
DEBUG_LIBVIRT_COREDUMPS=True
CINDER_VOLUME_CLEAR=none
LIBVIRT_TYPE=qemu
FORCE_CONFIG_DRIVE=False


ROOTSLEEP=0
ERROR_ON_CLONE=False
INSTALL_TEMPEST=False
NOVNC_FROM_PACKAGE=True


GIT_BASE=http://git.trystack.cn
LIBS_FROM_GIT=python-openstackclient
LIBS_FROM_GIT+=,python-tackerclient
LIBS_FROM_GIT+=,tacker-horizon


FLOATING_RANGE="192.168.10.0/24"
FIXED_RANGE="10.0.0.0/24"
Q_FLOATING_ALLOCATION_POOL=start=192.168.10.100,end=192.168.10.200
PUBLIC_NETWORK_GATEWAY="192.168.10.1"


ENABLE_BARBICAN=True
SWIFT_HASH=a


ENABLED_SERVICES=c-api,c-bak,c-sch,c-vol
ENABLED_SERVICES+=,ceilometer-acentral,ceilometer-acompute,ceilometer-alarm-evaluator
ENABLED_SERVICES+=,ceilometer-alarm-notifier,ceilometer-anotification,ceilometer-api,ceilometer-collector
ENABLED_SERVICES+=,cinder
ENABLED_SERVICES+=,dstat
#ENABLED_SERVICES+=,etcd3
ENABLED_SERVICES+=,g-api,g-reg
ENABLED_SERVICES+=,horizon
ENABLED_SERVICES+=,key
ENABLED_SERVICES+=,mysql
ENABLED_SERVICES+=,n-api,n-api-meta,n-cauth,n-cond,n-cpu,n-novnc,n-obj,n-sch
ENABLED_SERVICES+=,peakmen_tacker
ENABLED_SERVICES+=,placement-api
ENABLED_SERVICES+=,q-agent,q-dhcp,q-l3,q-meta,q-metering,q-svc
ENABLED_SERVICES+=,q-fwaas,q-lbaas,q-vpnaas
ENABLED_SERVICES+=,rabbit
#ENABLED_SERVICES+=,tempest
ENABLED_SERVICES+=,s-account,s-container,s-object,s-proxy
SKIP_EXERCISES=boot_from_volume,bundle,client-env,euca

enable_plugin heat http://git.trystack.cn/openstack/heat
enable_plugin heat-dashboard http://git.trystack.cn/openstack/heat-dashboard
enable_plugin aodh http://git.trystack.cn/openstack/aodh
enable_plugin ceilometer http://git.trystack.cn/openstack/ceilometer
enable_plugin networking-sfc http://git.trystack.cn/openstack/networking-sfc
enable_plugin mistral http://git.trystack.cn/openstack/mistral
enable_plugin barbican http://git.trystack.cn/openstack/barbican
enable_plugin tacker http://git.trystack.cn/openstack/tacker
```

## 11.执行stack.sh

```shell
./stack.sh
```

## 12.可在/opt/stack/logs/stack.sh.log中查看执行日志。

```shell
tailf /opt/stack/logs/stack.sh.log
```

## 13.问题集锦

### 13.1 运行stack.sh的过程中如果反复需要输入stack用户密码

运行stack.sh的过程中如果反复需要输入stack用户密码时，可以运行/home/devstack/tools/create-stack-user.sh脚本，给stack用户赋权
