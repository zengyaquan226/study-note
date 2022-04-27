

# NoSQL

### **优点**

1. 缓存数据库，完全在内存中，速度快，数据结构简单
2. 减少io操作，数据库和表拆分，虽然破坏业务逻辑，即外加一个缓存数据库，提高数据库速度，也可以用专门的存储方式，以及针对不同的数据结构存储



### 其他数据库（NoSQL）

#### Memcache	

​		NoSql数据库 /数据都在内存中，一般不持久化/// key-value模式，支持类型单一 ///一般是作为缓存数据库辅助持久化的数据库

#### Redis

​		几乎覆盖了Memcached的绝大部分功能 /数据都在内存中，支持持久化，主要用作备份恢复 /.除了支持简单的key-value模式，还支持多种数据结构的存储，比如 list、set、hash、zset等///一般是作为缓存数据库辅助持久化的数据库

#### MongoDB	

​		高性能、开源、模式自由(schema free)的文档型数据库//数据都在内存中， 如果内存不足，把不常用的数据保存到硬盘/虽然是key-value模式，但是对value（尤其是json）提供了丰富的查询功能/支持二进制数据及大型对象/可以根据数据的特点替代RDBMS ，成为独立的数据库。或者配合RDBMS，存储特定的数据。



# 软件安装

下载redis以及编译安装

1. 官网下载redis文件
2. 复制到/opt目录下cp /home/gaokaoli/Downloads/redis-6.2.3.tar.gz /opt
3. 查看是否安装了gcc编译输入gcc --version
4. 进入目录号`cd /opt/redis-6.2.3`进行`make`编译以及`make install`安装即可
5. 如果出错，采取的策略

```java
1.是否安装了gcc
2.重新输入make distclean
make
make install
```



**如果没有安装gcc，则输入yum install -y gcc，或者是apt-get install -y gcc，安装了忽视这一步**



### 将redis设置成后台启动

```
拷贝一份redis.conf到etc/redis.conf
设置redis.conf的daemonize值为yes
```



### 开启redis服务

在/usr/local/bin下执行

```xml
redis-server /etc/redis.conf  # 开启redis的服务
```

查看redis是否已经启动

```java
ps -ef | grep redis
```



### 客户端进行连接

```
redis-cli
```

### 客户端关闭的方式

```
shutdown
# 或者杀死进程
kill -9 进程号
```



### redis设置密码

```
一、编辑redis.conf文件
建议在修改前备份一下redis.conf文件，这样即使操作出错了，也可以从头再来
1.注释掉 bind 127.0.0.1
可以在 vim redis.conf 后进入指令模式（shift +：）
 然后输入：/单词
 快速找到单词的位置
 
2.修改protected-mode
在 redis.conf 中找到 protected-mode 将后面的 yes 改为 no
 
3.修改daemonize
在 redis.conf 中找到 daemonize 将后面的 no 改为 yes
 
4.修改密码（requirepass foobared）
在 redis.conf 中找到 requirepass foobared ，可在下面添加 requirepass 你的密码
 
执行完以上操作后进入指令模式  wq 保存退出
# 在客户端操作的时候需要 先执行 auth 密码
```



# Redis数据类型（都是相对于value来说的）

### key的常见指令

```
keys * 查看当前库所有key
set key value 设置key值与value
exists key 判断key是否存在
type key 查看key是什么类型
del key 删除指定的key数据
unlink key 根据value选择非阻塞删除
------仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。
expire key 10 10秒钟：为给定的key设置过期时间
ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期


========================================================================
    select 命令切换数据库（默认16个数据库（0-15））
    dbsize 查看当前数据库的key数量
    flushdb 清空当前库
    flushall 通杀全部库
```



### String字符串

- 一个key对应一个value
- 二进制安全的，即可包含任何数据
- value最多可以是512m

