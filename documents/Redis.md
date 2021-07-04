# Redis

## NoSQL（Not Only Sql） 泛指非关系型的数据库

http://redisdoc.com

数据存储的瓶颈：

1. 数据量的总大小，一个机器放不下时；
2. 数据的索引（B+Tree） 一个机器的内存放不下时；
3. 访问量（混合读写）一个实例不能承受

如果满足了上述 1 or 3个条件，就要进化；



为什么使用NoSql

Memcached（缓存）+Mysql+垂直拆分；

<img src="C:\Users\xiong\AppData\Roaming\Typora\typora-user-images\image-20200919193929038.png" alt="image-20200919193929038" style="zoom: 50%;" />

Mysql主从读写分离：读到从库去读数据，写到主库去写数据；

分表分库+水平拆分+mysql集群：

Mysql的扩展性瓶颈：存储视频等大型数据时，会导致mysql效率极低；

**Nosql**

数据存储不需要固定的模式，无需多余操作就可以横向扩展；

最终一致性，而非ACID属性；

非结构化和不可预知的数据；

键值对的存储；



NoSql数据模型简介：

聚合模型： K-V键值对；BSON；列族；图形；

NoSql数据库的四大分类：

KV键值： Redis用于缓存和日志系统

文档型数据库： MongoDB

列存储数据库：HBase

图关系数据库：InfoGrid  专注构建图形结构 例如人际关系

 

分布式数据库中CAP原理 CAP+BASE：

CAP：C（Consistency 强一致性） A（Availability 可用性） P（Partition tolerance 分布式容忍性）

CAP只能三进二：

CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大；   （传统Oracle数据库）

CP：满足一致性，分区容忍性的系统，通常性能不是特别高；                        （Redis、MongoDB）

AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低一些；          （大多数网站架构的选择）

 

 BASE：基本可用（Basically Available） 软状态（Soft state） 最终一致（Eventually consistent）

主要思想时通过让系统放松对某一是可数据一致性的要求来换取系统整体伸缩性和性能上的改观。



分布式+集群：

分布式：多个服务器分别具有不同的任务；

集群：多个计算器或服务器都在做同一件事；



## Redis：

REmote DIctionary Server（远程字典服务器）

redis之所以是单线程还这么快，是因为redis的数据都是存放在内存中进行操作的，对于内存系统来说没有上下文切换效率就是最高的。

完全开源免费，是C语言编写的，是一个高性能的K-V分布式内存数据库；

Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用；

Redis不仅仅支持简单的KV类型的数据，同时还提供list，set，zset，hash等数据结构的存储；

Redis支持数据的备份，即master-slave模式的数据备份；

q内存存储和'x持久化，reidis支持异步将n内存中的数据写到硬qujiuxing

内存存储和持久化，redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务；

取最新的n个数据的操作；

模拟类似于HttpSession这种需要设定过期时间的功能；

发布、订阅消息系统；

定时器、计数器；



单进程模型来处理客户端的请求；

默认端口是6379； （merz）

索引默认从0开始；



**基本命令：**

Select 命令切换数据库；

Dbsize查看当前数据库的Key的数量；

Flushdb：清空当前数据库；

Flushall：通杀全部库；

keys *： 查询；

move key db(数据库序号)：将某一kv键值对移动至目标数据库;

exists key：判断某个key是否存在；

expire key秒钟：为给定的key设置过期时间；

ttl key：查看还有多少秒过期，-1表示永不过期，-2表示已过期；



## **Redis五大数据类型：**

字符串，哈希，列表，集合，有序集合；

字符串：是二进制安全的，可以包含任何数据，最多可以是512M；

哈希：类似java终点map，是一个string类型的field和value的映射表；

列表：简单的字符串列表，按照插入顺序排序，可以双端插入；

集合：是string类型的无序集合。他是通过HashTable实现的；

有序集合：是String类型元素的集合，且不允许重复的成员。不同的是zsort会关联一个double类型的分数；



### **String：**

set/get/del key；

append key value：插入新数据

strlen key：返回key的value长度

Incr/decr/incrby/decrby（++/--/+x/-x）： 一定要是数字才能进行加减；

getrange key index1 index2：截取key中从第index1到index2个字符；

setrange key index value: 如k1原为12345 

setrange k1 0 xxx后为xxx12345；

setex（set with expire）键秒值

setnx（set if not exist）如果不存在才创建(分布式锁中常使用)：

setex k4 10 v4：k4在10秒后死亡； 

mset/mget：多值插入以及多值查询；

msetnx：要么都成功，要么都失败；



### **List：**

lpush/rpush/lrange（左进右出）：

lpop/rpop：

