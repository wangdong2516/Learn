<center><h1>Supervisor使用

> 安装

1. 推荐使用pip安装

```shell
pip install supervisor
```

> 创建配置文件

1. Supervisor安装完成后，运行 `echo_supervisord_conf`，会将Supervisor配置文件打印到终端的标准输出。
2. 可以使用重定向命令，将配置文件，重定向到指定的文件中去。

> 运行supervisor

1. 修改配置文件

```shell
启动之前，需要对配置文件的以下几个地方做一些修改

找到inet_http_server，取消掉注释，username和password填写你的用户名和密码

; 开启一个http服务
[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; ip_address:port specifier, *:port for all iface
username=wangdong              ; default is no username (open server)
password=1277431229               ; default is no password (open server)

；包含的配置文件(可以修改，也可以不修改)
[include]
files = /etc/supervisord.d/*.conf
```

2. 指定配置文件的位置

```shell
supervisord -c supervisord.conf
```

> 配置文件

1. 指令

```shell
command: 需要托管给supervisor指定的命令
user:执行命令的用户
autorestart：是否自动重启
autostart：是否随supervisor启动而启动
startretires：启动失败的重试次数，默认是3次
redirect_stderr：把stderr重定向到stdout，默认是false
startsecs：启动后几秒没有异常退出，就算是启动成功
stdout_logfile:日志文件存放的位置
stopasgroup：是否干掉程序的所有进程，包括子进程
killsgroup：是否干掉程序的所有进程，包括子进程
priority：程序启动的优先级，值越大优先
```

示例`使用supervisor管理uwsgi`

`配置文件如下`

```conf
[program:cms_uwsgi]
directory=/Users/wangdong/Desktop/Learn/practice/CMS/deploy/conf/uwsgi
user=wangdong
command=uwsgi --ini uwsgi.ini
priority=1
numprocs=1
autostart=true
autorestart=true
startretries=10
exitcodes=0
stopsignal=KILL
stopwaitsecs=10
redirect_stderr=true
environment=PATH=/Users/wangdong/miniconda3/envs/django/bin/
stdout_logfile=/Users/wangdong/Desktop/Learn/practice/CMS/logs/uwsgi.log
```

`步骤`：

1. 生成supervisor配置文件
2. 修改配置文件
3. 修改uwsgi配置，主要是log日志的位置，可能会和supervisor的log文件的位置冲突
4. 杀掉uwsgi进程
5. 启动supervisor
6. 在web界面查看进程的状态

![image-20201028154141228](/Users/wangdong/Library/Application Support/typora-user-images/image-20201028154141228.png)

![image-20201028154201722](/Users/wangdong/Library/Application Support/typora-user-images/image-20201028154201722.png)

示例`supervisor管理nginx进程`

配置文件

```conf
[program:cms_nginx]
command=nginx
user=wangdong
priority=999
autostart=true
autorestart=true
# 当开启多个进程的时候，必须指定process_name
process_name=%(program_name)s_%(process_num)02d
numprocs=5
startretries=20
exitcodes=0
stopsignal=KILL
stopwaitsecs=10
redirect_stderr=true
```

![image-20201028162141952](/Users/wangdong/Library/Application Support/typora-user-images/image-20201028162141952.png)