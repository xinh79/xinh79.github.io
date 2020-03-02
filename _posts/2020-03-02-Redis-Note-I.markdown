---
layout:      post
title:       "Redis基础知识-I"
subtitle:    "Redis"
author:      "Ashior"
header-img:  "img/ReadNotes/bg-C++ Primer.jpg"
catalog:     true
tags:
  - Redis
  - 工作
---

> 大体介绍Redis的基本操作与信息，有时间再去看书籍并更新博客。

----

## Redis基础

#### 前言

session存在哪里？

1. 存在cookie里：不安全、网络负担效率低
2. 存在文件服务器或者数据库里：大量的IO效率问题
3. session复制：session数据冗余、节点越多浪费越大
4. 缓存数据库：完全在内存中，速度快，数据结构简单

客户端发送请求到服务器，服务器先访问缓冲数据库，如果不命中再去访问数据库，更新缓存数据库，减少IO操作。

通过破坏一定的业务逻辑来换取性能。水平切分(一张表分成几张)、垂直切分(字段过多，分成多段存储)、读写分离(将读、写功能分离，能读的只能读，能写的能读能写)。

不支持ACID：

原子性(Atomicity):原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
一致性(Consistency):事务前后数据的完整性必须保持一致。
隔离性(Isolation)：事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
持久性(Durability)：持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

脏读：指一个事物读取了另外一个事务未提交的数据。

不可重复读：在一个事务内读取表中的某一行数据，多次读取结果不同。

幻读：是指一个事务内读取到了别人事务插入的数据，导致前后读取不一致。

NoSQL适用场景

- 对数据高并发的读写
- 海量数据的读书
- 对数据高可扩展性的

NoSQL不适用场景

- 需要事务支持
- 基于SQL的结构化查询存储，处理复杂的关系，需要即席查询(条件查询)。

由于拥有持久化能力，利用其多样的数据结构存储特定的数据。

- 最新N个数据：通过list实现按自然时间排序的数据
- 排行榜，top N：利用zset有序集合
- 时效性的数据，比如手机验证码：expire过期
- 计数器，秒杀：原子性，自增方法INCR、DECR
- 去除大量数据中的重复数据：利用set集合
- 构建队列：利用list集合
- 发布订阅消息系统：pub/sub模式

#### Redis入门

启动准备：

- 备份redis.conf：拷贝一份redis.conf到其他目录
- 修改redis.conf文件，将里面的daemonize no改为yes，设置为后台启动
- 启动命令：执行`redis-server /opt/my-redis/redis.conf`
- 启动用户：`redis-cli -h 127.0.0.1 -p 6379`
- 关闭服务：`redis-cli shutdown`
- 提示出错： `(error) ERR Errors trying to SHUTDOWN. Check logs.`

