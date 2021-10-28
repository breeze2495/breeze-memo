## D1: 应用优化

前面章节，我们介绍了很多数据库的优化措施。但是在实际生产环境中，由于数据库本身的性能局限，就必须要对前台的应用进行一些优化，来降低数据库的访问压力。

### P1: 使用连接池

对于访问数据库来说，建立连接的代价是比较昂贵的，因为我们频繁的创建关闭连接，是比较耗费资源的，我们有必要建立 数据库连接池，以提高访问的性能。



### P2: 减少对mysql的访问

#### d1: 减少对数据进行重复检索

在编写应用代码时，需要能够理清对数据库的访问逻辑。能够一次连接就获取到结果的，就不用两次连接，这样可以大大减少对数据库无用的重复请求。

```mysql
-- 比如 ，需要获取书籍的id 和name字段 ， 则查询如下:
select id , name from tb_book;

-- 之后在业务逻辑中能够需要用到数据状态信息
select id , status from tb_book;

-- 这要就需要向数据库提交两次请求,数据库就需要做两次查询操作.
select id, name , status from tb_book;
```



#### d2: 增加cache层

在应用中，我们可以在应用中增加缓存层来达到减轻数据库负担的目的。缓存层有很多种，也有很多实现方式，只要能达到降低数据库的负担又能满足应用需求就可以。 

因此可以部分数据从数据库中抽取出来放到应用端以文本方式存储， 或者使用框架(Mybatis, Hibernate)提供的一级缓存/二级缓存，或者使用redis数据库来缓存数据 。



### P3: 负载均衡

负载均衡是应用中使用非常普遍的一种优化方法，它的机制就是利用某种均衡算法，将固定的负载量分布到不同的服务器上， 以此来降低单台服务器的负载，达到优化的效果。

#### d1: 利用mysql复制分流查询

通过MySQL的主从复制，实现读写分离，使增删改操作走主节点，查询操作走从节点，从而可以降低单台服务器的读写压力。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2028%201635392608%201635392608355%20ZbDwM6%20image-20210705142334738.png" alt="image-20210705142334738" style="zoom:50%;" />

#### d2: 采用分布式数据库架构

分布式数据库架构适合大数据量、负载高的情况，它有良好的拓展性和高可用性。通过在多台服务器之间分布数据，可以实现在多台服务器之间的负载均衡，提高访问效率。





## D2: mysql查询缓存优化

### P1: 概述

开启Mysql的查询缓存，当执行完全相同的SQL语句的时候，服务器就会直接从缓存中读取结果，当数据被修改， 之前的缓存会失效，修改比较频繁的表不适合做查询缓存。

### P2: 操作流程

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2029%201635392609%201635392609847%20GqFE2K%20image-20210705142509713.png" alt="image-20210705142509713" style="zoom:50%;" />

 1. 客户端发送一条查询给服务器;
2. 服务器先会检查查询缓存，如果**命中**了缓存，则立即返回存储在缓存中的结果。否则进入下一阶段; 
 3. 服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划;
 4. MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询;
 5. 将结果返回给客户端

### P3: 查询缓存配置

1. 查看对当前mysql数据库是否支持查询缓存;

   ```mysql
   SHOW VARIABLES LIKE 'have_query_cache';
   ```

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2030%201635392610%201635392610513%20s8lWZc%20image-20210705142724759.png" alt="image-20210705142724759" style="zoom:50%;float:left" />

2. 查看mysql是否开启了查询缓存;

   ```mysql
   SHOW VARIABLES LIKE 'query_cache_type';
   ```

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2031%201635392611%201635392611343%208JJFdc%20image-20210705142829494.png" alt="image-20210705142829494" style="zoom:50%;float:left" />

3. 查看查询缓存的占用大小;

   ```mysql
   SHOW VARIABLES LIKE 'query_cache_size';
   ```

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2031%201635392611%201635392611943%20CyZAgq%20image-20210705142914676.png" alt="image-20210705142914676" style="zoom:50%;float:left" />

