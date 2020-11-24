<center><h1>
  Django内部权限判断的逻辑
</center>

1. 权限判断是根据`PermissionRequiredMixin`扩展类来完成的，需要校验的权限定义在permission_required类属性中，可以是字符串，可以是列表或者是其他的可迭代对象。
2. 在进行路由分发的时候`dispatch`方法会先进行权限判断，没有对应的权限，返回403

```python
def dispatch(self, request, *args, **kwargs):
  if not self.has_permission():
    return self.handle_no_permission()
  return super().dispatch(request, *args, **kwargs)

def has_permission(self):
  
  # 获取需要判断的权限列表
  perms = self.get_permission_required()
  
  # 根据用户对象判断是否有权限
  return self.request.user.has_perms(perms)


def has_perms(self, perm_list, obj=None):
  return all(self.has_perm(perm, obj) for perm in perm_list)


def has_perm(self, perm, obj=None):

  # Active superusers have all permissions.
  if self.is_active and self.is_superuser:
    return True

  # Otherwise we need to check the backends.
  return _user_has_perm(self, perm, obj)
```

3. 在获取权限的时候，调用的是用户对象的`has_prems`方法来进行权限判断的，具体的逻辑是

    + 当用户是超级管理员并且是有效的(is_activate=True),默认是有所有的权限的。
    + 获取权限的时候，调的又是backend(在settings中的配置项AUTHORIZATION_BACKENDS来确定，默认是ModelBackend)的has_perm()方法来判断的。
    + backend的has_perm方法的判断逻辑是1. `当前用户有效`， 2. `有对应的操作权限`

    ```python
    def _user_has_perm(user, perm, obj):
        for backend in auth.get_backends():
            if not hasattr(backend, 'has_perm'):
                continue
            try:
                if backend.has_perm(user, perm, obj):
                    return True
            except PermissionDenied:
                return False
        return False
    ```

    + backend的has_all_permissions的逻辑是 1. `用户有效、不是匿名用户、obj不是None`

```python
def get_all_permissions(self, user_obj, obj=None):
  
  # 匿名用户、无效用户，obj不是None的用户，没有任何权限。
  if not user_obj.is_active or user_obj.is_anonymous or obj is not None:
    return set()
  
  # 判断是否有权限的缓存，没有表示是第一次获取权限
  if not hasattr(user_obj, '_perm_cache'):
    
    # 设置权限缓存
    user_obj._perm_cache = {
      # 获取用户权限(个人)
      *self.get_user_permissions(user_obj),
      # 获取用户所在组的权限(组)
      *self.get_group_permissions(user_obj),
    }
    return user_obj._perm_cache
  
def get_user_permissions(self, user_obj, obj=None):

  return self._get_permissions(user_obj, obj, 'user')


def _get_permissions(self, user_obj, obj, from_name):
  if not user_obj.is_active or user_obj.is_anonymous or obj is not None:
    return set()

  perm_cache_name = '_%s_perm_cache' % from_name
  if not hasattr(user_obj, perm_cache_name):
    if user_obj.is_superuser:
      # 超级管理员默认有全部的权限，查询权限表auth_permission
      perms = Permission.objects.all()
    else:
      
      # 通过反射到_get_%s_permissions方法来完成组权限的获取操作
      perms = getattr(self, '_get_%s_permissions' % from_name)(user_obj)
      
      # 查询权限对应的app_label和权限的名称codename，content_type__app_label是关联查询的语法。
      perms = perms.values_list('content_type__app_label', 'codename').order_by()
      setattr(user_obj, perm_cache_name, {"%s.%s" % (ct, name) for ct, name in perms})
   return getattr(user_obj, perm_cache_name)

def _get_user_permissions(self, user_obj):
  # 获取用户权限
  return user_obj.user_permissions.all()

def _get_group_permissions(self, user_obj):
  # 获取组权限
  user_groups_field = get_user_model()._meta.get_field('groups')
  user_groups_query = 'group__%s' % user_groups_field.related_query_name()
  return Permission.objects.filter(**{user_groups_query: user_obj})
```

总结:整个调用栈

`view(dispatch`) --> `view(has_permission)` --> `user(has_perms)`--> `user(has_perm)` --> `user(_user_has_perm)`  --> `backend(has_perm)` --> `backend(get_all_permissions)` --> `backend(get_user_permissions, get_group_permissions)` --> `backend(_get_permissions)`--> `查询数据库获取权限`

展示权限的可读的名称

codename传的是权限的codename，name传递的是人类可读的名称



> Request.user对象的来源

1. request.user是由`django.contrib.auth.middleware.AuthenticationMiddleware`,这个中间件解析出来的。

```python
class AuthenticationMiddleware(MiddlewareMixin):
    def process_request(self, request):
        assert hasattr(request, 'session'), (
            "The Django authentication middleware requires session middleware "
            "to be installed. Edit your MIDDLEWARE%s setting to insert "
            "'django.contrib.sessions.middleware.SessionMiddleware' before "
            "'django.contrib.auth.middleware.AuthenticationMiddleware'."
        ) % ("_CLASSES" if settings.MIDDLEWARE is None else "")
        
        # get_user会获取用户对象
        request.user = SimpleLazyObject(lambda: get_user(request))

def get_user(request):
    if not hasattr(request, '_cached_user'):
      	# auth是django.contrib.auth包
        request._cached_user = auth.get_user(request)
    return request._cached_user
```

2. 会去调用auth包的get_user方法，然后会调用backend的get_user方法

```python
def get_user(request):
    from .models import AnonymousUser
    user = None
    try:
        user_id = _get_user_session_key(request)
        backend_path = request.session[BACKEND_SESSION_KEY]
    except KeyError:
        pass
    else:
        if backend_path in settings.AUTHENTICATION_BACKENDS:
            backend = load_backend(backend_path)
            user = backend.get_user(user_id)
            # Verify the session
            if hasattr(user, 'get_session_auth_hash'):
                session_hash = request.session.get(HASH_SESSION_KEY)
                session_hash_verified = session_hash and constant_time_compare(
                    session_hash,
                    user.get_session_auth_hash()
                )
                if not session_hash_verified:
                    request.session.flush()
                    user = None

    return user or AnonymousUser()

# backend.py 后端
def get_user(self, user_id):
   try:
      user = UserModel._default_manager.get(pk=user_id)
    except UserModel.DoesNotExist:
       return None
     return user if self.user_can_authenticate(user) else None
```

调用栈:

`request.user` --> `Middleware(process_request)`--> `Middleware(get_user)`--> `auth(get_user)`--> `backend(get_user)`--> `查询数据库得到用户对象`

