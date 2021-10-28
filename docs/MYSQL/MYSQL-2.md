## D1: 体系结构

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2017%201635392537%201635392537231%20tml5tO%20image-20210702111824314.png" alt="image-20210702111824314" style="zoom:50%;float:left" />

- 整个MySQL Server由以下组成
  - Connection Pool : 连接池组件
  - Management Services & Utilities : 管理服务和工具组件 
  - SQL Interface : SQL接口组件
  - Parser : 查询分析器组件
  - Optimizer : 优化器组件
  - Caches & Buffers : 缓冲池组件
  - Pluggable Storage Engines : 存储引擎
  - File System : 文件系统

**1) 连接层**

最上层是一些客户端和链接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于 TCP/IP的通 信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安 全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证 它所具有的操作权限。

**2) 服务层**

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数 的执行。所有跨存储引擎的功能也在这一层实现，如 过程、函数等。在该层，服务器会解析查询并创建相应的内部 解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等， 最后生成相应的执行操作。如果是 select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升 系统的性能。

**3) 引擎层**

存储引擎层， 存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存
储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。 

**4) 存储层**

数据存储层， 主要是将数据存储在文件系统之上，并完成与存储引擎的交互。

和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。



## D2: 存储引擎

### P1: 概述

和大多数的数据库不同, MySQL中有一个存储引擎的概念, 针对不同的存储需求可以选择最优的存储引擎。 

存储引擎就是**存储数据，建立索引，更新查询数据等等技术的实现方式** 。存储引擎是**基于表**的，而不是基于库的。
所以存储引擎也可被称为表类型。

Oracle，SqlServer等数据库只有一种存储引擎。MySQL提供了插件式的存储引擎架构。所以MySQL存在多种存储引擎，可以根据需要使用相应引擎，或者编写存储引擎。

MySQL5.0支持的存储引擎包含 : InnoDB 、MyISAM 、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、 ARCHIVE、CSV、BLACKHOLE、FEDERATED等，其中InnoDB和BDB提供事务安全表，其他存储引擎是非事务安 全表。

可以通过指定 show engines ， 来查询当前数据库支持的存储引擎 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2019%201635392539%201635392539381%20J37zCO%20image-20210702113821904.png" alt="image-20210702113821904" style="zoom:50%;float:left" />

创建新表时如果不指定存储引擎，那么系统就会使用默认的存储引擎，MySQL5.5之前的默认存储引擎是 MyISAM，5.5之后就改为了InnoDB。

查看Mysql数据库默认的存储引擎 ， 指令 :

```mysql
show variables like '%storage_engine%' ;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2020%201635392540%201635392540332%20Lr9FdT%20image-20210702113904631.png" alt="image-20210702113904631" style="zoom:50%;float:left" />



### P2: 各种存储引擎特性

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2021%201635392541%201635392541012%20obo0Sa%20image-20210702114231565.png" alt="image-20210702114231565" style="zoom:50%;float:left" />



#### d1: InnoDB

InnoDB存储引擎是Mysql的默认存储引擎。InnoDB存储引擎提供了具有提交、回滚、崩溃恢复能力的事务安全。
但是对比MyISAM的存储引擎，InnoDB写的处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引。

InnoDB不同于其他存储引擎的特点:

##### O1: 事务控制

```mysql
-- 环境
create table goods_innodb(
id int NOT NULL AUTO_INCREMENT, name varchar(20) NOT NULL, primary key(id)
)ENGINE=innodb DEFAULT CHARSET=utf8;

-- 测试
start transaction;
	
	insert into goods_innodb(id,name)values(null,'Meta20'); 

