---
title: kubernetes安装
categories: kubernetes
tags:
  - kubernetes
abbrlink: 50282
date: 2019-03-20 00:00:00
---

> kubeadm: 主要用于提供安装kubernetes的辅助工具
> master: nodes: 安装kubelet, kubeadm, docker, kubectl # kubectl仅master安装即可  
> master: kubeadm init  初始化集群主节点
> nodes: kubeadm join  将node节点加入集群
> Doc: <https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md>  


## 安装前配置  
### 关闭`iptables` 和``firewalld``  
```
$ systemctl stop iptables.service  
$ systemctl disable iptables.service  
$ systemctl stop firewalld.service   
$ systemctl disable firewalld.service  
```
<!--more-->

### 基于主机名  
master和nodes主机名添加至/etc/hosts  
### 主机网络桥接设置  
```
$ cat /proc/sys/net/bridge/bridge-nf-call-ip6tables  
1  
$ cat /proc/sys/net/bridge/bridge-nf-call-iptables  
1  
```
## 安装kubernetes和docker  
### 准备镜像源到/etc/yum.repos.d/  
- docker-ce源   
```
$ cd ; wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -P /etc/yum.repos.d/  
$ yum list docker-ce --showduplicates |sort -r
```
- kubernetes源  

```
$ (cat << EOF  
[kubernetes]  
name=Kubernets Repo  
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/  
enabled=1  
gpgcheck=1  
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg  
EOF  
) > /etc/yum.repos.d/kubernetes.repo  
  
wget https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg  
wget https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg  
rpm --import yum-key.gpg  
rpm --import rpm-package-key.gpg  
yum repolist  
```
  
### 安装kubernetes和docker  
- 安装kublete、kubeadm、docker、kubectl kubectl仅master安装即可  
```
$ yum install docker-ce-18.06.3 kubelet-1.12.0 kubeadm-1.12.0 kubectl-1.12.0  
```
  
### 配置kubernetes和docker  
- docker添加代理，[service]下添加,为了翻墙更改docker默认拉取镜像的源[这一步仅仅是为了翻墙下载镜像]  
```
$ vim /usr/lib/systemd/system/docker.service  
Environment="HTTPS_PROXY=http://www.ik8s.io:10080"  
Environment="NO_PROXY=127.0.0.0/8,172.16.0.0/16"  
```
### 配置开机自动启动  
```python
systemctl enable docker kubelet  
systemctl daemon-reload  
systemctl start docker  
```
### 禁用Swap功能  
```
$ vim /etc/sysconfig/kubelet  
KUBELET_EXTRA_ATGS="--fail-swap-on=false"   
```
### Master初始化  
```
kubeadm init --help;   
kubeadm init --kubernetes-version=v1.12.0 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap  
```
- 初始化参数  
-	--apiserver-advertise-address：表示apiserver对外的地址是什么，默认是0.0.0.0  
-	--apiserver-bind-port：表示apiserver的端口是什么，默认是6443  
-	--cert-dir：加载证书的目录，默认在/etc/kubernetes/pki  
-	--config：配置文件  
-	--ignore-preflight-errors：在预检中如果有错误可以忽略掉，比如忽略 IsPrivilegedUser,Swap.等  
-	--kubernetes-version：指定要初始化k8s的版本信息是什么  
-	--pod-network-cidr ：指定pod使用哪个网段，默认使用10.244.0.0/16  
-	--service-cidr：指定service组件使用哪个网段，默认10.96.0.0/12  
  
- 初始化产生信息  
- 提示在master上执行，和加入master的命令[也可以指定--ignore-preflight-errors忽略Swap]  
  
```
	………………………………………………………………  
	Your Kubernetes master has initialized successfully!  
  
To start using your cluster, you need to run the following as a regular user:  
  
  mkdir -p $HOME/.kube  
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
  sudo chown $(id -u):$(id -g) $HOME/.kube/config  
  
You should now deploy a pod network to the cluster.  
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:  
  https://kubernetes.io/docs/concepts/cluster-administration/addons/  
  
You can now join any number of machines by running the following on each node  
as root:  
  
  kubeadm join 192.168.1.171:6443 --token r9105w.5r5je2vko3jyn8be --discovery-token-ca-cert-hash sha256:d7a99553b49b88d8933785fee033663adebd6d6909323cea6173d009ad66a7f8  
  
  
```
- master继续执行  
```
$ mkdir -p $HOME/.kube  
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config  
```
- 若初始化失败：可先行下载镜像,下载脚本  

```
$ cat pull-images.sh   
#!/bin/bash  
images=(kube-apiserver:v1.12.0 kube-controller-manager:v1.12.0 kube-scheduler:v1.12.0 kube-proxy:v1.12.0 pause:3.1 etcd:3.2.24 coredns:1.2.2)  
  
for ima in ${images[@]}  
do  
   docker pull   registry.cn-shenzhen.aliyuncs.com/lurenjia/$ima  
   docker tag    registry.cn-shenzhen.aliyuncs.com/lurenjia/$ima   k8s.gcr.io/$ima  
   docker rmi  -f  registry.cn-shenzhen.aliyuncs.com/lurenjia/$ima  
done  
```
+ 查看状态信息  

```
$ kubectl get cs  
NAME                 STATUS    MESSAGE              ERROR  
scheduler            Healthy   ok                     
controller-manager   Healthy   ok                     
etcd-0               Healthy   \{"health": "true"\}   
  
$ kubectl get nodes  
NAME        STATUS     ROLES    AGE   VERSION  
master-01   NotReady   master   40m   v1.12.0  
```

master01为未就绪状态，需要一个重要的网络插件flannel  
  
### 安装flannel网络组件  
+ [参考] <https://github.com/coreos/flannel>  
```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  
podsecuritypolicy.extensions/psp.flannel.unprivileged created  
clusterrole.rbac.authorization.k8s.io/flannel created  
clusterrolebinding.rbac.authorization.k8s.io/flannel created  
serviceaccount/flannel created  
configmap/kube-flannel-cfg created  
daemonset.extensions/kube-flannel-ds-amd64 created  
daemonset.extensions/kube-flannel-ds-arm64 created  
daemonset.extensions/kube-flannel-ds-arm created  
daemonset.extensions/kube-flannel-ds-ppc64le created  
daemonset.extensions/kube-flannel-ds-s390x created  
```

## kubectl get -h  
  
| 命令 | 注释 |  
|:- |:--- |  
kubectl get cs | 查看状态信息 cs=componentstatus  
kubectl get nodes | 查看node信息  
kubectl get pods [-n kube-system -o wide] | 查看所有pod,-n 指定命名空间,-o wide 输出扩展信息  
kubectl get ns  | 查看所有命名空间  
  
  