<center> Django中请求流程的分析
  
</center>

1. wsgi.py是整个Django项目的入口

2. 路由注册的流程

    1. 我们知道在项目或者子应用的urls.py文件中，注册路由，在views编写视图函数之后，就可以在浏览器中访问了，那么背后到底发生了什么，让我们一起研究一下

        ```python
        # 这里项目的一个路由表，调用了path()方法或者是re_path()方法，并且添加到了urlpatterns列表中
        # 不管是path()还是re_path()，最后返回的都是URLResolver类的实例对象
        urlpatterns = [
            re_path('admin/', admin.site.urls),
            re_path('', include('user.urls')),
        ]
        ```

    2. 看看path()和re_path()都做了什么

        ```python
        # 可以看到path()和re_path()是两个偏函数
        # 偏函数的主要作用就是给函数传入预先默认的参数，如果再传入相同的参数，就会覆盖掉之前的参数值
        # 其实就是和默认值差不多
        path = partial(_path, Pattern=RoutePattern)
        re_path = partial(_path, Pattern=RegexPattern)
        
        
        def _path(route, view, kwargs=None, name=None, Pattern=None):
            if isinstance(view, (list, tuple)):
                # 处理include()函数
                pattern = Pattern(route, is_endpoint=False)
                urlconf_module, app_name, namespace = view
                return URLResolver(
                    pattern,
                    urlconf_module,
                    kwargs,
                    app_name=app_name,
                    namespace=namespace,
                )
            elif callable(view):
              	# 实例化默认传入的RegexPattern或者是RoutePattern
                pattern = Pattern(route, name=name, is_endpoint=True)
                # 实例化URLPattern并且返回
                return URLPattern(pattern, view, kwargs, name)
            else:
              	# 对于include（），视图必须是可调用的或列表/元组。
                raise TypeError('view must be a callable or a list/tuple in the case of include().')
        ```

        

