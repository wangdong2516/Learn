<center> Haystack使用

</center>

### 基本介绍

1. Haystack为Django提供模块化搜索。它具有统一的，熟悉的API，可让您插入不同的搜索后端（例如[Solr](http://lucene.apache.org/solr/)， [Elasticsearch](http://elasticsearch.org/)，[Whoosh](https://bitbucket.org/mchaput/whoosh/)，[Xapian](http://xapian.org/)等），而无需修改代码。

# 使用

> 安装

```bash
pip install django-haystack
```

> 注册

```python
将Haystack添加到INSTALLED_APPS中
INSTALLED_APPS= [
  "haystack"
]
```

> 配置

```python
# 配置hsystack使用的缓存后端，以Elasticsearch为例
HAYSTACK_CONNECTIONS = {
  'default': {
    'ENGINE' : 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
    'URL': 'http://127.0.0.1:9200/',
    'INDEX_NAME': 'haystack'
  }
}

# 其中URL和INDEX_NAME替换为你自己Elasticserch服务器的地址和你希望的索引名称即可。
```

> 处理数据，创建索引类

SearchIndex是决定需要在索引中存放那些数据并且如何处理数据的方式，它和Django的Models或者是Forms类似。

1. 创建索引类

	+ 定义一个类，继承indexed.SearchIndex和indexes.Indexable
	+ 指定要存储的字段
	+ 定义get_model方法

	```python
	from haystack import indexes
	from blog.models import Article
	
	
	class BlogIndex(indexes.SearchIndex, indexes.Indexable):
	  
	  text = indexes.CharField(document=True, use_template=True)
	  author = indexes.CharField()
	  pub_date = indexes.DateTimeField()
	  
	  def get_model(self):
	    return Article
	  
	  def index_queryset(self, using=None):
	    # 在更新模型的索引的时候使用
	    return self.get_model().objects.all()
	```

> 注意点

1. 每一个索引类有且只有一个document=True的字段，document=True表明搜索的时候使用的字段

2. use_template参数指明使用模版来构建搜索引擎将要建立的文档。

3. 在Django3.0版本中，删除了utils中的six模块，但是haysatck在导入的时候，需要从django.utils中导入six模块，从而引发错误，解决方式是首先安装six，然后将six复制到django.utils目录下，并且将haystack中的inputs文件中的8、9行代码修改为

	```python
	from django.utils.encoding import force_text
	from django.utils.six import python_2_unicode_compatible
	```

> 配置路由

1. 建议在总路由中配置

	```python
	path('search/', include('haystack.urls'))
	```

> 建立索引

1. haystack提供了一个命令来建立索引

	```bash
	python manage.py rebulid_index
	```

	