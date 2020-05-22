---
title: etcd-Group
categories: etcd
tags:
  - etcd
  - docker-compose
abbrlink: 13465
date: 2019-03-01 00:00:00
---

> docker-compose方式部署etcd集群

<!--more-->

```yaml
version: '3'
services:
        etcd1:
            container_name: etcd1
            image: registry.cn-hangzhou.aliyuncs.com/coreos_etcd/etcd:v3
            ports:
              - "12379:2379"
              - "14001:4001"
              - "12380:2380"
            environment:
              - TZ=CST-8
              - LANG=zh_CN.UTF-8
            command: 
              /usr/local/bin/etcd
              -name etcd1 
              -data-dir /etcd-data
              -advertise-client-urls http://172.33.0.11:2379,http://172.33.0.11:4001
              -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001
              -initial-advertise-peer-urls http://172.33.0.11:2380
              -listen-peer-urls http://0.0.0.0:2380 
              -initial-cluster-token docker-etcd 
              -initial-cluster etcd1=http://172.33.0.11:2380,etcd2=http://172.33.0.22:2380,etcd3=http://172.33.0.33:2380
              -initial-cluster-state new 
            volumes:
              - "/yibao/etcd/data1:/etcd-data"
            networks:
              test_net:
                  ipv4_address: 172.33.0.11
            labels:
              - project.source=
              - project.extra=public-image
              - project.depends=
              - project.owner=LHZ
        etcd2:
            container_name: etcd2
            image: registry.cn-hangzhou.aliyuncs.com/coreos_etcd/etcd:v3
            ports:
              - "22379:2379"
              - "24001:4001"
              - "22380:2380"
            environment:
              - TZ=CST-8
              - LANG=zh_CN.UTF-8
            command: 
              /usr/local/bin/etcd
              -name etcd2 
              -data-dir /etcd-data
              -advertise-client-urls http://172.33.0.22:2379,http://172.33.0.22:4001
              -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001
              -initial-advertise-peer-urls http://172.33.0.22:2380
              -listen-peer-urls http://0.0.0.0:2380 
              -initial-cluster-token docker-etcd 
              -initial-cluster etcd1=http://172.33.0.11:2380,etcd2=http://172.33.0.22:2380,etcd3=http://172.33.0.33:2380
              -initial-cluster-state new 
            volumes:
              - "/yibao/etcd/data2:/etcd-data"
            networks:
              test_net:
                  ipv4_address: 172.33.0.22
            labels:
              - project.source=
              - project.extra=public-image
              - project.depends=
              - project.owner=LHZ
        etcd3:
            container_name: etcd3
            image: registry.cn-hangzhou.aliyuncs.com/coreos_etcd/etcd:v3
            ports:
              - "32379:2379"
              - "34001:4001"
              - "32380:2380"
            environment:
              - TZ=CST-8
              - LANG=zh_CN.UTF-8
            command: 
              /usr/local/bin/etcd
              -name etcd3 
              -data-dir /etcd-data
              -advertise-client-urls http://172.33.0.33:2379,http://172.33.0.33:4001
              -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001
              -initial-advertise-peer-urls http://172.33.0.33:2380
              -listen-peer-urls http://0.0.0.0:2380 
              -initial-cluster-token docker-etcd 
              -initial-cluster etcd1=http://172.33.0.11:2380,etcd2=http://172.33.0.22:2380,etcd3=http://172.33.0.33:2380
              -initial-cluster-state new 
            volumes:
              - "/yibao/etcd/data3:/etcd-data"
            networks:
              test_net:
                  ipv4_address: 172.33.0.33
            labels:
              - project.source=
              - project.extra=public-image
              - project.depends=
              - project.owner=LHZ
networks:
    test_net:
        driver: bridge
        ipam:
            driver: default
            config:
             - subnet: 172.33.0.0/24
```
