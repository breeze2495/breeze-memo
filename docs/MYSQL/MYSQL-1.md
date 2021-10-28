## D1: 索引

### P1: 概述

MySQL官方对索引的定义为:索引(index)是帮助MySQL高效获取数据的数据结构(有序)。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向)数据， 这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引.



### P2: 优与劣

- **优点:**
  - 类似于书籍的目录索引,**提高数据检索效率**,较低数据库I/O成本
  - 通过索引列对数据进行排序,**降低数据排序的成本**,降低CPU的消耗

- **缺点:**
  - 索引本身也是一张表,该表保存了主键与索引字段,并指向实体类的记录,因此索引自身需占用空间
  - 索引虽然大大提高了查询效率,同时却也降低了更新表的速度,例如对表进行insert,update,delete.因为在更新表时,mysql不仅要保存数据,还要保存索引信息的变化.



### P3: 索引结构

索引是在MySQL的存储引擎层中实现的，而不是在服务器层实现的。所以每种存储引擎的索引都不一定完全相同，
也不是所有的存储引擎都支持所有的索引类型的。MySQL目前提供了以下4种索引:

- **BTREE 索引** : 最常见的索引类型，大部分索引都支持 B 树索引。
- **HASH 索引**: 只有Memory引擎支持 ， 使用场景简单 。
- R-tree 索引(空间索引):空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常 使用较少，不做特别介绍。
- Full-text (全文索引) :全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从 Mysql5.6版本开始支持全文索引。



### P4: 索引分类

- **单值索引:** 即一个索引只包含单个列,一个表可以有多个单列索引
- **唯一索引:** 索引列的值必须唯一,但允许有空值
- **复合索引:** 即一个索引包含多个列



### P5: 索引语法

`准备环境`

