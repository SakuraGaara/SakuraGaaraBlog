---
title: python之elasticsearch初试
categories: elasticsearch
tags:
  - elasticsearch
  - python
abbrlink: 57654
date: 2019-03-27 00:00:00
---

> python elasticsearch库应用
> Doc: <https://pypi.org/project/elasticsearch/>


<!--more-->

```python
# --coding: utf-8 --
from elasticsearch import Elasticsearch


def get_es_config():
    es = Elasticsearch(
        ['http://xxxxxxxxxxx.public.elasticsearch.aliyuncs.com'],
        http_auth=('elastic', '123456'),
    )
    return es


def search_body(index, doc):
    """搜索"""
    es=get_es_config()
    if es.indices.exists(index=index):
        res=es.search(index=index,body=doc)
        for line in res["hits"]["hits"]:
            print(line)


def close_index(index):
    """关闭/删除索引"""
    es = get_es_config()
    if es.indices.exists(index=index):
        if es.indices.close(index):
            print(True)
        if es.indices.delete(index):
            print(True)


search_body("my_index-2018.01.30",
            doc={
                "query": {
                    "match": {
                        "geoip.city_name": "Hangzhou"
                    }
                }
            }
            )

close_index("my_index-2018.01.30")

```