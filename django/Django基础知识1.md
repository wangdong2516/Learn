# Django
## Django框架的特点
	1. 重量级框架
		1. 相比较于flask框架，Django框架原生提供了众多的功能组件，使开发变得简单、快捷
		2. 提供项目工程管理的自动化脚本工具
		3. 数据库ORM支持（对象关系映射）
		4. 模板
		5. 表单
		6. admin管理站点
		7. 文件管理
		8. 认证权限
		9. session机制
		10. 缓存
	2. MVT模式
		1. MVT模式是由MVC模式发展而来的
		2. MVC设计模式的核心思想是分工、解耦，让不同代码块之间降低耦合度，增强代码的可扩展性和可移植性，实现向后兼容
		3. MVC模式说明：
			1. M：表示Model，主要封装对数据库层的访问，对数据库中的数据进行增、删、改、查的操作
			2. V：表示为View，用于封装结果，生成页面展示的html内容（页面渲染）
			3. C：表示为Controller，用于接收请求，处理业务逻辑，于Model和View进行交互，返回结果
	3. Django中的MVT设计模式：
		1. M：表示为Model，与MVC中Model的功能是相同的，负责和数据库进行交互，进行数据处理
		2. V：表示为View，于MVC中C的功能是相同的，负责接收请求，进行业务处理，返回结果
		3. T：表示为Template，与MVC中V的功能是相同的，负责封装构建要返回渲染的html页面 	  	 	  	
## 环境安装
	1. django版本：1.11.11
	2. 创建虚拟环境（首先安装mkvirtualenv命令）
		1. 虚拟环境命令：
			1. mkvirtualenv 虚拟环境名称  # 创建虚拟环境 	 		2. rmvirtualenv 虚拟环境名称  # 删除虚拟环境，注意：必须先退出当前的虚拟环境才能删除（类似于删除git分支）
			3. workon 虚拟环境名称  # 进入虚拟环境，查看所有的虚拟环境
			4. deactivate 退出虚拟环境
		2. pip命令：
			1. 安装依赖包： pip install 包名称
			2. 卸载依赖包： pip uninstall 包名称
			3. 查看已经安装的依赖：  pip list
			4. 冻结当前环境的依赖： pip freeze(将项目的依赖倒出，一般用于部署) 
		3. 创建不同python版本的虚拟环境：
			1. mkvirtualenv -p pytohn2 虚拟环境名称  # 创建python2的虚拟环境
			2. mkvirtualenv -p python3 虚拟环境名称  # 创建python3的虚拟环境
## 创建Django工程
	1. 创建工程： django-admin startproject 工程名称
## 项目的目录说明
	1. settings.py: 是项目的整体配置文件
	2. urls.py:是项目的url配置文件
	3. wsgi.py：是项目与WSGI兼容的web服务器的入口
	4. manage.py: 是项目的管理文件，通过它管理项目
## 创建子应用
	1. 创建子应用工作目录： python manage.py startapp 子应用名称
		django中的子应用相当于flask中的蓝图
## 子应用的目录结构  
	1. 子应用目录说明：
		1. admin.py：跟网站的后台管理站点配置相关
		2. apps.py：用于配置当前子应用的相关信息
		3. migrations：用于存档数据库迁移历史文件
		4. models.py： 用于保存数据库模型类
		5. tests.py： 用于开发测试用例，编写单元测试
		6. views.py: 用于编写view试图
	2. 注册安装子应用
		1. 创建出来的子应用目录虽然已经存在于项目目录中，但是django工程并不能字节立即使用该子应用，需要注册安装之后才能使用
		2. 在工程配置文件settings.py文件中，INSTALLED_APPS中保存了工程中已经注册安装的子应用
		3. 注册安装一个子应用的方法，就是将子应用的配置信息文件apps.py中的Config类添加到INSTALLED_APPS列表中
