# 三、Elasticsearch入门

## 1 基本概念
### 1.1 索引


在Elasticsearch中，存储数据的行为就叫做**索引(indexing)**，**文档**归属于一种**类型(type)**，而这些类型存在于**索引(index)**中



类比传统关系型数据库：



```plain
Relational DB -> Databases -> Tables -> Rows -> Columns

Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```



Elasticsearch集群可以包含多个**索引(indices)**（数据库）



每一个索引可以包含多个**类型(types)**（表）



每一个类型包含多个**文档(documents)**（行）



然后每个文档包含多个**字段(Fields)**（列）。



![1558701774209.png](./img/Ok5d7GbrRKj8HM_N/1639404196394-c508c030-485e-4ffe-859c-051e12669e65-924942.png)



Elasticsearch 是面向文档型数据库，一条数据在这里就是一个文档。



ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。



![image-20211208001159600.png](./img/Ok5d7GbrRKj8HM_N/1639404196563-ca5762be-6276-4556-bfc4-7441140ff4d0-990066.png)



<font style="color:#E8323C;">Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个type，Elasticsearch 7.X 中, Type 的概念已经被删除了。</font>



## 2 索引操作


### 2.1、创建索引


对比关系型数据库，创建索引就等同于创建数据库



![image-20211208002321611.png](./img/Ok5d7GbrRKj8HM_N/1639404196528-89e10e7a-ebcd-483e-a74a-49a26e39b71a-227441.png)



```json
{
    "acknowledged": true, # 【响应结果】 true 操作成功
    "shards_acknowledged": true, # 【分片结果】 分片操作成功
    "index": "shopping" #【索引名称】
}

# 注意：创建索引库的分片数默认 1 片，在 7.0.0 之前的 Elasticsearch 版本中，默认 5 片
```



如果重复添加索引，会返回错误信息



### 2.2、 查看所有索引


这里请求路径中的_cat 表示查看的意思，indices 表示索引，所以整体含义就是查看当前 ES服务器中的所有索引，就好像 MySQL 中的 show tables 的感觉



![image-20211208072843696.png](./img/Ok5d7GbrRKj8HM_N/1639438476041-8abbe1fc-5b0d-43ea-9a75-e7b177131bf5-043044.png)

| 表头 | 含义 |
| --- | --- |
| health | 当前服务器健康状态:   green(集群完整) yellow(单点正常、集群不完整) red(单点不正常) |
| status | 索引打开、关闭状态 |
| index | 索引名 |
| uuid | 索引统一编号 |
| pri | 主分片数量 |
| rep | 副本数量 |
| docs.count | 可用文档数量 |
| docs.deleted | 文档删除状态（逻辑删除） |
| store.size | 主分片和副分片整体占空间大小 |
| pri.store.size | 主分片占空间大小 |




### 2.3、查看单个索引


![image-20211208073805665.png](./img/Ok5d7GbrRKj8HM_N/1639438476053-c9eb93e9-bf9a-4337-9be3-c514e3307315-845519.png)



```json
{
    "shopping"【索引名】: {
        "aliases"【别名】: {},
        "mappings"【映射】: {},
        "settings"【设置】: {
            "index"【设置 - 索引】: {
                "creation_date"【设置 - 索引 - 创建时间】: "1614265373911",
                "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
                "number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
                "uuid"【设置 - 索引 - 唯一标识】: "eI5wemRERTumxGCc1bAk2A",
                "version"【设置 - 索引 - 版本】: {
                    "created": "7080099"
                },
                "provided_name"【设置 - 索引 - 名称】: "shopping"
			}
		}
	}
}
```



### 2.4、  删除索引


![image-20211208074248646.png](./img/Ok5d7GbrRKj8HM_N/1639438476018-dbc6f405-97dd-4b2a-b6e6-16f8d7f06dc1-610350.png)



## 3  文档操作


### 3.1、创建文档


索引已经创建好了，接下来我们来创建文档，并添加数据。



这里的文档可以类比为关系型数据库中的表数据，添加的数据格式为 JSON 格式



请求体内容为：



```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```



此处发送请求的方式必须为 POST，不能是 PUT，否则会发生错误



