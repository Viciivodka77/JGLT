

# elatsicSearch命令集

```http
GET /_search
{
  "query": {
    "match_all": {}
  }
}
```

## 索引

### 创建索引

`PUT /hello`

### 查看索引

`GET /_cat/indices`

>  带表头携带参数?v
>
> `GET /_cat/indices?v`

### 删除索引

`DELETE /hello`

## 类型与映射

### 创建类型与映射

> 类型操作 /index

```http
PUT /hello
{
  "mappings": {
    "emp":{
      "properties":{
        "id":{
          "type":"keyword" 
        },
        "name":{
          "type":"text"
        },
        "age":{
          "type":"integer"
        },
        "bir":{
          "type":"date"
        }
      }
    }
  }
}
```

### 查看创建的索引以及索引中的映射

> 查看索引
>
> `GET /hello`
>
> 查看索引中的映射
>
> `GET /hello/_mapping`



## 文档

### 插入文档

> ***插入一条文档 PUT /索引/类型/{id}***
>
> ```http
> PUT /hello/emp/2
> {
>   "id":2,
>   "name":"李四",
>   "age":25,
>   "bir":"2000-03-23"
> }
> ```



> ***不指定ID  随机生成  "_id" : "2jM6Y3gBsVEMiPvdjyzC"***
>
> ```http
> POST /hello/emp/
> {
>   "id":3,
>   "name":"王五",
>   "age":42,
>   "bir":"2020-03-23"
> }
> ```



### 查询文档

> ***根据id查询文档***‘ /hello/emp/{id}
>
> `GET /hello/emp/1`
> `GET /hello/emp/2`

> ***查询所有文档***
>
> `GET /hello/emp/_search`

### 删除文档

`DELETE /hello/emp/2`

### 更新文档

> ***1. 更新 这种更新不保留原始数据更新->先删除再插入***
>
> ```http
> POST /hello/emp/2jM6Y3gBsVEMiPvdjyzC
> {
>   "name":"法外狂徒"
> }
> ```
>
> 

> ***2.保留原始数据更新  也可以添加新字段***
>
> ```http
> POST /hello/emp/2jM6Y3gBsVEMiPvdjyzC/_update
> {
>   "doc":{
>     "name":"无情",
>     "dept":"好家伙"
>   }
> }
> ```
>
> 

## 全文索引

> *准备*
>
> ```http
> PUT /ems
> {
>   "mappings": {
>     "emp":{
>       "properties":{
>         "name":{
>           "type":"text"
>         },
>         "age":{
>           "type":"integer"
>         },
>         "bir":{
>           "type":"date"
>         },
>         "content":{
>           "type":"text"
>         },
>         "address":{
>           "type":"keyword"
>         }
>       }
>     }
>   }
> }
> ```
>
> > ***批处理***
> >
> > ```http
> > PUT   /ems/emp/_bulk
> >   {"index":{}}
> >   {"name":"小黑","age":23,"bir":"2012-12-12","content":"为开发团队选择一款优秀的MVC框架是件难事儿，在众多可行的方案中决择需要很高的经验和水平","address":"北京"}
> >   {"index":{}}
> >   {"name":"王小黑","age":24,"bir":"2012-12-12","content":"Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式","address":"上海"}
> >   {"index":{}}
> >   {"name":"张小五","age":8,"bir":"2012-12-12","content":"Spring Cloud 作为Java 语言的微服务框架，它依赖于Spring Boot，有快速开发、持续交付和容易部署等特点。Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区Spring 和Netflix 、Pivotal 两大公司的推动下越来越完善","address":"无锡"}
> >   {"index":{}}
> >   {"name":"win7","age":9,"bir":"2012-12-12","content":"Spring的目标是致力于全方位的简化Java开发。 这势必引出更多的解释， Spring是如何简化Java开发的？","address":"南京"}
> >   {"index":{}}
> >   {"name":"梅超风","age":43,"bir":"2012-12-12","content":"Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API","address":"杭州"}
> >   {"index":{}}
> >   {"name":"张无忌","age":59,"bir":"2012-12-12","content":"ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口","address":"北京"}
> > ```