4. 查看查询缓存的状态变量;

   ```mysql
   SHOW STATUS LIKE 'Qcache%';
   ```

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2032%201635392612%201635392612465%20bLdRdt%20image-20210705143100216.png" alt="image-20210705143100216" style="zoom:50%;float:left" />**

   





各个变量的含义:
<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2033%201635392613%201635392613118%203w5q7K%20image-20210705143214519.png" alt="image-20210705143214519" style="zoom:50%;float:Left" />















### P4: 开启查询缓存

MySQL的查询缓存默认是关闭的，需要手动配置参数 query_cache_type ， 来开启查询缓存。query_cache_type 该参数的可取值有三个 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2034%201635392614%201635392614232%20dEeJ1l%20image-20210705143356504.png" alt="image-20210705143356504" style="zoom:50%;float:Left" />

在 /usr/my.cnf 配置中，增加以下配置 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2034%201635392614%201635392614966%20ev6pN8%20image-20210705143426979.png" alt="image-20210705143426979" style="zoom:50%;float:Left" />

配置完毕之后，重启服务即可生效;

然后就可以在命令行执行SQL语句进行验证 ，执行一条比较耗时的SQL语句，然后再多执行几次，查看后面几次的 执行时间;获取通过查看查询缓存的缓存命中数，来判定是否走查询缓存。



### P5: 查询缓存SELECT选项

`可以在SELECT语句中指定两个与查询缓存相关的选项 :`

- SQL_CACHE : 如果查询结果是可缓存的，并且 query_cache_type 系统变量的值为ON或 DEMAND ，则缓存查询结果。
- SQL_NO_CACHE : 服务器不使用查询缓存。它既不检查查询缓存，也不检查结果是否已缓存，也不缓存查询结果。

```mysql
SELECT SQL_CACHE id, name FROM customer;
SELECT SQL_NO_CACHE id, name FROM customer;
```



### P6: 查询缓存失效的情况

1. SQL 语句不一致的情况， 要想命中查询缓存，查询的SQL语句必须一致。

   ```mysql
   SQL1 : select count(*) from tb_item;
   SQL2 : Select count(*) from tb_item;
   ```

2. 当查询语句中有一些不确定的时，则不会缓存。如 : now() , current_date() , curdate() , curtime() , rand() , uuid() , user() , database() 。

   ```mysql
   SQL1 : select * from tb_item where updatetime < now() limit 1; 
   SQL2 : select user();
   SQL3 : select database();
   ```

3. 不适用任何表查询语句

   ```mysql
   select "A"
   ```

4. 查询 mysql， information_schema或 performance_schema 数据库中的表时，不会走查询缓存。

   ```mysql
   select * from information_schema.engines;
   ```

5. 在存储的函数，触发器或事件的主体内执行的查询。

6. 如果表更改，则使用该表的所有高速缓存查询都将变为无效并从高速缓存中删除。这包括使用 MERGE 映射到 已更改表的表的查询。一个表可以被许多类型的语句，如被改变 INSERT， UPDATE， DELETE， TRUNCATE TABLE， ALTER TABLE， DROP TABLE，或 DROP DATABASE 。



## D3: mysql内存管理及优化

### P1: 内存优化原则

- 将尽量多的内存分配给mysql做缓存,但要给操作系统和其他程序预留足够内存
-  MyISAM 存储引擎的数据文件读取依赖于操作系统自身的IO缓存，因此，如果有MyISAM表，就要预留更多的内存给操作系统做IO缓存。
- 排序区、连接区等缓存是分配给每个数据库会话(session)专用的，其默认值的设置要根据最大连接数合理分配，如果设置太大，不但浪费资源，而且在并发连接较高时会导致物理内存耗尽。



### P2: MyISAM 内存优化

`myisam存储引擎使用 key_buffer 缓存索引块，加速myisam索引的读写速度。对于myisam表的数据块，mysql没有特别的缓存机制，完全依赖于操作系统的IO缓存。`

- **key_buffer_size**

  key_buffer_size决定MyISAM索引块缓存区的大小，直接影响到MyISAM表的存取效率。可以在MySQL参数文件中设置key_buffer_size的值，对于一般MyISAM数据库，建议至少将1/4可用内存分配给key_buffer_size。

  ```mysql
  -- 在 /usr/my.cnf 中做如下配置:
  key_buffer_size=512M
  ```

