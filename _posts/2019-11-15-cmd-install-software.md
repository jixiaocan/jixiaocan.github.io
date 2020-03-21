---
title: Linux 上安装软件的三种方式
date: 2019-11-15 21:35:20
tags: [Linux, command]
categories: [learn, Linux]
snippet: linux, 命令行, 安装软件, 配置仓库
seo:
  date_modified: 2020-02-22 17:02:55 +0800
---

在 Linux 上安装软件有三种常用的方式。

# 使用安装包
Linux 分为两种，一种是 RedHat 类型，使用 rpm 软件包。一种是 Ubuntu 类型，使用 deb 类型的软件包。

以 jdk 的安装为例：

rpm 软件包的管理方式：

```bash
# 安装
$ rpm -i jdk-XXX_linux-x64_bin.rpm

# 查看已安装的软件
$ rpm -qa
$ rpm -qa | grep jdk

# 卸载
$ rpm -e jdk
```

deb 软件包的管理方式：
```bash
# 安装
$ dpkg -i jdk-XXX_linux-x64_bin.deb

# 查看已安装的软件
$ dpkg -l
$ dpkg -l | grep jdk

# 卸载
$ dpkg -r jdk
```

# 使用软件管家
使用软件包安装软件，需要去网上找软件包，然后下载、安装，比如麻烦。Linux 也有软件管家，可以一键安装所需的软件，另外它也能处理软件之间的依赖关系，非常方便。
RedHat 下使用 yum，Ubuntu 下使用 apt。

```bash
# 搜索软件
$ yum search jdk

# 安装软件
$ yum install java-8-openjdk.x86_64

# 删除软件
$ yum erase java-8-openjdk.x86_64
```

```bash
# 搜索软件
$ apt search jdk

# 安装软件
$ apt install openjdk-8-jdk

# 删除软件
$ apt purge openjdk-9-jdk
```

配置仓库地址，对于 CentOS 来讲，配置文件在 `/etc/yum.repos.d/CentOS-Base.repo` 里。
```bash
[base]
name=CentOS-$releasever - Base - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

对于 Ubuntu 来讲，配置文件在 `/etc/apt/sources.list` 里。
```
deb http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
```

# 使用压缩包
有些软件可能即没有安装包，也没有在仓库里面，那就只能通过压缩包安装了。
首先，去网上找到软件的压缩包，可以使用 wget 工具下载。

```bash
# 下载
$ wget https://download.oracle.com/.../jdk-XXX_linux-x64_bin.tar.gz

# 解压缩
$ tar -xzvf jdk-XXX_linux-x64_bin.tar.gz

```
那怎么才能让命令行找到 java 可执行命令呢？

像 windows 配置环境变量一下，Linux 也需要配置：
```bash
export JAVA_HOME=/root/jdk-XXX_linux-x64
export PATH=$JAVA_HOME/bin:$PATH
```
export 命令仅在当前命令行的会话中管用，一旦退出重新登录，就不管用了。需要把配置写到配置文件中。

在当前用户的默认工作目录，有一个 `.bashrc` 文件。将上面两行添加到该文件中即可。
