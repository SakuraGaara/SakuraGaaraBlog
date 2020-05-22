---
title: kubernetes-Namespace资源限制
categories: kubernetes
tags:
  - kubernetes
abbrlink: 38928
date: 2019-11-20 00:00:00
---

http://docs.kubernetes.org.cn/

### 创建namespace
命令直接创建
```sh
kubectl create namespace dev
```
或者
<!--more-->
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
spec: {}
status: {}
```

### 为Namespace配置Pod配额限制
k8s可以为命名空间进行限制，如进行Pod，Service的数量限制等等, 配额通过`ResourceQuota`对象设置

1. 设置`ResourceQuota`对象配置文件，pod-default.yaml  
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-dev
spec:
  hard:
    pods: "2"
    services: "1"
```
2. 将配置应用到相应的命名空间
```sh
kubectl create -f pod-default.yaml --namespace=dev
```
3. 查看和验证资源限制
```sh
#查看
kubectl get quota --namespace=dev -o yaml
#创建pod
kubectl run flaskapp --image=sakuragaara/flaskapp:v3 --port=5000 --replicas=3 --namespace=dev
#查看pod及相关资源
kubectl get all -n dev -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
pod/flaskapp-5d4f4fb7c-bx5v2   1/1     Running   0          29s   172.30.91.36   192.168.1.36   <none>           <none>
pod/flaskapp-5d4f4fb7c-hx6j9   1/1     Running   0          29s   172.30.53.30   192.168.1.35   <none>           <none>

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                    SELECTOR
deployment.apps/flaskapp   2/3     2            2           29s   flaskapp     sakuragaara/flaskapp:v3   run=flaskapp

NAME                                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                    SELECTOR
replicaset.apps/flaskapp-5d4f4fb7c   3         2         2       29s   flaskapp     sakuragaara/flaskapp:v3   pod-template-hash=5d4f4fb7c,run=flaskapp
```
查看资源会发现，期望创建的pod3个，但就绪的状态只有2个.之后进行修改`ResourceQuota`资源pod数量，pod会自动更新成期望值3个.

### 为Namespace配置默认CPU和Memory限制

k8s可以为命名空间进行限制，如CPU,Memory等重要的资源, 配额通过`LimitRange`对象设置
1. 设置`LimitRange`对象配置文件，cpu-mem-default.yaml
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-range
spec:
  limits:
  - default:
      cpu: 0.5
      memory: 256Mi
    defaultRequest:
      cpu: 0.3
      memory: 128Mi
    type: Container
```
2. 将配置应用到相应的命名空间
```sh
kubectl create -f cpu-mem-default.yaml --namespace=dev
```
3. 查看验证限制规则
```sh
#查看
kubectl get limitranges -n dev -o yaml
#创建pod
kubectl run flaskapp-demo --image=sakuragaara/flaskapp:v3 --port=5000 --replicas=1 --namespace=dev
#查看pod是否默认应用资源限制
kubectl describe pod -n dev flaskapp-demo-5b9b976879-8qxbq
Name:               flaskapp-demo-5b9b976879-8qxbq
Namespace:          dev
Priority:           0
PriorityClassName:  <none>
Node:               192.168.1.36/192.168.1.36
Start Time:         Wed, 20 Nov 2019 17:39:46 +0800
Labels:             pod-template-hash=5b9b976879
                    run=flaskapp-demo
Annotations:        kubernetes.io/limit-ranger:
                      LimitRanger plugin set: cpu, memory request for container flaskapp-demo; cpu, memory limit for container flaskapp-demo
...........................
Containers:
  flaskapp-demo:
    Container ID:   docker://c457d1aa261af58265064b937c0725746d2ef590015ac5cd9b850d1ebfc323bc
    Image:          sakuragaara/flaskapp:v3
    Image ID:       docker-pullable://sakuragaara/flaskapp@sha256:fc52eb0cd726a3536babc2f0af8f3a55950665077b65e0da9c40c2d8d4561368
    Port:           5000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 20 Nov 2019 17:39:47 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        300m
      memory:     128Mi
    Environment:  <none>
...........................
```
之后对`LimitRange`对象进行修改并应用，但已经创建的pod是不会自动更新资源限制.
如果你在定义pod时，已经定义了资源限制，则不会应用默认的配置.


### 为Namespace配置单个pod最大CPU和Memory限制
1. 通过`LimitRange`对象设置, cpu-mem-container-max.yaml
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-container-max
spec:
  limits:
  - max:
      cpu: "400m"
      memory: "256Mi"
    min:
      cpu: "200m"
      memory: "128Mi"
    type: Container
```
2. 将配置应用到相应的命名空间
```sh
kubectl create -f cpu-mem-container-max.yaml --namespace=dev
```
3. 查看限制规则
```sh
#查看
kubectl get limitranges -n dev -o yaml
```
4. 创建一个超额的pod资源进行验证,pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
spec:
  containers:
  - name: default-mem-demo
    image: sakuragaara/flaskapp:v3
    resources:
      limits:
        memory: "768Mi"
```
创建时提示
```sh
kubectl create -f pod.yaml -n dev
Error from server (Forbidden): error when creating "pod.yaml": pods "default-mem-demo" is forbidden: maximum memory usage per Container is 256Mi, but limit is 768Mi.
```

### 为Namespace最大CPU和Memory限制
此配置为空间内运行的所有容器配置CPU和内存配额

namespace-limit.yaml
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-limit
spec:
  hard:
    requests.cpu: "0.5"
    requests.memory: 512Mi
    limits.cpu: "1"
    limits.memory: 1Gi
```
- 每个容器必须设置内存请求（memory request），内存限额（memory limit），cpu请求（cpu request）和cpu限额（cpu limit）。
- 所有容器的内存请求总额不得超过500 MiB。
- 所有容器的内存限额总额不得超过1 GiB。
- 所有容器的CPU请求总额不得超过0.5 CPU。
- 所有容器的CPU限额总额不得超过1 CPU。

deploy.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: flask-deploy
  namespace: dev
  labels:
    app: flask
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: flask
  template:
    metadata:
      name: flask-pod
      namespace: dev
      labels:
        app: flask
    spec:
      containers:
      - name: flask-container
        image: sakuragaara/flaskapp:v3
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 5000
        resources:
          requests:
            memory: "128Mi"
            cpu: "0.2"
          limits:
            memory: "256Mi"
            cpu: "0.4"
```
根据验证，3个pod将超过总资源限制，会创建成功两个
而资源限制满了之后，在此空间下创建其他pod，则会提示
```sh
kubectl create -f pod.yaml -n dev
Error from server (Forbidden): error when creating "pod.yaml": pods "default-mem-demo" is forbidden: exceeded quota: namespace-limit, requested: limits.cpu=400m,requests.cpu=400m, used: limits.cpu=800m,requests.cpu=400m, limited: limits.cpu=1,requests.cpu=500m
```