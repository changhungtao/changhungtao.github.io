---
title: "Windows设置PowerShell远程访问"
uuid: ec789d47-ad24-4837-9ea5-51f5d521ca15
excerpt: "本文介绍Windows下配置PowerShell远程访问的方法"
last_modified_at: 2019-02-20T10:52:00
toc: true
toc_label: "目录"
toc_icon: "columns"
categories:
  - 技术
tags:
  - Windows
---

### 1. Windows配置打开WinRM
在PowerShell环境下输入以下命令：
```
PS C:\Users\Administrator> winrm quickconfig
```
输出：

```
在此计算机上，WinRM 已设置为接收请求。
WinRM 没有设置成为了管理此计算机而允许对其进行远程访问。
必须进行以下更改:

在 HTTP://* 上创建 WinRM 侦听程序接受 WS-Man 对此机器上任意 IP 的请求。
启用 WinRM 防火墙异常。
配置 LocalAccountTokenFilterPolicy 以远程向本地用户授予管理权限。

进行这些更改吗[y/n]? y

WinRM 已经进行了更新，以用于远程管理。

在 HTTP://* 上创建 WinRM 侦听程序接受 WS-Man 对此机器上任意 IP 的请求。
WinRM 防火墙异常已启用。
已配置 LocalAccountTokenFilterPolicy 以远程向本地用户授予管理权限。
```

### 2. 开启防火墙命令或者直接关闭防火墙

```
C:\Windows\system32>netsh advfirewall firewall set rule group="Windows 远程管理" new enable=yes
```

### 3. 客户端使用PowerShell连接远程服务器
在客户端打开PowerShell，使用以下命令连接远程服务器的PowerShell
```
PS C:\Users\WW-PC>Enter-PSSession -computer {服务器名或者IP}
```
如果输出：

```
Enter-PSSession : 连接到远程服务器失败，错误消息如下: WinRM 客户端无法处理该请求。如果身份验证方案与 Kerberos 不同，或 
者客户端计算机未加入到域中， 则必须使用 HTTPS 传输或者必须将目标计算机添加到 TrustedHosts 配置设置。 使用 winrm.cmd 配 
置 TrustedHosts。请注意，TrustedHosts 列表中的计算机可能未经过身份验证。 通过运行以下命令可获得有关此内容的更多信息: wi 
nrm help config。 有关详细信息，请参阅 about_Remote_Troubleshooting 帮助主题。 
所在位置 行:1 字符: 16 
+ Enter-PSSession <<<<? 192.168.3.1 -Credential abc\administrator 
     + CategoryInfo            : InvalidArgument: (192.168.3.1:String) [Enter-PSSession], PSRemotingTransportException 
     + FullyQualifiedErrorId : CreateRemoteRunspaceFailed
```
网上一般都是说要添加一个TrustedHosts表（信任列表）。

在客户机执行如下命令，将IP为192.168.3.\*的主机都加入信任列表

```
Set-Item wsman:\localhost\Client\TrustedHosts -value 192.168.3.*
```

**注意：**这个命令需要在`客户端`上执行，不是在服务端执行，且客户端需要以`管理员权限`执行。
{: .notice--warning}

之后再用以下命令完成连接
```
Enter-PSSession 192.168.3.1 -Credential abc\administrator
```
> 没有域的情况下直接输入用户名
{: .notice--info}

![WinRM-1.png]({{site.url}}/assets/img/WinRM-1.png){: .align-center}

![WinRM-2.png]({{site.url}}/assets/img/WinRM-2.png){: .align-center}

### 4. 禁用WinRMB
如果你想在任何时间禁用WinRM,你可以使用这样的命令：

```
winrm delete winrm/config/listener?IPAdress=*+Transport=HTTP
```

### 5. 附录

参考资料：[https://www.pstips.net/](https://www.pstips.net/) 收集和分享 Windows PowerShell 相关教程,技术和最新动态