commit;
```



##### O2: 外键约束

MySQL支持外键的存储引擎只有InnoDB ， 在创建外键的时候， 要求父表必须有对应的索引 ， 子表在创建外键的时候， 也会自动的创建对应的索引。

下面两张表中 ， country_innodb是父表 ， country_id为主键索引，city_innodb表是子表，country_id字段为外 键，对应于country_innodb表的主键country_id 。

```mysql
create table country_innodb(
	country_id int NOT NULL AUTO_INCREMENT, 
  country_name varchar(100) NOT NULL, 
  primary key(country_id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

create table city_innodb(
    city_id int NOT NULL AUTO_INCREMENT,
    city_name varchar(50) NOT NULL,
    country_id int NOT NULL,
    primary key(city_id),
    key idx_fk_country_id(country_id),
    CONSTRAINT `fk_city_country` FOREIGN KEY(country_id) REFERENCES
country_innodb(country_id) ON DELETE RESTRICT ON UPDATE CASCADE
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into country_innodb values(null,'China'),(null,'America'),(null,'Japan');
insert into city_innodb values(null,'Xian',1),(null,'NewYork',2),
(null,'BeiJing',1);
```

在创建索引时， 可以指定在删除、更新父表时，对子表进行的相应操作，包括 RESTRICT、CASCADE、SET NULL 和 NO ACTION。

RESTRICT和NO ACTION相同， 是指限制在子表有关联记录的情况下， 父表不能更新; 

CASCADE表示父表在更新或者删除时，更新或者删除子表对应的记录;

SET NULL 则表示父表在更新或者删除的时候，子表的对应字段被SET NULL 。

针对上面创建的两个表， 子表的外键指定是ON DELETE RESTRICT ON UPDATE CASCADE 方式的， 那么在主表删除记录的时候， 如果子表有对应记录， 则不允许删除， 主表在更新记录的时候，如果子表有对应记录，则子表对应更新 。

表中数据如图:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2022%201635392542%201635392542115%201hGqaG%20image-20210702204442220.png" alt="image-20210702204442220" style="zoom:50%;float:left" />

查看外键信息:

```mysql
show create table city_innodb ;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2022%201635392542%201635392542899%20SFoqbX%20image-20210702204639790.png" alt="image-20210702204639790" style="zoom:50%;float:left" />

删除country_id为1 的country数据: (无法删除,因为子表有记录)

```mysql
delete from country_innodb where country_id = 1;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2023%201635392543%201635392543562%20psGQ1B%20image-20210702205301826.png" alt="image-20210702205301826" style="zoom:50%;float:left" />

更新主表country表的字段 country_id :

```mysql
update country_innodb set country_id = 100 where country_id = 1;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2024%201635392544%201635392544123%20htQJiZ%20image-20210702205400424.png" alt="image-20210702205400424" style="zoom:50%;float:left" />

更新后， 子表的数据信息为 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2024%201635392544%201635392544769%20AxPNjj%20image-20210702205421946.png" alt="image-20210702205421946" style="zoom:50%;float:left" />



##### O3: 存储方式

InnoDB 存储表和索引有以下两种方式 :
1. 使用共享表空间存储， 这种方式创建的表的表结构保存在.frm文件中， 数据和索引保存在 innodb_data_home_dir 和 innodb_data_file_path定义的表空间中，可以是多个文件。
2. 使用多表空间存储， 这种方式创建的表的表结构仍然存在 .frm 文件中，但是每个表的数据和索引单独保存在 .ibd 中。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2025%201635392545%201635392545414%20C8NYwF%20image-20210702205527456.png" alt="image-20210702205527456" style="zoom:50%;float:left" />



#### d2: MyISAM

MyISAM **不支持事务**、也**不支持外键**，其优势是**访问的速度快**，对事务的完整性没有要求或者以SELECT、INSERT
为主的应用基本上都可以使用这个引擎来创建表 。有以下两个比较重要的特点:

**文件存储方式**

每个MyISAM在磁盘上存储成3个文件，其文件名都和表名相同，但拓展名分别是 : 

- .frm (存储表定义);
- .MYD(MYData , 存储数据);
- .MYI(MYIndex , 存储索引);



#### d3: MEMORY

Memory存储引擎将表的数据存放在**内存**中。每个MEMORY表实际对应一个磁盘文件，格式是.frm ，该文件中只存储表的结构，而其数据文件，都是存储在内存中，这样有利于数据的快速处理，提高整个表的效率。MEMORY 类型的表访问非常地快，因为他的数据是存放在内存中的，并且默认使用HASH索引 ， 但是服务一旦关闭，表中的数据就会丢失。



#### d4: MERGE

MERGE存储引擎是一组MyISAM表的组合，这些MyISAM表必须结构完全相同，MERGE表本身并没有存储数据，对
MERGE类型的表可以进行查询、更新、删除操作，这些操作实际上是对内部的MyISAM表进行的。

对于MERGE类型表的插入操作，是通过INSERT_METHOD子句定义插入的表，可以有3个不同的值，使用FIRST 或 LAST 值使得插入操作被相应地作用在第一或者最后一个表上，不定义这个子句或者定义为NO，表示不能对这个 MERGE表执行插入操作。

可以对MERGE表进行DROP操作，但是这个操作只是删除MERGE表的定义，对内部的表是没有任何影响的。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2028%201635392548%201635392548803%20A50Owy%20image-20210702210650068.png" alt="image-20210702210650068" style="zoom:50%;float:left" />



**示例:**

1). 创建3个测试表 order_1990, order_1991, order_all , 其中order_all是前两个表的MERGE表 :

```mysql
create table order_1990( order_id int ,
    order_money double(10,2),
    order_address varchar(50),
    primary key (order_id)
)engine = myisam default charset=utf8;

create table order_1991(
    order_id int ,
    order_money double(10,2),
    order_address varchar(50),
    primary key (order_id)
)engine = myisam default charset=utf8;

create table order_all(
    order_id int ,
    order_money double(10,2),
    order_address varchar(50),
    primary key (order_id)
)engine = merge union = (order_1990,order_1991) INSERT_METHOD=LAST default
charset=utf8;
 
```

2). 分别向两张表中插入记录

```mysql
insert into order_1990 values(1,100.0,'北京'); 
insert into order_1990 values(2,100.0,'上海');

insert into order_1991 values(10,200.0,'北京'); 
insert into order_1991 values(11,200.0,'上海');
```

3). 查询3张表中的数据。