## 运行开发服务器
	1. django提供了一个python编写的轻量级的web服务器，仅在开发阶段使用
	2. 运行服务器的命令如下：
		1. python manage.py runserver ip:port：运行项目服务再某一个ip和端口，也可以不写ip和端口，默认是127.0.0.1:8000
		2. django默认工作在调试模式下，如果增加、删除、修改文件，服务器会自动重启
## web应用模式
	1. 前后端分离：在前后端分离的应用模式中，后端仅返回前端需要的数据，不再渲染html页面，不再控制前端的展示效果
	在前后端分离的应用模式下，我们将后端开发的每个试图都称为一个接口，或者是API
	2. 前后端不分离：前端页面看到的效果是由后端去控制的，由后端负责渲染页面或者是重定向，前端和后端的耦合度很高
## 创建视图
	1. Django中视图的定义是在子应用的views.py文件中定义的
	
		from django.http import HttpResponse
		
		def index(request):
			"""编写视图函数以及对应的逻辑，request表示包含了请求信息的请求对象"""
			return HttpResponse('Hello Python')  # HttpResponse表示响应对象
	2. 说明：
		1. 视图函数传入的第一参数必须定义，用于接收Django构造的包含请求数据的HttpRequest对象，通常叫做request
		2. 视图函数的返回值必须是一个响应对象，不能直接返回一个字符串，必须将字符串放到响应对象中去。
## 定义视图路由
	1. 在子应用中新建一个urls.py文件用于保存该子应用的路由（一般在创建子应用的时候，urls.py文件不会自动创建，需要手动来创建）
	2. 在子应用的urls.py文件中定义路由信息
		
		from django.conf.urls import url
		from . import views
		
		# urlpatterns是被django自动识别的路由列表变量
		# 每个路由信息都需要使用url函数来构造
		# url（路径， 视图）
		urlpatterns = [
			url(r'^index/$', views.index)
		]  	
	3. 在总路由文件中提加子路由的数据
		
		# 总路由文件urls.py
		from django.conf.urls import url, include
		from django.contrib import admin
		
		urlpatterns = [
			url(r'^admin/', admin.site.urls),  # django默认包含的
			url(r'^users/', include('users.urls')), # 自己添加的
		] 	
	4. 说明：
		1. 使用include来将子应用中的全部路由包含到总路由中
		2. r'^users/'决定了子应用中的所有路由全部以/users开头
		3. include函数除了能够传递字符串之外，还可以传递子应用的模块
## Django的配置文件
	1. BASE_DIR: 当前工程的根目录，Django依此来定位工程内的相关文件，我们也可以使用该参数来构造文件的路径
		这里就是os模块的使用：os.path.abspath(__file__)获取到当前文件的绝对路径
		os.path.dirname(os.path.abspath(__file__))获取到当前文件所在的目录
		os.path.dirname(os.path.dirname(os.path.abspath(__file__)))获取到的是当前文件所在目录的上一级目录，也就是工程的根目录
	2. 	DEBUG：创建工程之后，DEBUG默认位True，默认在DEBUG模式下工作
	3. 本地语言和时区：LANGUAGE_CODE = 'zh-hans'  中国大陆
						TIME_ZONE = 'Asia/Shanghai'  
## 静态页面
	1. 项目中的css，图片，js都是静态文件，一般静态文件都会放到一个单独的目录中，方便管理，Django中提供了一种解析方式来配置静态文件路径
	2. 为了提供静态文件，需要配置两个参数：
		1. STATICFILES_DIRS：存放查找静态文件的目录
		2. STATIC_URLS: 存放静态文件路由的url前缀
	3. 在settings中修改静态文件的两个参数：
		1. STATIC_URL = '/static/'  静态文件的路由匹配
		2. STATICFIELS_DIRS = [
			os.path.join(BASE_DIR, 'static_fiels'),  # 指定静态文件的存储路径
		]  
		3. Django仅在调试模式下能够对外提供静态文件
		4. 当DEBUG=False工作在生产模式下的时候，Django不能在对外提供静态文件，需要使用collectstatic命令来搜集静态文件并且交给其他静态文件服务器来提供
