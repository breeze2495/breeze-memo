## Nosql数据库:

### 	P1: Nosql数据库的四大分类

- **KV**

- **文档型数据库(bson格式比较多)**

  - couchDB:
  - **mongoDB**
    - 是一个基于分布式文件存储的数据库
    - 是一个介于关系数据库和非关系数据库之间的产品,是非关系型数据库中功能最丰富的,最像关系数据库的

- **列存储数据库:**

  - HBase
  - 分布式文件系统

- **图关系数据库**

  - 其中存放的关系例如:朋友圈社交网络,广告推荐系统

  - 社交网络,推荐系统. 专注于构建关系图谱

  - Neo4j,infoGrid

    ### 			

### P2: 四大类-对比

### <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210316193657721.png" alt="image-20210316193657721" style="zoom: 33%;" />	



### P3: 分布式数据库CAP原理

- **C:**consistency: 强一致性
- **A:**avalibility: 可用性
- **P:**partition tolerance: 分区容错性

- **cap理论就是说在分布式存储系统中,最多只能同时实现如上两点**

- 由于当前网络硬件肯定会出现延迟丢包,所以必须要实现 **分区容错**,因此需要在A,P中进行权衡

  

  - **CA:** 传统Oracle数据库 (单点集群,满足一致性,可用性的系统,通常在可扩展性上不太强大
  - **AP:** 大多数网站架构的选择 (满足可用性,分区容错性的系统,通常对一致性要求低一些
  - **CP:** redis mongoDB (满足一致性,分区容错性的系统,通常性能不是特别高

- 绝大多数web应用,其实并不需要强一致性,牺牲C换区P是分布式数据库产品的方向;



### 	P4: BASE

- BASE就是为了解决关系型数据库强一致性导致的可用性降低而提出的解决方案;

- **基本可用:** (Basically Available
- **软状态:** (Soft state
- **最终一致:** (Eventually consistent

- 思想:通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上的改观.



### 	P5: 分布式 + 集群

- **分布式:**
  - 不同的多台服务器部署不同的服务模块,他们之间通过RPC/RMI 进行通信与调用,对外提供服务和组内协作
- **集群:**
  - 不同的多台服务器部署相同的服务模块,通过分布式调度软件进行统一的调度,对外提供服务和访问





## Redis:

### P1: 安装

```linux命令
1.redis官网下载压缩文件,拷贝到 /opt 目录
2.在/opt 目录下 : sudo tar -zxvf redis-6.2.1.tar.gz 解压
3.进入redis安装后的目录 ls -l 查看  
4.sudo make命令
5.sudo make install命令
6.cd /usr/local/bin 目录 查看是否安装完毕
7.拷贝redis安装的目录里的 redis.conf文件到新建的文件夹: sudo cp redis.conf /myredis/
8.修改myredis中的redis.conf配置文件 sudo vim redis.conf  将daemon设置成yes
9.进入bin目录:  redis-server /myredis/redis.conf 接着 redis-cli -p 6379
10.ping命令 得到pong的回馈
11.ps -ef|grep redis命令查看redis运行状态

```



### P2: 杂项知识

- redis是**单进程**的
  - 单进程模型来处理客户端的请求。对读写等事件的响应 是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率
  - Epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本， 它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。
- 默认16个数据库,下标从0开始,可在配置文件中配置
- select 0/1/2/... 切换数据库
- DBSIZE: 查看当前数据库key的数量
- flushDB: 清空当前库
- flushall: 清空全部库
- 同一密码管理,16个库同样的密码
- Redis索引都是从零开始
- 默认端口: 6379



------



### P3: key关键字

- Redis键(key): 常用命令
  - keys *  
  - exists key名 : 判断某个key是否存在
  - move key名 db序号 : 将当前库key移除至index为db的库
  - expire key名 时间 : 指定key设置过期时间,过期完自动移除
  - ttl(time to live) key名 : 查看还有多少秒过期,-1代表永不过期,-2代表已过期
  - type key名: 查看key的类型



------



### P4: 五大数据类型

#### D1: String(字符串)

- **概念:**

  - String是redis最基本的类型,可以理解为Memcached一样的类型,一个key对应一个value
  - String类型是二进制安全的,意思是redis的string可以包含任何数据,如jpg图片或者序列化的对象
  - 一个string中字符串value最多可以是512M

- **常用命令:**

  ```
  * set/get/del/append/strlen  
  	 append key名 value  : 某个键值尾部追加
  	 strlen key名 : 返回键值长度
  
  * Incr/decr/incrby/decrby,一定要是数字才能进行加减
  	 Incr key名
  	 Incr key名 递增至
  	 
  * getrange/setrange
  	 getrange key名 开始index 结束index  : 获取下标开始index到结束index的值(包含端点
  	 					如果是   0      -1   : 表示获取全部
     setrange key名 开始index 内容 : 从index出开始覆盖
     
  * setex(set with expire)键秒值/setnx(set if not exist)
  		setex key名 时间 内容 
  		setnx key名 内容 : 当该key不存在时才生效
  		
  * mset/mget/msetnx
  		mset k1 v1 k2 v2 : 将key1设为v1 key2设为v2  如果key1已经存在 则该条指令不生效
  		
  * getset(先get在set)
  ```

  ​		





#### D2: List (列表)

- **概念:**

  - List是简单的字符串列表,按照插入顺序排序
  - 底层是一个链表,可以从头部和尾部插入元素

- **常用命令:**

  ```linux
  * lpush/rpush/lrange
  
  * lpop/rpop
  
  * lindex key名 index : 按照索引下标获得元素（从上到下）
  
  * llen: 列表长度
  
  * lrem key名 n个 value值 : 删除n个值
  
  * ltrim key名 开始index 结束index : 截取指定范围的值后再赋值给key
  
  * rpoplpush 源列表 目的列表
  
  * lset key名 index value : 将下标为index的值设置为value
  
  * linsert key名 before/after 值1 值2 : 在 值1 前/后 插入 值2
  ```

- **性能分析:**
  - 它是一个字符链表，left，right都可以插入添加。
  - 如果键不存在，创建新的链表。如果键已经存在，新增内容。
  - 如果值全移除，对应的键也就消失了。
  - 链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。



#### D3: Hash (哈希)

- **概念:**
  - Hash是一个键值对
  - Hash是一个String类型的field和value的映射表,hash特别适用于存储对象
  - 类似java中的 Map<String,Object>

- **常用命令:**

  ```linux
   * KV模式不变 V是一个键值对
   
   * hset/hget/hmset/hmget/hgetall/hdel
   	 	hset user name myl : 
   	 	hget user name 
   	 	hmset user name myl age 23 address nanjing 
   	 	hmget user name age address
   	 	hgetall user : 返回 name myl age 23 address nanjing
   	 	hdel user name : 删除user中的name内容
   	 	
   * hlen hash名 : 多少个键值对 name myl 算一对
   
   * hexists user name : 返回1(真 返回0
   
   * hkeys/hvals hash名 : 获取所有的键或值
   
   * hincrby/hincrbyfloat : 
   		hincrby user age 2 : 
   		hincrbyfloat user score 0.5
      
   * hsetnx customer email breeze@163.com  : 如果emain不存在,就设置成功
  ```

  

#### D4: Set (集合)

- **概念:**
  - Set是string类型的无序集合
  - 底层通过HashTable实现

- **常用命令:**

  ```linux
  * sadd/smembers/sismember
  	sadd set名 值1,值2,值3... : 去重添加
  	smenbers set名 : 查看元素
  	sismember set名 value : 查看value是不是set中的元素
  	
  * scard : 获取集合里面元素个数
  
  * srem set名 value : 删除集合元素
  
  * srandmember set名 int : set的元素中随机出int个元素
  
  * spop set名 : 随机出栈
  
  * smove set01 set02 set01中的value : 
  
  * (差集) sdiff set01 set02 : 在set01中且不再set02中的元素
  	(交集) sinter 
  	(并集) sunion
  ```

  

#### D5: ZSet (有序集合)

- **概念:**
  - 和set一样是string类型的集合,且不允许重复的元素
  - 不同的是每个成员都会关联一个double的分数,redis正式通过分数为集合中的元素从小到大排序
  - 元素唯一,但分数可以重复

- **常用命令:**

  ```linux
  * zadd/zrange
  		zadd zset01 60 v1 70 v2 80 v3
  		zrange zset01 0 -1 : 返回 v1 v2 v3
  		zrange zset01 0 -1 withscores : fanhui v1 60 v2 70 v3 80
     
  * zrangebyscore key 开始socre 结束score 
  		zrangebyscore zset01 60 90 : 60 - 90的
  		zrangebyscore zset01 60 (90 : 60 - 90(不包含90
  												(60 (90 :
  		zrangebyscore zset01 60 90 limit 2 2 : 从下标2开始截取两个
  		
  * zrem key v1  : 删除v1以及v1对应的score
  
  * zcard/zcount key score区间/zrank key values值，作用是获得下标值/zscore key 对应值，获得分数
  		
  		zcard  zset01 :   (返回个数
  		zcount zset01 70 80 : 返回70-80的个数
  		
  		
  		
  		
  ```

  

------

### P5: 解析配置文件redis.conf





------

### P6: 持久化

#### D1: RDB (Redis Database

- **概念:**
  - 简称快照,即在指定时间间隔内将内存中的数据集快照写入磁盘,恢复时将快照文件直接读到内存中
- **持久化过程:**
  - Redis会单独创建(Fork)一个子进程来进行持久化,会先将数据写入到一个临时文件中,等到持久化过程结束,再用这个临时文件替换上次持久化好的文件.	
    - **Fork**
      - Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量‘程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。
  - 整个过程中,主进程不进行任何I/O操作,这就确保了极高的性能
  - 最终保存的是 dump.rdb文件
- **如何触发RDB快照:**
  - 配置文件中的配置 (可更改
    - 默认:
      - 一小时1次
      - 五分钟100次
      - 一分钟10000次
  - save/bgsave命令:
    - save: sava只管保存,其他不管.
    - bgsave: redis会在后台异步进行快照操作,快照的同时还可以响应客户端请求.可以通过lastsave命令获取最后一次成功执行快照的时间。
  - flushall命令，也会产生dump.rdb文件，但里面是空的，无意义。
- **如何恢复:**
  - 将备份文件（dump.rdb）移动到redis安装目录并启动服务即可

- **优缺点:**
  - **优点:**
    - 如果需要进行大规模数据恢复,且对于数据恢复的完整性不是很敏感,那么RDB方式比AOF更加高效
  - **缺点:**
    - 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改。
    - Fork的时候，内存中的数据被克隆了一份，大约2倍的膨胀性需要考虑。



#### D2: AOF (Append Only File

- **概念:**
  - 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话根据日志文件的内容将写指令从前到后执行一次已完成数据的恢复工作。
  - AOF保存的是appendonly.aof文件
- **AOF启动/修复/恢复:**
  - **正常恢复:**
    - 启动: 配置文件冲appendonly 设置成yes
    - 将现有的aof文件复制一份到对应目录
    - 恢复:重启redis进行加载
  - **异常恢复:**
    - 启动:设置yes
    - 备份写坏的AOF文件
    - 修复:  Redis-check-aof --fix appendonly.aof
    - 恢复: 重启redis进行加载
- **Rewrite:**
  - **概念:**
    - AOF采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令gbrewriteaof
  - **重写原理:**
    - AOF文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后在rename），遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。
  - **触发机制:**
    - Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。
  - **优缺点:**
    - **优点:**
      - 每秒同步：appendfsync always  同步持久化  每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好
      - 每修改同步：appendfsync everysec  异步操作 ，每秒记录  如果一秒内宕机，有数据丢失。
      - 不同步：appendfsync no  从不同步
    - **缺点:**
      - 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
      - AOF运行效率要慢于rdb，每秒同步策略效率较好，不同步效率和rdb相同。

#### D3:总结

- ​	RDB和AOF可以共存，但是恢复的时候先找的是AOF，如果AOF文件异常，可以通过check-aof进行AOF修复。



------



### P7: Redis事务

- **概念:**

  - 可以一次执行多个命令,本质是一组命令的集合.
  - 一个事务中的所有命令都会序列化,按顺序地串行化执行而不会被其他命令插入,不许加塞

- **具体执行:**

  |     **命令**      |                             描述                             |
  | :---------------: | :----------------------------------------------------------: |
  |       MUTLI       |                     标记一个事务块的开始                     |
  |       EXEC        |                    直接所有事务块内的命令                    |
  |      DISCARD      |             取消事务,放弃执行事务块内的所有命令              |
  | WATCH KEY[key...] | 监视一个或多个key,如果在事务执行之前,key被其他命令改动,事务会被打断 |
  |      UNWATCH      |                      取消所有key的监视                       |

  - **case1:** 正常执行

    ```linux
    MUTLI
    
    set k1 v1 
    incr t1 
    ...
    
    EXEC
    ```

  - **case2:** 放弃事务

    ```linux
    MUTLI
    
    set k1 v1 
    incr t1 
    ...
    
    DISCARD
    ```

  - **case3:** 全体连坐 (类似java编译时异常

    <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317105303447.png" alt="image-20210317105303447" style="zoom:50%;" />

    

  - **case4:** 冤头债主 (类似java运行时异常

    <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317105501900.png" alt="image-20210317105501900" style="zoom:50%;" />

    

  - **case5:** watch监控

    - **悲观锁/乐观锁/CAS**

      - 悲观锁:
        - 每次拿数据的时候都认为会**被别人修改**,所以在每次拿数据的时候**都会加锁**,这样别人想拿这个数据时就会block直到他拿到锁,传统的关系型数据库就用到了很多这种锁机制,比如行锁,表锁,读锁,写锁等,都是在做操作之前先锁上
      - 乐观锁:
        - 乐观锁：每次去拿数据的时候都认为别人不会修改，**所以不会上锁**，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号等机制**。乐观锁适用于多度的应用类型，这样可以**提高吞吐量**.
        - 乐观锁策略：提交版本必须大于记录当前版本才能执行更新。
      - CAS

    - **具体执行:**

      - **case1:** 初始化信用卡可用余额和欠额

        - <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317114654098.png" alt="image-20210317114654098" style="zoom: 33%;" />

        

      - **case2:** 无加塞篡改，先监控再开启multi， 保证两笔金额变动在同一个事务内

        - <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317114824515.png" alt="image-20210317114824515" style="zoom:33%;" />

        

      - **case3:** 有加塞篡改,监控了key，**如果key被修改了**，后面一个事务的执行失效

        - <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317114957610.png" alt="image-20210317114957610" style="zoom: 33%;" />

      - **case4:** unwatch,将之前加的全部key的监控锁取消
        - <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210317115057384.png" alt="image-20210317115057384" style="zoom:33%;" />

      

      - **小结:**

        - Watch指令，类似乐观锁，事务提交时，如果Key的值已被别的客户端改变， 比如某个list已被别的客户端push/pop过了，整个事务队列都不会被执行

        - 通过WATCH命令在事务执行之前监控了多个Keys，倘若在WATCH之后有任何Key的值发生了变化， EXEC命令执行的事务都将被放弃，同时返回Nullmulti-bulk应答以通知调用者事务执行失败

          

- **3阶段**
  - **开启：**以MULTI开始一个事务
  - **入队：**将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面。
  - **执行：**由EXEC命令触发事务
- **3特性**
  - 单独的隔离操作：事务中的所有命令都会被序列化、按顺序地执行。事务在执行的构成中，不会被其他客户端发送来的命令请求所打断。
  - 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题。
  - 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。



------

### P8: Redis消息发布订阅

- **概念:**
  - 进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息。





------

### P9: Redis主从复制

- **概念:**	
  - 就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slver机制，Master以写为主，Slave以读为主。
- **作用:**
  - 读写分离
  - 容灾恢复

- **具体操作:**
  - 配从不配主
  - 从库配置:
    - 🔑: slaveof 主库IP 主库端口 
    - 每次与master断开之后,需要重新配置连接,除非将配置写进redis.conf文件中
    - 🔑: Info replication : 查看主从配置信息
  - 添加新从库操作:
    - 拷贝多个redis.conf文件
    - 开启daemonize yes
    - Pid文件名字
    - 指定端口
    - 更改log文件名字
    - 更改dump.rdb文件名字
  - **常用三招:**
    - **一主二从**
      - <img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vamFsbGVua3dvbmcvTGVhcm5SZWRpcy9yYXcvbWFzdGVyL2ltYWdlLzM0LnBuZw?x-oss-process=image/format,png" alt="img" style="zoom:50%;" />
      - 主从问题演示
        - 切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的123是否也可以复制？
          答：从头开始复制；123也可以复制
        - 从机是否可以写？set可否？
          答：从机不可写，不可set，主机可写
        - 主机shutdown后情况如何？从机是上位还是原地待命
          答：从机还是原地待命（咸鱼翻身，还是咸鱼）
        - 主机又回来了后，主机新增记录，从机还能否顺利复制？
          答：能
        - 其中一台从机down后情况如何？依照原有它能跟上大部队吗？
          答：不能跟上，每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件（具体位置：redis.conf搜寻#### REPLICATION ####）
    - **薪火相传 ** (去中心化
      - 上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力
      - 中途变更转向：会清除之前的数据，重新建立拷贝最新的
      - 🔑: slaveof 新主库IP 新主库端口
    - **反客为主:**
      - 🔑: SLAVEOF no one
      - 使当前数据库停止与其他数据库的同步，转成主数据库
    - **复制原理:**
      - slave启动成功连接到master后会发送一个sync命令
      - master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
      - 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
      - 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
      - 但是只要是重新连接master，一次完全同步（全量复制)将被自动执行
    - **哨兵模式:**
      - **概念:**
        - 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
      - **具体操作:**
        - 调整结构，6379带着6380、6381
        - bin目录,新建sentinel.conf文件，名字绝不能错
        - 配置哨兵,填写内容
          - sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
          - 上面最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机
        - 启动哨兵
          - redis-sentinel /sentinel.conf（上述目录依照各自的实际情况配置，可能目录不同）
        - 正常主从演示
        - 原有的master挂了
        - 投票新选
        - 重新主从继续开工，info replication查查看
        - 如果down掉的master重启回来,会变成slave
    - **复制存在的问题:**
      - 复制延时:
        - 由于所有的写操作都是先在Master上操作，然后同步更新到slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。













