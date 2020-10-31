Tornado的学习：

 1. 三大模块：
 	
 	tornado.web模块是tornado的基础web框架模块
 	RequestHandler封装了一个请求的所有信息和方法，write是返回相应的方法，对应的请求方式应该以请求方式的小写形式作为函数名定义出来
	Application是tornadoWeb框架的核心应用类，是和服务器对接的接口，里面保存了路由信息表，初始化接受的第一个参数是一个由路由信息映射元祖的列表
	
	tornado.ioloop是tornado的核心模块，封装了linux的epoll和bsd的kqueue，是tornado高性能的基础
	连接请求发送到socket，socket将请求发送给路由映射，找到对应的类视图执行，将结果返回给客户端
	Ioloop.current：返回当前线程的ioloop的实例
	ioloop实例.start():启动实例的i/o循环，服务器监听被打开
	
	httpserver是tornado的HTTP服务器的实现，创建一个http服务器实例，将接受到的客户端请求通过web应用对象中保存的路有映射列表绑定到handler中，在构建http_server的时候需要传入应用对象
	默认启动的是单进程的模式，我们也可以修改为多进程的模式，
	http_server.bind(端口):将服务器绑定到指定的端口  	http_server.start(num_processes)方法指定开启几个进程，参数num_processes的默认值是1，如果num_processes=None或者<=0，则自动根据机器硬件的cpu核数创建同等数目的子进程
	
	虽然提供了一次开启多个进程的方法，但是每个子进程都会从父进程复制一个ioloop实例，如果在创建子进程之前修改了ioloop实例，会影响到每一个子进程
	所有进程是由一个命令一次性开启的，无法做到在不停止服务的情况下更新代码
	所有进程共享一个端口，想要分别单独监控每一个进程就很困难，建议手动开启多个进程，并且绑定不同的端口
	app.listen()只能在单进程模式下使用
	
	tornado.options模块：用来定义options选项变量的方式，定义的变量可以在tornado.options.options中使用，传入的参数有name（ 表示选项的变量名，必须保证全局唯一性），default（选项变量的默认值，默认是None）type（选项变量的类型，可以通过设置type类型的字段控制输入）multipe（选项变量是否可以为多个，默认是False，如果设置为True，在设置选项变量的时候，使用，隔开，是一个列表）help（选项变量的帮助提示信息）
	
	tornado.options.parse_command_line()：转换命令行参数，并且将转换的值对应设置到options对象的属性上，追加命令行参数的方式是--myoption=myvalue
	
	tornado.options.parse_config_file(path):从配置文件中倒入options，配置文件的格式：option=value
	
	当我们在代码中调用parse_command_line()或者parse_config_file()的方法时，tornado会默认为我们配置标准logging模块，即默认开启了日志功能，并向标准输出（屏幕）打印日志信息。
	如果想关闭tornado默认的日志功能，可以在命令行中添加--logging=none 或者在代码中执行如下操作:
	
```python
from tornado.options import options, parse_command_line
options.logging = None
parse_command_line()

```
	
* 在使用配置文件的时候，通常会新建一个python文件（如config.py），
	然后在里面直接定义python类型的变量（可以是字典类型）；在需要配置文件参数的地方，将config.py作为模块导入，并使用其中的变量参数。
+	使用方式：app = tornado.web.Application([], **config.settings)
	
2. 	关于application对象的配置参数

	*  debug：自动重启（也可单独通过autoreload=True设置），取消缓存编译的模版（可以单独通过compiled_template_cache=False来设置；），取消静态文件的hash值（可以单独通过static_hash_cache=False来设置），提供追踪信息（可以单独通过serve_traceback=True来设置）	
	*  我们之前在构建路由映射列表的时候，使用的是二元元祖，对于这个路由映射列表，实际上可以传入多个信息，对于路由中的字典，会传入到对应的RequestHandler的initalize（）方法中
	*  对于路由中的name字段，不能在使用元祖，应该使用tornado.web.url来构建，name是给路由起名字，可以通过调用RequestHandler.reverse_url(name)来获取名字对应的url
*  获取查询字符串参数
	* get_query_argument(anme, default=_ARG_DEFAULT, strip=True) name表示从查询字符串中获取参数是name的值，如果出现同名参数的话，返回最后一个值             default表示没有传值的时候设置的默认值，如果没有设置default，会抛出tornado.web.MissingArguementError异常，strip表示是否过滤掉左右两边的空白字符，默认是True
	* get_query_arguments返回的是一个列表，获取不到，返回一个空列表  
