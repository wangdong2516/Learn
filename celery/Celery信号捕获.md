	<center><h1>Celery信号捕获
	  
	</center>

> 信号的种类

1. 在Celery中，信号的种类可以分为以下的7种类型，分别是：
    + 任务信号
    + 应用信号
    + 工人信号
    + 节拍信号
    + 事件信号
    + 记录信号
    + 命令信号

> 信号的作用

1. 当某些动作在应用程序中的其他位置发生的时候，信号可以解耦应用程序和通知之间的关联性，并且可以增加某些特殊的操作行为。

> 任务信号

1. `before_task_publish`：在任务发布之前触发，在发送任务的过程中执行的

    ```python
    before_task_publish(sender, body, exchange, routing_key, headers, properties, declare, retry_policy)
    ```

    参数介绍：

    ​	sender:执行任务的对象

    ​	body:消息的正文

    ​	exchange:要发送的交换的名称或者是Exchange对象

    ​	routing_key:发送消息的时候，需要使用的路由密钥。

    ​	headers:应用程序标头映射

    ​	properties:消息的属性

    ​	declare:实体列表，在发布消息之前声明，可以修改

    ​	retry_policy:重试选项的映射，可以是任何参数。

2. `after_task_publish`：

    ```python
    after_task_publish(sender, )
    ```

    参数介绍：

    ​	sender:执行任务的对象

    ​	body:消息的正文

    ​	exchange:要发送的交换的名称或者是Exchange对象

    ​	routing_key:发送消息的时候没，需要使用的路由密钥。

3. `task_prerun`:

    ```python
    task_prerun(sender, task_id, args, kwargs)
    ```

    参数介绍:

    ​	sender：正在执行的任务对象

    ​	task_id；任务id

    ​	args:执行任务的位置参数

    ​	kwargs：执行任务的关键字参数。

4. `task_postrun`：执行任务之后调度，发件人是执行的任务对象

    ```python
    task_posturn(sender, task_id, args, kwargs, retval, state)
    ```

    参数介绍:

    ​	sender：执行的任务对象

    ​	task_id：任务id

    ​	args:执行任务的位置参数

    ​	kwargs：执行任务的关键字参数

    ​	retval:任务的返回值

    ​	state:结果状态的名称

    示例代码

    ```python
    @task_postrun.connect
    def task_execute(sender=None, task_id=None, args=None, retval=None, state=None, **kwargs):
        """celery执行的任务信号，会在任务执行之后进行调度和信号的发送"""
        # 任务的执行者
        print(sender)
        # 任务id
        print(task_id)
        #
        print(args)
        print(kwargs)
        print(retval)
        print(state)
    
    ```

    执行结果

    ```python
    [2020-08-16 20:16:55,112: WARNING/ForkPoolWorker-8] <@task: api.task.add of task at 0x7fdd9c2336d0>
    [2020-08-16 20:16:55,113: WARNING/ForkPoolWorker-8] f2a8c498-3578-4ae2-ab14-167c5b1a1b72
    [2020-08-16 20:16:55,113: WARNING/ForkPoolWorker-8] [1, 2]
    [2020-08-16 20:16:55,113: WARNING/ForkPoolWorker-8] {'signal': <Signal: task_postrun providing_args={'task_id', 'args', 'task', 'retval', 'kwargs'}>, 'task': <@task: api.task.add of task at 0x7fdd9c2336d0>, 'kwargs': {}}
    [2020-08-16 20:16:55,113: WARNING/ForkPoolWorker-8] 3
    [2020-08-16 20:16:55,114: WARNING/ForkPoolWorker-8] SUCCESS
    ```

5. `task_retry`:在任务进行重试的时候调度

    ```python
    task_retry(sender, request, reason, einfo)
    ```

    参数介绍:

    ​	sender:执行的任务对象

    ​	request:当前的任务请求

    ​	reason：重试的原因

    ​	einfo:详细的错误信息

6. `task_success`:任务执行成功的时候触发

    ```python
    def task_success(result = None, *args, **kwargs)
    ```

    参数介绍:

    ​	result:任务的返回值

    示例代码

    ```python
    @task_success.connect
    def task_success(sender, result = None, *args, **kwargs):
        """celery执行的任务信号，会在任务执行成功之后触发信号的发送"""
        print(sender)
        print(f"当前的任务返回的结果是{result}")
        print(*args)
        print(kwargs)
    ```

    执行结果:

    ```python
    [2020-08-16 20:25:55,607: WARNING/ForkPoolWorker-8] <@task: api.task.add of task at 0x7fabb83f36d0>
    [2020-08-16 20:25:55,608: WARNING/ForkPoolWorker-8] 当前的任务返回的结果是3
    [2020-08-16 20:25:55,608: WARNING/ForkPoolWorker-8] {'signal': <Signal: task_success providing_args={'result'}>}
    ```

