---
title: ES通配符查询
date: 2023-01-31 00:46:00 +0800
categories: [技巧]
tags: [elasticsearch]
pin: false
---

Elasticsearch通配符查询语法如下

```text
GET /meituan_v1/_search

{
  "query": {
    "wildcard": {
      "uripath": {
        "value": "*collect_info/index/form/id*"
      }
    }
  }
}
```
