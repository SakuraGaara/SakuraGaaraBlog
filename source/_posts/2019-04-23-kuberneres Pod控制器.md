---
title: kubernetes Pod控制器
categories: kubernetes
tags:
  - kubernetes
abbrlink: 24000
date: 2019-04-23 00:00:00
---

> Pod是kubernetes的最小单元, 自主式创建的Pod删除之后就没有了，但是通过资源控制器创建的Pod如果被删除还会重建  
  **自主式Pod：** 创建一个Pod资源清单，``kubectl create -f xxxx.yaml``  
  **资源控制器：** ``kubectl run xxxx --image=xxxx --replicas=2 --port=80`` 默认属于 ``deployment``控制器管理  
  **Pod控制器**  就是用于实现代替我们去管理pod的中间层，并帮助我们确保每一个Pod资源处于我们所定义或期望的目标状态pod资源出现故障首先要重启，如果一直重启有问题的话会基于某种策略重新编排。自动适应期望pod数量  
  **[Pod资源清单](https://sakuragaara.github.io/kubernetes/2019/04/18/kubernetes-Pod%E8%B5%84%E6%BA%90%E6%B8%85%E5%8D%95%E6%B3%A8%E8%A7%A3/#) 作为模板内嵌在Pod控制器内进行创建**  


<!--more-->

## Pod控制器

### ReplicaSet  
**ReplicaSet: 新一代的ReplicationController**  
代用户创建指定数量的Pod副本数量，并确保Pod副本一直满足用户期望的数量状态，多退少补，而且还支持自动扩缩容机制  
***但是kuberneters不建议直接使用ReporcaSet***  
ReporcaSet主要由三个重要组件组成：  
（1） 管控用户期望的Pod副本数量  
（2） 标签选择器，判定归自己管理可控制的Pod副本  
（3） Pod资源模板（当现存的pod副本数量不足，会根据Pod资源模板进行新建，帮助用户管理无状态的Pod资源，精确反应用户定义的目标数量）  

### Deployment  
**Deployment： 无状态，守护进程类，只关注群体不关注个体**  
工作在ReplicaSet之上，通过ReplicaSet管理无状态Pod资源，是目前来说最好的控制器(意味着满足ReplicaSet的所有功能)    
除此之外，支持滚动更新和回滚等机制，而且还提供声明式配置，可随时通过修改声明来定义目标期望状态  

### DaemonSet
***DaemonSet：无状态，守护进程类，只关注群体不关注个体 Pod与Node一对一的关系***  
确保集群中每一个节点上只运行一个特定的Pod副本，系统级的后台任务，新增节点他都会自动添加pod  
也可以是满足条件的节点上运行特定的副本  

### Job
***Job：有状态，一次性任务***  
只要完成就立即退出，不需要重启或重建，没有完成重构job。只能执行一次性任务　  

### Cronjob
***Cronjob：有状态，周期性任务***  
周期性任务控制，不需要持续后台运行  

### StatefulSet
***StatefulSet：管理有状态应用***  
管理有状态应用（redis cluster）针对管理的应用器配置管理是不一样的，没有什么共通的规律，需要人为的封装在脚本中实行，相当之大的逻辑处理。（运维技能封装到运维脚本中）  



## Pod控制器应用
### ReplicaSet
help:``kubectl explain ReplicaSet`` 简写 ``rs ``   
ReplicaSet先定义控制器 ``apiVersion``,``kind``,``metadata``,``spec``等资源  
而在``spec``资源中，定义Pod数量，控制器标签选择器和Pod的模板  
模板中定义Pod``metadata``,``spec``等资源，metadata中Pod的标签必须需要继承控制器的标签属性  
支持动态修改``kubectl edit rs myapp`` , 但是并不支持修改模板中的Pod内容，因为控制器定义的数量不变,除非人为手动删除控制器中的pod  
  

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp
  namespace: default
spec:
  replicas: 2	# Pod数量
  selector:
    matchLabels:  # 定义ReplicaSet标签
      app: myapp
  template:
    metadata:
      name: flask-app
      namespace: default
      labels:	# 必须继承ReplicaSet标签，否则一直创建
        app: myapp
        ifram: flask
    spec:
      containers:
        - name: flaskapp
          image: sakuragaara/flaskapp:v1
          imagePullPolicy: IfNotPresent
          ports:
          - name: http
            containerPort: 5000
          livenessProbe:
            tcpSocket:
                port: 5000
            timeoutSeconds: 3
      restartPolicy: Always
```

### Deployment
help: ``kubectl explain deployment``  简写 ``deploy``   
Deployment与ReplicaSet一样，定义控制器``apiVersion``,``kind``,``metadata``,``spec``等资源  
而在``spec``资源中，定义Pod数量，控制器标签选择器和Pod的模板  
支持动态修改Pod副本数量，更新资源镜像，支持回滚  
  修改Pod副本数量,更新资源Pod镜像  
  可以使用``kubectl edit deployment flask-deploy``编辑修改，也可以编辑文件flask-deploy.yaml之后，``kubectl apply -f flask-deploy.yaml``更新，默认为滚动更新方式  
  更新Pod镜像资源后，可以使用``kubectl get rs -o wide ``查看,默认的rs模板被保留，这就是备份，随时可以使用它进行回滚  
#### 创建和查看  
  ![创建和查看](/images/img/20190423/kubectl_create.png)  
  
#### 更新副本数量  
  - 可使用修改yaml更新，可以``kubectl edit``或者``kubectl scale``更新  
  ![更新副本数量](/images/img/20190423/kubectl_update1.png)  

#### 更新镜像
  - 使用修改yaml更新，也可以使用``kubectl set image``的方式进行更新  
    ![更新镜像](/images/img/20190423/kubectl_update2.png)  
    ![更新镜像](/images/img/20190423/kubectl_update3.png)  

#### 回滚  
  - 使用``kubectl rollout history deployment flask-deploy``,可查看Deployment更新历史版本,``--revision=1`` 可查看Deployment的详细信息，在``kubectl apply``时添加``--record``才会查看到更新命令备注信息  
  - 使用``kubectl rollout undo deployment flask-deploy --to-revision=1`` 进行回滚  
    ![回滚](/images/img/20190423/kubectl_rollout.png)  

#### 使用修改补丁方式  
  如``kubectl patch deployment flask-deploy -p '{"spec":{"replicas":5}}'`` -p参数，只支持json格式修改  

#### 灰度发布（金丝雀发布）  
  Deployment默认为滚动更新方式，``kubectl describe deployment flask-deploy`` 可以查看，可以修改其发布方式为灰度发布
  ``spec.strategy.rollingUpdate.maxUnavailable``
  用来控制不可用Pod数量的最大值，从而在删除旧Pod时保证一定数量的可用Pod。当replicas=3，如果配置为1，则更新过程中会保证至少有2个可用Pod。默认为1  
  ``spec.strategy.rollingUpdate.maxSurge``
  用来控制超过期望数量的Pod数量最大值，从而在创建新Pod时限制总量。当replicas=3,如配置为1,则更新过着中会保证Pod总数量最多有4个。默认为1  
  （1）补丁方式更新为灰度发布  
    ``kubectl patch deploy  flask-deploy -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'``  
  ![灰度发布](/images/img/20190423/kubectl_strategyType.png)  
  （2）更新镜像，暂停,并监控pod更新状态
    ![k1](/images/img/20190423/k1.png)  
    ![k2](/images/img/20190423/k2.png)  
    ![k3](/images/img/20190423/k3.png)  


```yaml
cat flask-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: flask-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      name: flask-pod
      namespace: default
      labels:
        app: flask
        langure: python
    spec:
      containers:
      - name: flask-container
        image: sakuragaara/flaskapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 5000
        livenessProbe:
          httpGet:
            port: 5000
            scheme: HTTP
            path: /index
          initialDelaySeconds: 10
          timeoutSeconds: 5
```