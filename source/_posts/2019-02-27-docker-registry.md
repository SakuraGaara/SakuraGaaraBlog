---
title: docker-registry
categories: docker
tags:
  - docker
abbrlink: 14399
date: 2019-02-27 00:00:00
---

### 安装registry
```
#!/bin/bash
docker run -d \
    -p 5001:5000 \
    --restart=always \
    --name registry \
    -v $PWD/docker:/var/lib/registry \
    registry:2
```

<!--more-->

### 配置nginx代理，带密码验证
```
/etc/nginx/conf.d/docker-registry-htpasswd
htpasswd -c docker-registry-htpasswd username

cat registry.conf
upstream docker-registry {
    server 172.16.149.242:5001;
}

server {
    listen 80;
    #listen 443;
    server_name registry.ngames-dev.cn;
    add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;
    
    #ssl on;
    #ssl_certificate /etc/nginx/conf.d/server.crt;
    #ssl_certificate_key /etc/nginx/conf.d/server.key;

    location / {
        auth_basic         "Please Input username/password";
        auth_basic_user_file "/etc/nginx/conf.d/docker-registry-htpasswd";
        proxy_pass         http://docker-registry;
        proxy_set_header   Host             $http_host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarder-Porto $scheme;
        proxy_read_timeout         600;
        client_max_body_size 0;
    }
}

```

### 去https

- 验证登陆仓库

```
docker login registry.ngames-dev.cn
Username: sakura
Password: 
Error response from daemon: Get https://registry.ngames-dev.cn/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

```

- 从docker1.3.2版本开始默认docker registry使用的是https

```
cat /etc/docker/daemon.json 
{
    "insecure-registries": ["registry.ngames-dev.cn"]
}


systemctl daemon-reload
systemctl restart docker
```

- 之后再次登陆，成功后会在$HOME/.docker/config.json中auths添加一串登陆记录

```json
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "c2FrdXJhZ2FhcmE6R3VvanVuNTIzMDA="
                },
                "registry.ngames-dev.cn": {
                        "auth": "c2FrdXJhOjUyMzAwMxxxxxxxxxxxxxxxx"
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/18.09.2 (linux)"
        }
}
```

### 上传镜像
- 镜像更改名字
- 上传镜像

```
docker tag alpine:latest registry.ngames-dev.cn/sakura/alpine:v1
docker push registry.ngames-dev.cn/sakura/alpine:v1
```

- 查看镜像

```
curl -XGET username:password@registry.ngames-dev.cn/v2/_catalog
{"repositories":["sakura/alpine"]}
```
- 查看镜像标签信息

```
curl -XGET username:password@registry.ngames-dev.cn/v2/sakura/alpine/tags/list
{"name":"sakura/alpine","tags":["v1"]}
```

- 删除镜像
- 1,获取Docker-Content-Digest

```
curl --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I username:password@registry.ngames-dev.cn/v2/sakura/alpine/manifests/v1

HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Sun, 03 Mar 2019 02:32:15 GMT
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Content-Length: 528
Connection: keep-alive
Docker-Content-Digest: sha256:25b4d910f4b76a63a3b45d0f69a57c34157500faf6087236581eca221c62d214
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:25b4d910f4b76a63a3b45d0f69a57c34157500faf6087236581eca221c62d214"
X-Content-Type-Options: nosniff
Docker-Distribution-Api-Version: registry/2.0
```

- 2,删除

```
curl -XDELETE username:password@registry.ngames-dev.cn/v2/sakura/alpine/manifests/sha256:25b4d910f4b76a63a3b45d0f69a57c34157500faf6087236581eca221c62d214
```
