# Docker服务编排

## compose简介

> compose是用于定义和运行多容器Docker应用程序的命令，通过compose，可以使用yml文件来配置应用程序需要的所有的服务，然后，你可以仅仅使用一个命令，来从yml文件配置中创建并且启动所有的服务

> compose使用的三个步骤

1. 使用Dockerfile定义程序的运行环境

	```dock
	# 假设需要制作一个flask应用
	FROM  python:3.7-alpine
	WORKDIR   /code
	ENV FLASK_APP app.ay
	ENV FLASK_RUN_HOST 0.0.0.0
	RUN apk add --no-cache gcc musl-dev linux-headers
	COPY requirements.txt requirements.txt
	RUN pip install -r requiremnets.txt
	COPY . .
	CMD ['flask', 'run']
	```

+ FROM 指定基础镜像是python3.7
+ WORKDIR 设置工作目录为/code
+ ENV 设置需要使用的环境变量
+ RUN 构建过程中安装gcc加速编译
+ COPY 将当前目录下的requirements.txt文件拷贝到/code目录下的requirements.txt
+ RUN 构建过程中安装依赖
+ COPY 将当前目录下的文件复制到/code目录下
+ CMD 容器运行的时候，执行flask run

2. 使用docker-compose.yml定义构成应用程序的服务

	```yml
	version: '1'
	services:
		web:
			build: .
			ports:
				- "5000:5000"
		redis:
			image: "redis-apline"
	```

	+ web 该web服务使用从Dpckerfile目录中构建的镜像，然后暴露5000端口以供访问。
	+ redis 使用redis-apline镜像构建

3. 执行docker-compose up命令来启动并且运行整个应用程序。

	在目录中执行以下命令运行应用程序

	```shell
	docker-compose up
	```

	如果需要后台运行，可以添加-d参数

## yml配置指令

> version

指定compose的版本

> services

指定运行的服务

> build

构建镜像的上下文路径，会自动在该路径下寻找Dockerfile文件,会根据Dockerfile中的定义来构建镜像

或者可以使用指定具体上下文和dockerfile以及args

```yml
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

+ context: dockerfile构建的上下文路径
+ dockerfile: 指定dockerfile文件名
+ args: 添加构建参数，只能在构建过程中被访问和使用

> comamnd

覆盖容器启动时的默认命令

> container_name

指定容器名称而不是使用默认名称

> depends_on

设置依赖关系

> entrypoint

覆盖容器默认的entrypoint

> env_file

从文件中加载环境变量，可以是单个值或者是多个值的列表

> expose

指定需要暴露的端口，但不映射都到宿主机，只是被连接的服务所访问

> environment

添加环境变量，可以使用数组、字典、布尔值，布尔值需要使用引号括起来

> image

指定容器运行的镜像

> volumes

数据卷挂载

> ports

指定端口映射

> links

指定应用程序的链接