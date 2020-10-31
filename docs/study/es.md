<h2><center>ES相关操作</center></h2>

* ES创建索引

~~~json
put http://localhost:9200/commodity   commodity为索引名称
~~~
可以通过如下参数来指定分片数，副本数量

~~~json
{
	"settings":{
		"number_of_shards":3,
		"number_of_replicas":2
	}
}
~~~
通过curl命令创建一个商品的索引
curl -X put "localhost:9200/commodity?pretty"
索引创建成功会返回一下参数

~~~json
{
	"acknowledged": true,
	"shards_acknowledged": true,
	"index": "commodity"
}
~~~
创建带有索引类型以及映射的索引

~~~json
curl -X put "localhost:9200/commodity?pretty"
{
	"settings":{
		"number_of_shards": 3,
		"number_of_replicas": 2
	},
	"mapping":{
		"_doc":{
			"properties":{
				"commodity_id":{
					"type":"long"
				},
				"commodity_name":{
					"type":"text"
				},
				"picture_url":{
					"type":"keyword"
				},
				"price":{
					"type":"double"
				}
			}
		}
	}
}
~~~
修改索引副本数

~~~json
PUT http://localhost:9200/commodity/_settings
{
	"number_of_replicas": 3
}
~~~

列出索引索引状态
GET /_cat/indices?v
可以通过kibana的dev tools

查询所有详细信息
GET /index1,index2
GET /_all
GET /s*

删除索引
DELETE /index1
DELETE /_all
DELETE /*

在索引的映射中增加一个字段

~~~json
PUT /commodity/_mapping/tweet
{
	"properties":{
		"tag":{
			"type":"text",
			"index": false
		}
	}
}
~~~

查看某个type的映射关系

~~~json
GET /{index}/_mapping/{type}
~~~
往索引commodity里面添加一条数据指定id为1

~~~json
PUT /commodity/tweet/1
{
	"username": "liuym",
	"user_id": 16
}
~~~
更新索引

~~~json
PUT /{index}/{type}/{id}
POST /{index}/{type}/{id}/_update
~~~