lindex：按照索引获得元素 从上到下；

llen：长度

lrem key 删n个 元素：

ltrim key 开始index 结束index：截取指定范围内的值再复制给key；

rpoplpush 源列表 目的列表：rpoplpush list01 list02；

lset key index value：将第index位更新为value；

linsert key before/after value1 value2：在key的value1前/后新插入value2；



### Set

无序不重复集合

sadd key value1

Smembers key 查看 key中的所有元素

Sismember key value 查询key中是否又value存在

scard key 查看key中有几个元素

srem key value 移除key中的value

srandmember key 随机输出一个元素

spop 弹出

Sdiff key1 key2 输出差集

Sinter key1 key2 输出交集

Sunion key1 key2 输出并集



### **Redis哈希（Hash）：**

hset/hget/hmset/hmget/hgetall/hdel

hset： hset user name z3   ，  hmset customer id 11 name l4 age 26；

hget： hset user name；    hmget customer id name age；

hgetall；

hlen；

hexists key 在key里面某个值的key；

hkeys：得到所有的key；

hvals：得到所有的value；

hincrby customer key xx：对customer中的age+xx；

hincrbyfloat；

hsetnx；



### **Zset:**

在set的基础上 加一个score值；

之前：set k1 v1 v2 v3    现在：set k1 socre1 v1 score2 v2

zadd key score1 v1；

zrange key 查询范围 with scores；

（zREVRANGEBYSCORE 降序）

zrangebyscore key  开始score 结束score，通过score排序 -inf +inf最小值到最大值；

zrangebyscore key 开始score （结束score； 不包括结束score；

zrangebysocre key （开始socre 结束socre； 不包括开始socre；

zrangebysocre key 开始socre 结束socre limit index 个数：

zrangebysocre key1 score1 score2 limit 2 2：从前面的结果中的下标为2的地方开始截取两个；

zrem key value：删除value以及score

zcard：计算已有元素的个数；

zcount key score区间：计算区间内元素的个数；

zrank key values：获取对应value的下标；

zscore key value：获取对应value的score；

zrevrank key value：获取逆序的中value的下标；

zrevrange key 范围：逆序遍历范围中的value；

zrevrangebyscore key 开始score 结束score：zrevrangebyscore key 90 60；



## 三种特殊数据类型

### geospatial  地理位置

底层是zset类型 可以使用zset中的指令

GEOADD  :  geoadd china: city 116.40 39.90 beijing

GEOPOS： 获得一个坐标值  geopos china:city beijing

GEODIST :获取两个坐标之间的距离  geodist china : beijing shanghai [单位]

单位:m  km  mi英里  ft英尺

GEORADIUS：获得某位置周围指定距离内的所有坐标  georadius china:city 110 30 1000 km [withdist输出距离] [count输出个数]

GEORADIUSBYMEMBER  georediusbymember china:city shanghai 400km 找到 指定元素如上海周围400千米内的坐标

GEOHASH 返回一个或多个位置（返回的是一个字符串）



### Hyperloglog  基数 一个集合中不重复出现的元素

基数统计的方法 如：统计一个网页被多少人访问过，重复的用户只算一次

PFadd key value1 value2....... 添加n个value

PFcount key 统计key中的元素个数

PFmerge key3 key2 key1 将key1 和key2统计后生成到key3中



### Bitmap  位存储

setbit key  offset value

getbit key  offset

bitcount key 统计不为0的value



## 过期删除策略

**定时删除**：为指定的key创建一个定时器，当该key过期时，会由定时器来进行扫描，对key进行删除。

优点：保证内存尽快被释放

缺点：为每一个key设置一个定时器，会占用很多CPU的时间

**惰性删除**：key过期时不会主动删除，每次从数据库中去获取key时才检查，若过期就删除，返回null。

优点：对cpu的占用少

缺点：若大量过期的key在长时间内未被使用，则可能会发生内存泄漏（无用垃圾占用了大量的内存）

**定期删除**：每隔一段时间执行一次删除过期key操作。

优点：通过限制删除操作的时长和频率，减少对CPU的占用

缺点：内存方面不如定时删除，cpu时间方面不如惰性删除



## 内存淘汰机制

（1）volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。

（2）volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰。

（3）volatile-random：从已设置过期时间的数据集中任意选择数据淘汰。

（4）volatile-lfu：从已设置过期时间的数据集挑选使用频率最低的数据淘汰。

（5）allkeys-lru：从数据集中挑选最近最少使用的数据淘汰

（6）allkeys-lfu：从数据集中挑选使用频率最低的数据淘汰。

（7）allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