![image-20211208080520138.png](./img/Ok5d7GbrRKj8HM_N/1639438476003-bcad0f4a-4a39-441a-8cb4-add5549b2ae2-309928.png)



服务器响应结果如下：



```json
{
    "_index"【索引】: "shopping",
    "_type"【 类型-文档 】: "phone",
    "_id"【唯一标识】: "Xhsa2ncBlvF_7lxyCE9G", #可以类比为 MySQL 中的主键，随机生成
    "_version"【版本】: 1,
    "result"【结果】: "created", #这里的 create 表示创建成功
    "_shards"【分片】: {
        "total"【分片 - 总数】: 2,
        "successful"【分片 - 成功】: 1,
        "failed"【分片 - 失败】: 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



### 3.2、自定义唯一性标识


上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下，ES 服务器会随机生成一个。



如果想要自定义唯一性标识，需要在创建时指定：



![image-20211208080720498.png](./img/Ok5d7GbrRKj8HM_N/1639438476042-7a0ded4d-2a59-4836-9702-df75cb14da50-716279.png)



**此处需要注意：如果增加数据时明确数据主键，那么请求方式也可以为 PUT**



```json
{
    "_index": "shopping",
    "_type": "phone",
    "_id": "1", # 此处已经变为自定义ID
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```



### 3.3、主键查询


查看文档时，需要指明文档的唯一性标识，类似于 MySQL 中数据的主键查询



![image-20211208081105614.png](./img/Ok5d7GbrRKj8HM_N/1639710872945-e1bb5fe3-9a74-4673-a9fb-d0dd2f0907eb-272527.png)



```json
{
    "_index"【索引】: "shopping",
    "_type"【文档类型】: "phone",
    "_id": "1",
    "_version": 1,
    "_seq_no": 1,
    "_primary_term": 1,
    "found"【查询结果】: true, # true 表示查找到，false 表示未查找到
    "_source"【文档源信息】: {
        "title": "小米手机",
        "category": "小米",
        "images": "http://www.gulixueyuan.com/xm.jpg",
        "price": 4999.00
    }
}
```



### 3.4、查看所有文档


![image-20211208084045304.png](./img/Ok5d7GbrRKj8HM_N/1639710872944-6b3d7b81-1aa7-4b4c-89c1-82f18cb03056-805439.png)



```json
{
    "took": 45,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "shopping",
                "_type": "phone",
                "_id": "WYlal30BHTTsEIEiNYMf",
                "_score": 1.0,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                }
            },
            {
                "_index": "shopping",
                "_type": "phone",
                "_id": "1_update",
                "_score": 1.0,
                "_source": {
                    "doc": {
                        "price": 3000.00
                    }
                }
            }
        ]
    }
}
```



### 3.5、修改文档


和新增文档一样，输入相同的 URL 地址请求，如果请求体变化，会将原有的数据内容覆盖



请求体内容为:



```json
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4999.00
}
```



![image-20211208081454704.png](./img/Ok5d7GbrRKj8HM_N/1639710872965-4eb54b57-813d-48fc-bfb0-6907eef3aee0-523972.png)



修改成功后，服务器响应结果：



```json
{
    "_index": "shopping",
    "_type": "phone",
    "_id": "1",
    "_version"【版本】: 2,
    "result"【结果】: "updated", # updated 表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```



### 3.6、修改字段


修改数据时，也可以只修改某一条数据的局部信息



请求体内容为：



```json
{
	"doc": {
    	"price":3000.00
    }
}
```



![image-20211208082101854.png](./img/Ok5d7GbrRKj8HM_N/1639710872958-6da8088d-ced0-4cb8-bedf-ad57b830bbad-059047.png)



修改成功后，服务器响应结果：



![image-20211208082139284.png](./img/Ok5d7GbrRKj8HM_N/1639710873039-7288fe14-8dd5-468b-a40b-f8bff62d942b-710185.png)



根据唯一性标识，查询文档数据，文档数据已经更新



![image-20211208082314530.png](./img/Ok5d7GbrRKj8HM_N/1639710872969-ce11a830-e817-4f7a-a353-882aaf5886e6-458466.png)



### 3.7、删除文档


删除一个文档不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除）。



![image-20211208082442568.png](./img/Ok5d7GbrRKj8HM_N/1639710872963-5544fe43-f2ed-4614-b0bb-59649d57e3d0-402229.png)



删除后再查询当前文档信息



![image-20211208082625668.png](./img/Ok5d7GbrRKj8HM_N/1639710872969-fc765807-e246-4f45-9b00-f01a5fcf4172-691783.png)



如果删除一个并不存在的文档



![image-20211208082734548.png](./img/Ok5d7GbrRKj8HM_N/1639710872965-c91ede7e-e330-42ca-b6f1-9bc940ec2442-486189.png)



### 3.8、条件删除文档


一般删除数据都是根据文档的唯一性标识进行删除，实际操作时，也可以根据条件对多条数据进行删除



首先分别增加多条数据:



```json
{
	"title": "小米手机",
	"category": "小米",
	"images": "http://www.gulixueyuan.com/xm.jpg",
	"price": 4000.00
} 

{
	"title": "华为手机",
	"category": "华为",
	"images": "http://www.gulixueyuan.com/hw.jpg",
	"price": 4000.00
}
```



![image-20211208231115058.png](./img/Ok5d7GbrRKj8HM_N/1639710872956-eff2b171-a2d1-46c9-89e3-8d81bd27b256-458205.png)



![image-20211208231200228.png](./img/Ok5d7GbrRKj8HM_N/1639710873004-47613e2c-0b6d-474f-90dc-da04ffb123f9-388400.png)



向 ES 服务器发 **POST** 请求 ：[http://127.0.0.1:9200/shopping/_delete_by_query](http://127.0.0.1:9200/shopping/_delete_by_query)



请求体内容为：



```json
{
	"query": {
		"match": {
			"price": 4000.00
		}
	}
}
```



![image-20211209002713646.png](./img/Ok5d7GbrRKj8HM_N/1639710872952-9e827acc-0e4d-4157-bd67-5f24a79b1cbb-578774.png)



删除成功后，服务器响应结果：



```json
{
    "took": 568, 【耗时】
    "timed_out": false, 【是否超时】
    "total": 2, 【总数】
    "deleted": 2, 【删除数量】
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```



## 4、映射操作


有了索引库，等于有了数据库中的 database。



接下来就需要建索引库(index)中的映射了，类似于数据库(database)中的表结构(table)。



创建数据库表需要设置字段名称，类型，长度，约束等；



索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)。



### 4.1、创建映射


在 Postman 中，向 ES 服务器发 PUT 请求 ：[http://127.0.0.1:9200/student/_mapping](http://127.0.0.1:9200/student/_mapping)



请求体内容为：



```json
{
	"properties": {
		"name": {
			"type": "text",
			"index": true
		},
		"sex": {
			"type": "keyword",
			"index": false
		},
		"age": {
			"type": "long",
			"index": false
		}
	}
}
```



![image-20211212162357920.png](./img/Ok5d7GbrRKj8HM_N/1639710875360-a93314a2-b7f1-4049-8f6e-e87aed4e12b2-827104.png)



### 4.2、映射数据说明


+  字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price 



+  type：类型，Elasticsearch 中支持的数据类型非常丰富，说几个关键的： 
    - String 类型，又分两种： 
        * text：<font style="color:#E8323C;">可分词</font>
        * keyword：<font style="color:#E8323C;">不可分词</font>，数据会作为完整字段进行匹配
    - Numerical：数值类型，分两类 
        * 基本数据类型：long、integer、short、byte、double、float、half_float
        * 浮点数的高精度类型：scaled_float
    - Date：日期类型
    - Array：数组类型
    - Object：对象



+  index：是否索引，默认为 true，也就是说你不进行任何配置，所有字段都会被索引。 
    - true：字段会被索引，则可以用来进行搜索
    - false：字段不会被索引，不能用来搜索



+  store：是否将数据进行独立存储，默认为 false 
    - 原始的文本会存储在_source 里面，默认情况下其他提取出来的字段都不是独立存储的，是从_source 里面提取出来的。
    - 当然你也可以独立的存储某个字段，只要设置"store": true 即可，获取独立存储的字段要比从_source 中解析快得多，但是也会占用更多的空间，所以要根据实际业务需求来设置。



+  analyzer：分词器，这里的 ik_max_word 即使用 ik 分词器 



### 4.3、查看映射


![image-20211212163228324.png](./img/Ok5d7GbrRKj8HM_N/1639710875541-6a2eefc2-7983-42b0-a2c3-bc75ac192920-072127.png)



### 4.4、索引映射关联


```json
{
	"settings": {},
	"mappings": {
		"properties": {
			"name": {
				"type": "text",
				"index": true
			},
			"sex": {
				"type": "keyword",
				"index": false
			},
			"age": {
				"type": "long",
				"index": false
			}
		}
	}
}
```



![image-20211212164446998.png](./img/Ok5d7GbrRKj8HM_N/1639710875519-99aee0ce-ab97-40e3-aa1e-25b82b6f6433-645682.png)



服务器响应结果如下：



```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "user"
}
```



## 5、高级查询


Elasticsearch 提供了基于 JSON 提供完整的查询 DSL 来定义查询



### 5.1、查看所有文档 match_all


```json
{
	"query": {
		"match_all": {}
	}
}

# "query"：这里的 query 代表一个查询对象，里面可以有不同的查询属性
# "match_all"：查询类型，例如：match_all(代表查询所有)， match，term ， range 等等
# {查询条件}：查询条件会根据类型的不同，写法也有差异
```



![image-20211212215259517.png](./img/Ok5d7GbrRKj8HM_N/1639710875696-836cd9a2-38d9-481a-a5b5-9f19ee6d918d-806187.png)



```json
{
	"took": 1116, //【查询花费时间，单位毫秒】
	"timed_out": false, //【是否超时】
	"_shards": { //【分片信息】
		"total": 1, //【总数】
		"successful": 1, //【成功】
		"skipped": 0, //【忽略】
		"failed": 0 //【失败】
	},
	"hits": { //【搜索命中结果】
		"total": { //【搜索条件匹配的文档总数】
			"value": 3, //【总命中计数的值】
			"relation": "eq" //【计数规则】 eq 表示计数准确， gte 表示计数不准确
		},
		"max_score": 1.0,//【匹配度分值】
		"hits": [] //【命中结果集合】
	}
}
```



### 5.2、匹配查询 match


match 匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是 or 的关系



```json
{
	"query": {
		"match": {
			"name": "zhangsan"
		}
	}
}
```



![image-20211212231625874.png](./img/Ok5d7GbrRKj8HM_N/1639710875738-c4d50509-12ef-4a75-862d-6aaead9668cb-731764.png)



### 5.3、字段匹配查询 multi_match


multi_match 与 match 类似，不同的是它可以在多个字段中查询。



```json
{
	"query": {
		"multi_match": {
			"query": "zhangsan",
			"fields": ["name", "nickname"]
		}
	}
}
```



![image-20211212232006383.png](./img/Ok5d7GbrRKj8HM_N/1639710875751-9e93dd87-8d0e-4cea-b686-33885da5be2a-966004.png)



### 5.4、关键字精确查询 term


term 查询，精确的关键词匹配查询，不对查询条件进行分词。



```json
{
	"query": {
		"term": {
			"name": {
				"value": "zhangsan"
			}
		}
	}
}
```



![image-20211212232450084.png](./img/Ok5d7GbrRKj8HM_N/1639710875748-8316e7a8-43c5-482f-a0ed-c78029bef067-623114.png)



### 5.5、多关键字精确查询 terms


terms 查询和 term 查询一样，但它允许你指定多值进行匹配。



如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件，类似于 mysql 的 in



```json
{
	"query": {
		"terms": {
			"name": ["zhangsan", "lisi"]
		}
	}
}
```



![image-20211212232643074.png](./img/Ok5d7GbrRKj8HM_N/1639710880833-87a06eff-a1d5-415b-bbef-4723f2398de4-412745.png)



### 5.6、指定查询字段 _source


默认情况下，Elasticsearch 在搜索的结果中，会把文档中保存在 _source 的所有字段都返回。



如果我们只想获取其中的部分字段，我们可以添加_source 的过滤



```json
{
	"_source": ["name", "nickname"],
	"query": {
		"terms": {
			"nickname": ["zhangsan"]
		}
	}
}
```



![image-20211212232856702.png](./img/Ok5d7GbrRKj8HM_N/1639710881001-15b46e64-65bd-4f8e-87a9-d4c065fc5763-076377.png)



### 5.7、过滤字段 includes


我们也可以通过：



+  includes：来指定想要显示的字段  

```json
{
	"_source": {
		"includes": ["name", "nickname"]
	},
	"query": {
		"terms": {
			"nickname": ["zhangsan"]
		}
	}
}
```



![image-20211212233126895.png](./img/Ok5d7GbrRKj8HM_N/1639710881043-dc4cd061-d5cf-43e4-b76d-c175402551e1-917813.png)



+  excludes：来指定不想要显示的字段   
![image-20211212233302564.png](./img/Ok5d7GbrRKj8HM_N/1639710881088-faf498d7-1787-45aa-a26c-8387dcf74aae-065646.png) 

```json
{
	"_source": {
		"excludes": ["name", "nickname"]
	},
	"query": {
		"terms": {
			"nickname": ["zhangsan"]
		}
	}
}
```



### 5.8、组合查询 bool


`bool`把各种其它查询通过`must`（必须 ）、`must_not`（必须不）、`should`（应该）的方式进行组合



```json
{
	"query": {
		"bool": {
			"must": [{
				"match": {
					"name": "zhangsan"
				}
			}],
			"must_not": [{
				"match": {
					"age": "40"
				}
			}],
			"should": [{
				"match": {
					"sex": "男"
				}
			}]
		}
	}
}
```



![image-20211212233536310.png](./img/Ok5d7GbrRKj8HM_N/1639710881131-a77976c9-32a7-4164-ae43-68253130be16-002350.png)



### 5.9、范围查询 range


range 查询找出那些落在指定区间内的数字或者时间。range 查询允许以下字符

| 操作符 | 说明 |
| --- | --- |
| gt | 大于> |
| gte | 大于等于>= |
| lt | 小于< |
| lte | 小于等于<= |




```json
{
	"query": {
		"range": {
			"age": {
				"gte": 30,
				"lte": 35
			}
		}
	}
}
```



![image-20211212233927010.png](./img/Ok5d7GbrRKj8HM_N/1639710881118-faa28c17-bfde-4570-b7f2-c0001b0803a4-089630.png)



### 5.10、模糊查询


返回包含与搜索字词相似的字词的文档。



编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括：



+ 更改字符（box → fox）
+ 删除字符（black → lack）
+ 插入字符（sic → sick）
+ 转置两个相邻字符（act → cat）



为了找到相似的术语，fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。



```json
{
	"query": {
		"fuzzy": {
			"title": {
				"value": "zhangsan"
			}
		}
	}
}
```



![image-20211212234415188.png](./img/Ok5d7GbrRKj8HM_N/1639710881175-69e5041c-ddd1-454a-9584-2a1827682c0c-244420.png)



通过 fuzziness 修改编辑距离。一般使用默认值 AUTO，根据术语的长度生成编辑距离。



```json
{
	"query": {
		"fuzzy": {
			"title": {
				"value": "zhangsan",
				"fuzziness": 2
			}
		}
	}
}
```



![image-20211212234609067.png](./img/Ok5d7GbrRKj8HM_N/1639710881146-c06c3d76-8c34-4c06-b8d1-06df373f4206-052082.png)



### 5.11、单字段排序 sort 


sort 可以让我们按照不同的字段进行排序，并且通过 order 指定排序的方式。desc 降序，asc升序。



```json
{
	"query": {
		"match": {
			"name": "zhangsan"
		}
	},
	"sort": [{
		"age": {
			"order": "desc"
		}
	}]
}
```



![image-20211212234745866.png](./img/Ok5d7GbrRKj8HM_N/1639710881168-abdac54f-7033-41c9-a20d-10efc3448517-205284.png)



### 5.12、多字段排序


假定我们想要结合使用 age 和 _score 进行查询，并且匹配的结果首先按照年龄排序，然后按照相关性得分排序



```json
{
	"query": {
		"match_all": {}
	},
	"sort": [{
			"age": {
				"order": "desc"
			}
		},
		{
			"_score": {
				"order": "desc"
			}
		}
	]
}
```



![image-20211212234956615.png](./img/Ok5d7GbrRKj8HM_N/1639710881218-cf1609d2-8e54-401b-8c65-f757701adeed-383469.png)



### 5.13、高亮查询


在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮。



Elasticsearch 可以对查询内容中的关键字部分，进行标签和样式(高亮)的设置。



在使用 match 查询的同时，加上一个 highlight 属性：



+ pre_tags：前置标签
+ post_tags：后置标签
+ fields：需要高亮的字段
+ title：这里声明 title 字段需要高亮，后面可以为这个字段设置特有配置，也可以空



```json
{
	"query": {
		"match": {
			"name": "zhangsan"
		}
	},
	"highlight": {
		"pre_tags": "<font color='red'>",
		"post_tags": "</font>",
		"fields": {
			"name": {}
		}
	}
}
```



![image-20211212235244974.png](./img/Ok5d7GbrRKj8HM_N/1639710881320-dcc14287-2bd2-474c-8c3d-7f53b3cd43bc-539864.png)



### 5.14、分页查询


from：当前页的起始索引，默认从 0 开始。 from = (pageNum - 1) * size



size：每页显示多少条



```json
{
	"query": {
		"match_all": {}
	},
	"sort": [{
		"age": {
			"order": "desc"
		}
	}],
	"from": 0,
	"size": 2
}
```



![image-20211212235408366.png](./img/Ok5d7GbrRKj8HM_N/1639710881636-5a77e755-f43a-40fb-a8f9-3388954e2a1c-458274.png)



### 5.15、聚合查询


聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值、平均值等等。



+  对某个字段取最大值 max  

```json
{
	"aggs": {
		"max_age": {
			"max": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211212235550738.png](./img/Ok5d7GbrRKj8HM_N/1639710881483-333b5e64-d043-4414-aaa5-fff09413e29a-801938.png)



+ 对某个字段取最小值 min



```json
{
	"aggs": {
		"min_age": {
			"min": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211212235712347.png](./img/Ok5d7GbrRKj8HM_N/1639710881441-307f0604-bbe1-46dc-964a-0fcf013e16ea-666862.png)



+  对某个字段求和 sum  

```json
{
	"aggs": {
		"sum_age": {
			"sum": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211212235811413.png](./img/Ok5d7GbrRKj8HM_N/1639710881612-082ebee0-06cb-49eb-89f3-2fa509050d5b-429546.png)



+  对某个字段取平均值 avg  

```json
{
	"aggs": {
		"avg_age": {
			"avg": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211212235910696.png](./img/Ok5d7GbrRKj8HM_N/1639710881510-9b7de67b-5b36-4a1c-9205-48516bd7c967-826102.png)



+ 对某个字段的值进行去重之后再取总数



```json
{
	"aggs": {
		"distinct_age": {
			"cardinality": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211213000027368.png](./img/Ok5d7GbrRKj8HM_N/1639710881792-b352ffe6-bd8e-450a-8e83-63d3a1f3a302-190068.png)



+  State 聚合  
stats 聚合，对某个字段一次性返回 count，max，min，avg 和 sum 五个指标 



```json
{
	"aggs": {
		"stats_age": {
			"stats": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211213000135815.png](./img/Ok5d7GbrRKj8HM_N/1639710881827-1d46cd60-6979-4ade-a2f9-b0ed7d2f4443-882769.png)



## 6、桶聚合查询


桶聚和相当于 sql 中的 group by 语句



### 6.1、terms 聚合，分组统计


```json
{
	"aggs": {
		"age_groupby": {
			"terms": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211213000330174.png](./img/Ok5d7GbrRKj8HM_N/1639710881807-17ada810-b35d-4820-b8bb-a82b6db549b2-891671.png)



### 6.2、在 terms 分组下再进行聚合


```json
{
	"aggs": {
		"age_groupby": {
			"terms": {
				"field": "age"
			}
		}
	},
	"size": 0
}
```



![image-20211213000510650.png](./img/Ok5d7GbrRKj8HM_N/1639710881896-ec378ee2-a9b5-48f4-9d1d-d2c483e2d2cc-217293.png)



> 更新: 2023-10-31 14:33:35  
> 原文: <https://www.yuque.com/like321/fk7s34/slgpiv>