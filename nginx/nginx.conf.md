<center><h1>项目使用的Nginx配置文件解读

```shell
# nginx配置文件

# server配置段
server {
		# 设置监听的端口
    listen 8020;
    
    # 虚拟主机的域名
    server_name  localhost;
    
    # 指定客户端请求体的最大容量
    client_max_body_size 100m;
    
    # 指定访问日志文件的位置
    access_log  logs/smb_access.log  upstream;
    
    # 指定网站默认的根目录的位置
    root /home/smb/work/smb_web/dist;

    location ^~ /api/smb {
    		# 禁止所有的proxy_redirect指令，对重定向的url不做任何的修改
        proxy_redirect off;
        
        # 设置请求头并且将头信息传递给服务器端
        proxy_set_header Host $host;
        
        # 设置请求头扩展字段，表示和Nginx建立连接的IP地址
        proxy_set_header X-Real-IP $remote_addr;
        
        # 获取用户真实的IP地址
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # 等待后端服务器发送响应报文的超时时长(最大时间)
        proxy_read_timeout 300;
        
        # 把请求发送给后端服务器的超时时长 
        proxy_send_timeout 300;
				#proxy_http_version 1.1;
				
        proxy_set_header Connection "";
        
        # 客户端断网时，nginx服务器是否中断对被代理服务器的请求。默认为off。 
				proxy_ignore_client_abort on;
				proxy_connect_timeout 300;
				
				#  nginx跟后端服务器连接超时时间(代理连接超时)
        #proxy_pass http://smb_middle_server;
        
        # 指定代理的服务器
        proxy_pass http://10.0.0.207:9667;
    }

    location ^~ /fb_test {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_pass http://smb_middle_server;
        proxy_pass http://10.0.0.207:9667;
    }

    location ^~ /callback_gg {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_pass http://smb_middle_server;
        proxy_pass http://10.0.0.207:9667;
    }

    location ^~ /oe_api {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_pass http://smb_middle_server;
        proxy_pass http://10.0.0.207:9667;
    }

    location ^~ /callback_po {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_pass http://smb_middle_server;
        proxy_pass http://10.0.0.207:9667;
    }

    location ^~ /shopify_api {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://10.0.0.207:9667;
    }

    location / {
        try_files $uri /index.html =404;
    }
}
```

1. 配置段中server_name的作用：

    虚拟服务器的识别路径，不同的域名会通过匹配请求头中的HOST字段，匹配到特定的server配置段中的内容，然后转发到对应的应用服务器上去。如果都不匹配，优先选择listen配置项后有default或者是default_server的server配置段，找到匹配的第一个。

2. proxy_redirect配置项的作用：

    在Nginx作为代理服务器的时候，当web服务器返回的状态码是301或者是302重定向之类的指令的时候，proxy_redirect会修改url中location字段或者是refresh字段的值

3. Proxy-ser_header配置项的作用：

    主要是进行一些反向代理方面的配置，允许重新定义或者是添加发送给后端服务器的请求头，当且仅当当前的配置段中没有指定proxy_set_header配置项的时候，会从上级继承配置。

    ```shell
    语法:
    	proxy_set_header field value
    默认值：
    	proxy_set_header Host $proxy_host
    	proxy_set_header Connection close;
    	
    具体介绍：
    # 保持原来请求头中的Host字段不改变，如果客户端请求没有传递该头部，转发给后端服务器的时候，也不会包含该头部
    proxy_set_header   Host  $http_host;
    
    # 使用$host，如果客户端请求没有传递该头部的时候，为虚拟主机的域名，在懈携带了该头部的时候，为Host字段的值
    proxy_set_header   Host  $host；
    
    # 传递给后端服务器的Host字段为Nginx的域名(主机名)
    proxy_set_header   Host  $host；
    
    # 记录建立连接的IP地址，可以是代理服务器的IP，也可以是真实的IP地址(没有经过代理服务器)
    proxy_set_header   X-Real-IP $remote_addr;
    
    # 记录请求的真实IP地址
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
    # 设置转发的地址
    proxy_pass
    ```