```
set key value 设置key值
* NX 当数据库的key不存在的时候，可以将其加入到数据库
* XX 当数据库的key存在的时候，才可以将其加入到数据库
* EX key的超时时间单位为秒数
* PX key的超时毫秒数
get key 查询key值
append key value 将给定的value追加到原值末尾
strlen key 获取值的长度
setnx key value 只有在key不存在的时候，设置key值
incr key 将key值存储的数字增1，只对数字值操作，如果为空，新增值为1
decr key 将key值存储的数字减1，只对数字值操作，如果为空，新增值为1
decr key 将key值存储的数字减1
incrby/decrby key <步长> 将key值存储的数字增mset key value key value..同时设置一个或者多个key-value
==========================================================================================
mget key key...同时获取一个或多个value
msetnx key value key value..同时设置一个或者多个key-value.当且仅当所有给定key都不存在
getrange key <起始位置> <结束位置> 获取key的起始位置和结束位置的值
setrange key <起始位置> value 将value的值覆盖起始位置开始
setex key <超时时间> value 设置键值的同时,设置过期时间
getset key value 用新值换旧值

```

**底层原理扩容的机制**

1. 当字符串的长度小于1M的时候，扩容为当前的一倍
2. 当字符串的长度大于1M的时候，每次加1M最多512MB



### Redis列表

```
lpush/rpush key value value...从左或者右插入一个或者多个值(头插与尾插)

lpop/rpop key 从左或者右吐出一个或者多个值(值在键在,值都没,键都没)

rpoplpush key1 key2 从key1列表右边吐出一个值,插入到key2的左边

lrange key start stop 按照索引下标获取元素(从左到右)

lrange key 0 -1 获取所有值

lindex key index 按照索引下标获得元素

llen key 获取列表长度

linsert key before/after value newvalue 在value的前面插入一个新值

lrem key n value 从左边删除n个value值

lset key index value 在列表key中的下标index中修改值value
```

**底层原理**

当数据比较少的时候，redis将数据封装成一个ziplist（特点是在内存中是连续的存储），再用链表将这些ziplist进行连接

### redis的Set

<font color=red>特点是无序，去重，底层为一个hash表</font>

```
sadd key value value... 将一个或者多个member元素加入集合key中,已经存在的member元素被忽略
smembers key 取出该集合的所有值
sismember key value 判断该集合key是否含有改值
scard key 返回该集合的元素个数
srem key value value 删除集合中的某个元素
spop key 随机从集合中取出一个元素
srandmember key n 随即从该集合中取出n个值，不会从集合中删除
smove <一个集合a><一个集合b>value 将一个集合a的某个value移动到另一个集合b
sinter key1 key2 返回两个集合的交集元素
sunion key1 key2 返回两个集合的并集元素
sdiff key1 key2 返回两个集合的差集元素（key1有的，key2没有）
```



### redis的Map

非常适合存放对象

```
hset key field value 给key集合中的filed键赋值value
hget key1 field 集合field取出value
hmset key1 field1 value1 field2 value2 批量设置hash的值
hexists key1 field 查看哈希表key中，给定域field是否存在
hkeys key 列出该hash集合的所有field
hvals key 列出该hash集合的所有value
hincrby key field increment 为哈希表key中的域field的值加上增量1 -1
hsetnx key field value 将哈希表key中的域field的值设置为value，当且仅当域field不存在
```



### redis的ZSet

**底层的原理**

<font color=blue>底层通过score来进行排序</font>

```
zadd key score1 value1 score2 value2 将一个或多个member元素及其score值加入到有序key中
zrange key start stop (withscores) 返回有序集key，下标在start与stop之间的元素，带withscores，可以让分数一起和值返回到结果集。
zrangebyscore key min max(withscores) 返回有序集key，所有score值介于min和max之间（包括等于min或max）的成员。有序集成员按score的值递增次序排列
zrevrangebyscore key max min （withscores）同上，改为从大到小排列
zincrby key increment value 为元素的score加上增量
zrem key value 删除该集合下，指定值的元素
zcount key min max 统计该集合，分数区间内的元素个数
zrank key value 返回该值在集合中的排名，从0开始
```

### redis的发布和订阅（pub/sub）

**Redis可以订阅任意的数量的频道（信道）**

实现步骤：

```
(1)订阅一个客户端 
subscript channel(频道)
(2)另一个客户端给这个频道发送消息
public channel value
```



### redis的BitMaps类型

特点：

