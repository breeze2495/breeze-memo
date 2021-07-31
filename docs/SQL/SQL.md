# SQL：

## P1: SQL注入

- ##### **什么是sql注入：**

  - 比较常见的网络攻击方式之一，它不是利用操作系统的BUG来实现攻击，而是利用程序员编写的疏忽，通过SQL语句，实现无账号登录，甚至篡改数据；

- ##### sql注入攻击的**大体思路**：

  - 找到sql注入的位置；
  - 判断服务器类型和后台数据库类型；
  - 针对不同服务器和数据库的特点进行sql注入攻击；

- ##### **实例**：

  ```java
  String sql = "select * from user_table where username=
  ' "+userName+" ' and password=' "+password+" '";
  
  --当输入了上面的用户名和密码，上面的SQL语句变成：
  SELECT * FROM user_table WHERE username=
  '’or 1 = 1 -- and password='’
  
  /*
  --分析SQL语句：
  --条件后面username=”or 1=1 用户名等于 ” 或1=1 那么这个条件一定会成功；
  
  --然后后面加两个-，这意味着注释，它将后面的语句注释，让他们不起作用，这样语句永远都--能正确执行，用户轻易骗过系统，获取合法身份。
  --这还是比较温柔的，如果是执行
  SELECT * FROM user_table WHERE
  username='' ;DROP DATABASE (DB Name) --' and password=''
  --其后果可想而知…
  */
  ```

- ##### **如何防止？**

  - **注意**：但凡有SQL注入漏洞的程序，都是因为程序要接受来自客户端用户输入的变量或者URL传递的参数，并且这个变量或参数是组成SQL语句的一部分，对于用户输入的内容或传递的参数，我们应该要时刻保持警惕，这也是安全领域里面的 「外部数据不可信任」原则，纵观Web安全领域的各种攻击方式，大多数都是因为开发者违反了这个原则而导致，所以自然能想到，从变量的检测，过滤，验证下手，确保变量是开发者所预想到的；
  - **T1: 检查变量数据类型和格式**  ：
    - 如果你的SQL语句是类似where id = { $id } 这种形式数据库里面所有id都是数字，那么就应该在sql被执行之前，检查确保变量id是int类型；如果是接收邮箱，那就应该检查并严格确保变量一定是邮箱的格式，其他的类型比如日期，时间等也是同样的道理。
      - 总结起来：只要是有固定的格式的变量，在SQL语句执行前，应该严格按照固定格式去检查，确保变量是我们所预想的格式，这样很大程度上可以避免SQL注入攻击；
      - 例如：在接收userName的栗子中，我们的产品设计应该是在用户注册的时候，就有一个用户名的规则，比如5-20个字符，只能由大小写字母，数字以及一些安全的符号组成，不包含特殊字符。此时应该有一个check_username的函数来进行统一的检查。不过仍然有很多例外情况不能应用到这一准则，例如文章发布系统，评论系统等必须要允许用户提交任意字符串的场景，这就需要采用过滤等其他方案了；
  - **T2: 过滤特殊符号**
    - 对于无法固定格式的变量，一定要进行特殊符号过滤或转义处理；
  - **T3: 绑定变量，使用预编译语句**
    - MYSQL的mysqli驱动提供了预编译语句的支持，不同的程序语言，都分别有使用预编译语句的方法；
  - **实际上：**绑定变量使用预编译语句是预防SQL注入的最佳方式，使用预编译的SQL语句语义不会发生改变，在SQL语句中，变量用「  ？」表示，即使黑客本事再大，也不可能改变SQL语句的结构；

- **什么是SQL预编译？**

  1. **预编译语句是什么？**

     - 通常一条sql语句在db接收到最终执行完毕返回可以分为下面<u>三个过程</u>：
       1. 词法和语义解析；
       2. 优化sql语句，指定执行计划；
       3. 执行并返回结果；
     - 这种普通语句称作 Immediate Statements;
     - 但是很多情况，我们的一条sql语句可能会反复执行，或者每次执行的时候只有个别的值不同（例如query的where子句值不同，update的set子句值不同等）；
     - 如果每次都需要经过上面的三个过程，<u>效率就会低下</u>；
     - 所谓预编译语句就是将这类语句中的值用占位符替代，可以视为将sql语句模板化或者参数化，一般这类语句称为**Prepared Statements**；
     - 预编译语句的<u>优势</u>在于：<u>一次编译，多次运行</u>，省去了解析优化等过程；此外，预编译语句能够<u>防止sql注入</u>
     - 当然就优化来说，很多时候最优的执行计划不是光靠知道sql语句的模板就能决定，往往是需要通过具体值来预估成本代价；

  2. **MYSQL的预编译功能：**

     - **注意：**MySQL的老版本（4.1之前）是不支持服务端预编译的，但基于目前业界生产环境普遍情况，基本可以认为MySQL支持服务端预编译。

     1. **建表：**

        ```mysql
        mysql> show create table t\G
        *************************** 1. row ***************************
               Table: t
        Create Table: CREATE TABLE `t` (
          `a` int(11) DEFAULT NULL,
          `b` varchar(20) DEFAULT NULL,
          UNIQUE KEY `ab` (`a`,`b`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8
        ```

     2. **编译：**通过PREPARE stmt_name FROM preparable_stm的语法来预编译一条语句：

        ```mysql
        mysql> prepare ins from 'insert into t select ?,?';
        Query OK, 0 rows affected (0.00 sec)
        Statement prepared
        ```

     3. **执行：**通过`EXECUTE stmt_name [USING @var_name [, @var_name] ...]`的语法来执行预编译语句：

        ```mysql
        mysql> set @a=999,@b='hello';
        Query OK, 0 rows affected (0.00 sec)
         
        mysql> execute ins using @a,@b;
        Query OK, 1 row affected (0.01 sec)
        Records: 1  Duplicates: 0  Warnings: 0
         
        mysql> select * from t;
        +------+-------+
        | a    | b     |
        +------+-------+
        |  999 | hello |
        +------+-------+
        1 row in set (0.00 sec)
        ```

        可以看到数据已经成功插入；

        Mysql中的预编译语句作用域是session级，可以用过max_prepared_stmt_count变量来控制全局最大的存储的预编译语句。

        ```mysql
        mysql> set @@global.max_prepared_stmt_count=1;
        Query OK, 0 rows affected (0.00 sec)
         
        mysql> prepare sel from 'select * from t';
        ERROR 1461 (42000): Can't create more than max_prepared_stmt_count statements (current value: 1)
        ```

     4. **释放：**如果我们想要释放一条预编译语句，可以使用`{DEALLOCATE | DROP} PREPARE stmt_name`的语法进行操作；

     ```mysql
     mysql> deallocate prepare ins;
     Query OK, 0 rows affected (0.00 sec)
     ```

