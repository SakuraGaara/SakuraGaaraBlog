---
title: Docker_Registry
categories: docker
tags:
  - docker
  - harbor
abbrlink: 39521
date: 2019-02-14 00:00:00
---

### Docker registry

```
docker run -d -p 5001:5000 --restart=always --name registry -v /data/registry:/var/lib/registry registry:2
```

<!--more-->

- Nginx 代理

```
$ cat registry.conf

upstream docker-registry {
    server 172.16.149.242:5001;
}

server {
    listen 80;
    server_name registry.xxxxxxx.cn;
    add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;
    
    location / {
        auth_basic	   "Please Input username/password";
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

- docker-registry-htpasswd nginx认证

```
htpasswd -c /etc/nginx/conf.d/docker-registry-htpasswd $username
```

### harbor
- harbor是一个企业级的 Docker Registry，可以实现 images 的私有存储和日志统计权限控制等功能，并支持创建多项目(Harbor 提出的概念)，基于官方 Registry V2 实现
- 下载安装地址
<https://github.com/goharbor/harbor>
<https://github.com/goharbor/harbor/releases>