1. 合理使用操作位可以有效地提高内存使用率和开发使用率
2. 本身是一个字符串，不是数据类型，数组的每个单元只能存放0和1，数组的下标在Bitmaps叫做偏移量
3. 节省空间，一般存储活跃用户比较多

```
setbit key offset value # 设置bitmaps的key的偏移量（0/1）可以理解成数组的索引
getbit key offset  # 获取bitmaps中某个偏移量的值 获取键的第offset位的值
bitcount key （start end） #统计字符串从start到end之间比特值为1的数量 -1 表示最后一个位 -2 表示倒数第二个位
# redis的setbit设置或清除的是bit位置，而bitcount计算的是byte的位置
bitop and(or/not/xor）destkey key... # bitop是一个复合键 and（交集） or（或） not （非） xor（异或）
```



### Redis的类型（Hyperloglog）

1. 统计网页中页面访问量
2. 只会根据输入元素来计算基数，而不会储存输入元素本身，不能像集合那样，返回输入的各个元素
3. 基数估计是在误差可接受的范围内，快速计算（不重复元素的结算）
4. 基数的概论：计算不重复的个数

```
pfadd key element # 添加指定的元素到hyperloglog中
pfcount key..  # 计算key的近似基数
pfmerge destkey sourcekey sourcekey # 一个或多个key合并后的结果存在另一个key
```



### Redis的类型（Geospatial）

**提供经纬度设置，查询范围，距离查询等**

```
geoadd key longitude latitude member # 添加地理位置（经度纬度名称）当坐标超出指定的范围，命令会返回一个错误已经添加的数据，无法再添加
geopos key member # 获取指定地区的坐标值
geodist key member1 member2 (m km ft mi) # 获取两个位置之间的直线距离
georadius key longitude latitude radius (m km ft mi) # 以给定的经纬度为中心，找出某一半径的内元素

```



# Jedis操作Redis

### 引入相关的依赖

```
  <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.6.1</version>
        </dependency>
    </dependencies>
```

````
public class JedisDemo1 {
    public static void main(String[] args) {
        //创建Jedis对象
        Jedis jedis = new Jedis("192.168.242.110", 6379);
        //测试
        String ping = jedis.ping();
        System.out.println(ping);
    }
}
````



# Jedis整合springboot

**导入相关的场景启动器**