## 路由说明
	1. Django中的主要路由信息定义在工程目录同名目录中的urls.py文件中，该文件是Django解析路由的入口
	2. 每个子应用为了保持相对独立，可以在各个子应用中定义属于自己的urls.py文件来保存该子应用的路由，然后用主应用的路由文件包含各子应用的路由数据
	3. 路由解析顺序：
		1. Django在接收到一个请求的时候，从主路由文件中的urlpatterns列表中由上至下来查找路由，如果发现规则为include包含的时候，在进入被包含文件的urlpatterns列表中由上至下进行查询
		2. 注意：由上至下的顺序，可能会使上面的路由屏蔽掉下面的路由，导致路由找不到，所以需要注意路由的定义顺序，避免出现屏蔽效应。
	4. 路由命名和反解析：
		1. 在使用include函数定义路由的时候，可以使用namespace参数定义路由的命名空间，比如
			url(r'^users/', include('users.urls', namespace='users'))
			命名空间表示：凡是定义在users.urls中定义的路由，均属于namesapce指明的users名下
			命名空间的作用：避免了不用应用中使用了相同名字的路由导致了冲突
		2. 在定义普通路由的时候，可以使用name参数指明路由的名字，比如
			uelpatterns = [
				url(r'^index/$', views.index, name='index'),
				url(r'^say', views.say, name='say')
			] 
		3. reverse反解析：
			1. 使用reverse函数，可以根据路由名称，返回具体的路径
			2. 对于未指明namesapce的路由，使用reverse（路由name）
				对于指明了namespace的路由，使用reverse(命名空间.路由name)
		4. 路径结尾/的说明：
			1. 在Django中定义路由的时候，通常以斜线/结尾，其好处是用户访问不以斜线/结尾的路径的时候，Django会把用户重定向到以斜线/结尾的路径上，而不会返回404 NOT FOUND
	总结：django在进行路由处理的时候，先交给总路由进行处理，匹配到之后，进入到子路由中查找对应的匹配规则，有的话，进入到视图函数进行处理，没有的话，返回错误. 