* 获取请求参数
	* get_body_argument(name, default=_ARG_DEFAULT, strip=True)：name表示从请求体中获取参数名为name的值，如果出现多个同名参数，返回最后一个的值，strip表示过滤空白
	* get_body_arguments(name, strip=True):从请求体参数中获取参数名为name的值，返回的是一个列表
	* 对于请求体的数据要求必须是**字符串**，**并且格式是表单编码的格式**，即key1=value&key2=value2，对于请求体数据是json或者是lxml的无法获取
* 前两个方法的整合：
	* ***get _ argument(name, default=_ARG_DEFAULT, strip=True):从请求体或者是查询字符串中获取参数名是name的值，如果出现同名的参数，返回最后一个的值***
	* ***get_arguments(name, strip=True):从请求体和查询字符串中获取参数名是name的值，返回的是一个列表***
	* 这两种方式对于请求体的要求和上面的一致
*  关于请求的其他信息：
	* method：http的请求方式
	* host：被请求的主机名
	* uri：请求的完整资源表示，包括路径信息和查询字符串
	* path：请求的路径部分
	* query：请求的查询字符串部分
	* version：使用的http版本
	* headers：请求头。是类字典类型的对象，支持关键字索引的方式获取
	* **body：请求体数据,可以是表单数据，也可以是json数据**
	* remote_ip:远程客户端的IP地址
	* fiels：用户上传的文件，为字典类型
	* tornado.httputil.HTTPFile:是接受到的文件对象，他有三个属性，filename表示文件的实际名字，body表示文件的数据实体，content_type：表示文件的类型
* tornado中的正则提取路由
	* tornado中对于路由映射也支持正则提取uri，提取出来的参数会作为RequestHandler中对应请求方式的成员方法参数，若在正则表达式中定义了名字，则参数按名字传递，名字的定义和Django的方式是一样的，使用（？P<name>)的方式，若未定义名字，则参数按顺序传递，提取出来的参数会作为对应请求方式的成员方法的参数
* 关于输出
	* 将chunk数据写出到输出缓冲区，write是写到输出缓冲区的，可以像写文件一样多次使用write方法不断的追加响应的内容，最终写到缓冲区的内容作为本次输出的响应
	* 当write方法检测出我们传入的参数是字典的时候，会自动帮助我们转换为json字符串
	* 我们自己手动使用json模块序列化返回的类型是text/html，而使用write方法返回的类型是application/json
	* set_header(name, value)手动的设置一个名为name，值为value的字段
	* set_default_headers():该方法会在进入http处理方法前被调用，可以重写此方法来预先设置默认的headers。注意在http处理方法中使用set_header()方法会覆盖掉set_default_headers()方法中设置的值
	* set_status(status_code, reason=None):设置响应的状态码
	* redirect(url):跳转的url连接
	* send_error(status_code=500, **kwargs):抛出http错误状态码，默认是500， kwargs是可变参数，使用send_error抛出错误之后，tornado会调用相应的write_error()方法进行处理，并且返回给浏览器处理后的错误页面
	* wirte_error(status_code, **kwargs):用来处理send_error抛出的错误信息并返回给浏览器错误信息页面，可以重写此方法定制自己的错误信息页面
* 接口与调用的顺序
	* 	initialize()对应每个请求的处理类Handler在构造一个实例后首先执行initialize()方法，路由映射中的第三个字典型参数会作为该方法的命名参数传递，此方法是用来初始化参数（对像属性），很少使用
	*  prepare()预处理，即在执行对应请求方式的http方法前执行，不管是什么http请求方式，都会执行prepare方法
	*  http请求方法
		* get：请求指定的页面信息，并且返回实体主体
		* head：类似于get请求，只不过返回的响应中没有具体的内容，用于获取报文头
		* post：向指定的资源提交数据进行保存，数据包含在请求体中，post请求可能会导致新的资源的建立和已有资源的修改
		* delete：指定服务器删除指定的内容
		* patch：请求局部修改数据，并不需要提交全部的信息，只需要提交需要修改部分的信息
		* put：从客户端向服务器传送的数据取代指定文档的内容
		* options：返回给定url支持的所有http方法
	* on_finish():在请求处理结束的时候调用，在调用http方法之后调用，通常该方法用来进行资源清理释放或者处理日志，注意：不要在这个方法中响应输出
	* 调用的顺序：在没有错误的时候，set_default_headers()-->initialize()-->prepare()-->http请求方法 --->on finish()
	出现错误的时候，调用的顺序为：set_default_headers() -->initialize() ---> prepare()---> http请求方式 --->set_default_headers() ---> write_error() ---> on finish()
	
