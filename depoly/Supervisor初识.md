<center><h1>Supervisor初识

> 架构设计

1. Supervisor采用的是C/S(客户端/服务器)架构，允许用户控制操作系统的进程。
2. Supervisor将进程作为其子进程来进行启动，并且可以配置为在崩溃的时候，自动进行重启。也可以将其配置为自动调用启动进程。
3. Supervisor将进程作为其子进程启动，就可以很方便的知道子进程的pid是多少和子进程的上下文信息。
4. Supervisor允许对进程分配优先级，并且允许用户通过客户端发出命令。将会按照预先分配的优先级去启动这些进程。
5. Supervisor是使用Python编写的，使用简单的INI格式的配置文件就可以很方便的进行启动和管理。

> Supervisor中几个主要的组件

1. `Supervisord`负责在自己的调用中启动子进程，响应来自客户端的命令，重新启动奔溃的进程，记录进程的输入和输出，管理子进程的生命周期。

    使用/etc/supervisord.conf配置文件来进行启动，需要使用适当的权限保证配置文件的安全，因为其中可能包含未加密的用户名和密码。

2. `supervisorctl`：命令行客户端，可以使用命令行客户端连接`Supervisord`进程，获取子进程的状态信息。启动、停止、以及获取正在运行的所有的子进程。

    客户端通常使用和服务端一致的配置文件，但是任何带有[supervisorctl]的配置文件都是能够工作的。

3. `XML-RPC Interface`：给web程序提供服务的XML接口程序。

> 安装(以Centos为例)

1. `安装命令`：

```python
pip install supervisor
```

> 创建配置文件

1. `安装supervisor完成之后，运行echo_supervisord_conf将会打印出sample样例的配置文件`
2. `创建初始化的配置文件`：执行 echo_supervisord_conf > /etc/supervisord.conf(需要root权限)

3. 如果不想配置文件创建在/etc/supervisord.conf中的话，可以将文件创建在任意目录，然后使用-c参数指定配置文件的位置来启动supervisor。

4. supervisor默认搜索当前文件夹下的supervisord.conf配置文件来启动supervisor，使用-c参数指定之后，就会搜索指定位置的配置文件了。

> 启动supervisor

1. 命令:

```python
supervisord -c supervisord.conf
```

`默认的配置文件的搜索顺序`:

```python
$CWD/supervisord.conf
$CWD/etc/supervisord.conf
/etc/supervisord.conf
/etc/supervisor/supervisord.conf (since Supervisor 3.3.0)
../etc/supervisord.conf (Relative to the executable)
../supervisord.conf (Relative to the executable)

# CWD表示运行supervisor程序所在的目录，可以看到当前运行程序的目录下的配置文件的优先级是最高的，所以会覆盖其他地方的配置文件内容。
```

2. 在启动之后，supervisor并不会管理任何的项目或者是进程，如果你希望supervisor管理你的进程，你必须将你的项目添加到supervisor的配置文件中才能生效。

3. 简单的示例

```python
[program:foo] 
command=/usr/bin/cat
```

需要将上面的内容添加到配置文件中supervisord.conf，重启supervisor才能生效。

4. `查看当前管理的所有的进程`:  supervisorctl,进入supervisor客户端程序，将会列出当前管理的所有进程

![image-20200822132538489](/Users/wangdong/Desktop/bizhi/image-20200822132538489.png)

5. 在前台启动supervisord，supervisord -n
6. 在命令行传递的参数将会覆盖在配置文件中指定的参数。

> 配置文件参数说明

1. -c(--configuration=FILE):指定配置文件的位置，需要指定一个绝对路径，或者依靠默认的搜索路径。
2. -n(--nodaemon):指定supervisor在前台运行。
3. -s(--silent):以安静模式启动supervisor，不输出在终端。
4. -h(--help)：显示帮助信息。
5. -u(--user)：指定以哪个用户的身份运行supervisor，可以指定username或者是user_id。
6. -m(--umask)：八进制数，指定应该被使用的mask。
7. -d(--directory)：当supervisor以守护进程的方式运行的时候，会cd到该目录下。
8. -l(--logfile):指定日志文件的路径。
9. -y(--logfile_maxbytes)：指定日志文件的最大大小。
10. -z(--logfile_backups)：指定日志文件的备份数量。
11. -e(--loglevel)：指定日志文件记录日志的等级。
12. -j(--pidfile)：指定pidfile文件的路径。
13. -i(--identifier):由各种客户端ui为这个监控器实例公开的任意字符串标识符。
14. -q(--childlogdir):子进程日志记录的目录路径(必须已经存在)
15. -k(--nocleanup):指定supervisor在启动的时候是否需要删除旧的日志信息。
16. -a(--minfds):在启动之前日志文件描述符的最小数量。
17. -t(--strip_ansi):从所有子日志进程中去掉ANSI转义序列
18. -v(--version)：指定输出的supervisor的版本信息。

> supervisorctl的运行

