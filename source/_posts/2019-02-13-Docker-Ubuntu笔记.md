---
title: Docker-Ubuntu笔记
categories: docker
tags:
  - docker
  - ubuntu
abbrlink: 20606
date: 2019-02-13 00:00:00
---

# Docker ubuntu 笔记


## Docker安装Ubuntu sshd服务

### 启动容器,进入容器
```sh
docker run -it --name ubuntu_v1 ubuntu bash
```
### ununtu配置软件源
```sh
apt-get update
vi /etc/apt/soutces.list.d/163.list
deb http://mirrors.163.com/ubuntu/ wily main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ wily-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ wily-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ wily-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ wily-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ wily main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ wily-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ wily-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ wily-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ wily-backports main restricted universe multiverse

apt-get update
```

<!--more-->

### 安装配置启动ssh服务
```sh
agt-get install openssh-server
mkdir -p /var/run/sshd
/usr/sbin/sshd -D &
sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
```

### 将Docker Container保存为Image

创建run-ssh.sh脚本
```sh
#!/bin/bash
/usr/sbin/sshd -D
```
```
docker commit ubuntu_v1 sshd:ubuntu
启动新的sshd镜像
docker run -d -p 10022:22 sshd:ubuntu /run.sh
```