错误修正办法：[CSDN博客](https://blog.csdn.net/github_33809414/article/details/82531642) 需要注意的是，当设置完成之后需要使用kill命令杀死redis服务，之后再重新启动。不然他没有加载最新的配置。

Redis默认16个数据库，从0号位置开始使用，通过`select n`命令选择指定的库。

Redis是单线程+多路I/O复用技术

多路复用是指使用一个线程来检查多个文件描述符(socket)的就绪状态，比如调用select/epoll函数，传入多个文件描述符，如果一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行(比如线程池)。

串行 vs 多线程+锁 vs 单线程+多路I/O复用

**数据类型：**

key：string、set、list、hash、zset

+ `keys *`：查询当前所有的键值
+ `exists key`：判断某个键值是否存在
+ `type key`：查看某个键值的类型
+ `del key`：删除某个键值
+ `expire key second`：为键值key设置过期时间
+ `ttl key`：查看键值的还剩多少时间过期(-1，永不过期，-2，已过期)
+ `dbsize`：查看当前数据库的key的数量
+ `Flushdb`：清空当前库
+ `Flushall`：通杀全部库

#### string

string是Redis最基本的类型，一个key对应一个value。string是二进制安全类型。可以包含任何数据，比如jpg图片或者序列化的对象。string类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M。

+ `get key`：查询对应简直
+ `set key value`：添加对应键值对
+ `append key value`：将给定的value追加到原值的末尾
+ `strlen key`：获取值的长度
+ `setnx key value`：只有在key不存在时设置key的值
+ `incr key`：将key中存储的数字值增1，只能对数字值操作，乳沟为空，新增值为1
+ `decr key`：将key中储存的数字值减1，只能对数字值操作，如果为空，新增值为-1
+ `incrby/decrby key 步长`：将key中存储的数字值增减。自定义步长
+ `mset key1 value1 key2 value2`：同时设置一个或多个键值对
+ `mget key1 key2 key3`：同时获取一个或多个value
+ m`setnx key1 value1 key1 value2`：同时设置一个或多个键值对，当且仅当所有的key不存在时。
+ `getrange key 起始位置 结束位置`：获得值的范围
+ `setrange key 起始位置 value`：用value覆写key所存储的字符串值，从起始位置开始。
+ `setex key 过期时间 value`：在创建键值对的时候就设置过期时间。
+ `getset key value`：以新换旧，设置新的值，同时获得旧值。

#### list

单键多值，Redis列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部，或者尾部。它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

+ `lpush/rpush key value1 value2`：从左边右边插入一个或多个值
+ `lpop/rpop key`：从左边/右边吐出一个值，当值被pop完后，对应的key也将不存在
+ `rpoplpush key1 key2`：从key1列表右边吐出一个值，插入到key2的左边
+ `lrange key start stop`：按照索引下标获得元素(从左到右)
+ `lindex key index`：按照索引下标获得元素(从左到右)
+ `llen key`：获得列表长度
+ `linset key before value newvalue`：在value的后面插入newvalue
+ `lrem key n value`：从左删除n个value(从左到右)如果n为负值，则从右到左进行删除操作，删除值为value的数据，如果n为0，则删除所有值为value的数据。

#### set

Redis set对外提供的功能与list类似，是一个列表的功能，特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的set是string类型的无序集合。他的底层其实是一个value为null的hash表，所以添加、删除、查找的复杂度都是O(1)。

+ `sadd key value1 value2`：将一个或多个member元素加入到集合key中，已经存在于集合的member元素将被忽略。
+ `smembers key`：去除该集合的所有值。
+ `sismember key value`：判断集合key是否含有value的值，有返回1，没有返回0。
+ `scard key`：返回该集合的元素个数
+ `srem key value1 value2`：删除集合中某个元素
+ `spop key`：随机从该集合中吐出一个值(随机抽奖)
+ `srandmember key n`：随机从该集合去除n个值，不会从集合中删除
+ `sinter key1 key2`：返回两个集合的交集元素
+ `sunion key1 key2`：返回两个集合的并集元素
+ `sdiff key1 key2`：返回两个集合的差集元素

#### hash

Redis hash是一个键值对集合，Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。类似于map。

+ `hset key field value`：给key集合中的field键赋值value
+ `hget key1 field`：从key集合中取出value
+ `hmset key1 field1 value1 field2 value2`：批量设置hash的值
+ `hexists key field`：查看哈希表key中，给定域field是否存在
+ `hkeys key`：列出该hash集合的所有field
+ `hvals key`：列出该hash集合的所有value
+ `hincrby key field increment`：为哈希表key中的域field的值加上增量increment
+ `hsetnx key field value`：将哈希表key中的域field的值设置为value，当且仅当域field不存在

#### zset

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员关联了一个评分(score)，这个评分被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了。因为元素是有序的，所有你也可以很快的根据评分或者次序来获得一个范围的元素。访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表。

+ `zadd key socre1 value1 socre2 value2`：将一个或者多个member元素及其score值加入到有序集key当中。相同元素不同分数可以添加，相同元素不同分数，更新分数。元素为键值，分数为数值。
+ `zrange key start stop [WITHSCORES]`：返回有序集key中，下标在start和stop之间的元素，带后面的参数可以让分数一起喝值返回到结果集中。
+ `zrangebyscore key min max [WITHSCORES] [limit offset count]`：返回有序集key中，所有score值介于min和max之间(包括等于min或max)的成员。有序集成员按score值递增(从小到大)次序排列。
+ `zrevrangebyscore key max min [WITHSCORES] [limit offset count]`：同上，改为从大到小排列
+ `zincrby key increment value`：为元素的score加上增量
+ `zrem key value`：删除该集合下，指定值的元素
+ `zcount key min max`：统计该集合，分数区间内的元素个数
+ `zrank key value`：返回该值在集合中的排名，从0开始

----

#### Redis相关配置

**IP地址的绑定bind**

默认情况bind=127.0.0.1只能接受本机的访问请求。不写的情况下，无限制接受任何IP地址的访问。生产环境肯定要写你应用服务器的地址。如果开启了protected-mode，那么在没有设定bind IP且没有设密码的情况下，Redis只允许接受本机的响应。

```shell
# @ahior allow othre host access redis
# bind 127.0.0.1
protected-mode no
```

+ **tcp-backlog**： 一个请求到达后，到接受进程处理前的队列。backlog队列总和=未完成三次握手队列+已经完成三次握手队列。高并发环境tcp-backlog设置值跟超时时限内的Redis吞吐量决定。
+ **timeout**： 一个空闲的客户端维持多少秒会关闭，0为永不关闭。
+ **TCP keepalive**： 对访问客户端的一种心跳检测，每个n秒检测一次，官方推荐设为60秒。
+ **daemonize**： 是否为后台进程
+ **pidfile**： 存放pid文件的位置，每个实例会产生一个不同的pid文件
+ **log level**： 四个级别根据使用阶段来选择，生产环境选择notice或者warning
+ **logfile**： 日志文件名称
+ **syslog**： 是否将Redis日志输送到Linux系统日志服务中
+ **syslog-ident**： 日志的标志
+ **syslog-facility**： 输出日志的设备
+ **database**： 设定库的数量，默认16
+ **require**： 设置密码
+ **maxclient**： 最大客户端连接数
+ **maxmemory**： 设置Redis可以使用的内存量。一旦到达内存使用上限，Redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。如果Redis无法根据移除规则来移除内存中的数据，或者设置了"不允许移除"，那么Redis则会针对那些需要申请内存的指令返回错误信息，比如SET/LPUSH等。

**maxmemory-policy**

1. volatile-lru：使用LRU(最近最少使用)算法移除key，只对设置了过期时间的键
2. allkeys-lru：使用LRU算法移除key
3. volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
4. allkeys-random：移除随机的key
5. volatile-ttl：即将过期，移除那些TTL值最小的key，即那些最近要过期的key
6. noeviction：不进行移除，针对写操作，只是返回错误信息

**maxmemory-samples**

+ 设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小
+ 一般设置3到7的数字，数值越小样本越不准确，但是性能消耗也越小。

----

## Redis特性

#### Redis事务

Redis事物是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。Redis事务的主要作用就是串联多个命令防止别的命令插队。

从输入multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入exec后，Redis会将之前的命令队列中的命令依次执行。组队的过程中可以通过discard来放弃组队。

可以一次性执行多个指令，而且在执行阶段，不会因为执行出错而停止执行，Redis事务会继续执行，不会回滚。

**multi开始事务，exec执行事务，discard取消事务。**

关系型数据库用悲观锁比较多，Redis里面使用的是乐观锁。悲观锁会对被访问的数据加锁，别人无法访问。乐观锁可以进行访问，但是有一个版本号的概念，每当被访问的数据发生改变，版本号也会发生发生改变。在操作被访问的数据之前，会比较版本号，如果版本号不一致，则请求直接结束。

悲观锁：很悲观，每次都去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就是用到了很多这种锁机制，比如行锁，表锁，读锁，写锁，都是在操作之前先上锁。

乐观锁，就是很乐观，每次去拿数据的时候都认为别人不会改变，所以不会上锁，但是在更新的时候判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

`unwatch/watch key [key]`

在执行multi之前，先执行watch key1，可以监视一个或者多个key，如果在事务执行之前这个key被其他命令所改动，那么事务将被打断。

单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在"事务内的查询要看到事务里的更新，在事务外查询不能看到"，这个让人头痛的问题。

不保真原子性：Redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

秒杀系统：超卖，通过监视watch和事务来解决，在200个并发中只能有1个用户成功请求。库存遗留问题，使用lua脚本(作为嵌入式脚本语言)，每一个用户都会把事务完整的执行一遍，而不会中途退出。连接超时使用连接池完成。

#### Redis持久化

Redis持久化方式(将内存中的数据写入磁盘)：RDB(Redis database)和AOF(Append of file)。

RDB：在指定的时间间隔内将内存中的数据集快照写入磁盘，也是snapshot快照，它恢复时是将快照文件直接读到内存中。

**备份是如何执行的**

Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的数据恢复，且对于数据恢复的完整性不是很敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。(最后一次不满足保存策略的时候，就会丢失，正常关闭的时候会正常写入，如果是直接杀进程则会导致数据丢失)

**RDB方式**

应该修改dbfile文件(dump.rdb)的保存目录，不然默认是的是启动Redis时的目录位置。查找save有默认的保存方式。(save 900 1：表示每900秒进行一个持久化，save 300 10：表示每300秒内，更新了数据10次，就会把数据持久化到dump.rdb中)

`stop-writes-on-bgsave-error yes`：当Redis无法写入磁盘的话，直接关掉Redis的写操作。

`rdbcompression yes`：进行RDB保存时，将文件压缩

`rdbchecksum yes`：在存储快照后，还可以让Redis使用CRC64算法来进行数据校验，但是这样做回增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

**RDB的备份**

将备份文件拷贝一份到其他地方。RDB的恢复：关闭Redis，先把备份的文件拷贝到工作目录，启动Redis，备份数据会直接加载。

**RDB的有点**

节省磁盘空间，恢复速度快(与AOF比较，保存的是指令)

**RDB的缺点**

虽然Redis在fork时使用的写时拷贝技术，但是如果数据庞大时还是比较消耗性能。备份周期间隔一段时间做一次备份，所有如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

**AOF方式**

以日志的形式记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将读写指令从前到后执行一次以完成数据的恢复工作。

AOF方式默认是不开启的，需要将配置文件中的appendonly属性设置为yes才可以，同样可以通过appendfilename设置保存的文件名称，保存路径与RDB方式一致。

如果Redis中AOF和RDB方式同时开启，Redis恢复选择AOF方式。

**AOF文件故障备份**

AOF的备份机制和性能虽然和RDB不同，但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即可加载。AOF和RDB同时开启，系统默认取AOF的数据。

如果遇到AOF文件损坏，可通过`redis-check-aof --fix appendonly.aof`进行恢复。

查看AOF备份文件，命令是以`* $`开始表示一条命令，如果不小心使用了flushdb命令，可以在文件中删除。然后退出客户端模式，再次进入的时候就可以恢复数据了。

**AOF的同步频率**

始终同步，每次Redis的写入丢回立刻记录日志。每秒同步，每秒记录日志一次，如果宕机，本秒的数据可能丢失。配置文件中为：appendfsyn。

rewrite：AOF采用文件追加方式，文件会越来越大，避免出现此情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会自动启动AOF文件内容压缩，只保留可以恢复数据的最小指令值，可以使用命令bgrewriteaof。

**Redis如何实现重写**

AOF文件持续增长而过大时，会fork出一条新的进程来讲文件重写(也是先写零食文件，最后再rename)，遍历新进程的内存中数据，每条记录有一条的set语句。重写AOF文件的操作，并没有读取旧的AOF文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的AOF文件，这点和快照有点类似。

**何时重写**

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担。因此设定Redis要满足一定条件才会进行重写。

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size，如果Redis的AOF当前大小>=base_size+base_size*100%(默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。

**AOF优点**

备份机制更稳健，丢失数据概率低。可读的日志文本，通过操作AOF稳健，可以处理误操作。

**AOF的缺点**

比其RDB占用更多的磁盘空间；恢复备份速度慢；每次读写都同步的话，有一定的性能压力；存在个别BUG，造成恢复不能。

**如何使用**

1. 官方推荐两个都用
2. 如果对数据不敏感，可以选单独用RDB
3. 不建议单独用AOF，因为可能出现BUG
4. 如果只是做纯内存缓存，可以都不用

----

#### Redis主从复制

主从复制就是主机数据更新后根据配置和策略，自动同步到备份机的master/slaver机制，master以写为主，slaver以读为主。从而实现读写分离，性能扩展，容灾快速恢复。

**配从不配主**

- 拷贝多个redis.conf文件include
- 开启daemonize yes
- pid文件名字pidfile
- 指定端口port
- log文件名字
- dump.rdb名字dbfilename
- appendonly关掉或者换名字

info replication：打印主从复制的相关信息

slaveof ip port：成为某个实例的从服务器

需要记住从服务器永远和主服务器的数据一致，如果从服务器down了，那么重新启动后，需要重新设置为某个服务器的从服务器。如果主服务器down了，那么从服务器会原地待命，并且查看信息时显示主服务器down，等待主服务器up。

需要永远设置，需要在配置文件中写入主从关系。

**复制原理**

- 每次从机联通后，都会给主机发送sync指令；
- 主机立刻进行存盘操作，发送RDB文件给从机
- 从机收到RDB文件后，进行全盘加载
- 之后每次主机的写操作，都会立刻发送给从机，从机执行相同的命令

**薪火相传**

上一个slave可以是下一个slave的master，slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master，可以有效减轻master的写压力，去中心化降低风险。用`slaveof ip port`。中途变更转向：会清除之前的数据，重新建立拷贝最新的数据，风险是一旦某个slave宕机，后面的slave都没有办法进行备份。

**slaveof no one**将从机变为主机

**哨兵模式**

反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从从库转换为主库。

- 调整为一主二仆模式
- 自定义的/opt/my-redis/目录下新建sentinel.conf文件
- 在配置文件中填写内容：`sentinel monitor mymaster 127.0.0.1 6379 1`
- 其中mymaster为监控对象起的服务器名称，1为至少有多少个哨兵同意迁移的数量。通常设置一主二从三哨兵，主服务器肯定是需要的，从服务器两个，体现随机上位，三个哨兵进行投票，两票以上才表示主服务器已经down机。

从下线的主服务器的所有从服务里面挑选一个从服务，将其转为主服务。选择条件依次为：选择优先级靠前的；选择偏移量最大的；选择runid最小的从服务。优先级在redis.conf中`slave-priority 100`。偏移量是指获得原主数据最多的每个redis实例启动后都会随机生成一个40位的runid。

挑选出新的主服务之后，sentinel向原服务的从服务器发送slaveof新主服务器的命令，复制新master。

当已下线的服务器重新上线时，sentinel会向其发送slaveof命令，让其成为新主的从服务器。

----

#### Redis集群

Redis集群实现了对Redis的水平扩容，即启动N个Redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis集群通过分区来提供一定程度的可用性，即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求。

文件的基本配置过程：

```shell
#######################################
# 包含默认的配置文件
include /opt/my-redis/redis.conf
# 当Redis以守护进程方式运行时，即使该项没有配置，Redis也会默认把pid写入/var/run/redis.pid文件；而当Redis不是以守护进程凡是运行时，若该项没有配置，则redis不会创建pid文件。创建pid文件是尝试性的动作，即使创建写入失败，redis依然可以启动运行
pidfile "/var/run/redis6379.pid"
# 指定Redis监听端口，默认端口为6379
port 6379
# 绑定的主机地址，如果想全部访问不做限制的话，全为0.0.0.0,
# bind 127.0.0.1
#######################################
# 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 300
# 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
loglevel verbose
# 设置数据库的数量，默认数据库为0
databases 16
# 在后台启动
daemonize yes
# 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
save 900 1
save 30 5
save 60 10000
# 指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb
# 指定本地数据库存放目录
dir "/opt/my-redis"
# 设置当本机为slave服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
slaveof 127.0.0.1 6380
# 从服务器的优先级(越小优先级越高)
slave-priority 80
#######################################
# 打开集群模式
cluster-enabled yes
# 设置节点配置文件名
cluster-config-file nodes-6379.conf
# 设置节点失联时间，超过该时间(毫秒)，集群自动进行主从切换
cluster-node-timeout 15000
```

配置6个节点，合成一个集群，组合之前，请确保所有Redis实例启动后，nodes-xxxx.conf文件都生成正常。

执行命令：

```shell
cd /opt/redis-5.0.7/src
# 一个集群至少要有三个主节点
# --replicas 1表示我们希望为集群中的每个主节点创建一个从节点
# 在后面依次添加IP:PORT的信息
# 分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上
./redis-trib.rb create --replicas 1 IP:PORT [IP:PORT]
```

**插槽**

一个Redis集群包含16384个插槽(hash slot)，数据库中的每个键都属于16384个插槽的其中一个，集群使用公式CRC16(key)%16384来计算键key属于哪个槽，其中CRC16(16)语句用于计算key的CRC16校验和。

集群中的每个节点负责处理一部分插槽。举个例子，如果一个集群可以有主节点，其中：节点A负责处理0号至5500号插槽；节点B负责处理5501号至11000号插槽；节点C负责处理11001号至16383号插槽。

**在集群中录入值**

在redis-cli每次录入，查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应该前往的redis实例地址和端口。

redis-cli客户端提供了-c参数实现自动重定向。如`redis-cli -c -p 6379`登录后，再录入，查询键值对可以自动重定向。

不在一个slot下的键值是不能使用mget、mset等多键操作

可以通过`{}`来定义组的概念，从而使key中`{}`相同的内容的键值对放到一个slot中去。例如：`set a{shior} 1`

**查询集群中的值**

`cluster keyslot key`：计算键值key应该放置在哪个槽上
`cluster countkeysinslot slot`：返回槽slot目前包含的键值对数量
`cluster getkeyinslot slot count`：返回count个slot槽中的键

**故障恢复**

如果主节点下线，从节点自动升为主节点

主节点恢复后，会变成从节点

如果所有某一段插槽的主从节点都down了，redis服务中所对应的插槽无法使用了，会提示cluster is down。

redis.conf中的参数cluster-require-full-coverage。16384个slot都正常的时候才能对外提供服务

**Redis集群优点**

实现扩容、分摊压力、无中心配置相对简单

**Redis集群不足**

- 多键值操作是不被支持的
- 多键的Redis事务是不被支持的，lua脚本不被支持
- 由于集群方案出现比较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，需要整体迁移而不是逐步过渡，复杂度较大。