```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**在配置文件中配置**

```xml
#Redis服务器地址
spring.redis.host=172.22.109.205
#Redis服务器连接端口
spring.redis.port=6379
#Redis数据库索引（默认为0）
spring.redis.database= 0
#连接超时时间（毫秒）
spring.redis.timeout=1800000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#最大阻塞等待时间(负数表示没限制)
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=5
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```



# Redis的事务

### 事务操作的特点

- 单独的隔离操作
- 事务中的所有命令都会序列化
- 按顺序执行
- 执行的过程中，不会被其他命令请求所打断

### 事务的三大特性

- 单独的隔离操作（不会被打断）
- 没有隔离级别
- 不保证原子性



### redis的事务（Multi, Exec, discard）

1. 输入Multi开始，输入的指令都会依次进入命令队列中，直到输入exec后redis会将之前的队列中命令依次进行执行
2. 如果组队的过程中可以通过discard来放弃组队



### 事务的错误处理

1. 如果事务组队的某一个命令出现了错误，执行整个所有的队列都会被取消
2. 如果执行的时候某一个命令出了错误，只有报错的不会被执行，而其他的命令都会被执行



# 乐观锁和悲观锁

##### 悲观锁：

​		不能同时进行多人，执行的时候先上锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁

##### 乐观锁：

​		通过版本号一致与否，即给数据加上版本，同步更新数据以及加上版本号。不会上锁，判断版本号，可以多人操作，类似生活中的抢票。每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的



### redis进行乐观锁的配置

在执行multi之前，执行命令`watch`具体格式如下

```bash
watch key1 [key2]
unwatch #取消watch命令对所有的key的监视
```



### Lua脚本用于解决乐观锁的遗留问题

lua的脚本在执行的过程中，不允许其他的命令插队



# 持久化

### RDB

在指定的`时间间隔内`将内存中的`数据集快照`写入磁盘

**底层原理**

​		Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能

​		如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。
RDB的缺点是最后一次持久化后的数据可能丢失。

​		Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“写时复制技术”一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程

### 进行备份

将之前的dump.rdb进行拷贝一份，放到工作目录下，启动的时候会自动读取整个配置文件



### 配置文件redis.conf的配置

```
save 3600
save 300 10
save 60 10000
大概意思如下：save 秒钟 写操作次数，60秒传10000次的写操作。
不设置save指令，或者给save传入空字符串
```



### 优点

- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高更适合使用
- 节省磁盘空间
- 恢复速度快

### 缺点

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
- 虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。
- 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。



# aof

以日志的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件

- redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

关于redis.conf配置文件的部分解释

AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载



​		默认是不开启AOF，开启RDB可以在redis.conf中配置文件名称，默认为 appendonly.aofAOF文件的保存路径，同RDB的路径一致

    appendonly no，改为yes

插入其数据的时候，在日志里面会看到数据的添加如果直接在日志添加一些无法识别的数据，启动redis会启动不了

### 异常恢复

可以通通过/usr/local/bin/redis-check-aof--fix appendonly.aof进行恢复在当前目录下有redis-check-aof这个文件

```
/usr/local/bin/redis-check-aof --fix appendonly.aof
```



### 优点：

- 备份机制更稳健，丢失数据概率更低
- 可读的日志文本，通过操作AOF稳健，可以处理误操作

### 缺点：

- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。
- 每次读写都同步的话，有一定的性能压力。
- 存在个别Bug，造成恢复不能

**推荐两个都开启**



# 主从复制(一主多仆)

​		主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主



### 好处

- 读写分离，性能扩展
- 容灾快速恢复



### 在同一台主机上配置一主两仆的配置方式

```
(1)创建一个/myredis的文件夹
(2)复制redis.conf到这个文件夹
(3)配置一主两仆，创建三个配置文件加入一下内容
include /myredis/redis.conf
pidfile "/var/run/redis-6379.pid"
port 6379
dbfilename "dump6379.rdb"
# 如果配置了密码将下面的打入
masterauth "123"
(4)启动redis服务
(5)info replication 打印主从复制消息
(6)配置从机 在从机上执行
slavof <ip> <port>
```

### 特点

1. 从服务器挂了之后，再启就是主服务器了
2. 从服务器一旦加入主服务器，会主动请求主服务器的所有消息，将其传递给从服务器（全量复制）
3. 当主服务器添加了数据，将其传给从服务器（增量复制）
4. 主服务器宕机了，从服务器会等待主服务器的启动



# 薪火相传

**就是从服务器还可以有从服务器**



# 反客为主

**当主机宕机了，后面的从服务器变成主服务器**

```
slavof no one #将从服务器变成主服务器 但是只有他之前的从服务器还是他的服务器
```



# 哨兵模式（反客为主的自动版)

主要是为了监控主机宕机之后，从机可以立马变为主机，就和上面的反客为主一样，不用手动设置 能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库



### 实现方式

```
(1)在自定义的/myredis下新建sentinel.conf(名字固定)
# mymaster对主机监视的别名 1 表示至少需要多少哨兵同意迁移的数量
sentinel monitor mymaster 127.0.0.1 6380 1
# 设置了密码 
sentinel auth-pass mymaster 123

启动哨兵模式通过redis的bin目录下
命令如下：redis-sentinel /sentinel.conf
```



### 具体哪个从机会变成主机

（顺序依次往下，优先级》偏移量》runid

    优先级在redis.conf中默认：slave-priority 100，值越小优先级越高
    偏移量是指获得原主机数据最全的，也就是数据越多，变主机的机会越大
    每个redis实例启动后都会随机生成一个40位的runid


# redis集群

**无中心化集群**

任何一台主机都苦于成为集群的入口



### 实现步骤

```xml
(1)配置配置文件
include /myredis/redis.conf
pidfile "/var/run/redis-6379.pid"
port 6379
dbfilename "dump6379.rdb"
masterauth "012200abcdABC"
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
(2)启动redis服务
(3)将多个结点合成一个集群
cd /opt/redis-6.6/src

