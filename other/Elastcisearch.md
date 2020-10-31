<center> ElasticSearch搜索引擎入门
  
</center>

### 安装

1. ES的运行需要安装JAVA8以及以上的环境，设置正确的JAVA环境配置，然后你就可以开始下载和安装ES了。

2. ```bash
	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.zip
	unzip elasticsearch-5.5.1.zip
	cd elasticsearch-5.5.1/
	```

3. 下载完成之后，运行下面的命令，启动ES

	```bash'
	./bin/elasticsearch
	```

4. 或者使用官方文档中推荐的其他安装方式进行安装

### 基本使用

1. 当你安装完成ES的时候，就可以进行访问了，ES默认的运行端口是9200，当我们在浏览器地址栏中输入127.0.0.1:9200的时候，我们将会看到ES返回的数据。

	```json
	{
	  "name": "node-1",
	  "cluster_name": "elasticsearch",
	  "cluster_uuid": "9rCi6A6RQcSruGGsIWqPTg",
	  "version": {
	    "number": "7.7.0",
	    "build_flavor": "default",
	    "build_type": "rpm",
	    "build_hash": "81a1e9eda8e6183f5237786246f6dced26a10eaf",
	    "build_date": "2020-05-12T02:01:37.602180Z",
	    "build_snapshot": false,
	    "lucene_version": "8.5.1",
	    "minimum_wire_compatibility_version": "6.8.0",
	    "minimum_index_compatibility_version": "6.0.0-beta1"
	  },
	  "tagline": "You Know, for Search"
	}
	```

2. ES中几个重要的概念区分

	+ 索引(database)
		+ 索引是一个存储文档的地方，类似于关系型数据库中的数据库的概念
	+ 文档(row)
		+ 一条记录
	+ 类型(table)
		+ 类型设计索引的逻辑分区、分片
	+ 字段(field)
		+ 属性，相当于数据库的字段

3. python操作ES

	```python
	python能够操作es的第三方库有很多，比较常用的就是pyelasticsearch和elasticsearch
	
	from elasticsearch import Elasticsearch
	
	# 指定需要连接的主机信息
	es = Elasticsearch(hosts=[{'host': '123.56.102.64', "port": 9200}])
	# 需要向es服务器传递的参数
	body = {'name': 'wangdong2', 'age': 12}
	
	# 创建一个索索引库并且写入数据
	# index: 索引库的名称，相当于数据库名
	# doc_type: 索引的类型，相当于数据库表名
	# id: 文档的id值，可以不指定
	# body: 传递的参数
	es.index(index='test', doc_type='person', body=body, id=1)
	```

	