- **read_buffer_size**

  如果需要经常顺序扫描myisam表，可以通过增大read_buffer_size的值来改善性能。但需要注意的是 read_buffer_size是每个session独占的，如果默认值设置太大，就会造成内存浪费。

- **read_rnd_buffer_size**

  对于需要做排序的myisam表的查询，如带有order by子句的sql，适当增加 read_rnd_buffer_size 的值，可以改善此类的sql性能。但需要注意的是 read_rnd_buffer_size 是每个session独占的，如果默认值设置太大，就会造成内存费。



### P3: InnoDB 内存优化

`innodb用一块内存区做IO缓存池，该缓存池不仅用来缓存innodb的索引块，而且也用来缓存innodb的数据块。`

- **innodb_buffer_pool_size**

  该变量决定了 innodb 存储引擎表数据和索引数据的最大缓存区大小。在保证操作系统及其他程序有足够内存可用的情况下，innodb_buffer_pool_size 的值越大，缓存命中率越高，访问InnoDB表需要的磁盘I/O 就越少，性能也就越高。

  ```mysql
  innodb_buffer_pool_size=512M
  ```

- **innodb_log_buffer_size**

  决定了innodb重做日志缓存的大小，对于可能产生大量更新记录的大事务，增加innodb_log_buffer_size的大小， 可以避免innodb在事务提交前就执行不必要的日志写入磁盘操作。

  ```mysql
  innodb_log_buffer_size=10M
  ```

  



## D4: mysql并发参数调整

从实现上来说，MySQL Server 是多线程结构，包括后台线程和客户服务线程。多线程可以有效利用服务器资源， 提高数据库的并发性能。在Mysql中，控制并发连接和线程的主要参数包括 max_connections、back_log、 thread_cache_size、table_open_cahce。

### P1: max_connections

采用max_connections 控制允许连接到MySQL数据库的最大数量，默认值是 151。如果状态变量 connection_errors_max_connections 不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达到允许最大值而失败，这是可以考虑增大max_connections 的值。

Mysql 最大可支持的连接数，取决于很多因素，包括给定操作系统平台的线程库的质量、内存大小、每个连接的负荷、CPU的处理速度，期望的响应时间等。在Linux 平台下，性能好的服务器，支持 500-1000 个连接不是难事， 需要根据服务器性能进行评估设定。



### P2: back_log

 back_log 参数控制MySQL监听TCP端口时设置的积压请求栈大小。如果MySql的连接数达到max_connections时， 新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过 back_log，将不被授予连接资源，将会报错。5.6.6 版本之前默认值为 50 ， 之后的版本默认为 50 + (max_connections / 5)， 但最大不超过900。

如果需要数据库在较短的时间内处理大量连接请求， 可以考虑适当增大back_log 的值。



### P3: table_open_cache

该参数用来控制所有SQL语句执行线程可打开表缓存的数量， 而在执行SQL语句时，每一个SQL执行线程至少要打 开 1 个表缓存。该参数的值应该根据设置的最大连接数 max_connections 以及每个连接执行关联查询中涉及的表 的最大数量来设定 :

max_connections x N ;



### P4: thread_cache_size

为了加快连接数据库的速度,mysql会缓存一定数量的客户服务线程以备重用,通过参数thread_cache_size可控制mysql缓存客户服务线程的数量



### P5: innodb_lock_wait_timeout

该参数是用来设置InnoDB 事务等待行锁的时间，默认值是50ms ， 可以根据需要进行动态设置。对于需要快速反 馈的业务系统来说，可以将行锁的等待时间调小，以避免事务长时间挂起; 对于后台运行的批量处理程序来说， 可以将行锁的等待时间调大， 以避免发生大的回滚操作。





## D5: mysql锁问题

### P1: 锁概述

锁是计算机协调多个进程或线程并发访问某一资源的机制(避免争抢)。

在数据库中，除传统的计算资源(如 CPU、RAM、I/O 等)的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的 一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。



### P2: 锁分类

从对**数据操作的粒度**分 :