（8） no-enviction（驱逐）：禁止驱逐数据，这也是默认策略。意思是当内存不足以容纳新入数据时，新写入操作就会报错，请求可以继续进行，线上任务也不能持续进行，采用no-enviction策略可以保证数据不被丢失。

这八种大体上可以分为4中，lru、lfu、random、ttl。



## 配置文件

#### Security

config get requirepass "xxxxxxx" 输入完以后再进行操作时就得进行身份验证

默认为 ""

auth xxxxxxx 输入完成后才可以继续进行操作

#### LIMITS

Maxclients 最大客户数  默认10000

Maxmemory 最大内存 

到达最大内存以后，会选取一种清除策略进行清除

#### Daemonize 

持久化保存 yes为使用启用守护线程 no为不启用

#### port

端口号

#### 日志

loglevel notice  (有debug verbose notice warning)

#### databases 16

默认有16个数据库

#### logfile "" 

日志的文件位置名

always-show-logo yes 是否总是显示logo提示

#### 快照

save 900 1

save 300 10

save 60 10000

指定时间内对n个key进行操作就会执行持久化

stop--writes-on-bgsave-error yes 持久化如果吃错，是否还需要继续工作！

rdbcompression yes 是否对rdb文件进行压缩，需要消耗一些cpu

rdbchecksum yes 保存rdb文件的时候，进行错误的检查校验！

dir ./ rdb 文件保存的目录

Replication 复制



#### SECURITY

可以在此设置密码，修改密码。默认为空

config get requirepass 获取redis的密码

config set requirepass "123456" 设置redis的密码

auth  密码   发现没有权限后，使用该命令即可登录



maxclients 10000 设置能连接上redis的最大客户端的数量

maxmemory <bytes> redis配置最大的内存容量

maxmemory-policy noeviction  #内存到达上限之后的处理策列   移除一些过期的key 报错等



> APPEND ONLY  FILE MODE

appendonly no 默认不开启aof，默认是使用rdb方式持久化的，在大部分所有的情况下，rdb完全够用了

appendfilename "appendonly.aof" 持久化的文件的名字



#appendfsync always    #每次修改都会sync

appendfsync everysec   #每秒执行一次同步，可能会丢失这1s的数据

#appendfsync no            #不执行 sync，这个时候操作系统自己同步数据，速度最快！



## 持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会小时，所以很有必要去使用redis的持久化功能。

### RDB

在指定的时间间隔内将内存的数据集快照(snapshot)写入磁盘；

恢复时是将快照文件直接读到内存；

Redis会单独创建（fork）一个子进程来进行持久化，先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个文件替换上次持久化的文件；

如果需要进行大规模数据的恢复，且对数据恢复的完整性不是非常敏感，那RDB方式要比AOF更高效，缺点是最后一次持久化后的数据可能因为宕机丢失；

RDB保存的文件时dump.rdb

配置文件中 dbfilename dump.rdb      满足了save的规则时，会自动触发rdb规则，执行flushall命令，也会触发rdb规则，退出redis，也会产生rdb文件。

只需要将rdb文件放在redis的启动目录下，redis启动的时候就会自动检查dump.rdb恢复其中的数据！

#### FORK

作用是复制一个与当前进程一样的进程，新进程的所有数据数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程；

#### RDBCHECKSUM：

在存储快照后，还可以让redis使用64算法来进行校验，可以选择关闭此功能；

#### RDBcompression:

可以选择是否对存储到磁盘中的快照进行LZF算法压缩，可以选择关闭此功能；

#### SAVE/BGSAVE：

save立即开始存储，全部阻塞，bgsave进行异步存储；



### AOF

以日志的形式来记录每一个**写操作**，将redis执行过的所有写指令记录下来；

只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据；

默认为NO(关)；

**优先加载AOF**

#### Appendfsync:

Always 同步持久化 每次发生数据变更都会被立即记录到磁盘，性能较差但数据完整性好；

#### everysec：出厂默认推荐，异步操作，每秒记录，如果一秒内宕机，会有数据丢失；

no 不开启aof持久化；

#### Rewrite：

重新精简，优化；当Aof文件大小超过所设定的阈值，就会进行内容压缩；会记录上次重写时的AOF大小，当次AOF文件的大小是上次rewrite的一倍且大小大于64M时触发；

**no-appendfsync-on-rewrite**：重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性

**Auto-aof-rewrite-min-size**:设置重写的基准值

**Auto-aof-rewrite-percentage**:设置重写的基准值

优势：

每秒同步：Appendfsync always

每修改同步：Appendfsync everysec

不同步：no

劣势：

文件大小远大于rdb，且恢复速度慢于rdb

