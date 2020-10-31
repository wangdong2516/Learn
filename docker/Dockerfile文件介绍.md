# Dockerfile文件介绍

1. 文件内指令

	> FROM     nginx

	指定构建镜像的基础镜像

	> RUN   echo 'test'

	用于在docker bulid 的时候运行一些命令

	后面可以指定两种格式的命令:

	+ Shell格式

		RUN  命令行命令，相当于在shell终端执行的命令

	+ exec格式

		RUN ['echo', 'test'],相当于 RUN echo 'test'

	注意事项⚠️

	​	Dockerfile指定每执行一次就会构建一层镜像，如果指定的命令过多，会构建很多无意义的层，造成镜像庞大，不利于传输。

	​	可以使用&&符号连接命令，可以同时执行多个指令。

	> COPY  [--chown=<user>:<group>]  原路径  目标路径

	[--chown=<user>:<group>]，可选参数，用于改变复制到容器内文件的拥有者和所属的用户组

	目标路径可以不存在，不存在的话，会自动进行目录或者文件的创建。

	> ADD 原路径  目标路径

	ADD和COPY的使用方式是一样的，推荐使用COPY，不同之处在于

	+ ADD会在复制文件为tar压缩文件的时候，并且压缩格式为gzip, bzip2以及xz的情况下，自动复制并且解压到目标路径中去。
	+ ADD的缺点就是在不解压的前提下，无法复制tar压缩文件，从而导致镜像构建缓存失败，构建过程变得缓慢

	> CMD 指令

	CMD命令的作用类似于RUN命令，不同点在于

	+ 两者的执行时间点不同，RUN命令是在docker build的时候执行，CMD命令是在docker run的时候执行。

	作用

	+ 为启动中的容器指定默认要运行的程序，程序运行结束，容器也就结束，CMD命令指定的程序会被docker run的命令行指定的参数锁覆盖。

	注意⚠️：

	​	如果Dockerfile中存在多个CMD指令，仅最后一个会生效。

	> ENTRYPOINT

	类似于CMD命令，但是不会被命令行参数锁覆盖，并且命令行指定的参数会被当作参数传递给ENTRYPOINT指定的程序中去。会被docker run --entrypoint指定的参数覆盖

	注意⚠️

	​	在Dockerfile中存在多个ENTRYPOINT命令，仅最后一个会生效。

	可以搭配CMD命令进行使用，CMD相当于给ENTRYPOINT指定的程序传递参数

	例子:

	```dock
	FROM   nginx
	ENTRYPOINT   ['nginx', '-c']
	CMD.  ['/etc/nginx/nginx.conf']
	```

	不传递参数运行

	```bash
	docker run nginx:test
	
	# 容器内部会运行
	nginx -c /etc/nginx/nginx.conf
	```

	传递参数运行

	```bash
	docker run nginx:test -c /etc/nginx/new.conf
	
	# 在容器内部运行
	nginx -c /etc/nginx/new.conf
	```

	> ENV  key=value key2=value2

	设置一些环境变量，定义好的环境环境，在后面的指令中都可以使用

	> ARG key=value

	构建参数，作用和ENV命令的作用是一致的，但是区别在于

	+ 作用域不一致，ARG设置的环境变量仅在docker build过程中是有效的，在构建好的镜像中是无效的(构建好的镜像中不存在对应的环境变量)

	注意⚠️

	+ 在docker build命令中指定 --build-tag 参数名=值会将ARG中设置的环境变量覆盖

	> VOLUME  宿主机文件目录路径:容器内文件目录路径

	指定需要挂载的数据卷

	作用就是可以使容器内部的数据保存在宿主机上并且可以实现容器和宿主机之间的通信

	注意⚠️

	+ 在docker run命令中指定-v 参数和对应的路径，会将VOLUME中指定的参数覆盖

	> EXPOSE

	声明容器需要暴露的端口，相当于端口映射

	> WORKDIR   工作目录

	指定工作目录，WORKDIR指定的工作目录必须是提前创建好的，并且WORKDIR指定的目录会在构建镜像的每一层中都存在。

	> USER

	用于执行后续命令的用户和用户组，用于切换后续执行命令的用户

	>  HEALTHCHECK

	用于指定某个程序或者指令来监控docker服务的运行状态

	 