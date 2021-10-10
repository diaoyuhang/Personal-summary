# 倒排索引

### 数据结构

1. 包含这个关键词的document list
2. 关键词在每个doc中出现的次数TF term frequency
3. 关键词在整个索引中出现的次数IDF inverse doc frequency： <font color='red'>出现的次数越多越没有代表性</font>
4. 关键词在当前doc中出现的次数
5. 每个doc的长度，越长相关度越低
6. 包含这个关键词的所有doc的平均长度

# ES核心概念

1. Cluster:每个集群至少包含两个节点
2. Node:集群中的每个节点，一个节点不代表一台服务器
3. Field:一个数据字段，与index和type一起，可以定位一个doc
4. Document:ES中的最小的数据单元，json格式
5. Type：逻辑上的数据分类
6. Index:一类相同或类似的doc，比如一个员工索引，商品索引

**注：和关系型数据库的对应：Doc<>row;type<>table;index<>db**

## 容错机制

master宕机

1. master选举
2. replica容错
3. 重启故障机
4. 故障机启动后，数据增量同步

# ES常用查询

## Query DSL

- match all:

```
GET /product/_search
{
  "query": {
    "match_all": {}
  }
}

GET /product/_search
{
  "query": {
    "match": {
      "name": "nfc"
    }
  }
  , "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```

- multi_match多字段匹配

  ```
  GET /product/_search
  {
    "query": {
      "multi_match": {
        "query": "nfc",
        "fields": ["name","desc"] #name和desc字段中包含nfc的
      }
    }
  }
  
  ```

- _source查询指定字段

  ```
  GET /product/_search
  {
    "query": {
      "match": {
        "name": "xiao"
      }
    }
    , "_source": ["price"]
  }
  ```

- from,size分页

  ```
  GET /product/_search
  {
    "query": {
      "match_all": {}
    }
    , "from": 1,
    "size": 2
  }
  ```

## 全文检索

- query term 搜索的内容不会被拆分

  ```
  GET /product/_search
  {
    "query": {
      "term": {
        "name": "nfc phone" #这个依然会被作为一个词查询
      }
    }
  }
  ```

- terms

  ```
  GET /product/_search
  {
    "query": {
      "terms": {
        "name": [
          "nfc",
          "phone"
        ]
      }
    }
  }
  ```

- match会分词

## match_phrase

短语搜索:查询结果字段中完全包含

## Query and Filter :查询和过滤

