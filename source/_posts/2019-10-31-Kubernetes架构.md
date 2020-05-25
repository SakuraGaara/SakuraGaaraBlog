---
title: Kubernetes架构
categories: kubernetes
tags:
  - kubernetes
abbrlink: 44175
date: 2019-10-31 00:00:00
---

## Kubernetes的总架构图

![master的工作流程图](/images/img/20191031/Kubernetes-schema.png) 
<!--more-->

## Kubernetes各个组件

### kube-master[控制节点]

- master的工作流程图
![master的工作流程图](/images/img/20191031/master-process.png) 

1. Kubecfg将特定的请求，比如创建Pod，发送给Kubernetes Client。
2. Kubernetes Client将请求发送给API server。
3. API Server根据请求的类型，比如创建Pod时storage类型是pods，然后依此选择何种REST Storage API对请求作出处理。
4. REST Storage API对的请求作相应的处理。
5. 将处理的结果存入高可用键值存储系统Etcd中。
6. 在API Server响应Kubecfg的请求后，Scheduler会根据Kubernetes Client获取集群中运行Pod及Minion/Node信息。
7. 依据从Kubernetes Client获取的信息，Scheduler将未分发的Pod分发到可用的Minion/Node节点上

#### API Server[资源操作入口]
- 提供了资源对象的唯一操作入口，其他所有组件都必须通过它提供的API来操作资源数据，只有API Server与存储通信，其他模块通过API Server访问集群状态。
第一，是为了保证集群状态访问的安全。
第二，是为了隔离集群状态访问的方式和后端存储实现的方式：API Server是状态访问的方式，不会因为后端存储技术etcd的改变而改变。

- 作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。对相关的资源数据“全量查询”+“变化监听”，实时完成相关的业务功能。

#### Controller Manager[内部管理控制中心]
实现集群故障检测和恢复的自动化工作，负责执行各种控制器，主要有：
- endpoint-controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
- replication-controller：定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。

#### Scheduler[集群分发调度器]
1. Scheduler收集和分析当前Kubernetes集群中所有Minion节点的资源(内存、CPU)负载情况，然后依此分发新建的Pod到Kubernetes集群中可用的节点。
2. 实时监测Kubernetes集群中未分发和已分发的所有运行的Pod。
3. Scheduler也监测Minion节点信息，由于会频繁查找Minion节点，Scheduler会缓存一份最新的信息在本地。
4. 最后，Scheduler在分发Pod到指定的Minion节点后，会把Pod相关的信息Binding写回API Server。

#### etcd[存储组件]
支持一致性和高可用的名值对存储组件，Kubernetes集群的所有配置信息都存储在 etcd 中。

### kube-node[服务节点]

- kubelet结构图
![kubelet结构图](/images/img/20191031/kubelet-schema.png) 

#### Kubelet[节点上的Pod管家]
- 负责Node节点上pod的创建、修改、监控、删除等全生命周期的管理
- 定时上报本Node的状态信息给API Server。
- kubelet是Master API Server和Minion之间的桥梁，接收Master API Server分配给它的commands和work，与持久性键值存储etcd、file、server和http进行交互，读取配置信息。

- 具体的工作如下：
1. 设置容器的环境变量、给容器绑定Volume、给容器绑定Port、根据指定的Pod运行一个单一容器、给指定的Pod创建network 容器。
2. 同步Pod的状态、同步Pod的状态、从cAdvisor获取Container info、 pod info、 root info、 machine info。
3. 在容器中运行命令、杀死容器、删除Pod的所有容器。

#### Proxy[负载均衡、路由转发]
Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，运行在每个Node上。Proxy提供TCP/UDP sockets的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息（也可以从file获取），然后根据配置信息在Minion上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据Load Balancer将请求分发到后端正确的容器处理。

Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。

### kubectl（kubelet client）[集群管理命令行工具集]
通过客户端的kubectl命令集操作，API Server响应对应的命令结果，从而达到对kubernetes集群的管理。

### Addons
使用 Kubernetes 资源（DaemonSet、Deployment等）实现集群的功能特性。由于他们提供集群级别的功能特性，addons使用到的Kubernetes资源都放置在 kube-system 名称空间下。

- DNS（kube-dns / core dns）
除了 DNS Addon 以外，其他的 addon 都不是必须的，所有 Kubernetes 集群都应该有 Cluster DNS
- Web UI（Dashboard）
Dashboard 是一个Kubernetes集群的 Web 管理界面。用户可以通过该界面管理集群。
- Kuboard
Kuboard 是一款基于Kubernetes的微服务管理界面.