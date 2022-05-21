# Elasticsearch中search API

指定查询的索引

| 语法                    | 范围              |
| ----------------------- | ----------------- |
| /_search                | 集群上所有的索引  |
| /index1/_search         | index1            |
| /index1,index-2/_search | index1和index2    |
| /index*/_search         | 以index开头的索引 |





## URL  Search

  在 URL 使用查询参数，通过URL query实现搜索

```  
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
{
    "profile":true
}
```

-  q 指定查询语句，使用 Query String Syntax
- df 默认字段，不指定时查询所有字段
- sort 排序 /from 和 size 用于分页
- profile 可以查看查询如何执行



### Query String Syntax

- 指定字段  V.S  泛查询
  - q=title:2012 / q=2012  (置顶字段可代替df)
- Term  V.S  Phrase
  - Beautiful Mind 等效于 Beautiful OR Mind
  - “Beautiful Mind"，等效于 Beautiful AND Mind。 Phrase 查询，还要求前后顺序保持一致
- 分组与引号
  - title:(Beautiful AND Mind)
  - title="Beautiful Mind"
- 布尔操作
  - AND / OR / NOT 或者 && / ll / ！
    - 必须大写
    - title:(matrix NOT reloaded)
- 分组
  - +表示 must
  - -表示 must_not
  - title:(+matrix - reloaded)
- 范围查询
  - 区间表示：[]闭区间，{} 开区间
    - year: {2019 TO 2018}
    - year: [* TO 2018]
- 算数符号
  - year:>2010
  - year:(+>2010 && <=2018)
  - year:(+>2010 +<=2018)
- 通配符查询（通配符查询效率低，占用内存大，不建议使用。特别是放在最前面）
  - ？代表1个字符，*代表 0 或多个字符
    - title:mi?d
    - title:be*
- 正则表达
  - title:[bt]oy
- 模糊查询与近似查询
  - title:befutifl~1
  - title:"lord rings"~2

## Requset Body & Query DSL

知识点

- 分页 / 排序
- Source Filtering
- Match 查询和 Match Phrase 查询
- 调整 Precision & Recall

### Request Body Search

- 将查询语句通过 HTTP Requedt Body 发送给 Elasticsearch
- Query DSL

```
POST /movies,404_idx/_search/ignore_unavailable=true
{
	"profile":true,
	"query":{
	  "match_all":{}
	}
}
```

- 分页
  - From 从0 开始，默认返回10个结果
  - 获取靠后的翻页成本较高


```
POST /kibana_sample_data_ecommerce/_search
{
  "from": 10,
  "size": 20,
  "query": {
    "match_all": {}
  }
}
```



- 排序

  

```
POST /kibana_sample_data_ecommerce/_search
{
// 可简写  "sort":[{"order_date":"desc"}]
  "sort": [
    {
      "order_date": {
        "order": "desc"
      }
    }
  ], 
  "from": 0,
  "size": 2,
  "query": {
    "match_all": {}
  }
}
```



- _source filtering
  - 过滤 _source中不需要展示的字段

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["order_date"],
  "from": 10,
  "size": 5,
  "query": {
    "match_all": {}
  }
}
```



- 脚本字段
  - 合成原本_source中原本不存在的字段

```
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_fields": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  }, 
  "from": 10,
  "size": 5,
  "query": {
    "match_all": {}
  }
}
```

- 使用查询表达式 -- Match


```
GET /movies/_search
{
  "query": {
    "match": {
      // "title": "last christmas"
      "title": {
        "query": "last christmas",
        "operator":"or"
      }
    }
  }
}
```

```
GET /movies/_search
{
  "query": {
    "match": {
      // "title": "last christmas"
      "title": {
        "query": "last christmas",
        "operator":"and"
      }
    }
  }
}
```

- 短语搜索 -- Match Phrase
  - and 的关系且顺序有要求
  - slop 间隔步长


```
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love",
        "slop": 1

      }
    }
  }
}
```

### Query DSL

- Query String Query
  - 类似URL Query

```
POST users/_search
{
	"query":{
		"query_string":{
			"default_field":"name",
			"query":"Ruan AND Yiming"
		}
	}
}
```

```
POST users/_search
{
	"query":{
		"query_string":{
			"fields":["name","about"]
			"query":"(Ruan AND Yiming) OR (Java AND
				Elasticsearch)"
		}
	}
}
```

- Simple Query String Query
  - 类似 Query String，但是会忽略错误的语法，同时只支持部分查询语法
  - 不支持 AND OR NOT，会当作字符串处理
  - Term 之间默认的关系是 OR，可以指定 Operator
  - 支持部分逻辑
    - +替代 AND
    - |替代 OR
    - -替代 NOT

```
POST users/_search
{
	"query":{
		simple_query_string:{
			"query":"Ruan -Yiming",
			"fields":["name"],
			"default_operator":"AND"
		}
	}
}
```



