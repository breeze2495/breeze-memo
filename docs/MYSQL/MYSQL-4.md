## P1: mysql中常用工具

### d1: mysql

该mysql不是指mysql服务,而是mysql客户端工具

```mysql
mysql [options] [database]
```



#### O1: 连接选项

```mysql
参数 :
    -u, --user=name
    -p, --password[=name]
    -h, --host=name
    -P, --port=#
示例 :
指定用户名 指定密码 指定服务器IP或域名 指定连接端口
mysql -h 127.0.0.1 -P 3306 -u root -p
mysql -h127.0.0.1 -P3306 -uroot -p2143
```



#### O2: 执行选项

```mysql
-e, --execute=name 执行SQL语句并退出
-- 此选项可以在Mysql客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。

mysql -uroot -p2143 db01 -e "select * from tb_book";
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2016%201635392656%201635392656598%20XQuHjP%20image-20210709214851910.png" alt="image-20210709214851910" style="zoom:50%" />



### d2: mysqladmin

mysqladmin 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等.

可以通过 : mysqladmin --help 指令查看帮助文档

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2017%201635392657%201635392657881%20sxjQrZ%20image-20210709215258353.png" alt="image-20210709215258353" style="zoom:50%" />

```mysql
示例 :
    mysqladmin -uroot -p2143 create 'test01';
    mysqladmin -uroot -p2143 drop 'test01';
    mysqladmin -uroot -p2143 version;
```



### d3: mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到 mysqlbinlog 日志管理工具。

```MYSQL
mysqlbinlog [options]  log-files1 log-files2 ...
选项:
-d, --database=name : 指定数据库名称，只列出指定的数据库相关操作。
-o, --offset=# : 忽略掉日志中的前n行命令。
-r,--result-file=name : 将输出的文本格式日志输出到指定文件。
-s, --short-form : 显示简单格式， 省略掉一些信息。
--start-datatime=date1 --stop-datetime=date2 : 指定日期间隔内的所有日志。 
 --start-position=pos1 --stop-position=pos2 : 指定位置间隔内的所有日志。
```



### d4: mysqldump

mysqldump 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句.

```mysql
-- 语法
mysqldump [options] db_name [tables]
mysqldump [options] --database/-B db1 [db2 db3...] 
mysqldump [options] --all-databases/-A
```

```mysql
-- 连接选项:
参数 :
    -u, --user=name
    -p, --password[=name]
    -h, --host=name
    -P, --port=#
指定用户名 指定密码 指定服务器IP或域名 指定连接端口

-- 输出内容选项:
参数:
    --add-drop-database 	在每个数据库创建语句前加上 Drop database 语句
    --add-drop-table 			在每个表创建语句前加上 Drop table 语句 , 默认开启 ; 不开启 (--skip-add-drop-table)
    -n, --no-create-db 		不包含数据库的创建语句
    -t, --no-create-info	不包含数据表的创建语句
    -d --no-data					不包含数据
    -T, --tab=name				自动生成两个文件:一个.sql文件，创建表结构的语句; 一个.txt文件，数据文件，相当于														select into outfile
 
 -- 示例:
    mysqldump -uroot -p2143 db01 tb_book --add-drop-database --add-drop-table > a
    mysqldump -uroot -p2143 -T /tmp test city
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2019%201635392659%201635392659371%20uedRSv%20image-20210709220631583.png" alt="image-20210709220631583" style="zoom:50%" />



### d5: mysqlimport/source

mysqlimport 是客户端数据导入工具，用来导入mysqldump 加 -T 参数后导出的文本文件。

```mysql
-- 语法
mysqlimport [options] db_name textfile1 [textfile2...]

-- 示例
mysqlimport -uroot -p2143 test /tmp/city.txt

-- 如果需要导入sql文件
source /root/tb_book.sql
```



### d6: mysqlshow

mysqlshow 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

```mysql
-- 语法
mysqlshow [options] [db_name [table_name [col_name]]]

-- 参数
--count 显示数据库及表的统计信息(数据库，表 均可以不指定) 
-i 显示指定数据库或者指定表的状态信息

-- 示例
#查询每个数据库的表的数量及表中记录的数量 
mysqlshow -uroot -p2143 --count

#查询test库中每个表中的字段书，及行数 
mysqlshow -uroot -p2143 test --count

#查询test库中book表的详细情况
mysqlshow -uroot -p2143 test book --count
```



## P2: mysql日志

