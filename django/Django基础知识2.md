# Django基础知识2
## 类视图
	1. 类视图：以函数的方式定义的视图叫做函数视图，函数视图便于理解，但是当遇到一个视图中对应的路径中提供了多种不同的http请求方式的时候，需要在一个视图函数中编写不同的业务逻辑，代码的可读性和复用性不好
	2. Django中支持使用类来定义一个视图，称为类视图，使用类视图可以将视图对应的不同请求方式以类中不同的方法来区别定义
	使用：
		from django.views.generic import View
		
		class RegisterView(View):
		
			def get(self, request):
				"""处理get请求"""
				return HttpResponse('ok')
			
			def post(self, request):
				"""处理post请求""""
				return HttpResponse('post')
			
	3. 类视图的好处：
		1. 代码的可读性好
		2. 类视图相对于函数视图具有更高的复用性，需要使用的地方直接继承就好
	4. 类视图的使用：
		1. 定义类视图需要继承自Django提供的View，使用django.views.generic import View
		2. 配置路由的时候，使用类视图的as_view()方法来添加
		urlpatterns = [
			url(r'/index', views. RegisterView.as_view())
		] 
	5. 类视图的原理：
		1. 类视图是继承自Django中的View类，View类中有as_view()方法，as_view()中存在dispatch()方法，在dispatch()方法中有getattr()方法，判断类对象中是否有相应的属性或者是方法
		2. 伪代码：
			from django.view import View
			
			class View(object):
			
				def dispatch(self):
					func = getattr(self, 'get')
					return func()
					
				def as_view(self):
					return self.dispatch()
				
			class B(View):
				
				def get(self, request):
					
					return HttpResponse('get')
				
	6. 类视图使用装饰器
		在django中，装饰器是不能直接装饰类和方法的
		1. 为类视图添加装饰器，可以使用两种方式，一种是在url配置中装饰（不建议使用，不利于代码的完整性, 此种方式会为类视图中所有的请求方式都加上装饰器的行为），
		另一种是在类视图中装饰，不能直接添加装饰器，需要使用method_decorator将其转换为适用类视图方法的装饰器
		2. 可以使用name参数为指定的请求方式添加装饰器：method_decorator（自己定义的装饰器名， name='get'）
		3. 如果需要为类视图中的多个方法添加装饰器，但又不是全部的方法，可以直接在需要添加装饰器的方法上使用method_decorator。或者使用method_decorator(自己定义的装饰器, name='dispatch')
## 类视图Mixin扩展类
	1. 使用面向对象多继承的特性，可以通过定义父类，在父类中定义想要类视图补充的方法，类视图继承这些扩展父类，可以实现代码复用，定义的扩展类名称通常以Mixin结尾，便于区分。
## 中间件
	1. Django中的中间件是一个轻量级、底层的插件系统，可以介入Django的请求和响应处理过程，修改Django的输入和输出，中间件的设计
	为开发者提供了一种无侵入式的开发方式，增强了Django框架的健壮性
	2. 中间件的定义方式：
		1. 定义一个中间件工厂函数，然后返回一个可以被调用的中间件
		2. 中间件工厂函数需要接受一个可以调用的get_response对象
		3. 返回的中间件也是一个可以被调用的对象，并且像视图函数一样需要接受一个request请求参数，返回一个response对象。
		4. 伪代码：
			def simple_middleware(get_response):
				"""此处编写的代码仅在Django第一次配置和初始化的时候执行一次"""
				def middleware(request):
					"""此处编写的代码会在每个请求处理视图函数之前被调用"""
					response = get_response(request)
					
					"""此处编写的代码将会在每个请求处理完视图函数之后被调用"""
					return response
				return middleware
	3. 使用：
		1. 在定义好中间件之后，需要在配置文件settings.py文件中安装并且注册中间件
		2. 在MIDDLEWARE列表中，添加自己定义的中间件
	4. 中间件的执行流程：
		1. 在请求视图函数被处理之前，中间件的执行顺序是从上倒下执行的，这个和代码的加载顺序是有关系的
		2. 在请求视图函数被处理之后，中间件的执行顺序是从下到上执行的，这个和return有关系
