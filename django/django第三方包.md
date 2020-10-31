<center> Django项目开发使用的第三方包

>实现根据前端传递的数据进行筛选的功能

`django-filter`：自定义过滤器类，添加到django或者restframework中去，可以实现根据条件进行过滤

使用方式:

1. 安装：`pip install django-filter`
2. 注册: `django_filters`添加到INSTALLED_APPS中
3. 使用：

filters.py

```python
自定义过滤器类(以和restframework集成为例)

from django_filters import rest_framework as filters


class ProductFilter(filters.FilterSet):
    """自定义过滤器，实现筛选功能"""
    # 参数介绍:field_name：表示需要进行过滤的数据库字段
    # lookup_expr表示进行过滤的条件
    # 这里的name，age，age_gt表示需要从前端接受的参数名称
    name = filters.CharFilter(field_name='name', lookup_expr='icontains')
    age = filters.NumberFilter(field_name='age', lookup_expr='lte')
    age_gt = filters.NumberFilter(field_name='age', lookup_expr='gt')

    class Meta:
        # model表示需要参照的模型类
        model = Author
        # fields属性可以为空，也可以设置指定的值，但是必须要设置
        fields = []
```

Views.py

```python
class AuthorView(ListAPIView):

    filter_backends = (filters.DjangoFilterBackend, )
    filter_class = ProductFilter
    queryset = Author.objects.all()
    serializer_class = ProductionSerializer
```

也可以配置默认的过滤器后端

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
}
```

便捷方式(等效方式)

```python
class AuthorView(ListAPIView):

    filter_backends = (filters.DjangoFilterBackend, )
    filterset_fields = ('name', 'age')
    queryset = Author.objects.all()
    serializer_class = ProductionSerializer
    
等效于
class ProductFilter(filters.FilterSet):
    class Meta:
        model = Author
        fields = ('name', 'age')
```

`注意事项`: 请注意，不支持将`filterset_fields`and `filterset_class`一起使用。

`默认值说明`:如果在声明过滤器的时候没有指定`field_name`参数，将会使用过滤器的字段名作为field_name

如果没有指定`lookup_expr`则默认使用`exact`精确等于的方式。

如果期望进行范围过滤，可以指定`lookup_expr='range'`或者是`lookup_expr='in'`，前端进行筛选的时候，需要传递/?age=12,33这样形式的数据，就可以实现对应的功能了。

>集成登陆

`django-allauth`可以实现第三方登录，包括github，weibo，qq，weixin等登录方式。

>跨域解决方案

`django-cors-headers`可以实现跨域请求。