	<center>django-allauth的使用
	  
	</center>

1. Django-allauth是第一个Django的第三方库，它主要可以帮助你解决你的项目需要第三方登录的问题，可以实现快速接入并且是即插即用的，只需要简单的配置即可。

    下面附上django-allauth的官方文档[django-allauth](https://django-allauth.readthedocs.io/en/latest/installation.html)

2. 下面我介绍一下我使用django-allauth的用法

    + 我最近在写一个博客项目，想要支持第三方登录，所以用到了这个库

    + 首先我们需要下载django-allauth

        ```python
        pip install django-allauth
        ```

    + 然后在django配置文件中的INSTALLED_APPS中将其注册为django应用

        ```python
         # allauth启动必须，请检查你是否已经配置
         'django.contrib.auth',
         'django.contrib.messages',
         'django.contrib.sites',
         
         # allatuh应用
         'allauth',
         'allauth.account',
         'allauth.socialaccount',
        
          # 可添加需要的第三方登录,这里是gituhub，weibo以及微信的登录
         'allauth.socialaccount.providers.github',
         'allauth.socialaccount.providers.weibo',
         'allauth.socialaccount.providers.weixin',
        ```

    + 其次需要指定用户验证后端

        ```python
        AUTHENTICATION_BACKENDS = (
            # Django使用，独立于allauth
            'django.contrib.auth.backends.ModelBackend',
        
            # 配置 allauth 独有的认证方法，如 email 登录
            'allauth.account.auth_backends.AuthenticationBackend',
        )
        ```

    + 剩余的配置项可以根据需要添加

        ```python
        # 设置站点
        SITE_ID = 1
        
        # 登录成功后重定向地址
        LOGIN_REDIRECT_URL = '/article/article-list'
        
        # 配置注册成功后需要发送的邮箱
        ACCOUNT_EMAIL_VERIFICATION = os.getenv('IZONE_ACCOUNT_EMAIL_VERIFICATION', 'none') 
        
        # ACCOUNT_AUTHENTICATION_METHOD = 'username_email'的作用是当用户登录时，
        # 既可以使用用户名也可以使用email， 其他可选的值是'username'、'email'
        ACCOUNT_AUTHENTICATION_METHOD = "username_email"
        # 设置用户注册的时候必须填写邮箱地址
        ACCOUNT_EMAIL_REQUIRED = True
        # 登出直接退出，不用确认
        ACCOUNT_LOGOUT_ON_GET = True
        ```

    + 配置登录注册的url(建议在总路由中配置)

        ```python
        urlpatterns = [
            path('admin/', admin.site.urls),
            path('account/', include('allauth.urls')),  # django-allauthurl配置
        ]
        ```

    + 配置成功之后，你将可以免费的获得这些url

        ```python
        urlpatterns = [
            path("signup/", views.signup, name="account_signup"),
            path("login/", views.login, name="account_login"),
            path("logout/", views.logout, name="account_logout"),
            path("password/change/", views.password_change,
                 name="account_change_password"),
            path("password/set/", views.password_set, name="account_set_password"),
            path("inactive/", views.account_inactive, name="account_inactive"),
        
            # E-mail
            path("email/", views.email, name="account_email"),
            path("confirm-email/", views.email_verification_sent,
                 name="account_email_verification_sent"),
            re_path(r"^confirm-email/(?P<key>[-:\w]+)/$", views.confirm_email,
                    name="account_confirm_email"),
        
            # password reset
            path("password/reset/", views.password_reset,
                 name="account_reset_password"),
            path("password/reset/done/", views.password_reset_done,
                 name="account_reset_password_done"),
            re_path(r"^password/reset/key/(?P<uidb36>[0-9A-Za-z]+)-(?P<key>.+)/$",
                    views.password_reset_from_key,
                    name="account_reset_password_from_key"),
            path("password/reset/key/done/", views.password_reset_from_key_done,
                 name="account_reset_password_from_key_done"),
        ]
        ```

    + 我们可以看到，allauth已经帮助我们默认实现好了关于用户登录、注册、忘记密码、重制密码等许多功能，极大的简短了我们的开发时间。

    + 最后一步，就是我们需要执行迁移来创建对应的数据库

        ```python
        python manage.py makemigrations  # 生成迁移文件
        python manage.py migrate  # 指定迁移
        ```

2. django-allauth默认提供的登录，注册等模板是十分简陋的，通过查看源代码，我们可以很轻易的改写我们自己的模板来覆盖原生提供的模板,我已登录为例简单介绍一下。

    ```python
    class LoginView(RedirectAuthenticatedUserMixin,
                    AjaxCapableProcessFormViewMixin,
                    FormView):
        form_class = LoginForm  # 默认使用的登录表单
        template_name = "account/login." + app_settings.TEMPLATE_EXTENSION
        success_url = None
        redirect_field_name = "next"
    
    ```

    我们可以看到，allauth是通过寻找项目模板目录的account目录下的login.html结尾的文件，我们只需要将我们的模板文件放到这个位置，即可覆盖掉原生的模板