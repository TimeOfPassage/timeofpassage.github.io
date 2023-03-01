# ElasticSearch知识综述

## 查询分词信息: POST http://[ip]:9200/[index]/_analyze

```
{
    "field":"[analyze_field_name]",
    "text":"[analyze_field_value]"
}
```



## 查询全部: POST http://[ip]:9200/[index]/_search

```
{
    "query": {
        "match_all": {}
    }
}
```

## 单个字段查询: POST http://[ip]:9200/[index]/_search

```
{
    "query": {
        "match": {
            "[field_name]": "[field_value]"
        }
    }
}
```

## 多个字段联合查询: POST http://[ip]:9200/[index]/_search

### tips: must表示条件必须全部满足，如果换成should，表示条件满足其中任何一个, 记住用bool包裹

```
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "[field_name1]": "[field_value1]"
                    }
                },
                {
                    "term": {
                        "[field_name2]": "[field_value2]"
                    }
                },
                {
                    "term": {
                        "[field_name3]": [field_value3]
                    }
                }
            ]
        }
    }
}
```

## 查询增加排序字段: POST http://[ip]:9200/[index]/_search

```
{
    "sort": [
        {
            "[sort_field_name]": "desc"
        }
    ],
    "query": {
        "match": {
            "[field_name]": "[field_value]"
        }
    }
}
```

## 查询增加分页字段: POST http://[ip]:9200/[index]/_search

```
{
    "size": [page_size],
    "query": {
        "match": {
            "[field_name]": "[field_value]"
        }
    }
}
```

## 查询支持正则匹配: POST http://[ip]:9200/[index]/_search

```
{
    "query": {
        "wildcard": {
            "[regex_field_name]": "[regex_expression]"
        }
    }
}
```

## 查询支持查询范围: POST http://[ip]:9200/[index]/_search

### 范围匹配标识如下：(支持组合使用)

> gt: 大于
>
> lt: 小于
>
> gte: 大于并且等于
>
> lte: 小于并且等于

```
{
    "query": {
        "range" : {
            "[field_name]" : {
                "gte" : [range_condition_1],
                "lte" : [range_condition_2]
            }
        }
    }
}
```

## 查询索引中字段分词情况: GET http://[ip]:9200/[index]/_mapping

### tips: 类型为keyword需在index创建时初始化

```
{
    "comment_index": {
        "mappings": {
            "comment_type": {
                "properties": {
                    "[field_name_1]": { // 类型long
                        "type": "long"
                    },
                    "[field_name_2]": { // 类型text
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                  	"[field_name_3]": { // 类型关键字，支持wildcard搜索,
                        "type": "keyword"
                    },
                }
            }
        }
    }
}
```

### 初始化伪json如下:PUT http://[ip]:9200/[index]/_mapping/[index_type]

```
{
    "[index_type]": {
        "_all": {
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word",
            "term_vector": "no",
            "store": "false"
        },
        "properties": {
            "[field_name]": {
                "type": "text",
                "analyzer": "ik_max_word",// 可选
                "search_analyzer": "ik_max_word",// 可选
                "include_in_all": "true",// 可选
                "boost": 8 // 可选
            }
        }
    }
}
// 单独添加字段 PUT http://[ip]:9200/[index]/_mapping/[index_type]
{
  "properties": {
    "[field_name]": {
      "type": "text"
    }
}
```

### 根据id删除数据 DELETE http://[ip]:9200/[index]/[index_type]/[id]

### 返回json如下

```
{
    "_index": "comment_index",
    "_type": "comment_type",
    "_id": "user:6b63eae8a70d1524:commentId:4823213:replyId:52319:relatedReplyId:4823213:serverType:1",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 2,
        "failed": 0
    },
    "_seq_no": 19507,
    "_primary_term": 3
}
```