### ES 中高级查询

#### QueryString方式 

> ***QueryString方式      sort排序 :desc倒序 分页 from(offset) size大小(limit) ***
>
> ```http
> GET /ems/emp/_search?q=*&sort=age:desc&size=5&from=4&_source=name,age,bir
> ```
>
> ```http
> GET /ems/emp/_search?q=spring
> ```

#### QueryDSL方式查询

##### 查询所有

```http
GET /ems/emp/_search
{
  "query":{
    "match_all": {}
  }
}
```

##### 查询所有并排序 sort

```http
GET /ems/emp/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      },
      "address": {
        "order": "desc"
      }
    }
  ]
}
```

> *分页*
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "match_all": {}
>   },
>   "size": 2,
>   "from": 4
> }
> ```

##### 指定字段查找

```http
GET /ems/emp/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 4
  , "_source": ["name","address"]
}
```



##### 关键字查找  

> ***name是text类型 会分词查找 包含小就行***
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "term": {
>       "name": {
>         "value": "小"
>       }
>     }
>   }
> }
> ```



> ***bir是date类型 不会分词查找所以按照整体查寻，找不到数据***
>
> ```http
> GET /ems/emp/_search/
> {
>   "query": {
>     "term": {
>       "bir": {
>         "value": "2012-12-01"
>       }
>     }
>   }
> }
> ```



##### 范围查找

> ***查询age>=5,<=10的文档***
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "range": {
>       "age": {
>         "gte": 5,
>         "lte": 8
>       }
>     }
>   }
> }
> ```



 

##### prefix 关键字

> ***用来检索含有指定前缀的关键词的相关文档***
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "prefix": {
>       "name": {
>         "value": "无"
>       }
>     }
>   }
> }
> ```
>
> **注意： 指定的前缀并不是说元数据文档中name属性以"张"为前缀的，而是匹配的经过分词器分词后索引区的数据，这里"张无忌"经过分词后为：“张”,“无”,“忌”，无论匹配到哪个都会指向那份文档**


##### 通配符 查询

> > **?, which matches any single character**
> >
> > ***, which can match zero or more characters, including an empty one**
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "wildcard": {
>       "name": {
>         "value": "?小*"
>       }
>     }
>   }
> }
> ```



##### 多id查找

```http
GET /ems/emp/_search
{
  "query": {
    "ids": {
      "values": ["2zN4Y3gBsVEMiPvdQCym","3TN4Y3gBsVEMiPvdQCyn"]
    }
  }
}
```



##### fuzzy模糊查询

>```http
>GET /ems/emp/_search
>{
>  "query": {
>    "fuzzy": {
>      "content": {
>        "value": "clou"
>      }
>    }
>  }
>}
>```
>
>> ***模糊查询的规则：*** 
>>
>> **最大模糊错误 必须在0-2之间**
>>
>> **搜索关键词长度为 2 不允许存在模糊 0**
>>
>> **搜索关键词长度为3-5 允许一次模糊 0 1**
>>
>> **搜索关键词长度大于5 允许最大2模糊**



##### bool 查询

>>**must: 相当于&& 同时成立**
>>
>>**should: 相当于|| 成立一个就行**
>>
>>**must_not: 相当于! 不能满足任何一个**
>
>```http
>GET /ems/emp/_search
>{
>  "query": {
>    "bool": {
>      "must": [
>        {
>          "range": {
>            "age": {
>              "gte":8,
>              "lte": 9
>            }
>          }
>        }
>      ],
>      "must_not": [
>        {
>          "fuzzy": {
>            "address": {
>              "value": "*南*"
>            }
>          }
>        }
>      ]
>    }
>  }
>}
>```



##### highlight高亮查询

> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "term": {
>       "name": {
>         "value": "五"
>       }
>     }
>   },
>   "highlight": {
>     "fields": {
>       "name": {}
>     }
>   }
> }
> ```
>
> >***highlight 是对查询后的结果进行高亮，所以要放在"query"之后进行，同时，并不是在原数据上进行操作，而是新增了，并增加的为自定义高亮 html标签: 可以在highlight中使用pre_tags和post_tags***
>
> ```http
> GET /ems/emp/_search
> {
>   "query": {
>     "term": {
>       "name": {
>         "value": "五"
>       }
>     }
>   },
>   "highlight": {
>     "pre_tags": ["<span style='color:red'>"], 
>     "post_tags": ["</span>"], 
>     "fields": {
>       "name":{}
>     }
>   }
> }
> ```