1. 表锁:操作时，会锁定整个表。
2. 行锁:操作时，会锁定当前操作行。

从对**数据操作的类型**分:

1. 读锁(共享锁):针对同一份数据，多个读操作可以同时进行而不会互相影响。
2.  写锁(排它锁):当前操作没有完成之前，它会阻断其他写锁和读锁。



### P3: mysql锁

相对其他数据库而言，MySQL的锁机制比较简单，其最显著的特点是不同的存储引擎支持不同的锁机制。下表中罗 列出了各存储引擎对锁的支持情况:

| **存储引擎** | **表级锁** | 行级锁 | **页面锁** |
| ------------ | ---------- | ------ | ---------- |
| MyISAM       | 支持       | 不支持 | 不支持     |
| InnoDB       | 支持       | 支持   | 不支持     |
| MEMORY       | 支持       | 不支持 | 不支持     |
| BDB          | 支持       | 不支持 | 支持       |

**三种锁的特性**

| **锁类型** | **特点**                                                     |
| ---------- | ------------------------------------------------------------ |
| 表级锁     | 偏向MyISAM存储引擎,开销小,加锁快;不会出现死锁;锁的粒度大,发生锁冲突的概率最高,并发度最低 |
| 行级锁     | 偏向InnoDB存储引擎,开销大,加锁慢;会出现死锁;锁的粒度最小,发生锁冲突的概率最低,并发度最高 |
| 页面锁     | 开销和加锁时间介于表锁和行锁之间;会出现死锁;锁的粒度介于表锁和行锁之间,并发度一般 |

从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适!仅从锁的角度来说:表级 锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web 应用;而行级锁则更适合于有大量按索引 条件并发更新少量不同数据，同时又有并查询的应用，如一些在线事务处理(OLTP)系统。



### P4: MyISAM表锁

MyISAM 存储引擎只支持表锁，这也是MySQL开始几个版本中唯一支持的锁类型。

#### d1: 如何加表锁

MyISAM 在执行查询语句(SELECT)前，会自动给涉及的所有表加读锁，在执行更新操作(UPDATE、DELETE、 INSERT 等)前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此，用户一般不需要直接用 LOCK TABLE 命令给 MyISAM 表显式加锁。

```mysql
-- 显示加表锁语法:
加读锁 : lock table table_name read; 

加写锁 : lock table table_name write; 
```



#### d2: 读锁案例

`准备环境`

```mysql
create database demo_03 default charset=utf8mb4;
use demo_03;
CREATE TABLE `tb_book` (
  `id` INT(11) auto_increment,
  `name` VARCHAR(50) DEFAULT NULL,
  `publish_time` DATE DEFAULT NULL,
  `status` CHAR(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=myisam DEFAULT CHARSET=utf8 ;

INSERT INTO tb_book (id, name, publish_time, status) VALUES(NULL,'java编程思 想','2088-08-01','1');
INSERT INTO tb_book (id, name, publish_time, status) VALUES(NULL,'solr编程思 想','2088-08-08','0');

CREATE TABLE `tb_user` (
  `id` INT(11) auto_increment,
  `name` VARCHAR(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=myisam DEFAULT CHARSET=utf8 ;

INSERT INTO tb_user (id, name) VALUES(NULL,'令狐冲'); 
INSERT INTO tb_user (id, name) VALUES(NULL,'田伯光');
```



- **客户端一**

  1. 获得tb_book 表的读锁

     ```mysql
     lock table tb_book read;
     ```

  2. 执行查询操作

     ```mysql
     select * from tb_book;
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2035%201635392615%201635392615517%20JC95yx%20image-20210708213448439.png" alt="image-20210708213448439" style="zoom:50%;float:left" />

     可以正常执行 ， 查询出数据。

- **客户端二**

  3. 执行查询操作

     ```mysql
     select * from tb_book;
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2035%201635392615%201635392615517%20JC95yx%20image-20210708213448439.png" alt="image-20210708213448439" style="zoom:50%;float:left" />

