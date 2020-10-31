<center> Django中的快捷函数和通用的视图
</center>

1. django中的快捷函数定义在django.shortcuts模块中

2. render(request, template_name, context=None,content_type=None, status=None,using=None)

    1. render()函数用于模板的渲染

    2. 效果和使用loader()函数是一样的。

        ```python
        from django.tempalte import loader
        
        def index(request):
          ....
          template_obj = loader('user/index.html')  # 加载一个模板对象
          context = {'name': 'mmmm'}
          return tempalte_obj.render(context, request)  # 渲染一个模板并且传入上下文
        ```

    3. 接收的参数
        + request：表示当前的请求对象，用于生成响应。
        + template_name: 表示需要渲染的模板的文件名
        + context：用于模板渲染的数据
        + content_type：用于结果文档的MIME类型，默认是text/html
        + status:指定响应对应的状态码，默认是200。
        + using：指定模板引擎

3. redirect(to, *args, permanent=False, **kwargs)

    1. redirect()函数用于重定向。
    2. 默认的redirect函数重定向返回的状态码是302，也就是临时重定向，可以通过指定permanent=True来将其设置为永久重定向。
    3. 使用
        + redirect(obj): 当模型类对象定义了get_absolute_url()方法的时候，使用redirect(obj)将重定向到该方法指定的地址中去[^get]
        + redirect(view_func_name, name=value):直接传入视图函数名和对应的参数(可选)，在内部会调用reverse()进行反解析获取视图函数对应的url。
    4. 接受的参数
        1. to:需要重定向的url
        2. *args，**kwargs用于传递参数给url指定的视图函数

4. get_object_or_404(kclass, *args, **kwargs)

    1. get_object_or_404函数用于替代管理器的get方法

    2. 默认的管理器的get方法在对象不存在或者是存在多个的时候，会引发报错，get_object_or_404方法可以在对象不存在的时候返回404异常，而不是报错。

    3. 使用：

        + get_object_or_404接受的第一个参数可以是模型类，QuerySet、管理器对象或者是关联管理器对象

        + **kwrags是get方法的查询参数，应该指定get方法能能够识别的格式

            ```python
            get_object_or_404(User, first_name__startswith='wang')
            get_object_or_404(User.objects.filter())
            get_object_or_404(User.objects)
            get_object_or_404(heroinfo_set)
            ```

5. get_list_or_404(*klass***,** *args, **kwargs)

    1. get_list_or_404方法获取filter转换为列表的结果，如果列表为空，返回404
    2. 使用：
        + 使用的方式和get_object_or_404是一样的

[^get]:这里简单的说一下get_absolute_url的使用：当你在模型类中定义了这个方法的时候，在django默认的后台管理界面中编辑该对象的时候，会看到一个在站点上查看的标签，点击该标签就会跳到你定义的url地址，并且这也是避免硬编码查看对象的url的一种比较好的解决方式。