* tornado是如何提供静态资源的
	* 我们可以通过向web.Application类的构造函数中传入static_path参数来告诉tornado从哪里读取静态文件
	```python
	app = tornado.web.Application([(r'/', IndexHandler)], static_path=os.path.join(os.path.dirname(__file__), 'statics'))
	os.path.dirname(__file__):获取的是当前文件的绝对路径
```

静态文件目录的命名，为了便于部署，建议使用static
我们可以通过tornado.web.StaticFileHandler来映射静态文件和其访问路径的url
tornado.web.StaticFileHandler是tornado预置的用来提供静态文件的handler

```python
import os
current_path = os.path.dirname(__file__)
app = tornado.web.Application([(r'/', StaticFileHandler, {'path':os.path.join(current_path,'static/html')，"default_filename":"index.html"})])
path:用来指明提供静态文件的跟路径，并在此目录中寻找在路由中用正则表达式提取的文件名
default_filename:用来指定访问路由中未指明文件名的时候，默认提供的文件```

* 使用模板
* 1. 路径与渲染
	* 使用模板，需要像静态文件的路径设置一样，向web.Application类的构造函数中传递一个template_path的参数来告诉tornado从文件中读取模板文件
```python
app = tornado.web.Application(
    [(r'/', IndexHandler)],
    static_path=os.path.join(os.path.dirname(__file__), "statics"),
    template_path=os.path.join(os.path.dirname(__file__), "templates"),
)
```
* 模板的语法
	1. 在handler中使用render()方法来渲染模板并且返回给客户端
	tornado的模板中使用{{}}作为变量或者表达式的占位符，使用render渲染之后占位符{{}}会被替换为相应的值
	{% if %} {% elif %} {% else %}{% end %}
	{% for ... in ... %}{% end%}
	{% while ...%} {% end %}
	2. 给模板传递参数：在视图函数中，使用render('模板文件名'，在模板中使用的名字=传递的参数名)，参数需要构造成一个字典的形式传递给模板，模板中使用对象.属性或者是字典[key]的方式来获取
	3. static_url:tornado模板提供了static_url函数来生成静态文件目录下的url
优点：static_url函数创建了一个基于文件内容的hash值，并将其添加到url的末尾，这个hash值确保浏览器总是加载一个文件的最新版本而不是之前缓存的版本，另一个好处是你可以改变url的结构，而不需要改变模版中的代码，例如，可以通过设置static_url_prefix来改变tornado的默认静态路径前缀
	4. tornado默认开启了模板的自动转义功能，我们可以通过raw语句来输出不被转义的原始格式{% raw text %},若要关闭自动转义功能，一种方法是在Application构造函数中传递autoescape=None，另外一种方法是在每页模板中修改自动转义的行为，{% autoescape None %},关闭自动转义之后，可以使用escape函数对特定的变量进行转义，{{escape(要转义的变量名)}}
	5. 自定义函数
		* 在模板中我们可以编写自定义的函数，只需要将函数名作为模板的参数传递即可，比如 self.render('模板文件名'，参数=参数， 函数=函数)
		* 我们可以使用块来复用模板，块语法如下{% block blockname %} {% end %}

tornado中的数据库
1. 和Django相比，tornado没有自带orm，数据库需要自己配置
2. torndb是对mysql数据库的简单封装，不支持python3
3. ```python
	class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r"/", IndexHandler),
        ]
        settings = dict(
            template_path=os.path.join(os.path.dirname(__file__), "templates"),
            static_path=os.path.join(os.path.dirname(__file__), "statics"),
            debug=True,
        )
        super(Application, self).__init__(handlers, **settings)
        # 创建一个全局mysql连接实例供handler使用
        self.db = torndb.Connection(
            host="127.0.0.1",
            database="itcast",
            user="root",
            password="mysql"
        )
	```
	
