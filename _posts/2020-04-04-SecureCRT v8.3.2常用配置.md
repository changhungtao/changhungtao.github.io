---
title: "SecureCRT v8.3.2配色"
uuid: 4cf18dc7-7b84-4179-9311-0c8d76975dfb
excerpt: "介绍SecureCRT v8.3.2的配色主题"
last_modified_at: 2019-04-04T15:57:00
comments: true
toc: true
toc_label: "目录"
toc_icon: "columns"
categories:
  - 技术
tags:
  - 工具
  - Windows
---

## 下载安装
链接：[https://pan.baidu.com/s/1QAVOj_qZOOgQloLjeY-oaw](https://pan.baidu.com/s/1QAVOj_qZOOgQloLjeY-oaw)

提取码：znkk

## 配色方案
![绝佳配色]({{site.url}}/assets/img/SecureCRT-peise-1.png){: .align-center}

下载[Setting文件]({{ site.url }}/assets/file/绝佳配色方案.xml)，`Tools` -> `Import Settings`，选择Settings文件Import。

### 跳板机设置

跳板机：10.127.10.2，目标机：192.168.122.151，目标机是跳板机上的一台VM，跳板机可以直接连接，目标机无法直接连接。

#### 跳板机Session

新建跳板机Session，设置名称、IP、账号等信息
![CRT-set-1]({{site.url}}/assets/img/SecureCRT-set-1.png){: .align-center}

`Port Forwarding`增加`Local Port Forwarding`
![CRT-set-2]({{site.url}}/assets/img/SecureCRT-set-2.png){: .align-center}

#### Global Session

`Options` -> `Global Options` -> `Firewall`，添加一个Firewall

![CRT-set-3]({{site.url}}/assets/img/SecureCRT-set-3.png){: .align-center}

#### 目标机Session

新建目标机Session，设置名称、IP、账号等信息，在Firewall栏选择上一步新建的Firewall

![CRT-set-4]({{site.url}}/assets/img/SecureCRT-set-4.png){: .align-center}

#### 验证

连接跳板机之后，保持跳板机的连接，可以直接连接目标机。

## 参考资料
[https://changhungtao.github.io/%E6%8A%80%E6%9C%AF/2019/02/19/SecureCRT-SecureFX-v8.3.2%E7%A0%B4%E8%A7%A3.html](https://changhungtao.github.io/%E6%8A%80%E6%9C%AF/2019/02/19/SecureCRT-SecureFX-v8.3.2%E7%A0%B4%E8%A7%A3.html)