根据所使用的fsync策略，Aof的速度可能会慢于rdb



### 总结

RDB能够在指定的时间间隔内对你的数据进行快照存储

AOF记录每次对服务器写的操作，每当服务器重启的时候都会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件体积不至于过大；

只做缓存：如果只希望数据在服务器运行时存在，可以不适用任何持久化。

可以同时开启两种持久化方式：

这种情况下，优先使用AOF来恢复原始的数据，通常情况下AOF的数据要更完整；

不建议只使用AOF文件，因为RDB适合用于备份数据库快速重启，AOF文件在实时变化，所以不适合。而且不会有AOF可能存在的bug。



## 事物

Redis事务本质：一组命令的集合！一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行！

一次性、顺序性、排他性！执行一些列的命令！

**Redis单条命令是保证原子性的，但是事务是不保证原子性的**

Redis事务没有隔离级别的概念！

所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会执行！Exec

Redis的事务：

- 开启事务（multi）
- 命令入队（...）
- 执行事务（exec）

放弃事务:

multi 

set k1 v1 

set k2 v2

set k4 v4

DISCARD 放弃以上命令

事务命令中，如果是编译型异常，则整个事务都不会被执行。

如果是运行时异常，如自增一个字符串，则该条命令异常，其他命令正常执行。



## Redis实现乐观锁

watch key  监视对象

unwatch  取消监视

当被监视的对象被其他操作进行修改时，当前的事务会全部执行失败。

事务执行时会查看被监视的值是否被变动，如果被变动则事务失败。



## Redis发布订阅

使用场景： 1、实时信息系统  2、实时聊天 

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息

Redis 客户端可以订阅任意数量的频道。

第一个：消息发送者 第二个：频道  第三个：消息订阅者！

命令：

PSUBSCRIBE pattern [pattern]   订阅一个或多个符合给定模式的频道

PUBSUB subcommand [argument[argument]] 查看订阅与发布系统状态

PUBLISH channel message 将信息发送到指定的频道

PUNSUBSCRIBE [pattern[pattern...]] 退订所有给定模式的频道

SUBSCRIBE channel [channel] 订阅给定的一个或多个频道的信息

UNSUBSCRIBE [channel[channel]]  指退订给定的频道

 

## Redis 主从复制

主库不需要过多修改，从库配置文件进行修改。

1、端口

2、pid（进程号process id） 名字

3、log文件名字

4、dump.rdb名字





### 一主二从

默认情况下，每一个redis服务器都是主节点，只需要配置从机就好

info replication 查看主机和从机的信息

SLAVEOF  主机名字（host） 端口号



主机可以设置值，可以写，从机不能写只能读。主机中所有的信息和数据，都会被从机自动保存。

未设置哨兵时，主机宕机后，从机不会自动转换未主机，只能手动设置。

如果是使用命令进行配置的主从，当从机断开连接再重新连接后，会从slave变成master，但只要再连接主机后，就会立即从主机中获取已有的数据。



#### 复制原理

slave 启动连接到master后会发送一个sync同步命令。

Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集的命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

全量复制：slave服务在接受到数据库文件数据后，将其存盘并加载到内存中。

增量复制：Master 继续将新的所有收集到的修改命令一次传给slave，完成同步。

但是如果是初次连接至master，会有一次完全同步被自动执行。



如果主机断开了  从机可以使用 SLAVE NO ONE 使得自己成为主机



## 哨兵模式

sentinel

- 通过发送命令，让redis服务器返回监控其运行状态，包括主服务器和从服务器
- 当哨兵检测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他服务器，修改其配置文件，让它们切换主机。

当只使用一个哨兵时，可以会出现哨兵宕机的情况，可以通过使用多个哨兵相互监督，形成多哨兵模式。

1、哨兵的配置文件sentinel.conf：

sentinel  monitor myredis 127.0.0.1 6379 1

sentinel  monitor 被监控的主机名127.0.0.1 6379 1 

2、启动哨兵

redis-sentinel kconfig/sentinel.conf     

如果master节点宕机，那么就会从从机中随机选择一个从机作为主机（投票算法）；

如果之前的master重新连接后，只能作为从机连接已有的主机下；



优点：

1、哨兵集群，基于主从复制模式，所有的主从配置优点，它都有；

2、主从可以切换，故障可以转移，系统的可用性就会更好；

3、哨兵模式就是主从模式的升级，手动到自动，更加健壮；

缺点：

1、Redis不好在线扩容，当集群容量达到上限，在线扩容就十分麻烦，只能横向扩展；

2、实现哨兵模式的配置其实时很麻烦的，里面有很多选择！



## 布隆过滤器

是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力；





