在任何一种数据库中，都会有各种各样的日志，记录着数据库工作的方方面面，以帮助数据库管理员追踪数据库曾 经发生过的各种事件。MySQL 也不例外，在 MySQL 中，有 4 种不同的日志，分别是错误日志、二进制日志 (BINLOG 日志)、查询日志和慢查询日志，这些日志记录着数据库在不同方面的踪迹。

### d1: 错误日志

错误日志是 MySQL 中最重要的日志之一，它**记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息**。当数据库出现任何故障导致无法正常使用时，可以首先查 志。

该日志是默认开启的 ， 默认存放目录为 mysql 的数据目录(var/lib/mysql), 默认的日志文件名为
hostname.err(hostname是主机名)。 

**查看日志位置指令 :**

```mysql
show variables like 'log_error%';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2019%201635392659%201635392659957%20zmZq2f%20image-20210709221523647.png" alt="image-20210709221523647" style="zoom:50%" />

**查看日志内容:**

```mysql
tail -f /var/lib/mysql/xaxh-server.err
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2020%201635392660%201635392660879%20NVma3p%20image-20210709221613652.png" alt="image-20210709221613652" style="zoom:50%" />



### d2: 二进制日志

#### O1: 概述

二进制日志(BINLOG)记录了所有的 DDL(数据定义语言)语句和 DML(数据操纵语言)语句，但是不包括数 据查询语句。此日志对于灾难时的数据恢复起着极其重要的作用，MySQL的主从复制， 就是通过该binlog实现的。

二进制日志，默认情况下是没有开启的，需要到MySQL的配置文件中开启，并配置MySQL日志的格式。

配置文件位置 : /usr/my.cnf

日志存放位置 : 配置时，给定了文件名但是没有指定路径，日志默认写入Mysql的数据目录。

```mysql
#配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 : mysqlbin.000001,mysqlbin.000002
log_bin=mysqlbin

#配置二进制日志的格式 
binlog_format=STATEMENT
```



#### O2: 日志格式

- **STATEMENT**

  该日志格式在日志文件中**记录的都是SQL语句(statement)**，每一条对数据进行修改的SQL都会记录在日志文件中，通过Mysql提供的mysqlbinlog工具，可以清晰的查看到每条语句的文本。主从复制的时候，从库(slave)会将日志解析为原文本，并在从库重新执行一次。

- **ROW**

  该日志格式在日志文件中**记录的是每一行的数据变更**，而不是记录SQL语句。比如，执行SQL语句 : update tb_book set status='1' , 如果是STATEMENT 日志格式，在日志中会记录一行SQL文件; 如果是ROW，由于是对全表进行更新，也就是每一行记录都会发生变更，ROW 格式的日志中会记录每一行的数据变更。

- **MIXED**

  这是目前MySQL默认的日志格式，即混合了STATEMENT 和 ROW两种格式。默认情况下采用STATEMENT，但是在
  一些特殊情况下采用ROW来进行记录。MIXED 格式能尽量利用两种模式的优点，而避开他们的缺点。

#### O3: 日志读取

由于日志以二进制方式存储，不能直接读取，需要用mysqlbinlog工具来查看，语法如下 :

```mysql
mysqlbinlog log-file;
```



- **查看STATEMENT格式日志**

  - 执行插入语句:

  ```mysql
  insert into tb_book values(null,'Lucene','2088-05-01','0');
  ```

  - 查看日志文件:

    <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2021%201635392661%201635392661882%208RuyC6%20image-20210709222332079.png" alt="image-20210709222332079" style="zoom:50%" />

    mysqlbin.index : 该文件是日志索引文件 ， 记录日志的文件名

    mysqlbing.000001 :日志文件

  - 查看日志内容:

    ```mysql
    mysqlbinlog mysqlbing.000001;
    ```

    <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2022%201635392662%201635392662397%20X6A3GD%20image-20210709222606962.png" alt="image-20210709222606962" style="zoom:50%" />



- **查看ROW格式日志**

  ```mysql
  #配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 : mysqlbin.000001,mysqlbin.000002
  log_bin=mysqlbin
  
  #配置二进制日志的格式 
  binlog_format=ROW
  ```

  - 插入数据:

    ```mysql
    insert into tb_book values(null,'SpringCloud实战','2088-05-05','0');
    ```

  - 如果日志格式是 ROW , 直接查看数据 , 是查看不懂的 ; 可以在mysqlbinlog 后面加上参数 -vv

    ```mysql
    mysqlbinlog -vv mysqlbin.000002
    ```

    <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2023%201635392663%201635392663624%20BgtkQa%20image-20210709222745312.png" alt="image-20210709222745312" style="zoom:50%" />



#### O4: 日志删除

