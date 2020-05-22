---
title: 'kubernetes应用[一]'
categories: kubernetes
tags:
  - kubernetes
abbrlink: 63107
date: 2019-03-28 00:00:00
---

> kubectl相关命令操作

- 查看节点详细信息
```
[root@master-01 ~]# kubectl describe node node-01  
```

## pod  
### 创建  
- 创建pod ``kubectl run``   
```
[root@master-01 ~]# kubectl run nginx-deploy --image=nginx:latest --port=80 --replicas=1 --dry-run=true  
nginx-deploy 创建pod名称  
--image 指定镜像  
--port 指定端口  
--reolices 指定pod内创建几个容器  
--dry-run 干跑模式  
```
<!--more-->

### 查看  
- 查看pod ``kubectl get pods``  
```
[root@master-01 ~]# kubectl get pods  
NAME                           READY   STATUS    RESTARTS   AGE  
nginx-deploy-86bf78c77-ftscg   1/1     Running   0          4h17m  
```

- 查看pod详细信息 ``kubectl describe``
```
[root@master-01 ~]# kubectl describe pod nginx-deploy-86bf78c77-ftscg  
```
  
### 删除  
- 删除pod ``kubectl delete``  
```
[root@master-01 ~]# kubectl delete pod nginx-deploy-86bf78c77-ftscg  
pod "nginx-deploy-86bf78c77-ftscg" deleted  
```
删除之后发现会自动创建一个pod  
[root@master-01 ~]# kubectl get pod  
```
NAME                           READY   STATUS    RESTARTS   AGE  
nginx-deploy-86bf78c77-ffj8d   1/1     Running   0          35s  
```


### 彻底删除 ``kubectl delete deployment``  
```
[root@master-01 ~]# kubectl delete deployment nginx-deploy
deployment.extensions "nginx-deploy" deleted
```


## service  
### 创建  
- 创建服务 ``kubectl expose ``  
```
[root@master-01 ~]# kubectl expose deployment nginx-deploy --name=nginx --port=80 --target-port=80 --protocol=TCP  
nginx-deploy 为上面创建的pod名  
--name 指定服务名  
--port 指定pod端口  
--target-port 指定service端口映射pod端口  
--protocol 端口类型  
```

### 查看  
- 查看服务 ``kubectl get service``  
```
[root@master-01 ~]# kubectl get service  
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE  
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   21h  
nginx        ClusterIP   10.104.62.105   <none>        80/TCP    2m35s  
这时，在pod内可以使用地址<http://10.104.62.105>或者服务名<http://nginx>访问nginx,但是节点依旧无法访问  
```
- 查看服务具体信息 ``kubectl describe service``  
```
[root@master-01 ~]# kubectl describe service nginx  
Name:              nginx  
Namespace:         default  
Labels:            run=nginx-deploy  
Annotations:       <none>  
Selector:          run=nginx-deploy  
Type:              ClusterIP  
IP:                10.104.62.105  
Port:              <unset>  80/TCP  
TargetPort:        80/TCP  
Endpoints:         10.244.1.3:80  
Session Affinity:  None  
Events:            <none>  
```

## update  
### 更改pod规模  
- 动态更新pod规模，增加/减少pod ``kubectl scale`` 
```
[root@master-01 ~]# kubectl scale --replicas=5 deployment nginx-deploy  
[root@master-01 ~]# kubectl get pod -o wide  
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE      NOMINATED NODE  
client                         1/1     Running   0          26m   10.244.2.3   node-02   <none>  
nginx-deploy-86bf78c77-8mp2p   1/1     Running   0          57s   10.244.2.5   node-02   <none>  
nginx-deploy-86bf78c77-ffj8d   1/1     Running   0          51m   10.244.1.3   node-01   <none>  
nginx-deploy-86bf78c77-g7s7j   1/1     Running   0          57s   10.244.2.6   node-02   <none>  
nginx-deploy-86bf78c77-tsxmp   1/1     Running   0          57s   10.244.1.4   node-01   <none>  
nginx-deploy-86bf78c77-vddwx   1/1     Running   0          57s   10.244.2.4   node-02   <none>  
```

