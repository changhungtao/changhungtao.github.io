---
title: "Apache简单配置SSL的方法(HTTPS的实现)"
uuid: 97dccbd4-4923-4ccd-85cb-f513e6e058f4
excerpt: "本文主要介绍CentOS和Ubuntu系统下Apache配置HTTPS的方法"
last_modified_at: 2019-02-20T12:14:00
toc: true
toc_label: "目录"
toc_icon: "columns"
categories:
  - 技术
tags:
  - Linux
---

## CentOS下为Apache简单配置SSL的方法(HTTPS的实现)

### 1. 安装必备软件

```shell
yum install -y openssl
yum install -y httpd
yum install -y mod_ssl
```

### 2. 防火墙打开80、443端口（可选）

```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --reload
```

### 3. 设置Apache自动启动

```shell
systemctl enable httpd
systemctl start httpd
```

### 4. 创建CA签名(==不使用密码去除-des3选项==)

```shell
$ openssl genrsa -des3 -out server.key 1024
```

输出（密码随便输，但是后面会用到）：  

```shell
Generating RSA private key, 1024 bit long modulus
.++++++
......++++++
e is 65537 (0x10001)
Enter pass phrase for server.key:  
Verifying - Enter pass phrase for server.key:
```

### 5. 创建CSR(Certificate Signing Request)

```shell
$ openssl req -new -key server.key -out server.csr
```

输出（除了需要server.key密码，其余按回车键）:   

```shell
Enter pass phrase for server.key:  
You are about to be asked to enter information that will be incorporated
into your certificate request.  
What you are about to enter is what is called a Distinguished Name or a DN.  
There are quite a few fields but you can leave some blank  
For some fields there will be a default value,  
If you enter '.', the field will be left blank.  
Country Name (2 letter code) [AU]:  
State or Province Name (full name) [Some-State]:  
Locality Name (eg, city) []:  
Organization Name (eg, company) [Internet Widgits Pty Ltd]:  
Organizational Unit Name (eg, section) []:  
Common Name (e.g. server FQDN or YOUR name) []:  
Email Address []:  

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:  
An optional company name []:
```

### 6. 自己签发证书

```shell
$ openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
```

### 7. 保存密匙到安全的目录

```shell
mv server.* /etc/pki/tls
```

### 8. 修改ssl.conf配置文件
修改/etc/httpd/conf.d/ssl.conf配置文件 ,将文件中这二行信息修改为：

```shell
SSLCertificateFile /etc/pki/tls/server.crt
SSLCertificateKeyFile /etc/pki/tls/server.key
```

### 9. 重启httpd服务器

```shell
systemctl restart httpd
```

---

## Ubuntu下为Apache简单配置SSL的方法(HTTPS的实现)

### 1. 启用ssl模块

```shell
$ sudo a2enmod ssl
```

输出：

```shell
Considering dependency setenvif for ssl:  
Module setenvif already enabled  
Considering dependency mime for ssl:  
Module mime already enabled  
Considering dependency socache_shmcb for ssl:  
Enabling module socache_shmcb.  
Enabling module ssl.  
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.  
To activate the new configuration, you need to run:  
  service apache2 restart
```

输出上面信息说明ssl模块未启动，需要再次执行上述命令：  

```shell
$ sudo a2enmod ssl
```

输出：

```shell
Considering dependency setenvif for ssl:  
Module setenvif already enabled  
Considering dependency mime for ssl:  
Module mime already enabled  
Considering dependency socache_shmcb for ssl:  
Module socache_shmcb already enabled  
Module ssl already enabled
```
 
输出以上信息说明ssl模块成功启动  
### 2. 安装openssl（如果已经安装，可以跳过）

```shell
$ sudo apt-get install openssl
```


### 3. 创建CA签名(==不使用密码去除-des3选项==)

```shell
$ openssl genrsa -des3 -out server.key 1024
```
  
输出（密码随便输，但是后面会用到）：  

```shell
Generating RSA private key, 1024 bit long modulus
.++++++
......++++++
e is 65537 (0x10001)
Enter pass phrase for server.key:  
Verifying - Enter pass phrase for server.key:
```



### 4. 创建CSR(Certificate Signing Request)

```shell
$ openssl req -new -key server.key -out server.csr
```

输出（除了需要server.key密码，其余按回车键）:   

```shell
Enter pass phrase for server.key:  
You are about to be asked to enter information that will be incorporated
into your certificate request.  
What you are about to enter is what is called a Distinguished Name or a DN.  
There are quite a few fields but you can leave some blank  
For some fields there will be a default value,  
If you enter '.', the field will be left blank.  
Country Name (2 letter code) [AU]:  
State or Province Name (full name) [Some-State]:  
Locality Name (eg, city) []:  
Organization Name (eg, company) [Internet Widgits Pty Ltd]:  
Organizational Unit Name (eg, section) []:  
Common Name (e.g. server FQDN or YOUR name) []:  
Email Address []:  

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:  
An optional company name []:
```

### 5. 自己签发证书

```shell
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```


### 6. 复制到相应目录


```shell
$ cp server.crt /etc/ssl/certs  
$ cp server.key /etc/ssl/private
```


### 7. 修改配置文件

```shell
$ cp /etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-enabled/001-ssl.conf
```
  

```shell
$ vim /etc/apache2/sites-enabled/001-ssl.conf
```

在<VirtualHost \*:80>段中，DocumentRoot一行的下方加入内容:  

```shell
SSLEngine On  
SSLOptions +StrictRequire  
SSLCertificateFile /etc/ssl/certs/server.crt  
SSLCertificateKeyFile /etc/ssl/private/server.key
```
端口修改为：443，即<VirtualHost \*:443>(ssl的端口)


### 8. 重启apache

```shell
service apache2 restart
```
