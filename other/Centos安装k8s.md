<center> <h1> Centos安装k8s
</center>

> 安装kubectl

+ 更换k8s源

```bash
cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```

+ 查看可安装的版本

```bash
yum list kubectl –showduplicates
```

+ 安装kubectl,这个版本就是上条命令输出的版本

```bash
yum install -y kubectl.x86_64
```

+ 检测kubectl配置

```bash
kuubectl version
```

遇到报错信息

```bash
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

解决办法

```bash

```

> 安装MiniKube

```bash
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.11.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