对于比较繁忙的系统，由于每天生成日志量大 ，这些日志如果长时间不清除，将会占用大量的磁盘空间。下面我们
将会讲解几种删除日志的常见方法 :

- **方式一**

  通过 Reset Master 指令删除全部 binlog 日志，删除之后，日志编号，将从 xxxx.000001重新开始 。

  查询之前 ，先查询下日志文件 :

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2024%201635392664%201635392664451%20vVzZFE%20image-20210709222848444.png" alt="image-20210709222848444" style="zoom:50%" />

  执行删除指令:

  ```mysql
  Reset Master;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2025%201635392665%201635392665115%20F25rx7%20image-20210709222918062.png" alt="image-20210709222918062" style="zoom:50%" />

- **方式二**

  ```mysql
  执行指令 purge master logs to 'mysqlbin.******' ，该命令将删除 ****** 编号之前的所有日志。
  ```

- **方式三**

  ```mysql
  执行指令 purge master logs before 'yyyy-mm-dd hh24:mi:ss' ，该命令将删除日志为 "yyyy-mm-dd
  hh24:mi:ss" 之前产生的所有日志 。
  ```

- **方式四**

  设置参数 --expire_logs_days=# ，此参数的含义是设置日志的过期天数， 过了指定的天数后日志将会被自动删 除，这样将有利于减少DBA 管理日志的工作量。

  配置如下:

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2025%201635392665%201635392665794%207i1RqG%20image-20210709223049988.png" alt="image-20210709223049988" style="zoom:50%" />

  

### d3: 查询日志

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。 默认情况下， 查询日志是未开启的。如果需要开启查询日志，可以设置以下配置 :

```mysql
#该选项用来开启查询日志 ，  0 代表关闭， 1 代表开启 
general_log=1

#设置日志的文件名 ， 如果没有指定， 默认的文件名为 host_name.log 
general_log_file=file_name
```

在 mysql 的配置文件 /usr/my.cnf 中配置如下内容 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2026%201635392666%201635392666420%20qLVyzq%20image-20210709223435720.png" alt="image-20210709223435720" style="zoom:50%" />

配置完毕之后，在数据库执行以下操作 :

```mysql
select * from tb_book;
select * from tb_book where id = 1;
update tb_book set name = 'lucene入门指南' where id = 5; 
select * from tb_book where id < 8;
```

执行完毕之后， 再次来查询日志文件 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2027%201635392667%201635392667074%202VxrCI%20image-20210709223516579.png" alt="image-20210709223516579" style="zoom:50%" />



### d4: 慢查询日志

慢查询日志记录了所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于 min_examined_row_limit 的所有的SQL语句的日志。long_query_time 默认为 10 秒，最小为 0， 精度可以到微 秒。

#### O1: 文件位置和格式

慢查询日志默认是关闭的 。可以通过两个参数来控制慢查询日志 :

```mysql
# 该参数用来控制慢查询日志是否开启， 可取值: 1 和 0 ， 1 代表开启， 0 代表关闭 
slow_query_log=1

# 该参数用来指定慢查询日志的文件名 
slow_query_log_file=slow_query.log