## 模板
	1. 配置：在工程中创建模板目录templates，创建完成之后在配置文件settings.py文件中修改TEMPALTES配置项的DIRS值
	TEMPALTES = [
		'BACKEND': 'django,tempalte.backends.django.DjangoTempalte',
		'DIRS': [os.path.join(BASE_DIR, 'tempaltes')],
		'APP_DIRS': True,
		'OPTIONS': {
			'context_processors': [
				'django.template.context_processors.debug',
				'django.template.context_processors.request',
				'django.contrib.auth.context_processor.auth',
				'django.contrib.messages.context_processor.messages',
			]
		}
	] 
	2. 模板渲染：
		1. 调用模板分为两个步骤：
			1. 找到模板的相对路径 loader.get_tempalte('模板名')---> 返回模板对象
			2. 渲染模板： 模板对象.render(context=None, request=None) ---> 返回渲染之后的html页面，context为模板变量字典，默认是None，request为请求对象，默认是None  
		2. Django中提供了一个render()函数可以直接返回模板
			return render(request, '模板名', context)
	3. 模板语法：
		1. 获取变量的值: {{ 变量名 }}
		2. for循环： {% for item in 列表 %}. {% endfor %}
		3. if条件判断: {% if 条件1 %} {% elif 条件2 %} {% else %} {% endif %}
		4. 比较运算符： ==、 !=、 > 、<、 <=、 >=，在进行比较的时候，中间必须有空格
		5. 布尔运算符：and、 or、 not
		6. 过滤器： 变量 | 过滤器：参数
			使用：使用管道符号来应用过滤器，用于进行计算、转换操作，可以使用在变量和标签中
			如果过滤器需要参数，使用:传递参数
			列举几个模板过滤器：
				safe: 禁用转义，可以直接解释执行
				length：长度，返回字符串包含的字符的个数，或者是元祖、列表、字典中元素的个数、
				default：默认值，如果变量不存在的时候，返回的是默认值
				date：日期，用于对日期类型的值进行字符串格式化
		7. 注释：
			单行注释： {#   #}
			多行注释： {% comment %} {% endcomment %}
		8. 模板继承：
			1. 在父模板中定义block块 {% block 名称 %} {% endblock 名称 %}
			2. 在子模板中继承：{% extend '父模板的路径' %},继承要写在子模板中的第一行
			{% block 名称 %}
				子模板中的内容
			{{ block.super }}  # 继承父模板中的内容
				实际填充的内容
			{% endblock 名称 %}    	 
## 数据库
	1. ORM：对象关系映射，在ORM框架中，他帮助我们把类和数据表进行了一个映射，可以让我们通过类和类对象就能操作它对应的表中的数据，ORM框架还有一个功能，他可以根据我们设计的类自动帮助我们生成数据库中的表格，省去了我们自己建表的过程。同时也屏蔽了不同数据库之间的差异性。
	2. django中内嵌了ORM框架，不需要直接面向数据库编程，而是定义模型类，通过模型类和对象完成数据库表的增、删、改、查操作。
	3. 使用Django进行数据库开发的步骤：
		1. 配置数据库的连接信息
		2. 在models.py中定义模型类
		3. 迁移
		4. 通过类和对象完成数据的增、删、改、查操作
	4.  ORM的作用：
		1. orm可以将模型类的操作转换成sql语句并且执行
		2. 将执行后的结果转换为模型类对象
	5. 配置：
		1. 在settings.py中保存了数据库的连接信息，Django默认初始配置使用的是sqlite数据库
		DATABASES = {
			'default': {
				'ENGINE': 'django.db.backends.mysql',
				'NAME': '数据库名称'，
				'USER': '数据库连接用户名',
				'PASSWORD': '数据库连接密码',
				'HOST': '数据库服务IP',
				'PORT': '数据库连接端口'
			}
		}  	
		2. 使用：
			1. 使用mysql数据库之前需要首先安装mysql： pip install pymysql
			2. 在Django工程同名目录下的__init__包文件中添加：
				from pymysql import install_as_MySQLdb
				install_as_MySQLdb()
			3. 修改DATABASES的配置信息
			4. 在mysql中创建数据库
## 模型类的定义
	1. 模型类被定义在models.py文件中
	2. 模型类必须继承自Model类
	3. 使用：
		from django.db import models
		
		# 定义模型类
		class BookInfo(models.Model):
			btitile = model.CharField(max_length=20, verbose_name='名称')
			bpud_date = model.DateField(verbose_name='发布日期')
			bread = models.IntegerField(default=0, verbose_name='阅读量')
			bcomment = models.IntegerField(defalut=0, verbose_name='评论量')
			is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
		
		class Meta:
			db_table = 'tb_books'  # 指明数据库表名
			verbose_name = '图书'   # 在admin站点中显示的名称
			verbose_name_plural = verbose_name  # 在admin中显示的复数名称
		
		def __str__(self):
			"""定义每个数据对象的显示信息"""
				return self.btitle
	4. 说明：
		1. 模型类如果没有指定表名，Django默认以小写app应用名_小写模型类名的方式给数据库表命名
		2. 可以通过db_table指明数据库的表名
		3. 关于主键：
			1. django会为表创建自增主键，每个模型类只能有一个主键列，如果使用选项设置某属性为主键列之后，django不会再创建自增主键。
			2. 默认创建的主键列属性为id，可以使用pk替代
		4. 属性命名限制：
			1. 不能是python中保留的关键字
			2. 不允许使用连续的下划线，这是由django的查询方式决定的。
			3. 定义属性时，需要指明字段的类型，通过字段类型的参数指定选项，语法：属性=models.字段类型（选项）
		5. 字段类型：
			1. AutoField：自动增长，通常不需要指定
			2. BooleanField:布尔字段，值为True或者是False
			3. NullBooleanField: 支持Null，True，False三种值
			4. CharField：字符串，参数max_length表示最大字符个数
			5. TextField: 大文本字段，一般超过4000个字符时使用
			6. IntegerField: 整数
			7. DecimalField: 十进制浮点数，参数max_digits表示总位数，参数decimal_places表示小数位数
			8. FloatField: 浮点数
			9. DateFiled: 日期，参数auto_new表示每次保存对象的时候，自动设置该字段为当前时间
			10. TimeField: 时间
			11. DateTimeField: 日期时间
			12. FileField: 上传文本字段
			13. ImageField: 对上传的内容进行校验，确保是有效的图片
		6. 选项：
			1. null: 如果为True，表示允许为空，默认值是Flase
			2. blank：如果为True，表示该字段允许为空白，默认值是False
			3. db_column: 字段的名称，如果没有指定，则使用该属性的名称
			4. db_index: 若值为True，则在表中会为该字段创建索引，默认值是False
			5. default: 默认值
			6. primary_key: 若为True，该字段成为模型类的主键，默认值是False
			7. unique: 如果为True，这个字段在表中必须有唯一值，默认值是False
		7. 外键：
			1. 在设置外键的时候，需要通过on_delete选项指明删除主表数据的时候，外语外键引用表的数据该如何处理
			2. 选项：
				1. CASCADE:级联，删除主表的数据时连同删除外键表中的数据
				2. PROTECT:保护，通过抛出异常，来阻止删除主表中被外键引用的数据
				3. SET_NULL:设置为NULL，仅在该字段NULL=True的时候可用
				4. SET_DEFAULT:设置为默认值，仅在该字段设置了默认值的时候可用
				5. SET(): 设置为特定值或者调用特定的方法
				6. DO_NOTHING: 不做任何操作
		8. 迁移：
			1. 将模型类同步到数据库中
				1. 生成迁移文件： python manage.py makemigrations
				2. 同步到数据库中：python manage.py migrate
		9. shell工具的使用：
			python manage.py shell
## 数据库操作-- 增、删、改、查
	1. 增加：
		1. 增加数据有两种方式
			1. save()方法： 创建模型类对象，执行对象的save()方法保存数据到数据库
			2. create()方法：通过模型类.objects.create()来保存数据
		2. 查询：
			1. 基本查询：get查询单一结果，如果不存在，抛出异常
						 all查询多个结果，返回的是QuerySet类型的列表
						 count查询结果的数量
			 2. 过滤查询：
			 	 filter: 过滤多个结果
			 	 exclude:排除掉符合条件之后剩下的结果
			 	 get: 过滤单一结果
			 	 过滤条件的表达式语法： 属性名称__比较运算符=值  # 属性名称和比较运算符之间使用两个下划线，所以属性名称不能包括多个下划线
			 	 相等：exact：表示判等 模型类.objects.filter(id__exact=1)
			 	 	   contains：是否包含： 模型类.objects.filter(btitle__contains='传')
			 	 	   startswith、endswith:指定开头或者结尾： 模型类.objects.filter(btitle__startwith='部')
	 	 	   	 空查询：isnull: 是否为null 模型类.objects.filter(btitle__isnull=False)
	 	 	   	 范围查询：in: 是否包含在范围内：模型类.objects.filter(id__in=[1, 2, 3])
	 	 	   	 比较查询： gt:大于
	 	 	   	 			gte：大于等于
	 	 	   	 			lt：小于
	 	 	   	 			lte：小于等于
	 	 	   	 			exclude：不等于exclude(id=3)
   	 			日期查询：
   	 				year、month、day、week_day、hour、minute、second:对日期时间类型的属性进行运算: 模型类.objects,filter(bpub_date__year=1980)
			3. F对象：
				之前的查询都是对象属性和常量之间的比较，两个属性之间的比较就会使用到F对象。
				语法：F(属性名)
				使用：
					from django.db.models import F
					BookInfo.objects.filter(bread__gte=F('bcomment'))
					可以在F对象上进行算数运算
			4. Q对象
				多个过滤器逐个调用表示逻辑与关系
				语法：Q（属性名__运算符=值）
				使用：
					from django.db.models import Q
					 BookInfo.objects,filter(Q(bread__gt=20))
					 Q对象可以使用&、|连接，&表示与，|表示逻辑或， ～表示非
			 5. 聚合函数：
			 	使用aggregate()过滤器调用聚合函数，聚合函数包括：Avg（平均），Count（数量），Max（最大值），Min（最小值），Sum（求和）
			 	使用：
			 		from django.db.models import Sum
			 		
			 		BookInfo.objects.aggregate(Sum('bread'))
		 		注意：aggregate的返回值是一个字典类型的数据，格式如下：{'属性名__聚合类小写'：值}
		 		使用count时，一般不使用aggregate()过滤器，并且count函数的返回值是一个数字。
	 		6. 排序：
	 			使用order_by对结果进行排序
	 			BookInfo.objects.all().order_by('bread')
 			7. 关联查询：
 				一对多：
 					语法： 一对应的模型类对象.多对应的模型类名小写_set
 					使用：b = BookInfo.objects.get(id=1)
 						  b.heroinfo_set.all()
			  多对一：
			  		语法：多对应的模型类对象.多对应的模型类中的关系属性名
			  		使用：
			  			h = HeroInfo.objects.get(id=1)
			  			h.hbook
			  访问一对应模型类关联对象id的语法：
			  		多对应的模型类对象.关联属性_id
			  		使用：
			  			h = HeroInfo.objects.get(id=1)
			  			h.hbook_id
  			8. 关联过滤查询：
  				1. 由多模型类条件查询一模型类数据：
  					关联模型类名小写__属性名__条件运算符=值
				2. 由一模型类条件查询多模型类数据:
					一模型类关联属性名__一模型类属性名__条件运算符=值
			9.修改
				修改更新有两种方法
				1. 修改模型类对象的属性，然后执行save()方法
				2. 模型类.objects.filter().update()，会返回受影响的行数
			10. 删除
				删除有两种方法
				1. 模型类对象.delete()  # 物理删除
				2. 模型类.onjects.filter().delete()  # 物理删除
				3. 一般我们不会进行物理删除，我们一般进行的是逻辑删除
## 查询集QuerySet
	1. Django中的ORM中存在有查询集的概念
	2. 查询集：也叫查询结果，表示从数据库中获取的对象的集合
	3. 当调用以下方法的时候，会返回查询集
		1. all():返回所有数据
		2. filter(): 返回满足条件的数据
		3. exclude()：返回满足条件之外的数据
		4. order_by():对结果进行排序
	4. 对查询集可以再次使用过滤器进行过滤
	5. 判断一个查询集中是否有数据：exists(),如果有返回True，没有返回False
	6. 查询集的两大特性：
		1. 惰性执行：创建查询集的时候不会访问数据库，只有在调用数据的时候，才会访问数据库，调用数据的情况包括迭代、序列化等
		2. 缓存：使用同一个查询集，第一次使用的时候会发生数据库的查询操作，然后将结果缓存下来，再次使用这个查询集的时候，使用的就是缓存的数据，减少了数据库的查询次数
	7. 限制查询集：
		1. 可以对查询集进行取下标或者切片的操作，对查询集进行切片后，返回的是一个新的查询集，但不会立即执行查询
## 管理器Manager
	1. 管理器是Django的模型进行数据库操作的接口，每个模型类都拥有至少一个管理器
	2. 我们在通过模型类的objects属性提供的方法操作数据库的时候，就是在使用一个管理器对象，当没有为模型类定义管理器的时候，django会给每一个模型类
	生成一个名为objects的管理器，它是models.Manager的对象
	3. 自定义管理器：
		一旦为模型类指明自定义的管理器之后，Django不再生成默认的管理器对象objects
		1. 修改     
  				 