## 请求和响应
	1. 请求：
		1. url路径参数：
			1. 在定义路由url的时候，可以使用正则表达式提取参数的方法从url中获取请求参数，Django会将提取到的参数直接传递到视图函数的传如参数中
			2. 为命名参数按照定义的顺序来传递，命名参数按照名字来进行传递
	2. Django中的QueryDict对象
		1. 定义在django.http.QueryDict的HttpRequest对象的属性get，post都是QueryDict对象
		2. 于字典类型不同，QueryDict类型的对象用来处理同一个键带有多个值的情况
		3. get：根据键获取值：
			1. 如果一个键同时有多个值将获取最后一个值
			2. 如果键不存在则没有返回值，可以设置默认值进行后续处理
		4. getlist: 根据键获取值，值以列表的形式返回，可以获取指定键的所有值
			1. 如果键不存在，则返回空列表，可以设置默认值进行后续处理
		5. 获取查询字符串：
			1. 获取请求路径中的查询字符串可以通过request对象的GET属性获取，返回的是QueryDict对象
				request.GET.get('查询字符串参数名')
			2. 注意: 查询字符串参数的获取，不区分请求方式，统一使用request.GET属性获取查询字符串中的数据
		6. 请求体数据
			1. 请求体的数据格式不固定，可以是表单类型，可以是json字符串，可以是xml字符串，应该区别对待
			2. 可以发送请求体数据的请求方式有：post，put, patch, delete
			3. Django默认开启了CSRF防护，会对上述的请求方式进行CSRF防护验证，需要在settings.py文件中注释掉csrf中间件
			4. 获取表单数据：
				1. 通过使用request.POST属性获取，返回的是QueryDict对象
					request.POST.get('键')
				2. 注意：request.POST只能用来获取post请求方式中的表单数据
			5. 获取非表单类型数据：
				1. 非表单类型的请求体数据，Django是无法进行自动解析的，可以通过request.body属性获取最原始的请求体数据，自己解析，request.body返回bytes类型的数据
		7. 请求头中的数据：
			1. 可以通过request.META属性获取请求头中的数据，request.META是字典类型的数据
			2. 常见的请求头有以下几种：
				1. CONTENT_LENGTH: 请求正文的长度
				2. CONTENT_TYPE: 请求正文的类型
				3. HTTP_ACCEPT: 响应可以接受的文本类型
				4. HTTP_ACCEPT_ENCODING:响应可以接受的编码方式
				5. HTTP_ACCEPT_LANGUAGE:响应可以接受的语言
				6. HTTP_HOST: 客户端发送请求的主机
				7. HTTP_REFERER:引用页面，用于标识用户是从那个页面跳转来的
				8. HTTP_USER_AGENT: 客户端代理
				9. QUERY_STRING: 查询字符串
				10. REMOTE_ADDR: 客户端IP地址
				11. REMOTE_HOST: 客户端主机
				12. REMOTE_USER:验证的用户
				13. REQUEST_METHOD: 请求方式
				14. SERVER_NAME: 服务器主机
				15. SERVER_PORT: 服务器端口
		8. 其他常用的HttpRequest对象的属性：
			1. method: 获取请求的方式
			2. user: 获取请求的用户对象
			3. path: 获取请求的路径，不包含域名和参数部分
			4. encoding: 获取请求的编码方式
			5. FILES：获取请求上传的文件，是一个类似于字典的对象
	3. 响应
		1. 视图函数在接受请求并且处理之后，必须返回HttpResponse对象或者是子对象，HttpRequest对象是由Django创建的，HttpResponse对象是开发人员创建的
		2. HttResponse对象：
			1. 可以使用django.http.HttpResponse来构造响应对象
			2. 使用：
				1. HttpResponse(content=响应体, content_type=响应体数据类型, status=状态码)
				2. 响应头可以直接将HttpResponse对象当作字典进行响应头键值对的设置
			3. HttpResponse子类：
				1. HttpResponseRedirect  301
				2. HttpResponsePermanentRedirect  302
				3. HttpResponseNotModified  304
				4. HttpResponseBadRequest  400
				5. HttpResponseNotFound  404
				6. HttpResponseForbidden  403(没有认证)
				7. HttpResponseNotAllowed   405(不允许访问)
				8. HttpResponseGone  410
				9. HttpResponseServerError  500
			4. JsonResponse
				1. 若要返回json数据，可以使用JsonResponse来构造响应对象，作用就是帮助我们将数据转换成json字符串
				 	设置响应头Content-Type为application/json
			 	from django.http import JsonResponse
			 	
			 	def index(request):
			 		return JsonResponse({'name': 'wangdong'})
		 		2. 注意：使用JsonResponse返回的必须是字典格式的数据
	4. redirect重定向
		1. 使用：
			from django.shortcuts import redirect
			
			def demo_view(request):
				return redirect('/index.html')
## cookie
	1. cookie: 是指网站为了辨别用户身份，进行session跟踪而存储在用户本地浏览器上的数据，通常是经过加密的，cookie是由服务端生成的，cookie的名称和值可以由服务器端自己定义，cookie是存储在浏览器中的一段纯文本信息，建议不要存储敏感信息
	2. cookie的特点：
		1. cookie是以键值对的格式进行信息的存储
		2. cookie基于域名安全，不同域名之间的cookie是无法访问的
		3. 当浏览器请求某网站的时候，会将浏览器中存储的和该网站的cookie信息提交给网站服务器
	3. 设置cookie：
		1. 可以通过HttpResponse对象中的set_cookie方法来设置cookie
			HttpResponse.set_cookie(name=cookie名， value=cookie值， max_age=cookie有效期)
			max_age的单位为秒，默认是None， 如果是临时cookie，可以将max_age设置为None        
	4. 读取cookie
		1. 可以通过HttpRequest对象的COOKIES属性来读取本次请求携带的cookie值，request.COOKIES为字典类型的数据 
	5. 删除cookie
		response.delete_cookie('键') 
	6. 使用：
		设置cookie
		from django.http import HttpResponse
		
		def index(request):
			
			response = HttpResponse('ok') # 响应体内容
			response.set_cookie('name', 'wangdong', max_age=3600)  # 调用response对象的set_cookie方法设置cookie，参数1表示cookie名，参数二表示cookie的值
			参数三表示cookie的有效期，单位是秒，默认是None 
		读取cookie
		1. 使用request对象的COOKIES属性来获取cookie中的信息
			request.COOKIES.get('name')   COOKIES属性是一个字典，可以使用字典的方式获取
	总结： cookie的设置是依赖于response对象来完成的，response.set_cookie(name, value, max_age)
			cookie的获取是依赖于request对象来完成的，request.COOKIES.get('键')，request.COOKIES属性是一个字典
			不能通过js代码提取cookie
			cookie中存储的数据是字符串和整型数据