# --cluster-replicas 集群的方式 1 代表最简单的方式 1主1仆 IP 端口号 有密码在最后 加入 -a 密码
redis-cli --cluster create --cluster-replicas 1 192.168.242.110:6379 192.168.242.110:6380 192.168.242.110:6381 192.168.242.110:6389 192.168.242.110:6390 192.168.242.110:6391

(4)连接集群
redis-cli -c -p 6379

cluster nodes # 参看集群消息

```



分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。

# slots

​		一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个，
集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。
集群中的每个节点负责处理一部分插槽

**细节**

不在同一个slot的键值，不能使用mset，mget的方式进行多键操作，可以使用 {}定义组 

比如：

mset name{user} tom age{user} 20



### 常用指令

```
    查询集群中的值，CLUSTER KEYSLOT k1
    查询卡槽中key的数量，CLUSTER COUNTKEYSINSLOT 12706
    查询指定卡槽返回key的数量，CLUSTER GETKEYSINSLOT 5474 2
```



### 主机宕机的情况

1. 主机和从机都宕机了，redis的服务是否要继续
2. 配置文件中的 cluster-require-full-coverage 为 yes的时候 整个都挂 为no的时候只有这些插槽无法使用





# idea操作jedisCluster

```java
public class JedisClusterTest {
    public static void main(String[] args) {
        HostAndPort hostAndPort = new HostAndPort("192.168.242.110", 6381);
        JedisCluster jedisCluster = new JedisCluster(hostAndPort);
        jedisCluster.set("k5","v5");
        String k5 = jedisCluster.get("k5");
        System.out.println(k5);
    }
}

// 注意需要开启所有对应的端口才可以访问
```



# 应用问题

### 缓存穿透

**出现的原因**：redis的命中率突然的减低，大量的请求不在去redis中查询，大量的数据前往数据库去查找（简单的来说就是不断的查询的数据库中不存在的数据）



**解决的方案**

1. 对空值缓存，无论是否查询到是数据都存入到缓存中，设置他的存在时间
2. 设置可以访问的白名单（可以采取bitmaps的方式来继续存储）
3. 使用布隆过滤器（底层的方式也是白名单的方式）
4. 继续实时的进行监控（发现出现的出现空值的访问）直接将其拉人黑名单



### 缓存击穿

**出现的原因**

数据库的压力突然大幅度的增加，redis中的热点的数据过期了，突然大量的进行访问



**解决方案**

1. 预先设置热门的数据
2. 实时调整过期的时间
3. 使用锁



### 缓存雪崩

**出现原因**

数据库的压力突然变大，带着服务器一起崩溃，在极短的时间中，出现了大量的key过期的情况



**解决方案**

1. 构建多级缓存：nginx缓存+redis+其他的缓存（encacha）
2. 使用锁或队列
3. 设置过期时间标志更新缓存
4. 将缓存过期的时间分散开来



### 分布式锁

​		由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问

也就是在这个机器上了锁，另外一个机器也要可以识别到这个锁，也就是共享锁，都是同一把锁


**解决方案**

1. 基于数据库实现分布式锁
2. 基于缓存（Redis等）
3. 基于Zookeeper



### 基于redis实现分布式锁

1. setnx key value (当数据库中有这个的时候，我们就认为有人在操作，相当于上锁) 可以通过expire进行设置过期时间
2. set NX EX 100 key value 进行优化



**优化**

1. UUID防删（在进行del之前进行判断，将我们的value存放到value中）
2. 当判断完后，可以出现服务器卡了，下次进行删除的时候，将别人的锁给删除，我们可以使用lua脚本来进行保证原子性



### 保证分布式可用

- 互斥性。在任意时刻，只有一个客户端能持有锁。
- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。
- 加锁和解锁必须具有原子性



# ACL

ACL是Access Control List（访问控制列表）的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接。

```
    acl cat[String] 查看那些命令可以使用
    acl setuser xxx # 添加用户
    acl list命令展现用户权限列表
    acl cat，查看添加权限指令类别
    acl whoami命令查看当前用户
    acl set user命令创建和编辑用户ACL
    acl setuser userName on > password ~cached.*+get
```