- **为什么PrepareStatement可以防止sql注入：**

  - **原理：**采用了预编译的方法，先将SQL语句中可被客户端控制的参数集进行编译，生成对应的临时变量集，再使用对应的设置方法，为临时变量集里面的元素进行赋值，赋值函数setSring()，会对传入的参数进行强制类型检查和安全检查，所以就避免了SQL注入的产生。以下为具体分析：

    1. 为什么Statement会被sql注入：

       statement之所以会被sql注入是因为sql语句结构发生了变化：

       ```mysql
       select*from tablename where username='"+uesrname+  
       "'and password='"+password+"'
       ```

       在用户输入「  or true or  」之后sql语句结构改变

       ```mysql
       select*from tablename where username=''or true or'' and password=''
       ```

       这样本来是判断用户名和密码都匹配时才会计数，但是经过改变后变成了或的逻辑关系，不管用户名和密码是否匹配该式返回值永远为true；

    2. 为什么preparement可以防止sql注入？
       preparement样式为

       ```mysql
       select*from tablename where username=? and password=?
       ```

       该sql语句会在得到用户的输入之前先用数据库进行预编译，这样的话不管用户输入什么用户名和密码的判断使用都是与的逻辑关系，防止了sql注入；

    总结 ：参数化能防止注入的原因在于，语句是语句，参数是参数，参数的值并不是语句的与部分，数据库只按语句的跑，至于跑的时候是参数中带的是正常还是非正常参数，不会影响执行；

- **Mybatis如何防止sql注入：**

  ```mysql
  <select id="selectByNameAndPassword" parameterType="java.util.Map" resultMap="BaseResultMap">
  select id, username, password, role
  from user
  where username = #{username,jdbcType=VARCHAR}
  and password = #{password,jdbcType=VARCHAR}
  </select>
  
  -------------------------------------------------------------
  
  <select id="selectByNameAndPassword" parameterType="java.util.Map" resultMap="BaseResultMap">
  select id, username, password, role
  from user
  where username = ${username,jdbcType=VARCHAR}
  and password = ${password,jdbcType=VARCHAR}
  </select>
  ```

  - **mybatis中 # 和 $ 的区别：**

    1. #将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号；	
       - 如：where username=#{username}，如果传入的值是111,那么解析成sql时的值为where username="111", 如果传入的值是id，则解析成的sql为where username="id".　
    2. $将传入的数据直接显示在sql中
       - ：where username=${username}，如果传入的值是111,那么解析成sql时的值为where username=111；
         如果传入的值是;drop table user;，则解析成的sql为：select id, username, password, role from user where username=;drop table user;
    3. #方式能很大程度防止sql注入，$方式无法防止sql注入；
    4. $一般用于传入数据库对象，例如传入表明；
    5. **一般能用#就别用$**，若**不得不用「 ${xxx} 」这样的参数，要做好过滤工作**，以防止sql注入；
    6. 在Mybatis中，「 ${xxx} 」这样的格式参数会直接参与sql编译，从而不能避免注入攻击。但设计到动态表名和列名时，只能使用「 ${xxx} 」.因此这样的参数需要我们在代码中手工进行处理来防止sql注入;

  - **mybatis是如何做到防止sql注入的:**

    - Mybatis框架作为一款半自动化的持久层框架,其sql语句都要我们自己手动编写,这个时候防止sql注入是不要的.mybatis的sql是一个具有'输入+输出'的功能,类似于函数的结构.parameterTyoe表示了输入的参数类型,resultType表示输出的参数类型.如果我们要防止sql注入,理所当然要在输入参数上下功夫.
    - 上面代码中执行的sql语句即为:

    ```mysql
    select id, username, password, role from user where username=? and password=?
    ```

    不管输入什么参数,打印出的sql都是这样.

    这时因为mybatis启用了预编译功能,在sql执行前,会将上面的sql发送给数据库进行编译;在执行时,直接使用编译好的sql,替换占位符 ? 就可以了.因为sql注入只能对编译过程起作用,所以这样的方式就很好地 避免了sql注入的问题.

  - **底层实现原理:**

    - 在框架底层,是JDBC的preparedStatement类在起作用,PreparedStatement是我们熟悉的Statement的 子类,它的对象包含了编译好的sql语句.这种准备好的方式不仅能提高安全性,而且在多次执行统一sql时,能够提高效率.原因是sql已经编译好,再次执行时无需编译.























