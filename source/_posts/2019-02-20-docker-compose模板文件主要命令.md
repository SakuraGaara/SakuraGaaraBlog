---
title: docker-compose模板文件主要命令
categories: docker compose
tags:
  - docker compose
abbrlink: 60643
date: 2019-02-20 00:00:00
---

> docker-compose主要命令及功能

<!--more-->

|命令 | 功能|
|- |---- |
build | 指定Dockerfile所在文件的路径
cap_add, cap_drop | 指定容器的内核能力capacity分配
command | 覆盖容器启动后默认执行的命令
cgroup_parent | 指定cgroup组
container_name | 指定容器名称
devices | 指定设备映射关系
depends_on | 指定国歌服务之间依赖关系
dns | 自定义DNS服务器
dns_search | 配置DNS搜索域
dockerfile | 指定额外的编译镜像Dockerfile文件
entrypoint | 覆盖容器中默认的入口命令
env_file | 从文件中获取环境变量
environment | 设置环境变量
expose | 暴露端口，但不映射到宿主机，只被链接的服务访问
extends | 基于其他模板文件进行扩展
external_links | 链接到docker-compose.yml外部的容器
exter_hosts | 指定额外的host名称映射信息
healthcheck | 指定检测应用健康状态的机制
image | 指定镜像名称或镜像ID
isolation | 配置容器隔离机制
labels | 为容器添加Dockers元数据信息
links | 链接到其他服务器中的容器（旧用法，被移除）
logging | 跟日志相关的配置
network_mode | 设置网络模式
networks | 所加入的网络
pid | 跟宿主机系统共享进程命名空间
ports | 暴露端口信息
secrets | 配置应用的秘密数据
security_opt | 指定容器模板标签label机制的默认属性【用户，角色，类型，级别等】
stop_grace_period | 指定应用停止是，容器的优雅停止期限。过期通过SIGKILL强制退出.默认10s
stop_signal | 指定停止容器的信号，默认为SIGTERM
sysctls | 配置容器内核参数
ulimits | 配置容器的ulimits限制值
userns_mode | 指定用户命名空间模式
volumes | 数据卷所挂载路径设置
restart | 指定重启策略
deploy | 指定部署和运行时容器相关配置，命令只在Swarm模式下生效，且只支持docker stack deploy命令部署