1. 要想运行supervisorctl，直接输入`supervisorctl`该命令即可。
2. 可以控制当前由supervisor管理的进程。
3. 如果命令行中指定了参数，将不会调用shell。例如`supervisorctl stop all`
4. `supervisorctl status all`：查看进程的状态信息，正常运行返回0，非正常运行返回非零值。

> supervisor命令行参数选项

1. -c(--configuration):指定配置文件的位置。默认是/etc/supervisor.conf
2. -h(--help):显示帮助信息。
3. -i(--interactive):开启一个交互式的shell环境。
4. -s(--serverurl)：指定supervisor监听的url，默认是9001
5. -u(--username):指定用户名，用于身份验证。
6. -p(--password):指定密码，用于身份验证
7. -r(--history):指定历史日志文件。

>信号

1. 在supervisor运行的时候，将会发送一些信号，用于执行其他的操作。

> 配置文件

conf文件包含一个名为[unix http server]的部分，在该部分中应该插入侦听unix域套接字的http服务器的配置参数。如果配置文件没有[unix http server]节，则不会启动unix域套接字http服务器。允许的配置值如下所示

1. `[unix_http_server]` file

    echo_supervisord_conf输出的示例配置使用/tmp/supervisor.sock作为套接字文件。 该路径仅是示例，可能需要将其更改为更适合您的系统的位置。 某些系统会定期删除/ tmp中的旧文件。 如果套接字文件被删除，supervisorctl将无法连接到supervisor。

2. `chmod`：设置socket权限。

3. `chown`:设置执行supervisor的用户和用户组。

4. `username`：指定用于身份验证的用户和用户名

5. `password`：指定用于身份验证的密码

```shell
[unix_http_server]
file = /tmp/supervisor.sock
chmod = 0777
chown= nobody:nogroup
username = user
password = 123
```

2. `[inet_http_server]`

    supervisord.conf文件包含名为[inet_http_server]的部分，在该部分下应插入侦听TCP（互联网）套接字的HTTP服务器的配置参数。 如果配置文件没有[inet_http_server]节，则不会启动inet HTTP服务器。 允许的配置值如下。

    + port:指定运行服务的端口，host:port的形式。
    + username:指定用于身份验证的用户名
    + password:指定用于身份验证的密码

```shell
[inet_http_server]
port = 127.0.0.1:9001
username = user
password = 123
```

3. `[supervisord]`

    administratord.conf文件包含名为[supervisord]的部分，应在其中插入与受监管流程相关的全局设置。 这些如下

    + logfile：指定日志文件的路径如果将日志文件设置为不可搜索的特殊文件（如/ dev / stdout），则必须通过设置logfile_maxbytes = 0来禁用日志轮换。
    + logfile_maxbytes：指定日志文件的最大字节数。
    + logfile_backups：指定日志文件备份的最大数量
    + loglevel:指定记录的日志文件的等级，指定等级以及以上等级的日志信息会被记录，等于该等级的日志文件将不会被记录。
    + Pidfile:指定pid文件的存储位置。
    + umask;指定supervisor的umask
    + nodaemon:如果设置为true，supervisor将会在前台进行运行。
    + silent如果设置为true，将不会输出任何信息。
    + minfds：指定最小的文件描述符的数量
    + minprocs：指定最少进程的数量。
    + nocleanup：防止在启动时清除任何现有的自动子日志文件。用于调试。
    + childlogdir：子进程日志文件存储的目录位置
    + user:在进行任何有意义的处理之前，指示supervisord将用户切换到这个UNIX用户帐户。只有在以根用户的身份启动时才能切换该用户。
    + directory:当监督守护进程运行时，切换到这个目录
    + strip_ansi:从子日志文件中删除所有ANSI转义序列。
    + environment:指定运行的环境变量
    + identifier:RPC接口使用的这个监督进程的标识符字符串。

```shell
[supervisord]
logfile = /tmp/supervisord.log
logfile_maxbytes = 50MB
logfile_backups=10
loglevel = info
pidfile = /tmp/supervisord.pid
nodaemon = false
minfds = 1024
minprocs = 200
umask = 022
user = chrism
identifier = supervisor
directory = /tmp
nocleanup = true
childlogdir = /tmp
strip_ansi = false
environment = KEY1="value1",KEY2="value2"
```

4. `[supervisorctl]`

    配置文件可能包含对supervise orctl交互式shell程序的设置

    + serverurl:用于访问受管服务器的URL,默认是localhost:9001
    + username:用于身份验证的用户名
    + password:用于身份验证的密码
    + prompt：指定supervisorctl的提示符
    + history_file:指定持久化历史命令文件的路径。

```shell
[supervisorctl]
serverurl = unix:///tmp/supervisor.sock
username = chris
password = 123
prompt = mysupervisor
```

5. ``[program:x]``

    配置文件必须包含一个或多个程序部分，以便管理员知道应该启动和控制哪些程序。 标头值是复合值。 它是“程序”一词，紧接着是冒号，然后是程序名称。

    标头值[program：foo]描述名称为“ foo”的程序。 该名称用于控制该配置所创建的进程的客户端应用程序中

    