- **客户端一**

  4. 查询未锁定的表

     ```mysql
     select name from tb_seller;
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2040%201635392620%201635392620578%209tAzU7%20image-20210708213732064.png" alt="image-20210708213732064" style="zoom:50%;float:left" />

- **客户端二**

  5. 查询未锁定的表

     ```mysql
     select name from tb_seller;
     ```

     ![image-20210708213823704](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2042%201635392622%201635392622485%20Btpp6Z%20image-20210708213823704.png)

- **客户端一**

  6. 执行插入操作

     ```mysql
     insert into tb_book values(null,'Mysql高级','2088-01-01','1');
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2043%201635392623%201635392623488%20HbdUZE%20image-20210708214113477.png" alt="image-20210708214113477" style="zoom:50%;float:left" />
     
     执行插入， 直接报错 ， 由于当前tb_book 获得的是读锁， 不能执行更新操作。

- **客户端一**

  7. 执行插入操作

     ```mysql
     insert into tb_book values(null,'Mysql高级','2088-01-01','1');
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2044%201635392624%201635392624232%204Eqlvk%20image-20210708214631635.png" alt="image-20210708214631635" style="zoom:50%;float:left" />

     当在客户端一中释放锁指令 unlock tables 后 ， 客户端二中的 inesrt 语句 ， 立即执行 ;



#### d3: 写锁案例

- **客户端一**

  1. 获得tb_book 表的写锁

     ```mysql
     lock table tb_book write;
     ```

  2. 执行查询操作

     ```mysql
     select * from tb_bool; 
     ```

     <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2044%201635392624%201635392624839%20KbGG9j%20image-20210708214928582.png" alt="image-20210708214928582" style="zoom:50%;" />

     查询成功;

  3. 执行更新操作

     ```mysql
     update tb_book set name='java编程思想(第二版)' where id = 1;
     ```

     ![image-20210708215114401](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2045%201635392625%201635392625595%20y67Fan%20image-20210708215114401.png)

     更新成功;

- **客户端二**

  4. 执行查询操作

     ```mysql
     select * from tb_book;
     ```

     ![image-20210708215220428](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2046%201635392626%201635392626177%20TcFtTr%20image-20210708215220428.png)

当在客户端一中释放锁指令 unlock tables 后 ， 客户端二中的 select 语句 ， 立即执行 ;

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2046%201635392626%201635392626675%20j3ZmxC%20image-20210708215247275.png" alt="image-20210708215247275" style="zoom:50%;" />



**结论:** 

- 读锁会阻塞写,不会阻塞读; 写锁既会阻塞度又会阻塞写;

- MyISAM的读写锁调度是写优先,这也是MyISAM不适合作为写为主的存储引擎的原因.因为写锁后,其他线程不能做任何操作,大量的更新会使查询很难得到锁,从而造成永远阻塞;



#### d5: 查看锁的争用情况

```mysql
show open tables;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2047%201635392627%201635392627338%20MhnMsB%20image-20210708222059172.png" alt="image-20210708222059172" style="zoom:50%;" />

- In_user : 表当前被查询使用的次数。如果该数为零，则表是打开的，但是当前没有被使用。 
- Name_locked:表名称是否被锁定。名称锁定用于取消表或对表进行重命名等操作

