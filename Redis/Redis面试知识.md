<center><h1> Redis面试知识

> Redis集群内置了16384个哈希槽,当需要在redis集群中放置一个key-value的时候,redis先对key使用一个crc16的算法算出一个结果,然后把结果对16384取余,这样每个key都会对应一个编号在0-16384之间的哈希槽,redis会根据节点数量大致 均等的将哈希槽映射到不同的节点



> Redis集群投票机制:	

​	1. redis集群服务器之间通过互相的ping-pong判断是否节点可以连接上,如果有一半以上的节点去ping一个节点的时候没有回应,集群就会认为这个节点宕机了,此时会使用投票机制来确定下一个可用的节点。

 

> Redis数据库的使用：

Redis数据库的优点：	

1. 内存型数据库，基于键值对的形式存储数据

​	2. 读写性能高，具有丰富的数据类型，可以持久化存储

​		持久化存储的方案：快照和AOF文件追加写入

​	3. 运行redis数据库服务器：redis-server

​	4. 运行redis数据库客户端：redis-cli

​	5. 数据的类型以及内部实现的方式:

​		string ---->  int/raw/embstr

​		hash ------->  hash table / zip list1

​		list ------->  zip list / 双向链表

​		set -------->  intset / hash table 

​		zset -------->  zip list / skip list 和 dict的结合

> 数据的操作

数据的增删改查：

1. string(字符串类型)
    + 插入一条数据：set key value
    + 插入多条数据：mset key1 value1 key2 value2 ....
    + 获取一条数据：get key 
    + 获取多条数据：mget key
    + 设置键的有效期： setex key seconds value
    + 给键追加值： append key value，如果key已经存在并且值是字符串，新的值将会是拼接起来的字符串，如果key不存在，将会先创建一个空的字符串在执行拼接操作。
    + 给已经存在的键设置有效期：expire key second
    + 获取字符串的长度： strlen key
    + 获取字符串的切片：getrange key start end
    + 设置字符串的切片：setrange key start value,将从start其实的位置开始覆盖原来的value，覆盖的长度取决于新的字符串的长度。
    + 查看键的类型：type key
    + 查看键的有效期： ttl key
    + 查看键是否存在： exists key
    + 查看所有的键：keys 正则表达式

​		2. hash:

​			1. 插入一条数据： hset key field value

​			2. 插入多条数据： hmset key field1 vlaue1 field2 value2.....

​			3. 获取一条数据：hget key field 

​			4. 获取多条数据：hget key field1 field2 .....

​			5. 获取所有的属性：hkeys key

​			6. 获取所有属性的值：hvals key

​			7. 获取属性和属性值：hgetall key

​		3. list:

​			1. 插入一条数据：lpush key value/rpush key value

​			2. 获取一条数据：lrange key start stop

​			3. 插入一条数据：linsert key before/after oldvalue newvalue 

​			4. 设置指定位置的元素值：lset key index value

​		4. set：

​			1. 插入一条/多条数据：sadd key member/sadd key member1 member2

​			2. 获取一条/多条数据：smembers key

​		5. zset：

​			1. 插入一条数据：zadd key score1 member1 score2 member2

​			2. 获取范围之内的元素：zrange key start stop

​			3. 返回score值在min-max之间的成员：zrangebyscore key min max

​			4. 返回成员对应的权重值：zscore key member

 

​	7. redis的过期策略：

​		1. 定时过期:需要维护一个定时器,成本较大,一般不采用

​		2. 定期过期: 每隔一段时间,随机删除几个在expire字典中的key

​		3. 惰性过期:平时不删除,只有在访问这个键的时候,才会判断是否过期

​		在实际工作中,定期过期和惰性过期是我们经常使用的两种方案

 

​	8. redis的缓存淘汰策略(redis热数据):

​		1. 当内存不足以写入新的数据的时候,从已经设置了有效期的key中选择最近最少使用的key进行删除

​		2. 当内存不足以写入新的数据的时候,从已经设置了有效期的key中随机删除

​		3. 当内存不足以写入新的数据的时候,从已经设置了有效期的key中删除即将过期的key

​		4. 当内存不足以写入新的数据的时候,删除最近最少使用的key

​		5. 当内存不足以写入新的数据的时候,随机删除一个key

​		6. 当内存不足以写入新的数据的时候,报错

 

​	9. redis缓存更新的策略:

​		1. 先更新数据库,在更新缓存: 两个并发的写操作导致脏数据

​		2. 先删除缓存,在更新数据库:两个并发的读写操作导致脏数据

​		3. 先更新数据库,在删除缓存:可行

 

​	10. 缓存雪崩:

​		缓存在同一时间大面积失效，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩溃

​		解决办法:

​			1. 保证redis集群的高可用性,选择适合的内存淘汰机制

​			2. 采用分级存储,不同级别的数据设置不同级别的有效期