```mysql
create database demo_01 default charset=utf8mb4;
use demo_01;
CREATE TABLE `city` (
  `city_id` int(11) NOT NULL AUTO_INCREMENT,
  `city_name` varchar(50) NOT NULL,
  `country_id` int(11) NOT NULL,
  PRIMARY KEY (`city_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `country` (
  `country_id` int(11) NOT NULL AUTO_INCREMENT,
  `country_name` varchar(100) NOT NULL,
 PRIMARY KEY (`country_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into `city` (`city_id`, `city_name`, `country_id`) values(1,'西安',1); 
insert into `city` (`city_id`, `city_name`, `country_id`) values(2,'NewYork',2); 
insert into `city` (`city_id`, `city_name`, `country_id`) values(3,'北京',1); 
insert into `city` (`city_id`, `city_name`, `country_id`) values(4,'上海',1);

insert into `country` (`country_id`, `country_name`) values(1,'China');
insert into `country` (`country_id`, `country_name`) values(2,'America');
insert into `country` (`country_id`, `country_name`) values(3,'Japan');
insert into `country` (`country_id`, `country_name`) values(4,'UK');
 
```



#### d1: 创建索引

```mysql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name [USING index_type]
ON tbl_name(index_col_name,...)
index_col_name : column_name[(length)][ASC | DESC]
```

示例: 为city表中的city_name字段创建索引

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2031%201635392491%201635392491702%201xioej%20image-20210702083158062.png" alt="image-20210702083158062" style="zoom:50%" />



#### d2: 查看索引

```mysql
show index from table_name;
```

示例:查看city表中的索引信息

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2032%201635392492%201635392492568%20raN8Ag%20image-20210702083318456.png" alt="image-20210702083318456" style="zoom:50%" />

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2033%201635392493%201635392493547%20cKYC7S%20image-20210702083403288.png" alt="image-20210702083403288" style="zoom:50%" />



#### d3: 删除索引

```mysql
1 DROP INDEX index_name ON tbl_name;
```

示例:  删除city表上的索引 idx_city_name

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2034%201635392494%201635392494259%20OW8uz9%20image-20210702083516759.png" alt="image-20210702083516759" style="zoom:50%" />



#### d4: alter命令

```mysql
1). alter table tb_name add primary key(column_list); 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL
2). alter table tb_name add unique index_name(column_list); 这条语句创建索引的值必须是唯一的(除了NULL外，NULL可能会出现多次)
3). alter  table  tb_name  add  index index_name(column_list);
添加普通索引， 索引值可以出现多次。
4). alter table tb_name add fulltext index_name(column_list); 该语句指定了索引为FULLTEXT， 用于全文索引
```



### P6: 索引设计原则             

- 对查询频次较高，且数据量比较大的表建立索引。

- 索引字段的选择，最佳候选列应当从where子句的条件中提取，如果where子句中的组合比较多，那么应当挑选最常用、过滤效果最好的列的组合。

- 使用唯一索引，区分度越高，使用索引的效率越高。

- 索引可以有效的提升查询数据的效率，但索引数量不是多多益善，索引越多，维护索引的代价自然也就水涨船高。对于插入、更新、删除等DML操作比较频繁的表来说，索引过多，会引入相当高的维护代价，降低 DML操作的效率，增加相应操作的时间消耗。另外索引过多的话，MySQL也会犯选择困难病，虽然最终仍然会找到一个可用的索引，但无疑提高了选择的代价。

- 使用短索引，索引创建之后也是使用硬盘来存储的，因此提升索引访问的I/O效率，也可以提升总体的访问效 率。假如构成索引的字段总长度比较短，那么在给定大小的存储块内可以存储更多的索引值，相应的可以有效的提升MySQL访问索引的I/O效率。

- 利用最左前缀，N个列组合而成的组合索引，那么相当于是创建了N个索引，如果查询时where子句中使用了组成该索引的前几个字段，那么这条查询SQL可以利用组合索引来提升查询效率。

  ```mysql
  创建复合索引:
  CREATE INDEX idx_name_email_status ON tb_seller(name,email,status);
  
  就相当于
  对name 创建索引 ;
  对name , email 创建了索引 ;
  对name , email, status 创建了索引 ;
  ```



### P7: 索引的使用

索引是数据库优化最常用也是最重要的手段之一, 通过索引通常可以帮助用户解决大多数的MySQL的性能优化问题。

#### d1: 验证索引提升查询效率

在我们准备的表结构tb_item 中， 一共存储了 300 万记录;

- **根据id查询**

```mysql
select * from tb_item where id = 1999\G;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2034%201635392494%201635392494792%20Z2sfhh%20image-20210703150035142.png" alt="image-20210703150035142" style="zoom:50%" />

 查询速度很快， 接近0s ， 主要的原因是因为id为主键， 有索引;

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2035%201635392495%201635392495469%20ok05da%20image-20210703150127386.png" alt="image-20210703150127386" style="zoom:50%" />

- **根据title进行精确查询**

```mysql
select * from tb_item where title = 'iphoneX 移动3G 32G941'\G;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2036%201635392496%201635392496076%20aIg979%20image-20210703150221355.png" alt="image-20210703150221355" style="zoom:50%" />

查看SQL语句的执行计划 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2040%201635392500%201635392500500%20bAI2TB%20image-20210703150316700.png" alt="image-20210703150316700" style="zoom:50%" />

优化: 针对title字段,创建索引

```mysql
create index idx_item_title on tb_item(title);
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2041%201635392501%201635392501210%20Crm6O1%20image-20210703150448838.png" alt="image-20210703150448838" style="zoom:50%" />

索引创建完成之后，再次进行查询 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2041%201635392501%201635392501779%20vowCM9%20image-20210703150647228.png" alt="image-20210703150647228" style="zoom:50%" />

通过explain ， 查看执行计划，执行SQL时使用了刚才创建的索引

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2042%201635392502%201635392502441%20l9q1ib%20image-20210703150712051.png" alt="image-20210703150712051" style="zoom:50%" />



#### d2: 具体使用

##### O1: 环境准备

```mysql
create table `tb_seller` ( 
  	`sellerid` varchar (100), 
  	`name` varchar (100), 
  	`nickname` varchar (50), 
    `password` varchar (60), 
  	`status` varchar (1), 
  	`address` varchar (100), 
  	`createtime` datetime, 
  	primary key(`sellerid`)
)engine=innodb default charset=utf8mb4;

insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('alibaba','阿里巴巴','阿里小 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00'); 
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('baidu','百度科技有限公司','百度小 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00'); 
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('huawei','华为科技有限公司','华为小 店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00'); 
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itcast','传智播客教育科技有限公司','传智播 客','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itheima','黑马程序员','黑马程序 员','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('luoji','罗技科技有限公司','罗技小 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('oppo','OPPO科技有限公司','OPPO官方旗舰 店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('ourpalm','掌趣科技股份有限公司','掌趣小 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('qiandu','千度科技','千度小 店','e10adc3949ba59abbe56e057f20f883e','2','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('sina','新浪科技有限公司','新浪官方旗舰 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('xiaomi','小米科技','小米官方旗舰 店','e10adc3949ba59abbe56e057f20f883e','1','西安市','2088-01-01 12:00:00'); 
 insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('yijia','宜家家居','宜家家居旗舰 店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
 
create index idx_seller_name_sta_addr on tb_seller(name,status,address);
 
```



##### O2: 避免索引失效

- **全值匹配**

  对索引中所有列都执行具体值  `该情况下索引生效,执行效率高`

  ```mysql
  explain select * from tb_seller where name='小米科技' and status='1' and address='北京市'\G;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2043%201635392503%201635392503155%209ab1qe%20image-20210703151430065.png" alt="image-20210703151430065" style="zoom:50%" />



- **最左前缀法则**

  如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始，并且不跳过索引中的列。

  - 匹配最左前缀法则，走索引:

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2044%201635392504%201635392504053%20TPBNrU%20image-20210703151609268.png" alt="image-20210703151609268" style="zoom:50%" />

  - 违法最左前缀法则 ， 索引失效:

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2044%201635392504%201635392504959%20nG2Gba%20image-20210703154739261.png" alt="image-20210703154739261" style="zoom:50%" />
  
  - 如果符合最左法则，但是出现跳跃某一列，只有最左列索引生效:
  
  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2045%201635392505%201635392505665%208K82aX%20image-20210703154841895.png" alt="image-20210703154841895" style="zoom:50%" />



- **范围查询右边的列，不能使用索引**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2046%201635392506%201635392506405%20wBAR1d%20image-20210703154959578.png" alt="image-20210703154959578" style="zoom:50%" />

  根据前面的两个字段name ， status 查询是走索引的， 但是最后一个条件address 没有用到索引。



- **不要在索引列上进行运算操作,索引将失效**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2047%201635392507%201635392507102%20c8pUiw%20image-20210703155259293.png" alt="image-20210703155259293" style="zoom:50%" />



- **字符串不加单引号，造成索引失效。**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2047%201635392507%201635392507937%20MYfSD6%20image-20210703155409031.png" alt="image-20210703155409031" style="zoom:50%" />

  由于在查询时，没有对字符串加单引号，MySQL的查询优化器，会自动的进行类型转换，造成索引失效。



- **尽量使用覆盖索引，避免select ***

  尽量使用覆盖索引(只访问索引的查询(索引列完全包含查询列))，减少select * 。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2048%201635392508%201635392508821%20o5VNas%20image-20210703155522942.png" alt="image-20210703155522942" style="zoom:50%" />

  如果查询列，超出索引列，也会降低性能。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2049%201635392509%201635392509660%20jWGTBi%20image-20210703155739883.png" alt="image-20210703155739883" style="zoom:50%" />

  ```mysql
  using index:使用覆盖索引的时候就会出现
  
  using where:在查找使用索引的情况下，需要回表去查询所需的数据 
  
  using index condition:查找使用了索引，但是需要回表查询数据
  
  using index ; using where:查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据
  ```

  

- **用or分隔开的条件**

  如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

  

  示例，name字段是索引列， 而createtime不是索引列，中间是or进行连接是不走索引的 :

  ```mysql
  explain select * from tb_seller where name='黑马程序员' or createtime = '2088-01-01 12:00:00'\G;
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2050%201635392510%201635392510950%20jPD7XA%20image-20210703161202044.png" alt="image-20210703161202044" style="zoom:50%" />



- **以%开头的Like模糊查询，索引失效。**

  如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2051%201635392511%201635392511752%20YyxzRU%20image-20210703161341267.png" alt="image-20210703161341267" style="zoom:50%" />

  解决: 通过覆盖索引来解决

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2052%201635392512%201635392512865%20Dmd1ea%20image-20210703161504169.png" alt="image-20210703161504169" style="zoom:50%" />



- **如果mysql评估使用索引比全表更慢,则不适用索引**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2053%201635392513%201635392513951%20tHe6ml%20image-20210703161718955.png" alt="image-20210703161718955" style="zoom:50%" />



- **is NULL ， is NOT NULL 有时索引失效**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2041%2054%201635392514%201635392514930%20I6hQg4%20image-20210703161751993.png" alt="image-20210703161751993" style="zoom:50%" />



- **in 走索引， not in 索引失效**

  <img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2000%201635392520%201635392520173%20NDjcea%20image-20210703162009344.png" alt="image-20210703162009344" style="zoom:50%" />



- **单列索引和复合索引**

  尽量使用复合索引，而少使用单列索引。

  ```mysql
  # 创建复合索引
  create index idx_name_sta_address on tb_seller(name, status, address);
  
  就相当于创建了三个索引 :
      name
      name + status
      name + status + address
      
  # 创建单列索引
  create index idx_seller_name on tb_seller(name); 
  create index idx_seller_status on tb_seller(status); 
  create index idx_seller_address on tb_seller(address);    
  
  -- 数据库会选择一个最优的索引(辨识度最高索引)来使用，并不会使用全部索引 。    
  ```

  

#### d3: 查看索引使用情况

```mysql
show status like 'Handler_read%';
show global status like 'Handler_read%';
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2000%201635392520%201635392520894%203Lu9vF%20image-20210703162403285.png" alt="image-20210703162403285" style="zoom:50%" />

```mysql
Handler_read_first:索引中第一条被读的次数。如果较高，表示服务器正执行大量全索引扫描(这个值越低 越好)。

Handler_read_key:如果索引正在工作，这个值代表一个行被索引值读的次数，如果值越低，表示索引得到的 性能改善不高，因为索引不经常使用(这个值越高越好)。

Handler_read_next :按照键顺序读下一行的请求数。如果你用范围约束或如果执行索引扫描来查询索引列， 该值增加。

Handler_read_prev:按照键顺序读前一行的请求数。该读方法主要用于优化ORDER BY ... DESC。

Handler_read_rnd :根据固定位置读一行的请求数。如果你正执行大量查询并需要对结果进行排序该值较高。你可能使用了大量需要MySQL扫描整个表的查询或你的连接没有正确使用键。这个值较高，意味着运行效率低，应该建立索引来补救。

Handler_read_rnd_next:在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明你的表索引不正确或写入的查询没有利用索引。
```







## D2: 视图

### P1: 概述

视图(View)是一种**虚拟存在的表**。视图并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表， 并且是在使用视图时动态生成的。通俗的讲，**视图就是一条SELECT语句执行后返回的结果集**。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。

视图相对于普通的表的优势主要包括以下几项。

- 简单:使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤 好的复合条件的结果集。 
- 安全:使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但 是通过视图就可以简单的实现。 
- 数据独立:一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响;源表 修改列名，则可以通过修改视图来解决，不会造成对访问者的影响。

### P2: 视图语法

#### d1: 创建/修改视图

```mysql
#创建视图
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION]

#修改视图
ALTER [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION]

#选项:
WITH [CASCADED | LOCAL] CHECK OPTION 决定了是否允许更新数据使记录不再满足视图的条件。
LOCAL : 只要满足本视图的条件就可以更新。
CASCADED : 必须满足所有针对该视图的所有视图的条件才可以更新。 默认值.
```

示例 , 创建city_country_view视图 , 执行如下SQL :

```mysql
create or replace view city_country_view
as
select t.*,c.country_name from country c , city t where c.country_id = t.country_id;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2001%201635392521%201635392521849%20jcxfqg%20image-20210702092059635.png" alt="image-20210702092059635" style="zoom:50%" />



#### d2: 查看视图

从 MySQL 5.1 版本开始，使用 SHOW TABLES 命令的时候不仅显示表的名字，同时也会显示视图的名字，而不存 在单独显示视图的 SHOW VIEWS 命令。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2002%201635392522%201635392522585%20GocvMt%20image-20210702092141024.png" alt="image-20210702092141024" style="zoom:50%" />

同样，在使用 SHOW TABLE STATUS 命令的时候，不但可以显示表的信息，同时也可以显示视图的信息。

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2003%201635392523%201635392523236%20c1EJPN%20image-20210702092210586.png" alt="image-20210702092210586" style="zoom:50%" />

如果需要查询某个视图的定义，可以使用 SHOW CREATE VIEW 命令进行查看 :

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2003%201635392523%201635392523880%20QxJrOU%20image-20210702092232113.png" alt="image-20210702092232113" style="zoom:50%" />



#### d3: 删除视图

```mysql
DROP VIEW [IF EXISTS] view_name [, view_name] ...[RESTRICT | CASCADE]
```

示例 , 删除视图city_country_view :

```mysql
DROP VIEW city_country_view ;
```





## D3: 存储过程

### P1: 概述

存储过程和函数是 **事先经过编译并存储在数据库中的一段 SQL 语句的集合**，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程和函数的区别在于函数必须有返回值，而存储过程没有。 

- 函数 : 是一个有返回值的过程 ;
- 过程 : 是一个没有返回值的函数 ;



### P2: 存储过程语法

#### d1: 创建存储过程

```mysql
CREATE PROCEDURE procedure_name ([proc_parameter[,...]]) 
begin
-- SQL语句 
end ;
```

示例 :

```mysql
delimiter $

create procedure pro_test1()
begin
    select 'Hello Mysql' ;
end$

delimiter ;

#delimiter
该关键字用来声明SQL语句的分隔符 , 告诉 MySQL 解释器，该段命令是否已经结束了，mysql是否可以执行了。 默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，mysql将会执行该命令。
```



#### d2: 调用存储过程

```mysql
call procedure_name() ;
```



#### d3: 查看存储过程

```mysql
-- 查询db_name数据库中的所有的存储过程
select name from mysql.proc where db='db_name';

-- 查询存储过程的状态信息 
show procedure status;

-- 查询某个存储过程的定义
show create procedure test.pro_test1 \G;

```



#### d4: 删除存储过程

```mysql
DROP PROCEDURE [IF EXISTS] sp_name ;
```



#### d5: 语法

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构 ， 来完成比较复杂的功能。

##### O1: 变量

- DECLARE

通过 DECLARE 可以定义一个局部变量，该变量的作用范围只能在 BEGIN...END 块中。

```mysql
DECLARE var_name[,...] type [DEFAULT value]
```

示例:

```mysql
delimiter $

create procedure pro_test2()
begin
   declare num int default 5;
   select num+ 10;
end$

delimiter ;
```



- SET

直接m赋值使用 SET，可以赋常量或者赋表达式，具体语法如下:

```mysql
SET var_name = expr [, var_name = expr] ...
```

示例:

```mysql
DELIMITER $

CREATE  PROCEDURE pro_test3()
BEGIN
  DECLARE NAME VARCHAR(20);
  SET NAME = 'MYSQL';
  SELECT NAME ;
END$

DELIMITER ;
```

也可以通过select ... into 方式进行赋值操作 :

```mysql
DELIMITER $

CREATE  PROCEDURE pro_test5()
BEGIN
    declare  countnum int;
    select count(*) into countnum from city;
    select countnum;
END$

DELIMITER ;
```



##### O2: if

```mysql
if search_condition then statement_list
		
		[elseif search_condition then statement_list] ... 
		[else statement_list]
		
end if;
```

```mysql
# 根据定义的身高变量，判定当前身高的所属的身材类型 180 及以上 ----------> 身材高挑
# 170 - 180 ---------> 标准身材
# 170 以下 ----------> 一般身材

delimiter $

create procedure pro_test6()
begin
  declare  height  int  default  175;
  declare  description  varchar(50);
  
  if height >= 180 then
  	set description = '身材高挑';
  elseif height >= 170 and height < 180 then 
  	set description = '标准身材';
  else
  	set description = '一般身材';
  end if;
  
  select description ;
end$

delimiter ;
```

<img src="https://gitee.com/breeze1002/upic/raw/master/SQL/SQL/2021%2010%2028%2011%2042%2004%201635392524%201635392524758%20LNaE6T%20image-20210702094147803.png" alt="image-20210702094147803" style="zoom:50%" />



##### O3: 传递参数

```mssql
create procedure procedure_name([in/out/inout] 参数名 参数类型) 
...

IN : 该参数可以作为输入，也就是需要调用方传入值 , 默认 
OUT: 该参数作为输出，也就是该参数可以作为返回值 
INOUT: 既可以作为输入参数，也可以作为输出参数
```



- IN:

```mysql
# 根据定义的身高变量，判定当前身高的所属的身材类型
delimiter $

create procedure pro_test5(in height int)
begin
    declare description varchar(50) default '';
  if height >= 180 then
		set description='身材高挑';
	elseif height >= 170 and height < 180 then
		set description='标准身材'; 
  else
		set description='一般身材'; 
	end if;
	select concat('身高 ', height , '对应的身材类型为:',description); end$

delimiter ;
 
```



- OUT:

```mysql
# 根据传入的身高变量，获取当前身高的所属的身材类型
create procedure pro_test5(in height int , out description varchar(100)) 
begin
	if height >= 180 then
		set description='身材高挑';
	elseif height >= 170 and height < 180 then 
		set description='标准身材';
	else
		set description='一般身材';
	end if; 
end$

# 调用
call pro_test5(168, @description)$ 

select @description$

# 注:
@description : 这种变量要在变量名称前面加上“@”符号，叫做用户会话变量，代表整个会话过程他都是有作用 的，这个类似于全局变量一样。
@@global.sort_buffer_size : 这种在变量前加上 "@@" 符号, 叫做 系统变量
```



##### O4: case

```mysql
方式一 :
CASE case_value

  WHEN when_value THEN statement_list
  [WHEN when_value THEN statement_list] ...
  [ELSE statement_list]
  
END CASE;

方式二 : 
CASE

  WHEN search_condition THEN statement_list
  [WHEN search_condition THEN statement_list] ...
  [ELSE statement_list]
  
END CASE;
```

```mysql
# 给定一个月份, 然后计算出所在的季度

delimiter $

create procedure pro_test9(month int)
begin
  declare result varchar(20);
  case
		when month >= 1 and month <=3 then 
			set result = '第一季度';
		when month >= 4 and month <=6 then 
			set result = '第二季度';
		when month >= 7 and month <=9 then 
			set result = '第三季度';
		when month >= 10 and month <=12 then 
			set result = '第四季度';
	end case;

	select concat('您输入的月份为 :', month , ' , 该月份为 : ' , result) as content ; 
end$

delimiter ; 
```



##### O5: while

```mysql
while search_condition do 
	statement_list
end while;

# 计算从1加到n的值
delimiter $

create procedure pro_test8(n int)
begin
  declare total int default 0;
  declare num int default 1;
  while num<=n do
    set total = total + num;
    set num = num + 1;
  end while;
  select total;
end$

delimiter;
```



##### O6: repeat结构

有条件的循环控制语句, 当满足条件的时候退出循环 。while 是满足条件才执行，repeat 是满足条件就退出循环。

```mysql
REPEAT
	statement_list
	UNTIL search_condition
END REPEAT;

# 计算从1加到n的值

delimiter $

create procedure pro_test10(n int)
begin
  declare total int default 0;
  repeat
    set total = total + n;
    set n = n - 1;
    until n=0
end repeat;
  select total ;
end$

delimiter ;
```



##### O7: loop

LOOP 实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如

```mysql
[begin_label:] LOOP 
	statement_list
END LOOP [end_label]
```

如果不在 statement_list 中增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。



##### O8: leave

用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简 单例子 , 退出循环:

```mysql
delimiter $
CREATE PROCEDURE pro_test11(n int)
BEGIN
  declare total int default 0;
  
  ins: LOOP
  
    IF n <= 0 then
      leave ins;
    END IF;
    
    set total = total + n;
		set n = n - 1;
		
	END LOOP ins;
	
  select total;
END$
delimiter ;
```



##### O9: 游标/光标

游标是用来**存储查询结果集**的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用
包括光标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

```mysql
# 声明光标:
DECLARE cursor_name CURSOR FOR select_statement ;

# OPEN 光标:
OPEN cursor_name ;

# FETCH 光标:
FETCH cursor_name INTO var_name [, var_name] ...

# CLOSE 光标:
CLOSE cursor_name ;
```

**示例:**

```mysql
-- 初始化脚本
create table emp(
  id int(11) not null auto_increment , 
  name varchar(50) not null comment '姓名', 
  age int(11) 	comment '年龄',
  salary int(11) comment '薪水',
  primary key(`id`)
)engine=innodb default charset=utf8 ;

insert into emp(id,name,age,salary) values(null,'金毛狮王',55,3800),(null,'白眉鹰 王',60,4000),(null,'青翼蝠王',38,2800),(null,'紫衫龙王',42,1800);

-- 查询emp表中数据, 并逐行获取进行展示 
create procedure pro_test11() 
begin
  declare e_id int(11);
  declare e_name varchar(50);
  declare e_age int(11);
  declare e_salary int(11);
  declare emp_result cursor for select * from emp;
  
  open emp_result;
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
	select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
	select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
	select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
	select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
	select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
 
 	close emp_result;
end$

-- 通过循环结构,获取游标中的数据
DELIMITER $

create procedure pro_test12()
begin
  DECLARE id int(11);
  DECLARE name varchar(50);
  DECLARE age int(11);
  DECLARE salary int(11);
  DECLARE has_data int default 1;
  
  DECLARE emp_result CURSOR FOR select * from emp;
  DECLARE EXIT HANDLER FOR NOT FOUND set has_data = 0;
  
  open emp_result;
  
	repeat
		fetch emp_result into id , name , age , salary;
		select concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',salary);
    until has_data = 0
	end repeat;
	
  close emp_result;
end$

DELIMITER ;
```



#### d6: 存储函数

```mysql
CREATE FUNCTION function_name([param type ... ]) 
RETURNS type
BEGIN
	... 
END;
```

```mysql
delimiter $

create function count_city(countryId int)
returns int
begin
	declare cnum int ;
  select count(*) into cnum from city where country_id = countryId;
  return cnum;
end$
delimiter ;
```

```mysql
-- 调用
select count_city(1); 

select count_city(2);
```



## D4: 触发器

### P1: 概述

触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集 合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。

使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持 行级触发，不支持语句级触发。

| 触发器类型     | NEW和OLD的使用                                        |
| -------------- | ----------------------------------------------------- |
| INSERT型触发器 | NEW表示将要或者已经新增的数据                         |
| UPDATE型触发器 | OLD表示修改之前的数据,NEW表示将要或者已经修改后的数据 |
| DELETE型触发器 | OLD表示将要或者已经删除的数据                         |



### P2: 触发器语法

#### d1: 创建触发器

```mysql
create trigger trigger_name 
before/after insert/update/delete 
on tbl_name
[ for each row ] -- 行级触发器 
begin
    trigger_stmt ;
end;
```

示例:

```mysql
-- 通过触发器记录 emp 表的数据变更日志 , 包含增加, 修改 , 删除 ;

-- 首先创建一张日志表
create table emp_logs(
  id int(11) not null auto_increment,
  operation varchar(20) not null comment '操作类型, insert/update/delete', operate_time datetime not null comment '操作时间',
  operate_id int(11) not null comment '操作表的ID',
  operate_params varchar(500) comment '操作参数',
  primary key(`id`)
)engine=innodb default charset=utf8;

-- 创建 insert 型触发器，完成插入数据时的日志记录
DELIMITER $

create trigger emp_logs_insert_trigger
after insert
on emp
for each row
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
values(null,'insert',now(),new.id,concat('插入后(id:',new.id,', name:',new.name,', age:',new.age,', salary:',new.salary,')'));
end $

DELIMITER ;

-- 创建 update 型触发器，完成更新数据时的日志记录 :
DELIMITER $

create trigger emp_logs_update_trigger
after update
on emp
for each row
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,', age:',new.age,', salary:',new.salary,')'));
end $

DELIMITER ;
 
-- 创建delete 行的触发器 , 完成删除数据时的日志记录 
DELIMITER $

create trigger emp_logs_delete_trigger
after delete
on emp
for each row
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
values(null,'delete',now(),old.id,concat('删除前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,')'));
end $

DELIMITER ;

-- 测试
insert into emp(id,name,age,salary) values(null, '光明左使',30,3500); 
insert into emp(id,name,age,salary) values(null, '光明右使',33,3200);

update emp set age = 39 where id = 3;

delete from emp where id = 5;
```



#### d2: 删除触发器

```mysql
drop trigger [schema_name.]trigger_name
```

 `如果没有指定 schema_name，默认为当前数据库 。`

#### d3: 查看触发器

可以通过执行 SHOW TRIGGERS 命令查看触发器的状态、语法等信息。

```mysql
show triggers ;
```







