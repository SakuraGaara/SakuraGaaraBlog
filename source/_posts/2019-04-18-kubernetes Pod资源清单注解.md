---
title: kubernetes Pod资源清单注解
categories: kubernetes
tags:
  - kubernetes
abbrlink: 1922
date: 2019-04-18 00:00:00
---


> 创建资源的方法：  
> 定义yaml格式提供配置清单,将资源清单提交给apiServer  
  apiServer可自动将其转换为json格式，而后提交给Scheduler(集群中的调度器)  
  由Scheduler完成调度，调度目标节点完成创建，并启动相关服务  


## Pod核心资源配置清单：
> 资源清单定义帮助 ``kubectl explain`` ``kubectl explain pods`` 等查看可嵌套字段  

### apiVersion
> 格式为group/version，所属群组版本  
支持的版本 ``kubectl api-versions`` 可查看  

### kind
> 定义资源类别，Pod,Service,Deployment,Event,Secret等(注意大小写)  

<!--more-->

### metadata 
> 元数据 ``kubectl explain pods.metadata`` 查看帮助  

```yaml
metadata:
    name: pod_name
    namespace: 名称空间  
    labels: -- 标签  
       key: value
    annotations: 资源注解
```
 

### spec   
> disired state 期望状态, ``kubectl explain pods.spec``查看帮助  

```yaml
spec:
    containers:
    - name: <string>
      image: <string>
      imagePullPolicy: <string> [IfNotPresent,Always,Never]
        # Always：不管镜像是否存在都会进行一次拉取。  
        # Never：不管镜像是否存在都不会进行拉取  
        # IfNotPresent：只有镜像不存在时，才会进行镜像拉取,默认为IfNotPresent，但:latest标签的镜像默认为Always
       
      ports: <[]Object>
      - name: <string>
        containerPort: <integer>

      command: <[]string> # 运行的应用程序，类似docker的entrypoint,并且这里的命令不会允许中shell中
      args: <[]string> # args将参数传给command
      
      lifecycle:
        postStart: <Object> 
            # 容器创建成功后，运行前的任务，用于资源部署、环境准备等，在完成之前，容器处于ContainerCreating状态
        preStop: <Object> 
          # 在容器被终止前的任务，用于优雅关闭应用程序、通知其他系统等等
       
      livenessProbe: <Object> 
        # 存活探针,确定何时重启容器当应用程序处于运行状态但无法做进一步操作，
        # liveness探针将捕获到deadlock，重启处于该状态下的容器 
        # Kubernetes支持3种类型的应用健康检查动作，分别为HTTP Get、Container Exec和TCP Socket  
      readinessProbe: <Object>
        # 就绪探针,确定容器是否已经就绪可以接受流量,
        # 只有当Pod中的容器都处于就绪状态时kubelet才会认定该Pod处于就绪状态
        # 就绪状态, pod才会按照标签加入service  
        ########### livenessProbe,readinessProbe 中接受属性
        exec: <Object>
          command: <[]string>  
        httpGet: <Object>
          host: <string> # 连接的主机名，默认连接到pod的IP。你可能想在http header中设置”Host”而不是使用IP
          httpHeaders: <[]Object>  # 自定义请求的header。HTTP运行重复的header
          path: <string> # 访问的HTTP server的path
          port: <string> # 访问的容器的端口名字或者端口号。端口号必须介于1和65525之间
          scheme: <string> # 连接使用的schema，默认HTTP
        tcpSocket: <Object>
          host: <string>
          port: <string>
        initialDelaySeconds: <integer> # 容器启动后，等待多少秒之后进行第一次探测
        periodSeconds: <integer> # 执行探测的频率。默认是10秒，最小1秒
        successThreshold: <integer> # 探测失败后，最少连续探测成功多少次才被认定为成功。默认是1。对于liveness必须是1。最小值是1
        failureThreshold: <integer> # 探测成功后，最少连续探测失败多少次才被认定为失败。默认是3。最小值是1
        timeoutSeconds: <integer> # 探测超时时间。默认1秒，最小1秒

      resources: # CPU的单位是milicpu，单位后缀m代表“千分之一核心”，500mcpu=0.5cpu；而内存的单位则包括E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki等
        requests: # 请求
          cpu: "300m"
          memory: "64Mi"
        limits: # 上限
          cpu: "500m"
          memory: "128Mi"

    initContainers: # Init Container在所有容器运行之前执行（run-to-completion），常用来初始化配置
    - name: <string>

    nodeName: nodename 指定node节点
    nodeSelector: <map[string]string> 指定node的label标签
    restartPolicy: <string> Always, OnFailure, Never. Default to Always.
    # Always：只要退出就重启
    # OnFailure：失败退出（exit code不等于0）时重启
    # Never：只要退出就不再重启
```




### status 
> 当前状态 current state(只读), 由kubernetes集群维护,不需要用户自己定义



## Demo Code

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: flaskapp-pod-v1
  namespace: default
  labels:
    app: flaskapp
    version: v1
  annotations: 
    note: "This is Flask app."
spec:
  containers:
  - name: flaskapp-container-v1
    image: sakuragaara/flaskapp:v1
    imagePullPolicy: Never
    ports:
      - name: http
        containerPort: 5000
    livenessProbe:
      httpGet:
        port: 5000
        path: /index
      initialDelaySeconds: 30
      periodSeconds: 5
      successThreshold: 1
      timeoutSeconds: 8
    env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_APP_NAME
          value: Flask APP
    resources:
      requests:
        cpu: "0.1"
        memory: "56Mi"
      limits:
        cpu: "1"
        memory: "128Mi"
```

***tcpSocket没有加host，主要是host默认为containers的IP，而flask启动是0.0.0.0，使用127.0.0.1监听会报错***
