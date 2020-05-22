---
title: mysqlbinlog实时同步至elasticsearch
categories: elasticsearch
tags:
  - elasticsearch
  - canal
abbrlink: 22055
date: 2019-04-02 00:00:00
---

> canal从mysql中获取binlog日志信息，输出至kafka，logstash从kafka中获取日志信息，写入elasticsearch
> 不过看起来好像毫无意义,所以做的比较简易

<!--more-->

## demo环境介绍
- MYSQL: 使用docker做的一个5.7的环境，用于做自己网站的一个数据库
- Canal: 是阿里巴巴旗下的一款开源项目，纯Java开发。基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了MySQL（也支持mariaDB）
- kafka: 使用docker-compose创建的一个集群[[docker-compose.yml](https://sakuragaara.github.io/kafka/2019/03/07/kafka/)]
- elasticsearch+kibana: 使用docker-compose创建的单节点demo
- logstash: 用于从kafka写入elasticearch

## Canal简单配置
1.修改以下文件

```
vim conf/canal.properties
canal.serverMode = kafka # 配置输出至kafka
canal.mq.servers = 127.0.0.1:9091 # kafka server
```
```
vim conf/example/instance.properties
canal.instance.master.address=127.0.0.1:3000 # mysql数据库地址

canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
canal.instance.defaultDatabaseName = book # 可注释掉，则为所有数据库

canal.mq.topic=book # 定义kafka订阅主题
```
2.启动canal

```
./bin/startup.sh
```

3.以上则完成Canal配置，检查kafka是否可接受到canal传来的日志内容

```python
# -*- coding: utf-8 -*-
from kafka import KafkaConsumer

consumer = KafkaConsumer('book',bootstrap_servers='127.0.0.1:9092', auto_offset_reset='earliest')
for msg in consumer:
    print('topic: %s \n partition: %s \n offset: %s \n headers: %s \n timestamp: %s \n timestamp_type: %s \n key: %s \n value: %s ' % (
          msg.topic,
          msg.partition,
          msg.offset,
          msg.headers,
          msg.timestamp,
          msg.timestamp_type,
          msg.key,
          msg.value.decode('utf8'))
          )
```
向数据库中插入数据，运行脚步是否能得到输出结果

![mysqltest](/images/img/2019-04-02 2.16.59.png)  
![kafka-consumer](/images/img/2019-04-02 2.39.41.png)  

kafka既然能接受到canal传来的日志，接下来就可以配置logstash从Kafka接受消息写入es

## 配置logstash
1.准备logstash配置文件kafka-logstash-es.conf

```
input {
    kafka {
        bootstrap_servers => "172.16.149.242:9092"  # 5.x版本，写法bootstrap_servers
        group_id => "elastic_consumer"
        topics => ["book"]
        consumer_threads => 3
        decorate_events => true
        codec => "json"
    }
}

output {
    elasticsearch {
        hosts=> ["172.16.149.242:9200"]
        index => "logstash-book-%{[table]}-%{+YYYY-MM-dd}"  # 按照不同的表和日期做索引
        codec => "json"
    }
}

```
logstash-kafka-plugin 配置文件[参考](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-kafka.html)
配置与logstash版本相关

2.启动logstash,并查看es是否有索引

```
./bin/logstash -f kafka-logstash-es.conf
```
3.验证es是否有索引和相关数据
![check_es](/images/img/2019-04-02 3.06.28.png)