order_1990中的数据 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2029%201635392549%201635392549573%203d6usG%20image-20210702210914896.png" alt="image-20210702210914896" style="zoom:50%;float:left" />

order_1991中的数据 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2030%201635392550%201635392550155%20Qqydxb%20image-20210702210935372.png" alt="image-20210702210935372" style="zoom:50%;float:left" />

order_all中的数据 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2030%201635392550%201635392550736%20yYBlgm%20image-20210702210956147.png" alt="image-20210702210956147" style="zoom:50%;float:left" />

 4). 往order_all中插入一条记录 ，由于在MERGE表定义时，INSERT_METHOD 选择的是LAST，那么插入的数据会向最后一张表中插入。

```mysql
insert into order_all values(100,10000.0,'西安');
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2031%201635392551%201635392551361%20ZaT2co%20image-20210702211036786.png" alt="image-20210702211036786" style="zoom:50%;float:left" />



### P3: 存储引擎的选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选
择多种存储引擎进行组合。以下是几种常用的存储引擎的使用环境。

- **InnoDB** : 
  - 是Mysql的默认存储引擎，用于事务处理应用程序，支持外键。如果应用**对事务的完整性有比较高的要求**，在并发条件下要求**数据的一致性**，数据操作除了插入和查询意外，还包含很多的更新、删除操作， 那么InnoDB存储引擎是比较合适的选择。InnoDB存储引擎除了有效的降低由于删除和更新导致的锁定， 还可以确保事务的完整提交和回滚，对于类似于计费系统或者财务系统等对数据准确性要求比较高的系统， InnoDB是最合适的选择。
- **MyISAM** : 
  - 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且**对事务的完整性、并发性要求不是很高**，那么选择这个存储引擎是非常合适的。 
- **MEMORY**:
  - 将所有数据保存在RAM中，在需要快速定位记录和其他类似数据环境下，可以提供极快的访问。 MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，其次是要确保表的数据可以恢复，据库异常终止后表中的数据是可以恢复的。MEMORY表通常用于更新不太频繁的小表，用以快速得到访问结 果。 
- **MERGE**:
  - 用于将一系列等同的MyISAM表以逻辑方式组合在一起，并作为一个对象引用他们。MERGE表的优 点在于可以突破对单个MyISAM表的大小限制，并且通过将不同的表分布在多个磁盘上，可以有效的改善 MERGE表的访问效率。这对于存储诸如数据仓储等VLDB环境十分合适。



## D3: 优化sql步骤

在应用的的开发过程中，由于初期数据量小，开发人员写 SQL 语句时更重视功能上的实现，但是当应用系统正式上线后，随着生产数据量的急剧增长，很多 SQL 语句开始逐渐显露出性能问题，对生产的影响也越来越大，此时这些有问题的 SQL 语句就成为整个系统性能的瓶颈，因此我们必须要对它们进行优化，本章将详细介绍在 MySQL 中优化 SQL 语句的方法。

当面对一个有 SQL 性能问题的数据库时，我们应该从何处入手来进行系统的分析，使得能够尽快定位问题SQL 并 尽快解决问题。

### P1: 查看SQL执行频率

MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。

show [session|global] status 可以根据需要加上参数“session”或者“global”来显示 session 级(当前连接)的结果和 global 级(自数据库上次启动至今)的统计结果。如果不写，默认使用参数是“session”。

下面的命令显示了当前 session 中所有统计参数的值:

```mysql
show status like 'Com_______';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2032%201635392552%201635392552142%20NJodi9%20image-20210702212355196.png" alt="image-20210702212355196" style="zoom:50%;float:left" />

```mysql
show status like 'Innodb_rows_%';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2032%201635392552%201635392552915%204VTgE7%20image-20210702212458044.png" alt="image-20210702212458044" style="zoom:50%;float:left" />

Com_xxx 表示每个 xxx 语句执行的次数，我们通常比较关心的是以下几个统计参数。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2034%201635392554%201635392554483%20Zy1TX5%20image-20210702212530375.png" alt="image-20210702212530375" style="zoom:50%;float:left" />

Com_*** : 这些参数对于所有存储引擎的表操作都会进行累计。
Innodb_*** : 这几个参数只是针对InnoDB 存储引擎的，累加的算法也略有不同。



### P2: 定位低效率执行SQL

- **慢查询日志** : 通过慢查询日志定位那些执行效率较低的 SQL 语句，用--log-slow-queries[=file_name]选项启动时，mysqld 写一个包含所有执行时间超过 long_query_time 秒的 SQL 语句的日志文件。

- **show processlist** : 慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询 日志并不能定位问题，可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态、是否 锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2035%201635392555%201635392555352%20j14UjV%20image-20210702213149408.png" alt="image-20210702213149408" style="zoom:50%;float:left" />

```mysql
1) id列，用户登录mysql时，系统分配的"connection_id"，可以使用函数connection_id()查看 