4. 数据库的使用
	创建数据库：create database '数据库名' default character set utf8;
	使用数据库：use 数据库名
	
	创建数据库的表：create table 数据库表名 (
		id bigint(20) unsigned bot null auto_increment comment '介绍'，
		title varchar(64) not null default '' commnet '介绍'
		primary key（id）
	)ENGINE=Innodb default charset=utf8 comment=‘介绍’
 	执行数据库的语句：
 		execute（query， parameters，kwargs）：返回影响的最后一条自增字段的值
 		execute_rowcount(query, parameters, kwargs):返回受影响的行数
 		query为要执行的sql语句，parameters和kwargs是要绑定的参数
 		get(query, parameters ,kwargs):返回单行结果或者是None，若结果是多行就会报错，返回值是tornado.Row类型，是一个类字典的对象，即同时支持字典的关键字索引和对象的属性访问
 		query(query, parameters, kwargs):返回多行结果，tornado.Row类型的列表
 		
5. 操作cookie
	* set_cookie(name, value, domain=None, expires=None, path='/', expires_days=None)
	* 参数的说明：name：cookie名， value：cookie值， domain：提交cookie时匹配到的域名，path：提交cookie时匹配到的路径，expires：cookie的有效期，可以是时间戳整数，时间元祖或者是datetime类型，expires_days:cookie的有效期，天数，优先级低于expires
	* 获取cookie：get_cookie(name, default=None):获取名为name的cookie值，可以设置默认值
	* 清除cookie：clear_cookie(name, path='/', domain=None):删除名为name，并同时匹配domain和path的cookie
	* clear_all_cookies(path='/', domain=None):删除同时匹配domain和path的所有cookie
	* 注意：执行清除cookie的操作之后，并不是立即删除了浏览器中的cookie值，而是把cookie的值置空，并且改变其有效期使其失效，真正的删除cookie是由浏览器自己去清理的
	* cookie是存储在浏览器中的，很容易被篡改，tornado提供了一种对cookie进行建议签名加密的方法来防止cookie被恶意篡改
	* 使用安全的cookie需要为应用配置一个用来给cookie进行混淆的密钥cookie_secret,将其传递给Application的构造函数，我们可以使用如下的方法生成一个随机字符串作为cookie_secret的值
	```python 
	import base64, uuid
	base64,b64encode(uuid.uuid4().bytes + uuid.uuid4().bytes)
	base64作为一种基于64个可打印的字符串来表示二进制数据的表示方法，uuid是通用唯一识别码，是由一组32个16进制数字构成的，uuid模块的uuid4()函数可以随机产生一个uuid码，bytes属性将此uuid码作为16字节字符串
	将生成的cookie_secret传入Application构造函数
	app = tornado.web.Application([(r'/', IndexHandler),], cookie_secret='随机字符串')
	```
	* 设置安全的cookie：set_secure_cookie(name, value, expire_days=30):设置一个带签名和时间戳的cookie
	* 获取安全的cookie：get_secure_cookie(name, value=None, max_age=31):如果cookie存在并且验证通过，返回cookie的值，否则返回None，max_age_day不同于expires_days。expires_days是设置浏览器中cookie的有效期，而max_age_day是过滤安全cookie的时间戳
	* 签名之后的cookie值："2|1:0|10:1476412069|5:count|4:NQ==|cb5fc1d4434971de6abf87270ac33381c686e4ec8c6f7e62130a0f8cbe5b7609"
	* 安全的cookie的版本，默认是2，默认是0，时间戳，cookie名，base64编码之后的cookie值，签名值
	* 注意：tornado的安全cookie只是一定程度的安全，仅仅是增加饿了恶意修改的难度，Tornado的安全cookies仍然容易被窃听，而cookie值是签名不是加密，攻击者能够读取已存储的cookie值，并且可以传输他们的数据到任意服务器，或者通过发送没有修改的数据给应用伪造请求。因此，避免在浏览器cookie中存储敏感的用户数据是非常重要的。
6. XSRF（跨站请求伪造）
	* 开启XSRF保护，需要在Application的构造函数中添加xsrf-cookies参数app = tornado.web.Application(
    [(r"/", IndexHandler),],
    cookie_secret = "2hcicVu+TqShDpfsjMWQLZ0Mkq5NPEWSk9fi0zsSt3A=",
    xsrf_cookies = True
)
	* 用不带xsrf的请求时，会报403错误