# 该选项用来配置查询的时间限制， 超过这个时间将认为值慢查询， 将需要进行日志记录， 默认10s 
long_query_time=10
```



#### O2: 日志的读取

和错误日志、查询日志一样，慢查询日志记录的格式也是纯文本，可以被直接读取。

- 查询long_query_time 的值。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2027%201635392667%201635392667778%20rm5FMD%20image-20210709223746704.png" alt="image-20210709223746704" style="zoom:50%" />

- 执行查询操作

  ```mysql
  select id, title,price,num ,status from tb_item where id = 1;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2028%201635392668%201635392668485%20TDpJhp%20image-20210709223825022.png" alt="image-20210709223825022" style="zoom:50%" />

  由于该语句执行时间很短，为0s ， 所以不会记录在慢查询日志中。

  ```MYSQL
  select * from tb_item where title like '%阿尔卡特 (OT-927) 炭黑 联通3G手机 双卡双待 165454%' ;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2029%201635392669%201635392669008%20rOLDwv%20image-20210709223916705.png" alt="image-20210709223916705" style="zoom:50%" />

  该SQL语句 ， 执行时长为 26.77s ，超过10s ， 所以会记录在慢查询日志文件中。

- 查看慢查询日志文件

  直接通过cat 指令查询该日志文件 :

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2029%201635392669%201635392669696%20u0orJP%20image-20210709223952473.png" alt="image-20210709223952473" style="zoom:50%" />

  如果慢查询日志内容很多， 直接查看文件，比较麻烦， 这个时候可以借助于mysql自带的 mysqldumpslow 工 具， 来对慢查询日志进行分类汇总。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2030%201635392670%201635392670461%20DhNNtQ%20image-20210709224013455.png" alt="image-20210709224013455" style="zoom:50%" />



## P3: mysql复制

### d1: 概述

复制是指将主数据库的DDL 和 DML 操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行 (也叫重做)，从而使得从库和主库的数据保持同步。
MySQL支持一台主库同时向多台从库进行复制， 从库同时也可以作为其他从服务器的主库，实现链状复制。



### d2: 原理

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2031%201635392671%201635392671146%20lzSSb4%20image-20210710102715263.png" alt="image-20210710102715263" style="zoom:50%;" />

从上层来看,复制分成三步:

- Master主库在事务提交时,会把数据变更作为时间Events记录在二进制日志文件Binlog中
- 主库推送二进制日志文件Binlog中的日志事件到从库的中继日志Relay Log
- Slave重做中继日志中的事件,将改变反应它自己的数据



### d3: 复制优势

- 主库出现问题,可以快速切换到从库提供服务;
- 可以在从库上执行查询操作,从库中更新,实现读写分离,降低主库的访问压力;
- 可以在从库中执行备份,以避免备份期间影响主库的服务;



### d4: 搭建步骤

#### O1: Master

1. 在master 的配置文件(/usr/my.cnf)中，配置如下内容:

```mysql
#mysql 服务ID,保证整个集群环境中唯一 
server-id=1

#mysql binlog 日志的存储路径和文件名 
log-bin=/var/lib/mysql/mysqlbin

#错误日志,默认已经开启 
#log-err

#mysql的安装目录 
#basedir

#mysql的临时目录 
#tmpdir

#mysql的数据存放目录 
#datadir

#是否只读,1 代表只读, 0 代表读写 
read-only=0

#忽略的数据, 指不需要同步的数据库 
binlog-ignore-db=mysql

#指定同步的数据库 
#binlog-do-db=db01
```

2. 执行完毕之后,需要重启mysql:

```mysql
service mysql restart ;
```

3. 创建同步数据的账户，并且进行授权操作:

```mysql
grant replication slave on *.* to 'itcast'@'192.168.192.131' identified by 'itcast'; 

flush privileges;
```

4. 查看master状态

```mysql
show master status;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2032%201635392672%201635392672366%20Z0hP7S%20image-20210710104230470.png" alt="image-20210710104230470" style="zoom:50%" />

字段含义:

```mysql
File : 从哪个日志文件开始推送日志文件 
Position : 从哪个位置开始推送日志 
Binlog_Ignore_DB : 指定不需要同步的数据库
```



#### O2: Slave

1. 在slave端配置文件中,配置:

```mysql
#mysql服务端ID,唯一 
server-id=2

#指定binlog日志 
log-bin=/var/lib/mysql/mysqlbin
```

2. 执行完毕之后,需要重启mysql:

```mysql
service mysql restart;
```

3. 执行如下命令:

```mysql
change master to master_host= '192.168.192.130', master_user='itcast', 
master_password='itcast', master_log_file='mysqlbin.000001', master_log_pos=413;
-- 指定当前从库对应的主库的IP地址，用户名，密码，从哪个日志文件开始的哪个位置开始同步推送日志。
```

4. 开启同步操作:

```mysql
start slave;
show slave status;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2033%201635392673%201635392673314%20Le9Vqp%20image-20210710104727163.png" alt="image-20210710104727163" style="zoom:50%" />

5. 停止同步操作

```mysql
stop slave;
```



#### O3: 验证同步操作

1. 在主库中创建数据库,数据表,并插入数据:

   ```mysql
   create database db01;
   
   user db01;
   
   create table user(
       id int(11) not null auto_increment,
       name varchar(50) not null,
       sex varchar(1),
     	primary key (id)
   )engine=innodb default charset=utf8;
   
   insert into user(id,name,sex) values(null,'Tom','1');
   insert into user(id,name,sex) values(null,'Trigger','0');
   insert into user(id,name,sex) values(null,'Dawn','1');
   ```

2. 在从库中查询数据,进行验证:

   在从库中，可以查看到刚才创建的数据库:

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2034%201635392674%201635392674164%20jT8mES%20image-20210710104948484.png" alt="image-20210710104948484" style="zoom:50%" />

   在该数据库中，查询user表中的数据:

   <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2044%2034%201635392674%201635392674748%20ZDbkUs%20image-20210710105009180.png" alt="image-20210710105009180" style="zoom:50%" />