6. Django中的视图

    1. 基础视图View

        + 功能

            + 提供了as_view()方法，主要是实例化了我们定义的类视图，并且调用了dispatch方法。

                ```python
                # classonlymethod装饰器是Django基于clasmethod进行封装的装饰器
                # 保证了as_view方法只能被类对象调用而不能被实例对象调用
                @classonlymethod
                def as_view(cls, **initkwargs):
                  # 遍历传入的参数，避免像get这样的参数覆盖请求方式
                  for key in initkwargs:
                    if key in cls.http_method_names:
                      raise TypeError("You tried to pass in the %s method name as a "
                                      "keyword argument to %s(). Don't do that."
                                      % (key, cls.__name__))
                    if not hasattr(cls, key):
                      raise TypeError("%s() received an invalid keyword %r. as_view "
                 "only accepts arguments that are already ""attributes of the class." % (cls.__name__, key))
                
                    def view(request, *args, **kwargs):
                      	# 实例化我们定义的类视图，生成一个对象
                        self = cls(**initkwargs)
                        if hasattr(self, 'get') and not hasattr(self, 'head'):
                          self.head = self.get
                          # 给类视图对象设置了request属性和参数
                          self.setup(request, *args, **kwargs)
                          if not hasattr(self, 'request'):
                            # 如果重定义了setup方法并且没有设置request属性的时候，抛出异常
                            raise AttributeError(
                              "%s instance has no 'request' attribute. Did you override "
                              "setup() and forget to call super()?" % cls.__name__
                            )
                            # 调用dispatch方法，分发请求
                            return self.dispatch(request, *args, **kwargs)
                          # 保存数据给view函数对象
                          view.view_class = cls
                          view.view_initkwargs = initkwargs
                
                          # 从类中获取名称和文档字符串
                          update_wrapper(view, cls, updated=())
                
                          # 设置装饰器的属性
                          update_wrapper(view, cls.dispatch, assigned=())
                    # 返回了view引用，在匹配到对应路由的时候，调用view方法
                    return view
                ```

                

            + 提供了dispatch()方法

                ```python
                def dispatch(self, request, *args, **kwargs):
                	
                  # 判断请求方式的小写形式是否在类属性中允许的请求方式中
                  if request.method.lower() in self.http_method_names:
                    # 获取类中定义的请求方式作为方法名的属性，也就是方法的引用
                    handler = getattr(self,request.method.lower(),self.http_method_not_allowed)
                  else:
                    handler = self.http_method_not_allowed
                  # 调用该方法并且传入request和一些其他的参数
                  return handler(request, *args, **kwargs)
                ```

                

            + 类属性http_method_names中定义了所有请求方式

                ```python
                http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
                ```

    2. 基于模板的视图TemplateView

        + 功能

            + 可以快速的渲染一个给定的模板，模板中将含有url中捕获的参数

                ```python
                 class TemplateView(TemplateResponseMixin, ContextMixin, View):
                 		
                    def get(self, request, *args, **kwargs):
                      	# 获取模板的上下文数据，包含路由匹配中的关键字参数
                        context = self.get_context_data(**kwargs)
                        # 获取指定的template_name并且设置响应
                        return self.render_to_response(context)
                ```

            + 使用

                ```python
                class HomePageView(TemplateView):
                
                    template_name = "home.html"
                
                    def get_context_data(self, **kwargs):
                        context = super().get_context_data(**kwargs)
                        context['latest_articles'] = Article.objects.all()[:5]
                        return context
                ```

    3. 重定向视图

        + 功能

            + 可以快速重定向到指定的url,如果给定的url为None，将返回410

                ```python
                class RedirectView(View):
                    permanent = False  # 是否为永久重定向
                    url = None  # 重定向的url
                    pattern_name = None  # 重定向到的URL模式的名称
                    query_string = False
                
                    def get_redirect_url(self, *args, **kwargs):
                        if self.url:
                            url = self.url % kwargs
                        elif self.pattern_name:
                            url = reverse(self.pattern_name, args=args, kwargs=kwargs)
                        else:
                            return None
                				# 获取请求中的查询字符串
                        args = self.request.META.get('QUERY_STRING', '')
                        # 判断query_string为True的时候，将获取到的查询字符串传递给
                        # 重定向的url，否则查询字符串将被丢弃
                        if args and self.query_string:
                            url = "%s?%s" % (url, args)
                        return url
                
                    def get(self, request, *args, **kwargs):
                        url = self.get_redirect_url(*args, **kwargs)
                        if url:
                          	# 判断是否为永久重定向的响应
                            if self.permanent:
                                return HttpResponsePermanentRedirect(url)
                            else:
                                return HttpResponseRedirect(url)
                        else:
                            logger.warning(
                                'Gone: %s', request.path,
                                extra={'status_code': 410, 'request': request}
                            )
                            return HttpResponseGone()
                ```

            + 使用

                ```python
                class ArticleCounterRedirectView(RedirectView):
                
                    permanent = False
                    query_string = True
                    pattern_name = 'article-detail'
                
                    def get_redirect_url(self, *args, **kwargs):
                        article = get_object_or_404(Article, pk=kwargs['pk'])
                        article.update_counter()
                        return super().get_redirect_url(*args, **kwargs)
                ```

7. 通用显示视图

    1. DetailView详情视图
        + 功能：快速实现一个对象的详情展示
        + 

    