> Nginx命令行参数

1. Nginx仅有几个命令行参数，其他的完全使用配置文件来指定。

    ```shell
    -c 指定Nginx运行使用的配置文件，默认是/etc/nginx/nginx.conf
    -t 检查配置文件是否存在错误。并且尝试打开配置文件中引用到的其他的文件。
    -v 显示Nginx的版本号
    -V 显示Nginx的版本号，编译器版本和配置参数。
    ```

2. 可以使用信号系统来控制Nginx的主进程，主进程可以处理的信号如下
    + TREM， INT：快速关闭
    + QUIT：从容关闭,安静模式
    + HUP：重新加载配置，用新的配置开始新的工作进程，从容关闭旧的工作进程。
    + USER1:重新打开日志文件
    + USER2：平滑升级可执行程序
    + WINCH：从容关闭工作进程
3. Nginx的启动、停止、重启命令
    + sudo /usr/local/nginx/nginx   (nginx二进制文件绝对路径，可以根据自己安装路径实际决定)
    + nginx从容停止命令，等所有请求结束后关闭服务ps -ef |grep nginx    kill -QUIT  nginx主进程号
    + nginx 快速停止命令，立刻关闭nginx进程：ps -ef |grep nginx     kill -TERM nginx主进程号

4. 使用信号重新加载配置文件、重新启动Nginx

    当Nginx收到HUP信号的时候，他会尝试先解析配置文件(在重启Nginx之前，最好使用Nginx -t检查一下配置文件的语法是否正确)，成功的话，就使用新的配置文件，之后，Nginx就会运行新的配置文件来创建新的工作进程，来替代原来的工作进程，处理完成客户端的所有请求之后，旧的工作进程就会被关闭，新的工作进程用来替代旧的工作进程。

5. 平滑升级Nginx

    平滑升级Nginx指的是在不中断服务的情况下，使用新的Nginx可执行程序替换旧的可执行程序。

    首先，使用新的可执行程序替换旧的可执行程序，然后发送USER2信号给主进程，主进程会将原来的pid文件重命名为.oldbin，然后执行新的可执行程序，依次启动新的工作进程和主进程。在平滑升级的时候，两个Nginx实例将会被运行，一起处理请求，然后旧的实例会被逐步取代。

> Nginx模块

1. `http模块`

    + alias

        + 语法：alias file_path/driectory_path

        + 默认值:无

        + 上下文: location

        + 作用：这个指定为指定的位置分配了一个路径，看起来和root指令类似，但是文档的根目录路径不改变，只是用于请求的文件系统的路径。

        + 示例

            ```shell
            location  /i/ {
            	alias /spool/w3/images/;
            }
            
            请求/i/top/gif,将会返回文件/spool/w3/images/top.gif
            ```

    + client_body_in_file_only
        + 语法: client_body_in_file_only on/off
        + 默认值:off
        + 上下文:http, server,location
        + 作用: 该指令允许在文件中存储客户端请求主体，如果指令被启用，在请求完成的时候，文件将不会被删除。

> Nginx负载均衡

1. 配置

    ```ngin
    在http块中，配置upstream块，里面配置后台服务器的列表，然后在location配置段中，添加proxy_pass参数，指向upstream块的地址
    
    http {
    	upstream myserver {
    			ip_hash;
    			server 192.168.10.123:8080;
    			server 192.169.10.123:8081;
    	}
    	
    	location {
    		proxy_pass  http://myserver;
    		root /html;
    		index index.html;
    	}
    }
    
    
    负载均衡的策略有以下几种:
    轮询（默认的）
    weight(权重)，默认的权重是1
    ip_hash(基于IP地址的Hash值)，同一个IP地址的hash值是一样的，可以解决session共享的问题
    fair(响应速度)，基于服务器的响应速度，响应速度快的节点会被分配到更多的请求。
    ```

    