2) user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句

3) host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户

4) db列，显示这个进程目前连接的是哪个数据库

5) command列，显示当前连接的执行的命令，一般取值为休眠(sleep)，查询(query)，连接 (connect)等

6) time列，显示这个状态持续的时间，单位是秒

7) state列，显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态才可以完成

8) info列，显示这个sql语句，是判断问题语句的一个重要依据
```



### P3: explain 分析 ✨

通过以上步骤查询到效率低的 SQL 语句后，可以通过 EXPLAIN或者 DESC命令获取 MySQL如何执行 SELECT 语句
的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

查询SQL执行计划:

```mysql
explain select * from tb_item where id = 1;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2036%201635392556%201635392556070%20EWJtyt%20image-20210702213745664.png" alt="image-20210702213745664" style="zoom:50%;float:left" />

```mysql
explain select * from tb_item where title = '阿尔卡特 (OT-979) 冰川白 联通3G手机3';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2036%201635392556%201635392556674%20qwmA7c%20image-20210702213818179.png" alt="image-20210702213818179" style="zoom:50%;float:left" />

**概览:**

| **字段**      | **含义**                                                     |
| :------------ | :----------------------------------------------------------- |
| id            | select查询的序列号,是一组数字,表示的是查询中执行select子句或者是操作表的顺序 |
| select_type   | 表示select类型,常见的值为SIMPLE(简单表,即不适用表连接或者子查询),PRIMARY(主查询,即外层的查询),UNION(union中的第二个或者后面的查询语句),SUBQUERY(子查询中的第一个select)等 |
| table         | 输出结果集的表                                               |
| type          | 表示访问类型                                                 |
| possible_keys | 查询时可能用到的索引                                         |
| key           | 查询时实际用到的索引                                         |
| key_len       | 索引字段长度                                                 |
| rows          | 扫描行的数量                                                 |
| extra         | 执行情况的说明和描述                                         |



**示例:**

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2037%201635392557%201635392557469%20WnUB0s%20image-20210702214848468.png" alt="image-20210702214848468" style="zoom:50%;" />

```mysql
CREATE TABLE `t_role` (
`id` varchar(32) NOT NULL,
`role_name` varchar(255) DEFAULT NULL, `role_code` varchar(255) DEFAULT NULL, `description` varchar(255) DEFAULT NULL, PRIMARY KEY (`id`),
UNIQUE KEY `unique_role_name` (`role_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `t_user` (
  `id` varchar(32) NOT NULL,
  `username` varchar(45) NOT NULL,
  `password` varchar(96) NOT NULL,
  `name` varchar(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_user_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `user_role` (
  `id` int(11) NOT NULL auto_increment ,
  `user_id` varchar(32) DEFAULT NULL,
  `role_id` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_ur_user_id` (`user_id`),
  KEY `fk_ur_role_id` (`role_id`),
  CONSTRAINT `fk_ur_role_id` FOREIGN KEY (`role_id`) REFERENCES `t_role` (`id`) ON
DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `fk_ur_user_id` FOREIGN KEY (`user_id`) REFERENCES `t_user` (`id`) ON
DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into `t_user` (`id`, `username`, `password`, `name`) values('1','super','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe',' 超级管理员');
insert into `t_user` (`id`, `username`, `password`, `name`) values('2','admin','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe',' 系统管理员');
insert into `t_user` (`id`, `username`, `password`, `name`) values('3','itcast','$2a$10$8qmaHgUFUAmPR5pOuWhYWOr291WJYjHelUlYn07k5ELF8ZCrW0Cui', 'test02');
insert into `t_user` (`id`, `username`, `password`, `name`) values('4','stu1','$2a$10$pLtt2KDAFpwTWLjNsmTEi.oU1yOZyIn9XkziK/y/spH5rftCpUMZa','学 生1');
insert into `t_user` (`id`, `username`, `password`, `name`) values('5','stu2','$2a$10$nxPKkYSez7uz2YQYUnwhR.z57km3yqKn3Hr/p1FR6ZKgc18u.Tvqm','学 生2');
insert into `t_user` (`id`, `username`, `password`, `name`) values('6','t1','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','老师 1');

INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('5','学 生','student','学生');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('7','老 师','teacher','老师');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('8','教 学管理员','teachmanager','教学管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('9','管 理员','admin','管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('10','超 级管理员','super','超级管理员');

INSERT INTO user_role(id,user_id,role_id) VALUES(NULL, '1', '5'),(NULL, '1', '7'),
(NULL, '2', '8'),(NULL, '3', '9'),(NULL, '4', '8'),(NULL, '5', '10') ;
```



#### d1: explain 之 id

d 字段是 select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。id 情况有三种:

1) id 相同表示加载表的顺序是从上到下。

```mysql
explain select * from t_role r, t_user u, user_role ur where r.id = ur.role_id and u.id = ur.user_id ;
```

![image-20210702215234091](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2038%201635392558%201635392558259%20TKCQHR%20image-20210702215234091.png)

2) id 不同id值越大，优先级越高，越先被执行。

```mysql
EXPLAIN SELECT * FROM t_role WHERE id = (SELECT role_id FROM user_role WHERE user_id = (SELECT id FROM t_user WHERE username = 'stu1'))
```

![image-20210702215317547](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2039%201635392559%201635392559020%20ashSIQ%20image-20210702215317547.png)

3) id 有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行;在所有的组中，id的值越 大，优先级越高，越先执行。

```mysql
EXPLAIN SELECT * FROM t_role r , (SELECT * FROM user_role ur WHERE ur.`user_id` = '2') a WHERE r.id = a.role_id ;
```

![image-20210702215406434](https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2039%201635392559%201635392559607%20h4pUv7%20image-20210702215406434.png)



#### d2: explain 之 select_type

表示 SELECT 的类型，常见的取值，如下表所示:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2040%201635392560%201635392560242%20eG4iRy%20image-20210702215448018.png" alt="image-20210702215448018" style="zoom:50%;float:left" />



#### d3: explain 之 type ✨

`表示访问类型,较为重要的指标`

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2041%201635392561%201635392561035%20ppHy8d%20image-20210702215721265.png" alt="image-20210702215721265" style="zoom:50%;float:left" />

结果值从最好到最坏以此是:

```mysql
NULL > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

system > const > eq_ref > ref > range > index > ALL

-- 一般来说,我们需要保证查询至少达到range级别,最好达到ref
```



#### d4: explain 之 key

```mysql
possible_keys : 显示可能应用在这张表的索引， 一个或多个。 

key : 实际使用的索引， 如果为NULL， 则没有使用索引。

key_len : 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前 提下， 长度越短越好 。
```



#### d5: explain 之 extra

其他的额外的执行计划信息，在该列展示 。

| **extra**        | **含义**                                                     |
| ---------------- | ------------------------------------------------------------ |
| using filesort   | 说明mysql会对数据使用一个外部的索引排序,而不是按照表内的索引顺序进行读取,称为"文件排序",效率低 |
| using temeporary | 使用了临时表保存中间结果,mysql在对查询结果排序时使用临时表.常见于order by 和group by,效率低 |
| using index      | 表示相应的select操作使用了覆盖索引,避免访问表的数据行,效率不错 |





### P4: show profile 分析 sql

Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。show profiles 能够在做SQL优化时
帮助我们了解时间都耗费到哪里去了。

```mysql
-- have_profiling 查看当前mysql是否支持profile
select @@have_prifiling	
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2041%201635392561%201635392561928%20w89g89%20image-20210703143915123.png" alt="image-20210703143915123" style="zoom:50%;float:left" />

默认profiling是关闭的，可以通过set语句在Session级别开启profiling:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2042%201635392562%201635392562600%205r97kL%20image-20210703143938247.png" alt="image-20210703143938247" style="zoom:50%;float:left" />

```mysql
set profiling=1; //开启profiling 开关;
```

通过profile，我们能够更清楚地了解SQL执行的过程。

**示例:**

```mysql
show databases;
use db01;
show tables;
select * from tb_item where id < 5; 
select count(*) from tb_item;
```

执行完上述命令之后，再执行show profiles 指令， 来查看SQL语句执行的耗时:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2043%201635392563%201635392563082%20lmPZ7T%20image-20210703144105572.png" alt="image-20210703144105572" style="zoom:50%;float:left" />

通过show profile for query query_id 语句可以查看到该SQL执行过程中每个线程的状态和消耗的时间:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2043%201635392563%201635392563815%20kItyRA%20image-20210703144145885.png" alt="image-20210703144145885" style="zoom:50%;float:left" />

[注]: Sending data 状态表示MySQL线程开始访问数据行并把结果返回给客户端，而不仅仅是返回给客户端。由于
在Sending data状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。

在获取到最消耗时间的线程状态后，MySQL支持进一步选择all、cpu、block io 、context switch、page faults等 明细类型类查看MySQL在使用什么资源上耗费了过高的时间。例如，选择查看CPU的耗费时间 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2044%201635392564%201635392564632%204yzCo1%20image-20210703144648300.png" alt="image-20210703144648300" style="zoom:50%;float:Left" />

| **字段**   | **含义**                      |
| ---------- | ----------------------------- |
| Status     | sql语句执行的状态             |
| Duration   | sql执行过程中每一个步骤的耗时 |
| CPU_user   | 当前用户占有的cpu             |
| CPU_system | 系统占有的cpu                 |



### P5: trace分析优化器执行计划

MySQL5.6提供了对SQL的跟踪trace, 通过trace文件能够进一步了解为什么优化器选择A计划, 而不是选择B计划。

打开trace ， 设置格式为JSON，并设置trace最大能够使用的内存大小，避免解析过程中因为默认内存过小而不能 够完整展示

```mysql
SET optimizer_trace="enabled=on",end_markers_in_json=on;
set optimizer_trace_max_mem_size=1000000;
```

```mysql
-- 执行sql
select * from tb_item where id < 4;

--最后， 检查information_schema.optimizer_trace就可以知道MySQL是如何执行SQL的:
select * from information_schema.optimizer_trace\G;
```

```mysql
*************************** 1. row *************************** 
QUERY: select * from tb_item where id < 4
TRACE: {
"steps": [ {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `tb_item`.`id` AS
`id`,`tb_item`.`title` AS `title`,`tb_item`.`price` AS `price`,`tb_item`.`num` AS
`num`,`tb_item`.`categoryid` AS `categoryid`,`tb_item`.`status` AS
`status`,`tb_item`.`sellerid` AS `sellerid`,`tb_item`.`createtime` AS
`createtime`,`tb_item`.`updatetime` AS `updatetime` from `tb_item` where
(`tb_item`.`id` < 4)"
}
] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "(`tb_item`.`id` < 4)",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`tb_item`.`id` < 4)"
}, {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`tb_item`.`id` < 4)"
                },
 # 仅展示部分
```





## D4: SQL优化

### P1: 大批量插入数据

`环境准备:`

```mysql
CREATE TABLE `tb_user_2` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`username` varchar(45) NOT NULL,
`password` varchar(96) NOT NULL,
`name` varchar(45) NOT NULL,
`birthday` datetime DEFAULT NULL,
`sex` char(1) DEFAULT NULL,
`email` varchar(45) DEFAULT NULL,
`phone` varchar(45) DEFAULT NULL,
`qq` varchar(32) DEFAULT NULL,
`status` varchar(32) NOT NULL COMMENT '用户状态', `create_time` datetime NOT NULL,
`update_time` datetime DEFAULT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `unique_user_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ;
```

当使用load 命令导入数据的时候，适当的设置可以提高导入的效率。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2045%201635392565%201635392565668%20zWcdpg%20image-20210704102100837.png" alt="image-20210704102100837" style="zoom:50%;float:left" />

对于 InnoDB 类型的表，有以下几种方式可以提高导入的效率:



#### d1: 主键顺序插入

因为InnoDB类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效的提高导入数据的效率。如果InnoDB表没有主键，那么系统会自动默认创建一个内部列作为主键，所以如果可以给表创建一个主键，将可以利用这点，来提高导入数据的效率。

```mysql
脚本文件介绍 :
sql1.log ----> 主键有序 
sql2.log ----> 主键无序
```

插入ID顺序排列数据:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2046%201635392566%201635392566574%20MmjxJt%20image-20210704102321527.png" alt="image-20210704102321527" style="zoom:50%;float:left" />

插入ID无序排列数据:

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2047%201635392567%201635392567236%20Welk5j%20image-20210704102349047.png" alt="image-20210704102349047" style="zoom:50%;float:left" />



#### d2: 关闭唯一校验

在导入数据前执行 SET UNIQUE_CHECKS=0，关闭唯一性校验，在导入结束后执行SET UNIQUE_CHECKS=1，恢复唯一性校验，可以提高导入的效率。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2047%201635392567%201635392567950%20e48c0K%20image-20210704102559534.png" alt="image-20210704102559534" style="zoom:50%;float:Left" />



#### d3: 手动提交事务

如果应用使用自动提交的方式，建议在导入前执行 SET AUTOCOMMIT=0，关闭自动提交，导入结束后再执行 SET
AUTOCOMMIT=1，打开自动提交，也可以提高导入的效率。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2048%201635392568%201635392568670%20NMU4uP%20image-20210704102709232.png" alt="image-20210704102709232" style="zoom:50%;float:Left" />



### P2: 优化insert语句

`可以采取以下几种优化方案`

- 如果需要同时对一张表插入很多行数据时，应该尽量使用多个值表的insert语句，这种方式将大大的缩减客户端与数据库之间的连接、关闭等消耗。使得效率比分开执行的单个insert语句快。

  ```mysql
  -- 原始方式:
  insert into tb_test values(1,'Tom'); 
  insert into tb_test values(2,'Cat'); 
  insert into tb_test values(3,'Jerry');
  
  -- 优化后:
  insert into tb_test values(1,'Tom'),(2,'Cat')，(3,'Jerry');
  ```

  

- 在事务中进行数据插入。

  ```mysql
  start transaction;
  insert into tb_test values(1,'Tom'); 
  insert into tb_test values(2,'Cat'); 
  insert into tb_test values(3,'Jerry'); 
  commit;
  ```

  

- 数据有序插入

  ```mysql
  -- 原始方式
  insert into tb_test values(4,'Tim'); 
  insert into tb_test values(1,'Tom'); 
  insert into tb_test values(3,'Jerry'); 
  insert into tb_test values(5,'Rose'); 
  insert into tb_test values(2,'Cat');
  
  -- 优化后:
  insert into tb_test values(1,'Tom'); 
  insert into tb_test values(2,'Cat'); 
  insert into tb_test values(3,'Jerry'); 
  insert into tb_test values(4,'Tim'); 
  insert into tb_test values(5,'Rose'); 
  ```

  



### P3: 优化order by语句

`环境准备`

```mysql
CREATE TABLE `emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT, 
  `name` varchar(100) NOT NULL,
  `age` int(3) NOT NULL,
  `salary` int(11) DEFAULT NULL, 
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4;

insert into `emp` (`id`, `name`, `age`, `salary`) values('1','Tom','25','2300');
insert into `emp` (`id`, `name`, `age`, `salary`) values('2','Jerry','30','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('3','Luci','25','2800');
insert into `emp` (`id`, `name`, `age`, `salary`) values('4','Jay','36','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('5','Tom2','21','2200');
insert into `emp` (`id`, `name`, `age`, `salary`) values('6','Jerry2','31','3300');
insert into `emp` (`id`, `name`, `age`, `salary`) values('7','Luci2','26','2700');
insert into `emp` (`id`, `name`, `age`, `salary`) values('8','Jay2','33','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('9','Tom3','23','2400');
insert into `emp` (`id`, `name`, `age`, `salary`)
values('10','Jerry3','32','3100');
insert into `emp` (`id`, `name`, `age`, `salary`) values('11','Luci3','26','2900');
insert into `emp` (`id`, `name`, `age`, `salary`) values('12','Jay3','37','4500');

create index idx_emp_age_salary on emp(age,salary);
```



#### d1: 两种排序方式

-  第一种是通过对返回数据进行排序，也就是通常说的 filesort 排序，所有不是通过索引直接返回排序结果的排序
  都叫 FileSort 排序。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2049%201635392569%201635392569298%20rfO6NJ%20image-20210704103813935.png" alt="image-20210704103813935" style="zoom:50%;float:left" />

- 第二种通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2050%201635392570%201635392570134%20g76rQN%20image-20210704104123328.png" alt="image-20210704104123328" style="zoom:50%;float:left" />

  

  了解了MySQL的排序方式，优化目标就清晰了:尽量减少额外的排序，通过索引直接返回有序数据。where 条件 和Order by 使用相同的索引，并且Order By 的顺序和索引顺序相同， 并且Order by 的字段都是升序，或者都是 降序。否则肯定需要额外的操作，这样就会出现FileSort。



#### d2: FileSort的优化

通过创建合适的索引，能够减少 Filesort 的出现，但是在某些情况下，条件限制不能让Filesort消失，那就需要加快 Filesort的排序操作。

对于Filesort ， MySQL 有两种排序算法:

- **两次扫描算法**: MySQL4.1 之前，使用该方式排序。首先根据条件取出排序字段和行指针信息，然后在排序区 sort buffer 中排序，如果sort buffer不够，则在临时表 temporary table 中存储排序结果。完成排序之后，再根据行指针回表读取记录，该操作可能会导致大量随机I/O操作。
- **一次扫描算法**: 一次性取出满足条件的所有字段，然后在排序区 sort buffer 中排序后直接输出结果集。排序时内存开销较大，但是排序效率比两次扫描算法要高。

MySQL 通过比较系统变量 max_length_for_sort_data 的大小和Query语句取出的字段总大小， 来判定是否那种排序算法，如果max_length_for_sort_data 更大，那么使用第二种优化之后的算法;否则使用第一种。

可以适当提高 sort_buffer_size 和 max_length_for_sort_data 系统变量，来增大排序区的大小，提高排序的效率。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2051%201635392571%201635392571118%20Guv5R0%20image-20210704105043550.png" alt="image-20210704105043550" style="zoom:50%;float:left" />



### P4: 优化group by语句

由于GROUP BY 实际上也同样会进行排序操作，而且与ORDER BY 相比，GROUP BY 主要只是多了排序之后的分 组操作。当然，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算。所以，在 GROUP BY 的实现过程中，与 ORDER BY 一样也可以利用到索引。

- 如果查询包含 group by 但是用户想要避免排序结果的消耗， 则可以执行order by null 禁止排序。如下 :

  ```mysql
  drop index idx_emp_age_salary on emp;
  explain select age,count(*) from emp group by age;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2051%201635392571%201635392571856%20OilHFT%20image-20210704105814085.png" alt="image-20210704105814085" style="zoom:50%;float:left" />

  ```mysql
  -- 优化后
  explain select age,count(*) from emp group by age order by null;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2052%201635392572%201635392572642%20M52y3T%20image-20210704105949023.png" alt="image-20210704105949023" style="zoom:50%;float:left" />

  从上面的例子可以看出，第一个SQL语句需要进行"filesort"，而第二个SQL由于order by null 不需要进行 "filesort"， 而上文提过Filesort往往非常耗费时间。

创建索引:

```mysql
create index idx_emp_age_salary on emp(age,salary);
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2053%201635392573%201635392573360%20FESLQH%20image-20210704110130164.png" alt="image-20210704110130164" style="zoom:50%;float:left" />



### P5: 优化嵌套查询

Mysql4.1版本之后，开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把 这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL 操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，**有些情况下，子查询是可以被更高效的连接 (JOIN)替代**。

示例: 

```mysql
explain select * from t_user where id in (select user_id from user_role );
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2054%201635392574%201635392574298%20r5pnmE%20image-20210704111327855.png" alt="image-20210704111327855" style="zoom:50%;float:left" />

```mysql
-- 优化后
explain select * from t_user u , user_role ur where u.id = ur.user_id;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2055%201635392575%201635392575326%20olK3IS%20image-20210704111557645.png" alt="image-20210704111557645" style="zoom:50%;float:left" />

连接(Join)查询之所以更有效率一些 ，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。



### P6: 优化or条件

对于包含OR的查询子句，如果要利用索引，则OR之间的每个条件列都必须用到索引 ， 而且不能使用到复合索 引; 如果没有索引，则应该考虑增加索引。

获取emp表中的所有索引:
<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2056%201635392576%201635392576029%20pRsLAu%20image-20210705124955980.png" alt="image-20210705124955980" style="zoom:50%;float:left" />





示例:

```mysql
explain select * from emp where id = 1 or age = 30;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2056%201635392576%201635392576894%20Sr6VyA%20image-20210705125103419.png" alt="image-20210705125103419" style="zoom:50%;float:Left" />

建议使用 union 替换 or :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2058%201635392578%201635392578063%20vpesBC%20image-20210705125200376.png" alt="image-20210705125200376" style="zoom:50%;float:left" />

重要指标主要区别在type和ref两项:

- UNION语句的type值为const,OR语句的type值为range,这是一个明显的差距

- UNION语句的ref值为const,OR语句的type值为null




### P7: 优化分页查询

一般分页查询时，通过创建覆盖索引能够比较好地提高性能。一个常见又非常头疼的问题就是 limit 2000000,10 ， 此时需要MySQL排序前2000010 记录，仅仅返回2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2058%201635392578%201635392578897%206AHytB%20image-20210705130004651.png" alt="image-20210705130004651" style="zoom:50%;float:Left" />



- **优化思路一**

  在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2059%201635392579%201635392579613%20YCNzxB%20image-20210705130214630.png" alt="image-20210705130214630" style="zoom:50%;" />

- **优化思路二**

  该方案适用于主键自增的表,可以把limit查询转换成某个位置的查询

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2006%201635392586%201635392586797%20bqoF4F%20image-20210705130415011.png" alt="image-20210705130415011" style="zoom:50%;" />



### P8: 使用SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的.

#### d1: USE INDEX

在查询语句中表名的后面，添加 use index 来提供希望MySQL去参考的索引列表，就可以让MySQL不再考虑其他 可用的索引。

```mysql
create index idx_seller_name on tb_seller(name);
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2010%201635392590%201635392590870%20KAsgLq%20image-20210705130648261.png" alt="image-20210705130648261" style="zoom:50%;float:left" />

#### d2: IGNORE INDEX

如果用户只是单纯的想让MySQL忽略一个或者多个索引，则可以使用 ignore index 作为 hint 。

```mysql
explain select * from tb_seller ignore index(idx_seller_name) where name = '小米科技';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2011%201635392591%201635392591727%208k6Cxb%20image-20210705130843180.png" alt="image-20210705130843180" style="zoom:50%;float:left" />

#### d3: FORCE INDEX

为强制mysql使用一个特定的索引,可在查询中使用force index作为hint

使用场景:如果一个address地址列(为索引列)中大部分值为"北京",少部分为"南京",在where子句中指定"北京"时,仍然走全表扫描,不会走索引,此时可以强制mysql走索引

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2043%2012%201635392592%201635392592574%20nnBc8J%20image-20210705131440113.png" alt="image-20210705131440113" style="zoom:50%;float:left" />







