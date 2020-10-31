<center><h1> K8s
</center>

> 发展经历

1. IAAS：基础设施即服务。
2. PAAS:平台即服务。
3. SAAS:软件设施即服务。

>特点

1. 轻量级，消耗的资源小
2. 开源的
3. 弹性伸缩，平缓升级
4. 负载均衡，模块之间的负载均衡。IPVS框架

> 组件说明

1. Borg的架构

    + BorgMaster：负责请求的分发(大脑)，接受请求
    + BorgLet：工作的节点，监听Paxos数据库，看有没有自己的任务。有的话，就执行消费。
    + 访问的方式：
        + 浏览器
        + 命令行
        + 文件的读取
        + 配置文件
    + scheduler: 调度器，任务的调度和分发，不是直接和BorgLet直接交互，而是将分发的数据写入到Paxos(键值对的数据库)中 

2. K8S架构

    ![image-20200910000020755](/Users/wangdong/Library/Application Support/typora-user-images/image-20200910000020755.png)

    + Master
        + scheduler：调度器，分发交给api server，api server将任务写入到etcd。接受任务，选择合适的node节点进行任务的分配。
        + replication controller：控制器，维护副本的数目(期望值),创建或者删除对应的Pod
        + api server:服务访问的入口，基本上所有的服务都要和api server进行交互。
        + etcd：类似于Borg系统的Paxos键值对数据库，也是键值对存储系统。存储K8S集群的所有重要信息，需要持久化的数据。
    + Node
        + kubelet: 跟容器进行交互，操作容器，维持Pod的生命周期。管理容器的生命周期。
        + Kube proxy:负责均衡，访问。操作防火墙实现Pod映射。负责写入规则到IPTABLES或者是IPVS实现服务映射访问。
        + container: 容器

> 其他插件说明

1. CoreDNS：可以为集群中的SVC创建一个域名IP的对应关系解析。
2. Dashboard:给K8S集群提供一个B/S结构的访问体系。
3. Ingress Controller:官方实现了四层代理，Ingress实现了七层代理
4. Fedetation:提供一个可以跨集群中心多K8S统一管理的功能。
5. prometheus：提供一个K8S集群的监控能力
6. Elk：提供K8S集群日志的统一分析和接入的平台。

> Pod的概念

1. 自主式Pod：不是被控制器管理的Pod，一旦死亡，不会被拉起。

2. Pod是K8S中能够被运行的最小的逻辑单元，也叫做原子单元。

3. 一个Pod中可以运行多个容器，它们之间共享UTS+NET+IPC名称空间，只隔离了Mount、Pid和User。

4. 可以将Pod理解为豌豆荚，每个Pod中的容器理解为豌豆。

5. `Pod控制器`： 是Pod启动的一种模板，用来保证在K8S里启动的Pod始终按照人们的预期来运行。

    K8S中提供了很多的Pod控制器，常用的有下面的几种：

    `Depolyment`:部署

    `DaemonSet`：

    `ReplicaSet`：不直接对外提供服务，

    `StatefulSet:`管理有状态的应用

    `Job`: 管理任务

    `Cronjob`:管理定时周期任务

6. 一个Pod中运行多个容器，叫做边车模式。

7. 控制器管理的Pod:

    只要有Pod运行，就会有pause容器被运行，共用pause的网络栈和存储卷

8. RC控制器:用来确保容器应用的副本数始终保持在用户定义的副本数，保持一个期望值， 即如果有容器异常退出，会自动创建新的Pod进行替换，多出来的容器也会被自动回收，在新版本的K8S中建议使用RS控制器取代RC控制器。

9. RS控制器:和RC没有本质的不同，但是支持集合式的selector。

10. Dep控制器：虽然RS可以独立使用，但是建议还是使用Deployment来自动管理RS，不需要担心不兼容的问题。RS不支持关东更新但是Deployment支持。Deployment不直接创建Pod

11. HPA：平滑扩展。

12. statefulset:
     + 稳定的持久化存储，Pod重新调度之后还是能访问到相同的持久化数据
     + 稳定的网络标示，Pod重新调度后PodName和HostName不变，
     + 有序部署：有序扩展，Pod是有顺序的，在部署或者是扩展的时候要根据定义的顺序依次进行
     + 有序收缩和有序删除。

13. DaemonSet:确保全部Node上运行一个Pod的副本，当Node加入集群的时候，也会创建一个新的Pod，当有Node从集群中移除的时候，这些Pod也会被回收，删除DaemonSet监核会删除他创建的所有的Pod。

14. Job：负责批处理的任务，仅执行一次，保证批处理任务的一个或者多个Pod成功结束。