7. 用户验证
	* 用户验证是指在收到用户请求之后进行处理前先判断用户的认证状态，若通过验证则正常处理，否则强制用户跳转至认证页面
	* authenticated装饰器
	* 为了使用tornado的认证功能，我们可以使用tornado.web.authenticated装饰器完成，tornado将确保这个方法的主体只有在合法的用户被发现的时候才会调用
	```
	class ProfileHandler(RequestHandler):
		@tornado.web.authenticated
		def get(self):
			self.write('index')```
	
	* get_current_user()方法
		* 装饰器的判断执行依赖于请求处理类中的self.current_user属性，如果current_user值为假，任何get或者head请求都将把访客重定向到应用设置中login_url指定的url，而非法的用户的post请求将返回一个带有403状态的http响应
		* 在获取self.current_user属性的时候，tornado会调用get_current_user()方法来返回current_user的值，也就是说，我们的验证用户的逻辑应该写在get_current_user()方法中，若该方法返回非假值则验证通过，否则验证失败
```python
		class ProfileHandler(RequestHandler):
			def get_current_user(self):
				'''在此完成用户的逻辑认证'''
				username = self.get_argument('name', None) 				return user_name
			@tornado.web.authenticated
			def get(self):
				self.write('index')			
		```
	* login_url设置
		* 当用户验证失败的时候，将用户重定向到login_url设置的url中，我们还需要在Application中配置的login_url
		* app = tornado.web.Application([(r'/', IndexHandler)], 'login_url = '/login')
8. 异步和websockets
	* 理解同步和异步
		* 同步就是按部就班的依次执行，始终按照同一个步调来执行，上一个步骤没有执行完毕，不能进入下一个步骤
		* 异步就是返回一个可调用的状态，并不会马上收到结果，当结果执行完毕的时候，将结果返回给我们，异步的特点是程序存在多个步调，即原本属于同一个过程的代码可以在不同的步调上同时执行
	* 协程写法实现原理
		* 使用带参数的装饰器gen_coroutine
		*  tornado实现异步的机制不是线程，而是epoll，即将异步过程交给epoll执行并进行监视回调，需要注意的一点是，我们实现的版本严格意义上来说并不能管事协程，因为两个程序的刮起与唤醒是在两个线程上实现的，而tornado利用epoll来实现异步，程序的挂起与唤醒始终在一个线程上，由tornado自己来调用，属于真正意义上的协程
	* tornado的异步网络请求
		* tornado提供了一个异步web请求客户端tornado.httpclient.AsyncHTTPClient用来实现异步web请求
		* fetch(request, callback=None)：用于执行一个web请求request，并且异步返回一个tornado.httpclient.HTTPResponse响应
		* request可以是一个url，也可以是一个tornado.httpclient.HTTPRequest对象，如果是url，fetch会自己构造一个HTTPRequest对象
		* HTTPRequest：HTTP请求类，HTTPRequest的构造函数可以接受很多参数，最常用的如下：
		url:访问的url，必传
		method:http的请求方式，默认是get
		headers:http请求头
		body: http请求体
		code：http状态码
		reason：状态码的详细描述
		body：响应体字符串
		error：异常
		* tornado.web.asynchronous：此装饰器用于回调形势的异步方法，并且仅用于http的方法上
		* 此装饰器不会让被装饰的方法变为异步，而是告诉框架并装饰的方法是异步的，当方法返回时响应尚未完成，只有在request handler调用了finish方法之后，才会结束本次请求的处理，发送响应
		* 不带此装饰器的请求在get、post等方法返回的是欧自动完成结束请求的处理
		* tornado可以用来执行多个异步，并发的异步可以使用列表或者字典
* 1. Tornado的WebSocket模块
Tornado提供支持WebSocket的模块是tornado.websocket，其中提供了一个WebSocketHandler类用来处理通讯。

WebSocketHandler.open()
当一个WebSocket连接建立后被调用。

WebSocketHandler.on_message(message)
当客户端发送消息message过来时被调用，注意此方法必须被重写。

WebSocketHandler.on_close()
当WebSocket连接关闭后被调用。

WebSocketHandler.write_message(message, binary=False)
向客户端发送消息messagea，message可以是字符串或字典（字典会被转为json字符串）。若binary为False，则message以utf8编码发送；二进制模式（binary=True）时，可发送任何字节码。

WebSocketHandler.close()
关闭WebSocket连接。

WebSocketHandler.check_origin(origin)
判断源origin，对于符合条件（返回判断结果为True）的请求源origin允许其连接，否则返回403。可以重写此方法来解决WebSocket的跨域请求（如始终return True）。	 
	
			 	