##### 多字段查询

>```http
>GET /ems/emp/_search
>{
>  "query": {
>    "multi_match": {
>      "query": "中国", 
>      "fields": ["name","content"]
>    }
>  }
>}
>```
>
>> **注意： 检索的关键词是否需要拆开来检索还需要看指定的字段是否有分词**
>
>> ***多字段增加分词***
>
>```http
>GET /ems/emp/_search
>{
>  "query": {
>    "multi_match": {
>      "query": "水平决策",
>      "analyzer": "standard", 
>      "fields": ["content","name"]
>    }
>  }
>}
>```



GET _cat/plugins

GET /_analyze
{
  "text": ["中华人民共和国国歌"],
  "analyzer": "ik_smart"
}

## Ik分词器

> 准备
>
> 删除原有的ems索引数据
>
> `DELETE /ems`
>
> 新建索引/类型/约束
>
> **同时针对属性增加了使用IK分词器**
>
> ```http
> PUT /ems
> {
>   "mappings":{
>     "emp":{
>       "properties":{
>         "name":{
>           "type":"text",
>            "analyzer": "ik_max_word",
>            "search_analyzer": "ik_max_word"
>         },
>         "age":{
>           "type":"integer"
>         },
>         "bir":{
>           "type":"date"
>         },
>         "content":{
>           "type":"text",
>           "analyzer": "ik_max_word",
>           "search_analyzer": "ik_max_word"
>         },
>         "address":{
>           "type":"keyword"
>         }
>       }
>     }
>   }
> }
> ```
>
> 添加数据
>
> ```http
> PUT /ems/emp/_bulk
>   {"index":{}}
>   {"name":"小黑","age":23,"bir":"2012-12-12","content":"为开发团队选择一款优秀的MVC框架是件难事儿，在众多可行的方案中决择需要很高的经验和水平","address":"北京"}
>   {"index":{}}
>   {"name":"王小黑","age":24,"bir":"2012-12-12","content":"Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式","address":"上海"}
>   {"index":{}}
>   {"name":"张小五","age":8,"bir":"2012-12-12","content":"Spring Cloud 作为Java 语言的微服务框架，它依赖于Spring Boot，有快速开发、持续交付和容易部署等特点。Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区Spring 和Netflix 、Pivotal 两大公司的推动下越来越完善","address":"无锡"}
>   {"index":{}}
>   {"name":"win7","age":9,"bir":"2012-12-12","content":"Spring的目标是致力于全方位的简化Java开发。 这势必引出更多的解释， Spring是如何简化Java开发的？","address":"南京"}
>   {"index":{}}
>   {"name":"梅超风","age":43,"bir":"2012-12-12","content":"Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API","address":"杭州"}
>   {"index":{}}
>   {"name":"张无忌","age":59,"bir":"2012-12-12","content":"ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口","address":"北京"}
> ```
>
> 测试
>
> ```http
> GET /ems/emp/_search
> {
>   "query":{
>     "term":{
>       "content":"个"
>     }
>   },
>   "highlight": {
>     "pre_tags": ["<span style='color:red'>"],
>     "post_tags": ["</span>"],
>     "fields": {
>       "*":{}
>     }
>   }
> }
> ```

### 类型

IK分词器提供了两种mapping类型用来做文档的分词分别是**ik_max_word**  和 **ik_smart**

**ik_max_word 和 ik_smart 什么区别?**

>***ik_max_word:*** 
>
>会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合；
>
>***ik_smart:*** 
>
>会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”



