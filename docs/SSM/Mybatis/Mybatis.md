## D1:  Mybatis缓存机制

`Mybatis包含了一个非常强大的查询缓存特性,它可以非常方便地配置和定制.缓存可以极大地提升查询效率.`

**Mybatis中默认定义了两级缓存**:

- 默认情况下只有一级缓存开启
- 二级缓存需要手动开启和配置,他是基于namespace级别的缓存
- 为了提高扩展性.Mybatis定义了缓存接口Cache.我们可以通过实现Cache接口来自定义二级缓存





### P1: 一级缓存

#### d1: 概述

- 一级缓存(local cache), 即本地缓存, 作用域默认为sqlSession。

- 本地缓存不能被关闭, 但可以调用 clearCache() 来清空本地缓存, 或者改变缓存的作用域.

- 在mybatis3.1之后, 可以配置本地缓存的作用域. 在 mybatis.xml 中配置

  <img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Mybatis/2021%2010%2028%2014%2022%2011%201635402131%201635402131731%20sW35Bi%20image-20210717133104047.png" alt="image-20210717133104047" style="zoom:50%" />



#### d2:一级缓存失效情况:

- 不同的sqlSession对应不同的一级缓存
- 同一个sqlSession但是查询条件不同
- 同一个sqlSession大师两次查询期间执行了任意一次的增删改操作
- 同一个sqlSession两次查询期间手动清空了缓存



### P2: 二级缓存

#### d1: 概述

- 二级缓存(second level cache)，全局作用域缓存
- 二级缓存默认不开启，需要手动配置
- MyBatis提供二级缓存的接口以及实现，缓存实现要求POJO实现Serializable接口
- 二级缓存在 SqlSession 关闭或提交之后才会生效



#### d2: 使用步骤

```xml
–1、全局配置文件中开启二级缓存

<setting name=*"cacheEnabled" value="true"/>*

–2、需要使用二级缓存的映射文件处使用cache配置缓存

<cache />

–3、注意：POJO需要实现Serializable接口
```



#### d3: 缓存相关属性

- **eviction=“FIFO”：缓存回收策略：**
  - LRU – 最近最少使用的：移除最长时间不被使用的对象。
  - FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
  - SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
  - WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
  - 默认的是 LRU。

- **flushInterval：刷新间隔，单位毫秒**
  - 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新

- **size：引用数目，正整数**
  - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出

- **readOnly：只读，true/false**

  - true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。

  - false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是 false。



#### d4: 缓存有关设置

- 全局setting的**cacheEnable**：
  - 配置二级缓存的开关。一级缓存一直是打开的。

- select标签的**useCache**属性：
  - 配置这个select是否使用二级缓存。一级缓存一直是使用的

- sql标签的**flushCache**属性：
  - 增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。查询默认flushCache=false。

- **sqlSession.clearCache()**：
  - 只是用来清除一级缓存。

- 当在某一个作用域 (一级缓存Session/二级缓存Namespaces) 进行了 C/U/D 操作后，默认该作用域下所有select中的缓存将被clear。



### P3: 第三方缓存整合

- EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。

- MyBatis定义了Cache接口方便我们进行自定义扩展。

- **步骤**：

  - 导入ehcache包，以及整合包，日志包

    ehcache-core-2.6.8.jar、mybatis-ehcache-1.0.3.jar

    slf4j-api-1.6.1.jar、slf4j-log4j12-1.6.2.jar

  - 编写ehcache.xml配置文件

  - 配置cache标签

    ```xml
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
    ```

- **参照缓存**：若想在命名空间中共享相同的缓存配置和实例。可以使用 cache-ref 元素来引用另外一个缓存。

  ```xml
  <cache-ref namespace="com.breeze.mybatis.example.CustomerMapper"/>
  ```



### P4:  缓存使用流程

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Mybatis/2021%2010%2028%2014%2022%2012%201635402132%201635402132719%20rkEOqs%20image-20210717140410954.png" alt="image-20210717140410954" style="zoom:50%" /> 