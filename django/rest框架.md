<center> RestFrameWork之序列化器

> 在序列化器中，如果需要给返回的字段，定义一个和数据库不相同的名称，可以使用source来指定数据的源是什么，默认是序列化器定义的字段名

```python
# 模型类的定义如下
class Article(models.Model):

    title = models.CharField(max_length=100)
    content = models.TextField()
    category = models.ForeignKey(
        'Category', on_delete=models.CASCADE,
    )

# 序列化器的定义如下，不指定source参数
class BlogSerializer(serializers.Serializer):

    name = serializers.CharField(max_length=100)
    content = serializers.CharField(max_length=200)
# 这个使用的时候会报错，因为数据库模型类Article中不存在名为name的属性
Got AttributeError when attempting to get a value for field `name` on serializer `BlogSerializer`.
The serializer field might be named incorrectly and not match any attribute or key on the `Article` instance.
Original exception text was: 'Article' object has no attribute 'name'.

# 我们可以使用source来指定字段对应的数据源，这样就可以实现更改字段的名称，但是又不会报错
class BlogSerializer(serializers.Serializer):
		name = serializers.CharField(max_length=100, source='title')
  	content = serializers.CharField(max_length=200)
```

	>并且source还可以使用.的形式继续获取关联对象的其他属性

```python
class Category(models.Model):
    name = models.CharField(max_length=100)

class BlogSerializer(serializers.Serializer):

    name = serializers.CharField(max_length=100, source='title')
    content = serializers.CharField(max_length=200)
    category = serializers.CharField(source='category.name')
```

获取到的数据如下

```json
"results": [
        {
            "name": "dd",
            "content": "111",
            "category": "最新文章"
        },
        {
            "name": "最新文章2",
            "content": "222",
            "category": "最新文章"
        },
        {
            "name": "今日头条",
            "content": "是啥",
            "category": "热门话题"
        }
]
```

>不仅可以获取属性，还可以执行方法

```python
# 这里假设gender是Blog模型类的一个字段，并且是一个ChoiceField
class BlogSerializer(serializers.Serializer):

    name = serializers.CharField(max_length=100, source='title')
    content = serializers.CharField(max_length=200)
    category = serializers.CharField(source='category.name')
    choices = serializer.CharField(source='get_gender_display')
# 注意: 我们在一般时候获取ChoiceField字段的人类可读名称的时候，是直接调用的get_gender_display()方法，这里不需要加括号的原因是，在序列化器内部会判断source指定的字段是否可调用，如果可调用的话，会直接调用该方法，不可调用的话，直接返回数据库中的值
```

> 还可以使用序列化器的serializers.SerializerMethodField()来指定提取字段数据的方法
>
> 注意方法名的定义必须是:get_{fieldname}形式的才可以被识别，否则会报错

```python
class BlogSerializer(serializers.Serializer):

    fixed = serializers.SerializerMethodField()

    def get_fixed(self, rows):
        return {'ok'}
```

```json
"results": [
        {
            "fixed": [
                "ok"
            ]
        },
        {
            "fixed": [
                "ok"
            ]
        }
]
```

> 序列化器关系

`关系字段用于定义序列化器，他们可以应用到ForeignKey、ManyToManyField和OneToOneField`关系以及反向关系，自定义关系等。

使用ModelSerializer类的时候，将自动生成序列化器字段和关系。

`StringRelatedField`关联对象将被序列化为其__str__表示形式。该字段是只读的。`如果序列化的对象有多个，需要指定many=True`

`PrimaryKeyRelatedField`关联对象将被序列化为主键的形式，默认情况下，该字段是只读写的，可以指定`read_only和write_only`来更改此关系。

`HyperlinkedRelatedField`关联对象将被序列化为超链接的形式，默认情况下，该字段是可读写的

`参数`：

	1. view_name:必须指定，表示应该用做关系的视图函数的名称。
 	2. query_set：必须显示指定或者指定read_only=True
 	3. many: 序列化的对象是多个的时候需要指定该参数。
 	4. allow_null：是否允许为空。

> 字段验证器

对于序列化器字段的验证，我们可以依赖于默认提供的验证方式(ModelSerializer)或者使用在序列化器中提供的验证方法(validate_{filename}(单一字段验证), validate(多字段验证))，但是这些方法不能够在多个序列化器中重复使用，要想定义能够重复使用的验证器，我们可以编写专门的验证方法或者是验证类。

`好处`： 可以实现验证逻辑的复用，代码结构好

1. 使用ModelSerializer将会生成默认的字段验证方法，比如唯一字段将使用UniqueValidator来验证唯一性。

`RestFrameWork框架默认提供的验证器`

1. UniqueValidator用于模型类unique=True字段的验证，需要传入queryset作为参数
2. UniqueTogetherValidator用于模型类unique_together字段的验证，需要传入queryset和fields作为参数，fields应为字段名称的列表或元组，应组成唯一的集合。
3. UniqueForDateValidator、UniqueForMonthValidator、UniqueForYearValidator分别用于unique_for_date`，`unique_for_month`和`unique_for_year模型字段的验证，需要传入queryset、fields、date_field参数。

`默认情况下，UniqueTogetherValidator将强制所有字段为required=True,如果需要禁止这种行为，需要在序列化器类中将验证器置空，然后显示的编写字段的验证方法。`

> 自定义验证器或者是验证器类

1. `自定义验证器函数需要我们定义一个方法，在验证失败的时候抛出异常`
2. `自定义验证器类，需要我们在__call__方法中实现我们的验证逻辑，并且在验证失败的时候抛出对应的异常`