### 更改Pod镜像  
- 滚动更新image ``kubectl set image``  
- 首先查看一下当前的  
```
[root@master-01 ~]# kubectl describe pod nginx-deploy|grep -E "^Name:|Image:"  
Name:               nginx-deploy-86bf78c77-5h4rh  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-ffj8d  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-qn7zh  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-tkmtj  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-vddwx  
    Image:          nginx:1.14-alpine  
```

- 使用 ``kubectl set image`` 更新镜像  
```
[root@master-01 ~]# kubectl set image deployment nginx-deploy nginx-deploy=nginx:1.15.10-perl  
deployment.extensions/nginx-deploy image updated  
```
- 更新的同时可使用``kubectl rollout status``查看镜像升级状态  
```
[root@master-01 ~]# kubectl rollout status deployment nginx-deploy  
Waiting for deployment "nginx-deploy" rollout to finish: 3 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 3 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 3 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 3 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 3 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 out of 5 new replicas have been updated...  
Waiting for deployment "nginx-deploy" rollout to finish: 2 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 2 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 2 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 1 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 1 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 1 old replicas are pending termination...  
Waiting for deployment "nginx-deploy" rollout to finish: 4 of 5 updated replicas are available...  
deployment "nginx-deploy" successfully rolled out  
再次查看Pod信息，镜像已变动  
[root@master-01 ~]# kubectl describe pod nginx-deploy|grep -E "^Name:|Image:"           
Name:               nginx-deploy-7596d8b647-58xcs  
    Image:          nginx:1.15.10-perl  
Name:               nginx-deploy-7596d8b647-dlf76  
    Image:          nginx:1.15.10-perl  
Name:               nginx-deploy-7596d8b647-shkcn  
    Image:          nginx:1.15.10-perl  
Name:               nginx-deploy-7596d8b647-tttkd  
    Image:          nginx:1.15.10-perl  
Name:               nginx-deploy-7596d8b647-znf24  
    Image:          nginx:1.15.10-perl  
```
  
### 回滚镜像  
- 回滚 ``kubectl rollout undo``  
```
[root@master-01 ~]# kubectl rollout undo deployment nginx-deploy  
deployment.extensions/nginx-deploy  
[root@master-01 ~]# kubectl describe pod nginx-deploy|grep -E "^Name:|Image:"  
Name:                      nginx-deploy-7596d8b647-dlf76  
    Image:          nginx:1.15.10-perl  
Name:                      nginx-deploy-7596d8b647-tttkd  
    Image:          nginx:1.15.10-perl  
Name:                      nginx-deploy-7596d8b647-znf24  
    Image:          nginx:1.15.10-perl  
Name:               nginx-deploy-86bf78c77-55bdr  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-9q54l  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-h6qlf  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-h9fgd  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-vfdzs  
    Image:          nginx:1.14-alpine  
[root@master-01 ~]# kubectl describe pod nginx-deploy|grep -E "^Name:|Image:"  
Name:               nginx-deploy-86bf78c77-55bdr  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-9q54l  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-h6qlf  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-h9fgd  
    Image:          nginx:1.14-alpine  
Name:               nginx-deploy-86bf78c77-vfdzs  
    Image:          nginx:1.14-alpine  
```

## 设置外部访问  
- 首先查看服务  
```
[root@master-01 ~]# kubectl get svc nginx  
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE  
nginx   ClusterIP   10.104.62.105   <none>        80/TCP    5d23h  
```
- 更改服务clusterIP类型``kubectl edit svc``  
```
[root@master-01 ~]# kubectl edit svc nginx  
apiVersion: v1  
kind: Service  
metadata:  
  creationTimestamp: 2019-03-22T07:50:32Z  
  labels:  
    run: nginx-deploy  
  name: nginx  
  namespace: default  
  resourceVersion: "98592"  
  selfLink: /api/v1/namespaces/default/services/nginx  
  uid: 2b765b06-4c77-11e9-a182-000c29bb0a84  
spec:  
  clusterIP: 10.104.62.105  
  ports:  
    port: 80  
    protocol: TCP  
    targetPort: 80  
  selector:  
    run: nginx-deploy  
  sessionAffinity: None  
  type: NodePorts  
status:  
  loadBalancer: {}  
```
- 再次查看服务  
```
[root@master-01 ~]# kubectl get svc nginx  
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE  
nginx   NodePort   10.104.62.105   <none>        80:32660/TCP   5d23h 
```
此时可以使用集群内任意Node节点通过32660端口访问  
  