```mysql
show status like 'Table_locks%';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2048%201635392628%201635392628651%203Gw1ar%20image-20210708222314923.png" alt="image-20210708222314923" style="zoom:50%;" />

- Table_locks_immediate : 指的是能够立即获得表级锁的次数，每立即获取锁，值加1。
- Table_locks_waited : 指的是不能立即获取表级锁而需要等待的次数，每等待一次，该值加1，此值高说明存在着
  较为严重的表级锁争用情况。



### P5: InnoDB行锁

#### d1: 行锁介绍

- 行锁特点 :偏向InnoDB 存储引擎，开销大，加锁慢;会出现死锁;锁定粒度最小，发生锁冲突的概率最低,并发度也最高。
- InnoDB 与 MyISAM 的最大不同有两点:一是支持事务;二是采用了行级锁

#### d2: 背景知识

`事务是由一组SQL语句组成的逻辑处理单元。`

- **ACID**

| **ACID属性**        | **含义**                                                     |
| ------------------- | ------------------------------------------------------------ |
| 原子性 (Atomicity)  | 事务是一个原子操作单元，其对数据的修改，要么全部成功，要么全部失败。 |
| 一致性 (Consistent) | 事务中包含的处理要满足数据库提前设置的约束，如主键约束或者not null等； |
| 隔离性(Isolation)   | 数据库系统提供一定的隔离机制,保证事务在不受外部并发操作影响的'独立'环境下运行; |
| 持久性(Durable)     | 事务完成之后,对数据的修改时永久的;                           |

- **并发事务处理带来的问题**

| **问题**                         | **含义**                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| 丢失更新(Lost Update)            | 当两个或多个事务选择同一行,最初的事务修改的值,会被后面的事务修改的值覆盖 |
| 脏读(Dirty Reads)                | A事务访问数据并进行修改,未提交;B事务也访问并使用了该数据     |
| 不可重复读(Non-Repeatable Reads) | 一个事务在读取数据后,再一次读取该数据,发现和之前读取的不同   |
| 幻读(Phantom Reads)              | 一个事务按照相同的查询条件重新读取以前查询过的数据,却发现其他事务插入了满足      其查询条件的新数据 |

- **事务隔离级别**

为了解决上述提到的事务并发问题，数据库提供一定的事务隔离机制来解决这个问题。数据库的事务隔离越严格， 并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使用事务在一定程度上“串行化” 进行，这显然 与“并发” 是矛盾的。

| **隔离级别**         | 丢失更新    | 脏读        | 不可重复读 | 幻读 |
| -------------------- | ----------- | ----------- | ---------- | ---- |
| Read UnCommitted     | ❎(不会出现) | ✅(可能出现) | ✅          | ✅    |
| Read Committed       | ❎           | ❎           | ✅          | ✅    |
| Reaptable read(默认) | ❎           | ❎           | ❎          | ✅    |
| Serializable         | ❎           | ❎           | ❎          | ❎    |

- 查看mysql默认隔离界别

  ```mysql
  show variable like 'tx_isolation';	
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2049%201635392629%201635392629376%20GYHs7d%20image-20210709151526412.png" alt="image-20210709151526412" style="zoom:50%;float:left" />



#### d3: InnoDB的行锁模式

`InnoDB实现了以下两种类型的锁`

- 共享锁: 读锁,简称S锁.共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- 排他锁: 写锁,简称X锁,排他锁就是不能与其他锁并存,如一个事务获取了一个数据行的排他锁,其他事务就不能再获取该行的其他锁,包括共享锁和排他锁,但是获取排他锁的事务是可以对数据就行读取和修改。

对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁(X);

对于普通SELECT语句，InnoDB不会加任何锁;

可以通过以下语句显式得给记录集加共享锁或排他锁 。

```mysql
共享锁(S): SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE 
排他锁(X): SELECT * FROM table_name WHERE ... FOR UPDATE
```



#### d4: 行锁基本演示

`环境准备`

```mysql
create table test_innodb_lock( id int(11),
    name varchar(16),
    sex varchar(1)
)engine = innodb default charset=utf8;

insert into test_innodb_lock values(1,'100','1');
insert into test_innodb_lock values(3,'3','1');
insert into test_innodb_lock values(4,'400','0');
insert into test_innodb_lock values(5,'500','1');
insert into test_innodb_lock values(6,'600','0');
insert into test_innodb_lock values(7,'700','0');
insert into test_innodb_lock values(8,'800','1');
insert into test_innodb_lock values(9,'900','1');
insert into test_innodb_lock values(1,'200','0');

create index idx_test_innodb_lock_id on test_innodb_lock(id);
create index idx_test_innodb_lock_name on test_innodb_lock(name);
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2050%201635392630%201635392630082%2056Dkyi%20image-20210709152339221.png" alt="image-20210709152339221" style="zoom:50%;" />



#### d5: 无索引行锁升级为表锁

如果不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，实际效果跟表锁一样。

查看当前表的索引 : 

```mysql
show index from test_innodb_lock ;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2051%201635392631%201635392631446%207iIPS9%20image-20210709152502306.png" alt="image-20210709152502306" style="zoom:50%;" />

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2052%201635392632%201635392632365%207LMw2X%20image-20210709152524745.png" alt="image-20210709152524745" style="zoom:50%;" />

