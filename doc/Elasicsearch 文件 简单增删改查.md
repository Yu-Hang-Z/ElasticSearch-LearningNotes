Elasicsearch 文档简单增删改查



![CleanShot 2022-05-16 at 21.11.59@2x](http://typoraimages.oss-cn-beijing.aliyuncs.com/img/202205162112157.png)



```
GET users/_doc/1

#自动生成id
POST users/_doc
{
	"user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
}

#create document. 指定Id。如果id已经存在，报错
PUT users/_doc/1?op_type=create
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

#create document. 指定 ID 如果已经存在，就报错
PUT users/_create/1
{
     "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

#Update 指定 ID  (先删除，在写入)
PUT users/_doc/1
{
	"user" : "Mike"

}
#在原文档上增加字段
POST users/_update/1/
{
    "doc":{
        "post_date" : "2019-05-15T14:12:22",
        "message" : "trying out Elasticsearch"
    }
}

# 删除文档
DELETE users/_doc/2
```

