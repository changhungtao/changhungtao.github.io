---
title: "CentOS安装jekyll环境"
excerpt: "本文主要介绍在CentOS系统下安装jekyll环境的过程"
last_modified_at: 2019-02-20T10:00:00
toc: true
toc_label: "目录"
toc_icon: "columns"
categories:
  - 技术
tags:
  - Linux
---

#### 1. 安装依赖包
```shell
yum -y install ruby ruby-devel gcc make
```

---

#### 2. 安装jekyll
```shell
gem install jekyll
```

---

#### 3. 安装依赖包

```shell
gem install bundler
gem install rails
gem install json_pure
```

---

#### 4. 遇到错误信息可以考虑运行下面命令
```
bundle install
bundle exec bin/sbapp new APP_SLUG
```

---

#### 5. 初始化目录

```
jekyll new blog
```

---