3. 路由匹配的流程

    1. 首先设置环境变量DJANGO_SETTINGS_MODULE指向了当前Django项目的配置文件

    2. 获取一个wsgi_application

        ```python
        application = get_wsgi_application()
        ```

        + 在get_wsgi_application方法内部做了以下的事情：

        + ```python
            def get_wsgi_application():
              django.setup(set_prefix=False)
              return WSGIHandler()
            ```

            + django.setup(set_prefix=False)启动了一个Django应用，主要做的事情是加载一些配置，初始化应用的配置和导入对应的应用模块从insatlled_apps中之类的一些操作

            + 返回了一个满足wsgi协议的可调用对象，该对象在初始化的时候，加载了在配置文件中定义的中间件，实现了请求钩子（process_view， process_template_response， process_exception）

                ```python
                def load_middleware(self):
                  """加载中间件"""
                  self._view_middleware = []
                  self._template_response_middleware = []
                  self._exception_middleware = []
                	
                  # 我们重点关注self._get_response，这里实现了路由的匹配
                  # convert_exception_to_response就是一个闭包，返回的结果是_get_response函数的引用
                  handler = convert_exception_to_response(self._get_response)
                  # 这里大概就是Django中的中间件，在视图函数执行之前是从上到下，但是在视图函数
                  # 执行之后是从下到上返回的原因吧
                  for middleware_path in reversed(settings.MIDDLEWARE):
                    middleware = import_string(middleware_path)
                    try:
                      mw_instance = middleware(handler)
                    except MiddlewareNotUsed as exc:
                      if settings.DEBUG:
                         ...
                      if hasattr(mw_instance, 'process_view'):  django提供的请求钩子
                            self._view_middleware.insert(0, mw_instance.process_view)
                      if hasattr(mw_instance, 'process_template_response'):  django提供的请求钩子
                                    		self._template_response_middleware.append(mw_instance.process_template_response)
                      if hasattr(mw_instance, 'process_exception'):  django提供的请求钩子
                                      		     self._exception_middleware.append(mw_instance.process_exception)
                
                       handler = convert_exception_to_response(mw_instance)
                  self._middleware_chain = handler  ** 将handler保存在了_middleware_chain属性中，
                ```

                

            + 当这个可调用的wsgi对象WSGIHandler被调用的时候，执行\__call()__方法中的逻辑，当请求进入的时候，wsgi就会调用这个可调用对象了。

                ```python
                class WSGIHandler(base.BaseHandler):
                  request_class = WSGIRequest
                  ...
                  def __call__(self, environ, start_response):
                    set_script_prefix(get_script_name(environ))  设置脚本前缀在当前的线程
                    signals.request_started.send(sender=self.__class__, environ=environ)  发送信号
                    request = self.request_class(environ)  实例化WSGIRequest()传入环境变量
                    response = self.get_response(request)  获取wsgi的响应
                ```

                + self.request_class实际上就是WSGIRequest的引用，self.request_class(environ)实际上就是在初始化WSGIRequest类

                ```python
                class WSGIRequest(HttpRequest):
                    def __init__(self, environ):
                        script_name = get_script_name(environ)  # 获取当前线程的脚本前缀
                        # If PATH_INFO is empty (e.g. accessing the SCRIPT_NAME URL without a
                        # trailing slash), operate as if '/' was requested.
                        path_info = get_path_info(environ) or '/' # 从wsgi应用中解析到的请求路径
                        self.environ = environ  # 保存一些环境变量
                        self.path_info = path_info  # 保存请求路径
                        self.path = '%s/%s' % (script_name.rstrip('/'),
                                               path_info.replace('/', '', 1))
                        self.META = environ
                        self.META['PATH_INFO'] = path_info
                        self.META['SCRIPT_NAME'] = script_name
                        self.method = environ['REQUEST_METHOD'].upper()  # 解析请求方式并且转换为大写形式，保存到了method属性中
                        # Set content_type, content_params, and encoding.
                        self._set_content_type_params(environ)
                        try:
                            content_length = int(environ.get('CONTENT_LENGTH'))
                        except (ValueError, TypeError):
                            content_length = 0
                        self._stream = LimitedStream(self.environ['wsgi.input'], content_length)
                        self._read_started = False
                        self.resolver_match = None
                ```

                + 现在看看get_response方法内部做了些什么

                    ```python
                    def get_response(self, request):
                        set_urlconf(settings.ROOT_URLCONF)  # 从配置文件中读取ROOT_URLCONF，设置项目的根路由
                        # self._middleware_chain保存的就是handler，也就是_get_response的引用
                        response = self._middleware_chain(request)
                        response._closable_objects.append(request)
                        if response.status_code >= 400:
                          log_response(
                            '%s: %s', response.reason_phrase, request.path,
                            response=response,
                            request=request,
                          )
                          return response
                    ```

                + 我们可以看到上面的response = self._middleware_chain(request)这句其实是调用了BaseHandler内部定义的\_get_response方法

                    ```python
                    def _get_response(self, request):
                    
                      response = None
                    
                      if hasattr(request, 'urlconf'):
                        urlconf = request.urlconf
                        set_urlconf(urlconf)  # 设置路由
                        resolver = get_resolver(urlconf)  # 获取路由解析器
                      else:
                          resolver = get_resolver()  # 获取路由解析器
                      resolver_match = resolver.resolve(request.path_info)  # 进行路由匹配
                      # 指定的视图函数，匹配到的位置参数和关键字参数
                      callback, callback_args, callback_kwargs = resolver_match
                      request.resolver_match = resolver_match
                      # 进入注册的process_request处理视图函数
                      for middleware_method in self._view_middleware:
                        response = middleware_method(request, callback, callback_args, callback_kwargs)
                        if response:
                          break
                      if response is None:
                        wrapped_callback = self.make_view_atomic(callback)
                        try:
                          # 这一步就是在调用我们的视图函数，并且传入相应的请求参数，获取响应
                          response = wrapped_callback(request, *callback_args, **callback_kwargs)
                        except Exception as e:
                            response = self.process_exception_by_middleware(e, request)
                    ```

                    ```python
                    def get_resolver(urlconf=None):
                        if urlconf is None:
                            urlconf = settings.ROOT_URLCONF
                        return _get_cached_resolver(urlconf)
                    
                    @functools.lru_cache(maxsize=None)
                    def _get_cached_resolver(urlconf=None):
                      	# 这里的URLResolver实际上就是在项目路由文件中定义的urlpattern序列中的内容
                        # 因为path函数和re_path函数就是返回一个URLResolver的实例，并且添加到urlpattern的序列中
                        return URLResolver(RegexPattern(r'^/'), urlconf)
                    ```

                + 到目前为止我们已经获取到了路由解析器，下面就要执行resolver.resolve(request.path_info)方法来进行路由匹配了，让我们看看内部大概做了什么

                    ```python
                        def resolve(self, path):
                            path = str(path)
                            tried = []
                            match = self.pattern.match(path)  # 进行乐友匹配
                            if match:
                              	# 获取匹配到的路径、位置参数和关键字参数
                                new_path, args, kwargs = match
                                for pattern in self.url_patterns:
                                    try:
                                      	# 子路由的匹配,匹配不到引发404异常
                                        sub_match = pattern.resolve(new_path)
                                    except Resolver404 as e:
                                        sub_tried = e.args[0].get('tried')
                            				...
                    ```

                    

                    

            