7. `task_failure`：任务失败的时候触发

    ```python
    task_failure(sender, task_id, exception, args, kwargs, traceback, einfo)
    ```

    参数介绍:

    ​	sender;任务执行的对象

    ​	task_id：任务id

    ​	exception：引发异常的实例

    ​	args:任务调用的位置参数

    ​	kwargs:任务调用的关键字参数

    ​	traceback:堆栈跟踪对象

    ​	info：错误信息

    示例代码

    ```python
    @task_failure.connect
    def task_failure(sender, task_id, exception, traceback, einfo, args, **kwargs):
        """任务执行失败之后发送的信号"""
        print(sender)
        print(task_id)
        print(exception)
        print(traceback)
        print(einfo)
        print(args)
        print(kwargs)
    ```

    执行结果

    ```python
    [2020-08-16 20:41:58,857: WARNING/ForkPoolWorker-8] <@task: api.task.add of task at 0x7fd3f02f2790>
    [2020-08-16 20:41:58,857: WARNING/ForkPoolWorker-8] b0ddfeb7-a554-42a4-9832-340eb15f109a
    [2020-08-16 20:41:58,857: WARNING/ForkPoolWorker-8] division by zero
    [2020-08-16 20:41:58,857: WARNING/ForkPoolWorker-8] <traceback object at 0x7fd3f06431e0>
    [2020-08-16 20:41:58,858: WARNING/ForkPoolWorker-8] Traceback (most recent call last):
      File "/Users/wangdong/Library/Caches/pypoetry/virtualenvs/api-1YLpJCrU-py3.7/lib/python3.7/site-packages/celery/app/trace.py", line 412, in trace_task
        R = retval = fun(*args, **kwargs)
      File "/Users/wangdong/Library/Caches/pypoetry/virtualenvs/api-1YLpJCrU-py3.7/lib/python3.7/site-packages/celery/app/trace.py", line 704, in __protected_call__
        return self.run(*args, **kwargs)
      File "/Users/wangdong/Desktop/i_api/api/api/task.py", line 10, in add
        return x + y / 0
    ZeroDivisionError: division by zero
    [2020-08-16 20:41:58,858: WARNING/ForkPoolWorker-8] [1, 2]
    [2020-08-16 20:41:58,858: WARNING/ForkPoolWorker-8] {'signal': <Signal: task_failure providing_args={'exception', 'args', 'traceback', 'einfo', 'task_id', 'kwargs'}>, 'kwargs': {}}
    ```

8. `task_internal_error`：在内部发生错误的时候调度

    ```python
    task_interna_error(sender, task_id, args, kwargs, request, exception, traceback, einfo)
    ```

    参数介绍:

    ​	sender：任务的执行者

    ​	task_id:任务id

    ​	args:调用任务的位置参数

    ​	kwargs：调用任务的关键字参数

    ​	request：原始请求字典

    ​	exception:触发的异常

    ​	traceback:堆栈跟踪对象

    ​	einfo:错误信息

9. `task_received`:从代理接受到任务准备执行的时候调度

    ```python
    task_received(sender, request)
    ```

    参数介绍:

    ​	sender：执行的任务对象

    ​	request:Request的实例对象

10. `task_revoked`：任务取消或者是终止的时候调度

    ```python
    task_revoked(sender, request。 terminated, signum, expired)
    ```

    参数介绍:

    ​	sender：执行的任务对象

    ​	request:Request的实例对象

    ​	terminated：任务是否已经终止

    ​	signum:终止任务的信号

    ​	expired：任务是否过期

11. `task_unknown`：收到未注册的任务的时候触发

    ```python
    task_unknown(sender, name, id, message, exc)
    ```

    参数介绍:

    ​	sender：执行任务的对象

    ​	name:任务名称

    ​	id：任务id

    ​	message：任务的原始消息

    ​	exc：发生的错误

12. `task_rejected`：收到未知类型的任务的时候触发

    ```python
    task_rejected(sender, message, exc)
    ```

    参数介绍:

    ​	sender：执行任务对象

    ​	message：原始消息

    ​	exc:错误信息

> 应用信号

`import_modules`：当导入include或者是import设置中的模块的时候，将发送此信号

```python
import_modules(sender)
```

参数介绍:

​	sender:执行的任务对象

> 工人信号

`celeryd_after_setup`：在设置工作程序实例但是在调用运行之前发送的信号，这通常意味着已启用了该选项中的所有队列，已经设置了日志记录等



> 节拍信号

(详见Celery官方文档)[https://docs.celeryproject.org/en/stable/userguide/signals.html#task-success]

> 事件信号

(详见Celery官方文档)[https://docs.celeryproject.org/en/stable/userguide/signals.html#task-success]

> 记录信号

(详见Celery官方文档)[https://docs.celeryproject.org/en/stable/userguide/signals.html#task-success]

> 命令信号

(详见Celery官方文档)[https://docs.celeryproject.org/en/stable/userguide/signals.html#task-success]