​			3. 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。

​			4. 可以通过缓存reload机制，预先去更新缓存，再即将发生大并发访问前手动触发加载缓存

​			5. 不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀

​			6. 做二级缓存，或者双缓存策略。

 

​	11. 缓存穿透:

​		黑客故意请求缓存中不存在的数据，导致所有请求都落到数据库上，造成数据库短时间内承受大量的请求而崩溃 

​		解决办法:

​			1. 缓存层缓存空值。 

​			–缓存太多空值，占用更多空间。（优化：给个空值过期时间） 

​			–存储层更新代码了，缓存层还是空值。（优化：后台设置时主动删除空值,并缓存把值进去）

​			2、将数据库中所有的查询条件，放到布隆过滤器中。当一个查询请求来临的时候，先经过布隆过滤器进行检查，如果请求存在这个条件中，那么继续执行，如果不在，直接丢弃。

​	12. redis和memcached的区别和比较:

​		1. redis不仅仅支持简单的key:value型数据,同时还提供了list,set,zset,hash等数据结构的存储,mamcache支持简单的数据类型:string

​		2. redis支持数据的备份,主从模式的备份

​		3. redis支持数据的持久化,可以将内存 中的数据保存在磁盘当中,重启的时候可以再次进行加载进行使用,二mamcache把数据全部存储在内存中

​		4. redis的速度比mamcache快

​		5. memcache是多线程非阻塞IO的复用网络模型,redis使用单线程的IO复用模型

​	13. redis中常见的数据结构以及使用场景:

​		1. string:常用命令:set.get.mset.mget.incr.decr

​			应用场景:常规的key-value缓存应用,常规计数:微博数,粉丝数等

​		2. hash:常用命令:hget. hset. hgetall

​			应用场景:存储用户信息,商品信息

​		3. list:常用命令:lpush,rpush,lpop,rpop.lrange

​			应用场景:微博的关注列表,粉丝列表,最先消息排行

​		4.set:常用命令:sadd,spop,smembers,sunion等

​			应用场景:共同关注,共同爱好,

​		5. zset:常用命令:zadd, zrange,zrem.zcard

​			应用场景:直播平台,礼物排行榜,弹幕消息

​	14. redis的并发问题怎么解决:

​		1. redis本事是单线程单进程的模式,采用队列将并发访问变为串行访问,redis本身没有锁的概念

​		2. 从客户端的角度来讲:为了保证每个客户端和redis能够进行正常的通信,可以使用连接池,同时对客户端的读写操作提供锁

​		3. 从服务器的角度,利用setnx实现锁

 

​	15. redis的回收进程是怎么工作的,使用的是什么算法:

​		1. 使用的是LRU算法(最近最久未使用算法):

​			在内存有限的情况下,当内存容量不足的情况下,为了保证程序的正常运行,这时候就不得不淘汰内存中的一些对象,释放这些对象所占用的空间

​		2. 引用计数算法

​	16. redis如何尽可能快的处理数据:

​		1. 使用Luke协议

​		2. 生成redis协议

​		3. redis提供了pipe mode的新模式用于执行大量的数据插入工作

​	17. redis分区的优势,不足,以及分区的类型:

​		1. 分区是分割数据到多个redis实例的处理的过程.每隔实力只保存key的一个子集

​		2. 分区的优势:通过利用多台计算机的内存的和值,允许我们构造更大的数据库,通过多核和多台计算机,允许我们扩展计算能力,通过多台计算机和网络适配器,允许 我们扩展网络带宽

​	18. 分区的不足:

​		1. 涉及多个key的操作是不被支持的,举例来说,当两个映射到不同的redis实例上的时候,不能对这两个set集合进行交集操作,涉及多个key的redis事务不被支持

​		2. 当使用分区的时候数据处理比较复杂

​	19. 分区的类型;

​		1. 范围分区:映射一定范围内的对象到redis实例中去

​		2. 哈希分区:用一个hash函数将key转换成一个数字,对这个整数取模,映射到redis实例中去

​	20.redis持久化数据和缓存怎么做扩容:

​		1. 通过redis集群来实现扩容

​	21.redis常见性能问题和解决方案

​		1. master做好不要做任何持久化工作,如rdb内存快照和AOF日志文件,会阻塞主线程的工作,当快照较大的时候,对性能的影响是很大的,会间断性暂停服务,AOF文件过大会影响 master重启的时候的恢复速度.AOF文件在重写的时候回占用大量的CPU和内存资源导致 服务的短暂暂停

​		2. 如果数据 比较重要,主从同步的策略设置 每秒同步一次

​		3. 为了主从复制的速度和连接的稳定性,master和slav'e最好在同一个局域网内

​		4. 尽量避免在压力很大的主库上增加从库

​		5. 为了master的稳定性,主从复制不要使用图状结构,单向链表的结构更稳定

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 