> 服务发现

1. 

> 网络通讯的方式

1. K8S的网络模型假定了所有的Pod都在一个可以直接连通的扁平的网络空间中，这在GCE里面是现成的网络模型，K8S假定这个网络已经存在。
2. 同一个Pod内的多个容器之间：lo
3. 各Pod之间的通讯：Overlay Network
4. Pod于Service之间的通信：各节点的IPTables规则

> Name和Namespace

1. `Name`：由于K8S内部使用`资源`来定义每一种逻辑的概念(功能)，所以每种资源都有自己的名称。

    资源有api版本类别，元数据，定义清单，状态等配置信息。

    名称通常定义在资源的元数据信息中

2. `Namespace`:

    + 随着项目的增多，人员的增加以及集群规模的扩大，需要一种能够隔离K8S内部各种资源的方法，这就是名称空间Namespace。
    + 名称空间可以理解为K8S内部的虚拟集群组。
    + 不同名称空间内的资源，名称可以相同，想通名称空间内的同种资源名称不能相同。
    + 合理的使用K8S的名称空间，可以使得集群管理员能够更好的对交付到K8S里的服务进行分类管理和浏览。
    + K8S里存在的默认名称空间有：default、kube-system、kube-public
    + 查询K8S里特定资源的时候，需要带上相应的名称。 

> Label和Label选择器

1. `Label`：

    + 标签是K8S中特色的管理方式，便于分类管理资源对象。
    + 一个标签可以对应多个资源，一个资源可以有多个标签，它们之间是多对多的关系。
    + 一个资源拥有多个标签，可以实现不同纬度的管理。
    + 标签的组成:key=value
    + 与标签类似的还有注解

2. `Label选择器`：

    + 给资源打上标签之后，可以使用标签选择器过滤指定的标签。

    + 标签选择器目前有两个：基于等值关系（等于、不等于）和基于集合关系（属于、不属于、存在）。

    + 许多资源支持内嵌标签选择器字段

        matchLabels

        matchExpressions

> Service和Ingress

1. `Service`:在K8S 的世界中，虽然每个Pod都会被分配一个单独的IP地址，但是这个IP地址会随着Pod的销毁而消失。
    + Service就是用来解决这个问题的核心概念。
    + 一个Service可以看作一组提供相同服务的Pod的对外访问的接口。
    + Service作用于那些Pod是用过标签选择器定义的
2. `Ingress`
    + Ingress是K8S集群理工作在OSI网络参考模型下，第7层的应用，对外暴露的接口
    + Service只能进行L4流量调度，表现的形式是IP+PORT
    + Ingress可以调度不同的业务域，不同的URL访问路径的业务流量。 

> 核心组件

1. 配置存储中心-> etcd服务，存储集群的元数据信息，key-value数据库

2. 主控(master)节点：

    + Kube-apiserver：提供了集群管理的REST API接口，包括鉴权，数据校验以及集群的状态变更。负责其他模块之间的数据交互，承担通信枢纽的功能。是资源配置控制的入口。提供完备的集群安全机制。大脑🧠

    + Kube-controller-manager服务：由一系列控制器组成，通过apiserver监控整个集群的状态，并且确保集群处于预期的工作状态 

    + Kube-scheduler服务：主要的功能是接受调度的Pod到适合的运算节点上

        预算策略(predict)

        优选策略(prioritiesen)

3. 运算节点：

    + Kube-kubelet服务：简单的说，kubeklet的主要功能就是定时的从某个地方获取节点上的Pod的期望状态（运行什么容器，运行的副本数量、网络或者存储如何配置的等等），并且调用对应的容器平台接口达到这个状态

        定时汇报当前节点的状态给apiserver，以供调度的时候使用。

        镜像和容器的清理工作，保证节点上的镜像不会占满磁盘空间，退出的容器不会占用太多的资源。

    + Kube-proxy服务：是K8S在每个系统上运行网络代理，service资源的载体。

        简历了Pod网络和集群网络的关系

        常用三种流量调度模式

        ​	Usersapce(废弃)

        ​	Iptables(濒临废弃)

        ​	Ipvs推荐

        负责建立和删除包括更新调度规则，通知apiserver自己的更新，或者从apiserver哪里获取其他kube-proxy的调度规则变化来更新自己的

4. CIL客户端

    + kubectl

![image-20200913132557234](/Users/wangdong/Library/Application Support/typora-user-images/image-20200913132557234.png)

> 核心附件

1. CNI网络插件 ---> flannel/calico

2. 服务发现用插件 ---> coredns

3. 服务暴露用插件 ---> traefik

4. GUI管理插件 ---> Dashboard

    