由于执行更新时,name字段本来为varchar类型,我们是作为数组类型使用,存在类型转换,索引失效,最终行锁变为表锁 ;



#### d6: 间隙锁危害

当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据进行加锁; 对于键值在条件范围内但并不存在的记录，叫做 "间隙(GAP)" ， InnoDB也会对这个 "间隙" 加锁，这种锁机制就是所谓的 间隙锁(Next-Key锁) 。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2053%201635392633%201635392633256%20nCFNgP%20image-20210709153306599.png" alt="image-20210709153306599" style="zoom:50%;" />



#### d7: InnoDB行锁争用情况

```mysql
show status like 'innodb_row_lock%';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2054%201635392634%201635392634324%20rYDK7S%20image-20210709154016985.png" alt="image-20210709154016985" style="zoom:50%;float:left" />

```mysql
Innodb_row_lock_current_waits: 当前正在等待锁定的数量 
Innodb_row_lock_time: 从系统启动到现在锁定总时间长度 
Innodb_row_lock_time_avg:每次等待所花平均时长 
Innodb_row_lock_time_max:从系统启动到现在等待最长的一次所花的时间 
Innodb_row_lock_waits: 系统启动后到现在总共等待的次数

当等待的次数很高，而且每次等待的时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根
据分析结果着手制定优化计划。
```



#### d8: 总结

InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面带来了性能损耗可能比表锁会更高一些，但是在整体并发处理能力方面要远远由于MyISAM的表锁的。当系统并发量较高的时候，InnoDB的整体性能和MyISAM 相比就会有比较明显的优势。
但是，InnoDB的行级锁同样也有其脆弱的一面，当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

**优化建议:**

- 尽可能让所有数据检索都能通过索引来完成，避免无索引行锁升级为表锁。 
- 合理设计索引，尽量缩小锁的范围 
- 尽可能减少索引条件，及索引范围，避免间隙锁 
- 尽量控制事务大小，减少锁定资源量和时间长度 
- 尽可使用低级别事务隔离(但是需要业务层面满足需求)



## D6: 常用SQL技巧

### P1: SQL执行顺序

```mysql
-- 编写顺序
SELECT DISTINCT <select list>
FROM
    <left_table> <join_type>
JOIN
    <right_table> ON <join_condition>
WHERE
    <where_condition>
GROUP BY
    <group_by_list>
HAVING
    <having_condition>
ORDER BY
    <order_by_condition>
LIMIT
    <limit_params>
    
-- 执行顺序
FROM <left_table>

ON <join_condition>

<join_type> JOIN <right_table> 

WHERE <where_condition>

GROUP BY <group_by_list>

HAVING      <having_condition>

SELECT DISTINCT     <select list>

ORDER BY    <order_by_condition>

LIMIT       <limit_params>
```



### P2: 正则表达式的使用

正则表达式(Regular Expression)是指一个用来描述或者匹配一系列符合某个句法规则的字符串的单个字符串。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2055%201635392635%201635392635023%20R36s5Z%20image-20210709162031239.png" alt="image-20210709162031239" style="zoom:50%;" />

```mysql
select * from emp where name regexp '^T'; 

select * from emp where name regexp '2$'; 

select * from emp where name regexp '[uvw]';
```



### P3: 常用函数

- **数字函数**

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2055%201635392635%201635392635994%20zuCIHa%20image-20210709162127628.png" alt="image-20210709162127628" style="zoom:50%;" />



- **字符串函数**

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2056%201635392636%201635392636957%20AxY9gz%20image-20210709162143247.png" alt="image-20210709162143247" style="zoom:50%;" />



- **日期函数**

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2058%201635392638%201635392638011%20Ym5TAv%20image-20210709162159880.png" alt="image-20210709162159880" style="zoom:50%;" />



- **聚合函数**

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2059%201635392639%201635392639174%20yiOnPV%20image-20210709162219291.png" alt="image-20210709162219291" style="zoom:50%;" />