## session（实现状态保持）
	session在redis中，存储的形式是字符串类型，key：value
	key是一个随机的字符串，唯一值， value值是字符串类型
	随机字符串又叫做sessionID，返回session的时候，会将sessionID进行返回，存储在cookie中，sessionID：随机字符串
	1. django项目默认启用session，使用的是session中间件
	2. 存储方式：在settings.py文件中，可以设置session数据的存储方式，可以保存在数据库中，可以存储在本地缓存中， 默认是存储在数据库中的。
	3. 存储在数据库的方式：
		1. SESSION_ENGINE='django.contrib.session.backends.db'  # 这是session默认的存储方式
		2. 如果存储在数据库中，需要在INSTALLED_APPS中注册安装session应用
	4. 存储在内存中，比数据库的方式读写更加快（临时存储）
		1. 在settings.py文件中设置：SESSION_ENGINE='django.contrib.session.backends.cache'
	5. 混合存储：
		1. 优先从本地内存中读取，如果没有的话，从数据库中读取 
	6. 在redis中保存sesion的方法：
		1. 安装扩展： pip install django-redis
		2. 配置：在settings.py文件中：
			CACHES = {
				'default': {
					'BACKEND': 'django_redis.cache.RedisCache',
					'LOCATION': 'redis://127.0.0.1:6379/1',
					'OPTIONS': {
						'CLIENT_CLASS': 'django_redis.client.DefaultClient'
					}
				}
			}    	
			SESSION_ENGINE='django.contrib.session.backends.cache'
			SESSION_CACHE_ALIAS='default'
	7.如果redis的连接地址不是127.0.0.1，而是其他的地址，Django可能会出现连接错误
		解决办法就是修改redis的配置文件，添加特定的ip地址
		sudo vim /etc/redis/redis.conf
		添加： bind ip port
	8. 关于session的一些操作：
		1. 可以通过HttpRequest对象的session属性进行读写操作
		2. 以键值对的形式写session：request.session['键'] = 值
		3. 根据键获取值：request.session.get('键'， 默认值)
		4. 清除所有的session值，在存储中删除值的部分：request.session.clear()
		5. 清除session数据，在存储中删除整条session数据：request.session.flush()
		6. 删除session中指定的键和值： del requests.session['键']
		7. 设置session有效期：request.session.set_expire(value)
			如果value是一个整数，那么session将在value秒之后过期
			如果value是0，将在浏览器关闭之后过期
			如果value的值是None，将采用默认值，默认为两周，可以通过在settings.py文件中设置SESSION_COOKIE_AGE来设置全局默认值 
	总结：session的设置和获取是依赖于request请求对象来完成的，session的设置			request.session['键'] = 值， 
			session的获取：request.session.get('键') session的删除，request.session.clear()/request.session.flush()/ del request.session['键'], 
			设置session有效期： request.session.set_expire(value)
## project和app的区别
	1. 一个app实现某个功能
	2. 一个project是配置文件和多个app的集合，它们组合成整个站点
	3. 	