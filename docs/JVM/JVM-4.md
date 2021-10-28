

## 第十二章: 垃圾回收概述及相关算法

### P1: 垃圾回收概述

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2034%201628506954%201628506954068%20CGPvwc%20image-20210413211224577.png" alt="image-20210413211224577" style="zoom:33%" />

1. Java 和 C++语言的区别，就在于垃圾收集技术和内存动态分配上，C++语言没有垃圾收集技术，需要程序员手动的收集。
2. 垃圾收集，不是Java语言的伴生产物。早在1960年，第一门开始使用内存动态分配和垃圾收集技术的Lisp语言诞生。
3. 关于垃圾收集有三个经典问题：
   - 哪些内存需要回收？
   - 什么时候回收？
   - 如何回收？
4. 垃圾收集机制是Java的招牌能力，极大地提高了开发效率。如今，垃圾收集几乎成为现代语言的标配，即使经过如此长时间的发展，Java的垃圾收集机制仍然在不断的演进中，不同大小的设备、不同特征的应用场景，对垃圾收集提出了新的挑战，这当然也是面试的热点。

#### p1: 什么是垃圾?

1. 垃圾是指**在运行程序中没有任何指针指向的对象**，这个对象就是需要被回收的垃圾。
2. 外文：An object is considered garbage when it can no longer be reached from any pointer in the running program.
3. 如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其他对象使用。甚至可能导致内存溢出。

#### p2: 为什么需要GC?

1. 对于高级语言来说，一个基本认知是如果不进行垃圾回收，**内存迟早都会被消耗完**，因为不断地分配内存空间而不进行回收，就好像不停地生产生活垃圾而从来不打扫一样。
2. 除了释放没用的对象，垃圾回收也可以清除内存里的记录碎片。碎片整理将所占用的堆内存移到堆的一端，**以便JVM将整理出的内存分配给新的对象**。
3. 随着应用程序所应付的业务越来越庞大、复杂，用户越来越多，**没有GC就不能保证应用程序的正常进行**。而经常造成STW的GC又跟不上实际的需求，所以才会不断地尝试对GC进行优化。

#### p3: 早期垃圾回收

1. 在早期的C/C++时代，垃圾回收基本上是手工进行的。开发人员可以使用new关键字进行内存申请，并使用delete关键字进行内存释放。比如以下代码：

   ```c++
    MibBridge *pBridge= new cmBaseGroupBridge（）；
    //如果注册失败，使用Delete释放该对象所占内存区域
    if（pBridge->Register（kDestroy）！=NO ERROR）
        delete pBridge；Copy to clipboardErrorCopied
   ```

2. 这种方式可以灵活控制内存释放的时间，但是会给开发人员带来**频繁申请和释放内存的管理负担**。倘若有一处内存区间由于程序员编码的问题忘记被回收，那么就会产生**内存泄漏**，垃圾对象永远无法被清除，随着系统运行时间的不断增长，垃圾对象所耗内存可能持续上升，直到出现内存溢出并造成**应用程序崩溃**。

3. 有了垃圾回收机制后，上述代码极有可能变成这样

   ```c++
    MibBridge *pBridge=new cmBaseGroupBridge(); 
    pBridge->Register(kDestroy);Copy to clipboardErrorCopied
   ```

4. 现在，除了Java以外，C#、Python、Ruby等语言都使用了自动垃圾回收的思想，也是未来发展趋势，可以说这种自动化的内存分配和来及回收方式已经成为了现代开发语言必备的标准。

#### p4: Java垃圾回收机制

##### D1: 自动内存管理

> **官网介绍**：https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html

- **自动内存管理的优点**
  1. 自动内存管理，无需开发人员手动参与内存的分配与回收，这样降低内存泄漏和内存溢出的风险
  2. 没有垃圾回收器，java也会和cpp一样，各种悬垂指针，野指针，泄露问题让你头疼不已。
  3. 自动内存管理机制，将程序员从繁重的内存管理中释放出来，可以更专心地专注于业务开发

- **关于自动内存管理的担忧**
  1. 对于Java开发人员而言，自动内存管理就像是一个黑匣子，如果过度依赖于“自动”，那么这将会是一场灾难，最严重的就会**弱化Java开发人员在程序出现内存溢出时定位问题和解决问题的能力**。
  2. 此时，了解JVM的自动内存分配和内存回收原理就显得非常重要，只有在真正了解JVM是如何管理内存后，我们才能够在遇见OutofMemoryError时，快速地根据错误异常日志定位问题和解决问题。
  3. 当需要排查各种内存溢出、内存泄漏问题时，当垃圾收集成为系统达到更高并发量的瓶颈时，我们就必须对这些“自动化”的技术**实施必要的监控和调节**。

##### D2: 应该关心哪些区域的回收?

​	<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2035%201628506955%201628506955016%20UCB5if%20image-20210413212043870.png" alt="image-20210413212043870" style="zoom:33%" />	





1. 垃圾收集器可以对年轻代回收，也可以对老年代回收，甚至是全栈和方法区的回收，
2. 其中，**Java堆是垃圾收集器的工作重点**
3. 从次数上讲：
   1. 频繁收集Young区
   2. 较少收集Old区
   3. 基本不收集Perm区（元空间）



### P2: 垃圾回收相关算法

#### p1: 标记阶段: 引用计数算法

##### D1: 标记阶段的目的

​	**垃圾标记阶段：主要是为了判断对象是否存活**

1. 在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先**需要区分出内存中哪些是存活对象，哪些是已经死亡的对象。**只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为**垃圾标记阶段**。
2. 那么在JVM中究竟是如何标记一个死亡对象呢？简单来说，当一个对象已经不再被任何的存活对象继续引用时，就可以宣判为已经死亡。
3. 判断对象存活一般有两种方式：**引用计数算法**和**可达性分析算法**。

##### D2: 引用计数算法 

1. 引用计数算法（Reference Counting）比较简单，对每个对象保存一个整型的引用计数器属性。用于记录对象被引用的情况。
2. 对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收。
3. 优点：实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。
4. 缺点：
   1. 它需要单独的字段存储计数器，这样的做法增加了**存储空间的开销**。
   2. 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了**时间开销**。
   3. 引用计数器有一个严重的问题，即**无法处理循环引用**的情况。这是一条致命缺陷，导致在Java的垃圾回收器中没有使用这类算法。

- **循环引用**

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2036%201628506956%201628506956453%20OeWbIj%20image-20210413212511907.png" alt="image-20210413212511907" style="zoom:33%" />

  当p的指针断开的时候，内部的引用形成一个循环，计数器都还算1，无法被回收，这就是循环引用，从而造成内存泄漏

- **Java使用的不是引用计数算法**

  ```java
  /**
   * -XX:+PrintGCDetails
   * 证明：java使用的不是引用计数算法
   */
  public class RefCountGC {
      //这个成员属性唯一的作用就是占用一点内存
      private byte[] bigSize = new byte[5 * 1024 * 1024];//5MB
  
      Object reference = null;
  
      public static void main(String[] args) {
          RefCountGC obj1 = new RefCountGC();
          RefCountGC obj2 = new RefCountGC();
  
          obj1.reference = obj2;
          obj2.reference = obj1;
  
          obj1 = null;
          obj2 = null;
          //显式的执行垃圾回收行为
          //这里发生GC，obj1和obj2能否被回收？
          System.gc();
  
      }
  }
  ```

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2038%201628506958%201628506958001%2001e5cV%20image-20210413212616408.png" alt="image-20210413212616408" style="zoom:33%" />

  - 如果不小心直接把`obj1.reference`和`obj2.reference`置为null。则在Java堆中的两块内存依然保持着互相引用，无法被回收

- **没有进行GC时**

  把下面的几行代码注释掉，让它来不及

  ```java
          System.gc();//把这行代码注释掉
  ```

  ```
  Heap
   PSYoungGen      total 38400K, used 14234K [0x00000000d5f80000, 0x00000000d8a00000, 0x0000000100000000)
    eden space 33280K, 42% used [0x00000000d5f80000,0x00000000d6d66be8,0x00000000d8000000)
    from space 5120K, 0% used [0x00000000d8500000,0x00000000d8500000,0x00000000d8a00000)
    to   space 5120K, 0% used [0x00000000d8000000,0x00000000d8000000,0x00000000d8500000)
   ParOldGen       total 87552K, used 0K [0x0000000081e00000, 0x0000000087380000, 0x00000000d5f80000)
    object space 87552K, 0% used [0x0000000081e00000,0x0000000081e00000,0x0000000087380000)
   Metaspace       used 3496K, capacity 4498K, committed 4864K, reserved 1056768K
    class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
  
  Process finished with exit code 0
  ```



- **进行GC**

  打开那行代码的注释

  ```java
  [GC (System.gc()) [PSYoungGen: 13569K->808K(38400K)] 13569K->816K(125952K), 0.0012717 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  [Full GC (System.gc()) [PSYoungGen: 808K->0K(38400K)] [ParOldGen: 8K->670K(87552K)] 816K->670K(125952K), [Metaspace: 3491K->3491K(1056768K)], 0.0051769 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  Heap
   PSYoungGen      total 38400K, used 333K [0x00000000d5f80000, 0x00000000d8a00000, 0x0000000100000000)
    eden space 33280K, 1% used [0x00000000d5f80000,0x00000000d5fd34a8,0x00000000d8000000)
    from space 5120K, 0% used [0x00000000d8000000,0x00000000d8000000,0x00000000d8500000)
    to   space 5120K, 0% used [0x00000000d8500000,0x00000000d8500000,0x00000000d8a00000)
   ParOldGen       total 87552K, used 670K [0x0000000081e00000, 0x0000000087380000, 0x00000000d5f80000)
    object space 87552K, 0% used [0x0000000081e00000,0x0000000081ea7990,0x0000000087380000)
   Metaspace       used 3498K, capacity 4498K, committed 4864K, reserved 1056768K
    class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
  
  Process finished with exit code 0
  ```

  1、从打印日志就可以明显看出来，已经进行了GC

  2、如果使用引用计数算法，那么这两个对象将会无法回收。而现在两个对象被回收了，说明Java使用的不是引用计数算法来进行标记的。

- **小结**
  1. 引用计数算法，是很多语言的资源回收选择，例如因人工智能而更加火热的Python，它更是同时支持引用计数和垃圾收集机制。
  2. 具体哪种最优是要看场景的，业界有大规模实践中仅保留引用计数机制，以提高吞吐量的尝试。
  3. Java并没有选择引用计数，是因为其存在一个基本的难题，也就是很难处理循环引用关系。
  4. Python如何解决循环引用？
     - 手动解除：很好理解，就是在合适的时机，解除引用关系。
     - 使用弱引用weakref，weakref是Python提供的标准库，旨在解决循环引用。



#### p2: 标记阶段: 可达性分析算法

**可达性分析算法：也可以称为根搜索算法、追踪性垃圾收集**

1. 相对于引用计数算法而言，可达性分析算法不仅同样具备实现简单和执行高效等特点，更重要的是该算法可以有效地**解决在引用计数算法中循环引用的问题，防止内存泄漏的发生**。
2. 相较于引用计数算法，这里的可达性分析就是Java、C#选择的。这种类型的垃圾收集通常也叫作**追踪性垃圾收集**（Tracing Garbage Collection）

##### D1: 可达性分析实现思路

- 所谓"GCRoots”根集合就是一组必须活跃的引用
- 其基本思路如下：

1. 可达性分析算法是以根对象集合（GCRoots）为起始点，按照从上至下的方式**搜索被根对象集合所连接的目标对象是否可达。**
2. 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为**引用链**（Reference Chain）
3. 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象。
4. 在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2039%201628506959%201628506959322%20ylX81v%20image-20210413215841972.png" alt="image-20210413215841972" style="zoom:33%" />



##### D2: GC Roots可以是哪些元素?

1. 虚拟机栈中引用的对象
   - 比如：各个线程被调用的方法中使用到的参数、局部变量等。
2. 本地方法栈内JNI（通常说的本地方法）引用的对象
3. 方法区中类静态属性引用的对象
   - 比如：Java类的引用类型静态变量
4. 方法区中常量引用的对象
   - 比如：字符串常量池（StringTable）里的引用
5. 所有被同步锁synchronized持有的对象
6. Java虚拟机内部的引用。
   - 基本数据类型对应的Class对象，一些常驻的异常对象（如：NullPointerException、OutofMemoryError），系统类加载器。
7. 反映java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2040%201628506960%201628506960193%20obW2yi%20image-20210413215933660.png" alt="image-20210413215933660" style="zoom:25%" />

1. 总结一句话就是，除了堆空间的周边，比如：虚拟机栈、本地方法栈、方法区、字符串常量池等地方对堆空间进行引用的，都可以作为GC Roots进行可达性分析

2. 除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。比如：

   分代收集

   和局部回收（PartialGC）。

   - 如果只针对Java堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入GC Roots集合中去考虑，才能保证可达性分析的准确性。

**小技巧**

​	由于Root采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root。

- **注意**
  1. 如果要使用可达性分析算法来判断内存是否可回收，那么分析工作必须在一个能保障一致性的快照中进行。这点不满足的话分析结果的准确性就无法保证。
  2. 这点也是导致GC进行时必须“Stop The World”的一个重要原因。即使是号称（几乎）不会发生停顿的CMS收集器中，**枚举根节点时也是必须要停顿的**。



#### p3: 对象的finalization机制

##### D1: finalize() 方法机制

**对象销毁前的回调函数：finalize()**

1. Java语言提供了对象终止（finalization）机制来允许开发人员提供**对象被销毁之前的自定义处理逻辑**。
2. 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的finalize()方法。
3. finalize() 方法允许在子类中被重写，**用于在对象被回收时进行资源释放**。通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。

Object 类中 finalize() 源码

```java
// 等待被重写
protected void finalize() throws Throwable { }
```

1. 永远不要主动调用某个对象的finalize()方法，应该交给垃圾回收机制调用。理由包括下面三点：
   1. 在finalize()时可能会导致对象复活。
   2. finalize()方法的执行时间是没有保障的，它完全由GC线程决定，极端情况下，若不发生GC，则finalize()方法将没有执行机会。
   3. 一个糟糕的finalize()会严重影响GC的性能。比如finalize是个死循环
2. 从功能上来说，finalize()方法与C++中的析构函数比较相似，但是Java采用的是基于垃圾回收器的自动内存管理机制，所以finalize()方法在**本质上不同于C++中的析构函数**。
3. finalize()方法对应了一个finalize线程，因为优先级比较低，即使主动调用该方法，也不会因此就直接进行回收

##### D2: 生存还是死亡？

由于finalize()方法的存在，**虚拟机中的对象一般处于三种可能的状态。**

1. 如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。

   一个无法触及的对象有可能在某一个条件下“复活”自己

   ，如果这样，那么对它立即进行回收就是不合理的。为此，定义虚拟机中的对象可能的三种状态。如下：

   1. 可触及的：从根节点开始，可以到达这个对象。
   2. 可复活的：对象的所有引用都被释放，但是对象有可能在finalize()中复活。
   3. 不可触及的：对象的finalize()被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，**因为finalize()只会被调用一次**。

2. 以上3种状态中，是由于finalize()方法的存在，进行的区分。只有在对象不可触及时才可以被回收。

##### D3: 具体过程

判定一个对象objA是否可回收，至少要经历两次标记过程：

1. 如果对象objA到GC Roots没有引用链，则进行第一次标记。
2. 进行筛选，判断此对象是否有必要执行finalize()方法
   1. 如果对象objA没有重写finalize()方法，或者finalize()方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，objA被判定为不可触及的。
   2. 如果对象objA重写了finalize()方法，且还未执行过，那么objA会被插入到F-Queue队列中，由一个虚拟机自动创建的、低优先级的Finalizer线程触发其finalize()方法执行。
   3. finalize()方法是对象逃脱死亡的最后机会，稍后GC会对F-Queue队列中的对象进行第二次标记。如果objA在finalize()方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，objA会被移出“即将回收”集合。之后，对象会再次出现没有引用存在的情况。在这个情况下，finalize()方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的finalize()方法只会被调用一次。

**通过 JVisual VM 查看 Finalizer 线程**

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2041%201628506961%201628506961696%206W8F47%20image-20210413220248214.png" alt="image-20210413220248214" style="zoom:33%" />



##### D4: 代码演示finalize()方法可复活对象

我们重写 CanReliveObj 类的 finalize()方法，在调用其 finalize()方法时，将 obj 指向当前类对象 this

```java
/**
 * 测试Object类中finalize()方法，即对象的finalization机制。
 *
 */
public class CanReliveObj {
    public static CanReliveObj obj;//类变量，属于 GC Root


    //此方法只能被调用一次
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("调用当前类重写的finalize()方法");
        obj = this;//当前待回收的对象在finalize()方法中与引用链上的一个对象obj建立了联系
    }


    public static void main(String[] args) {
        try {
            obj = new CanReliveObj();
            // 对象第一次成功拯救自己
            obj = null;
            System.gc();//调用垃圾回收器
            System.out.println("第1次 gc");
            // 因为Finalizer线程优先级很低，暂停2秒，以等待它
            Thread.sleep(2000);
            if (obj == null) {
                System.out.println("obj is dead");
            } else {
                System.out.println("obj is still alive");
            }
            System.out.println("第2次 gc");
            // 下面这段代码与上面的完全相同，但是这次自救却失败了
            obj = null;
            System.gc();
            // 因为Finalizer线程优先级很低，暂停2秒，以等待它
            Thread.sleep(2000);
            if (obj == null) {
                System.out.println("obj is dead");
            } else {
                System.out.println("obj is still alive");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**如果注释掉finalize()方法**

```java
 //此方法只能被调用一次
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("调用当前类重写的finalize()方法");
        obj = this;//当前待回收的对象在finalize()方法中与引用链上的一个对象obj建立了联系
    }
```

输出结果：

```java
第1次 gc
obj is dead
第2次 gc
obj is dead
```

**放开finalize()方法**

输出结果：

```java
第1次 gc
调用当前类重写的finalize()方法
obj is still alive
第2次 gc
obj is dead
```

第一次自救成功，但由于 finalize() 方法只会执行一次，所以第二次自救失败



#### p4: MAT与JProfiler的GC Roots溯源

##### D1: MAT 介绍

1. MAT是Memory Analyzer的简称，它是一款功能强大的Java堆内存分析器。用于查找内存泄漏以及查看内存消耗情况。
2. MAT是基于Eclipse开发的，是一款免费的性能分析工具。
3. 大家可以在http://www.eclipse.org/mat/下载并使用MAT

> 1、虽然Jvisualvm很强大，但是在内存分析方面，还是MAT更好用一些
>
> 2、此小节主要是为了实时分析GC Roots是哪些东西，中间需要用到一个dump的文件

##### D2: 获取 dump 文件方式

**方式一：命令行使用 jmap**

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2043%201628506963%201628506963129%204zMYUb%20image-20210413220652601.png" alt="image-20210413220652601" style="zoom:33%" />

**方式二：使用JVisualVM**

1. 捕获的heap dump文件是一个临时文件，关闭JVisualVM后自动删除，若要保留，需要将其另存为文件。可通过以下方法捕获heap dump：
2. 操作步骤下面演示

##### D3: 捕捉dump示例

- **使用JVisualVM捕捉heap dump**

  代码：

  - numList 和 birth 在第一次捕捉内存快照的时候，为 GC Roots
  - 之后 numList 和 birth 置为 null ，对应的引用对象被回收，在第二次捕捉内存快照的时候，就不再是 GC Roots

  ```java
  public class GCRootsTest {
      public static void main(String[] args) {
          List<Object> numList = new ArrayList<>();
          Date birth = new Date();
  
          for (int i = 0; i < 100; i++) {
              numList.add(String.valueOf(i));
              try {
                  Thread.sleep(10);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
  
          System.out.println("数据添加完毕，请操作：");
          new Scanner(System.in).next();
          numList = null;
          birth = null;
  
          System.out.println("numList、birth已置空，请操作：");
          new Scanner(System.in).next();
  
          System.out.println("结束");
      }
  }
  ```

  **如何捕捉堆内存快照**

  1、先执行第一步，然后停下来，去生成此步骤dump文件

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2043%201628506963%201628506963986%20lfSQWX%20image-20210413220910714.png" alt="image-20210413220910714" style="zoom:33%" />

  2、 点击【堆 Dump】

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2044%201628506964%201628506964889%20agtaA3%20image-20210413220934118.png" alt="image-20210413220934118" style="zoom:33%" />

  3、右键 --> 另存为即可

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2045%201628506965%201628506965969%20NJpvEM%20image-20210413220955735.png" alt="image-20210413220955735" style="zoom:33%" />

  4、输入命令，继续执行程序

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2047%201628506967%201628506967203%20LhJoUC%20image-20210413221017368.png" alt="image-20210413221017368" style="zoom:33%" />

  5、我们接着捕获第二张堆内存快照

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2049%201628506969%201628506969293%20WAL2Ya%20image-20210413221038770.png" alt="image-20210413221038770" style="zoom:33%" />

- **使用MAT查看堆内存快照**

  1、打开 MAT ，选择File --> Open File，打开刚刚的两个dump文件，**我们先打开第一个dump文件**

  > 点击Open Heap Dump也行

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2050%201628506970%201628506970574%202PZb2b%20image-20210413221307451.png" alt="image-20210413221307451" style="zoom:33%" />

  2、选择Java Basics --> GC Roots

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2051%201628506971%201628506971603%202TxFdd%20image-20210413221334680.png" alt="image-20210413221334680" style="zoom:33%;" />

  3、第一次捕捉堆内存快照时，GC Roots 中包含我们定义的两个局部变量，类型分别为 ArrayList 和 Date，Total:21

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2053%201628506973%201628506973021%20KcYUB4%20image-20210413221401729.png" alt="image-20210413221401729" style="zoom:33%" />

  4、打开第二个dump文件，第二次捕获内存快照时，由于两个局部变量引用的对象被释放，所以这两个局部变量不再作为 GC Roots ，从 Total Entries = 19 也可以看出（少了两个 GC Roots）

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2054%201628506974%201628506974653%20gQZmSv%20image-20210413221423053.png" alt="image-20210413221423053" style="zoom:33%" />

##### D4: JProfiler GC Roots溯源

1、在实际开发中，我们很少会查看所有的GC Roots。一般都是查看某一个或几个对象的GC Root是哪个，这个过程叫**GC Roots 溯源**

2、下面我们使用使用 JProfiler 进行 GC Roots 溯源演示

依然用下面这个代码

```java
public class GCRootsTest {
    public static void main(String[] args) {
        List<Object> numList = new ArrayList<>();
        Date birth = new Date();

        for (int i = 0; i < 100; i++) {
            numList.add(String.valueOf(i));
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("数据添加完毕，请操作：");
        new Scanner(System.in).next();
        numList = null;
        birth = null;

        System.out.println("numList、birth已置空，请操作：");
        new Scanner(System.in).next();

        System.out.println("结束");
    }
}
```

1.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2056%201628506976%201628506976746%206MuMGj%20image-20210413221617485.png" alt="image-20210413221617485" style="zoom:33%;" />

2. 

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2057%201628506977%201628506977867%2009eURx%20image-20210413221640889.png" alt="image-20210413221640889" style="zoom:33%;" />

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2002%2059%201628506979%201628506979290%20krEKIi%20image-20210413221652496.png" alt="image-20210413221652496" style="zoom:33%;" />

可以发现颜色变绿了，可以动态的看变化

3、右击对象，选择 Show Selection In Heap Walker，单独的查看某个对象

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2000%201628506980%201628506980688%20gAqfVS%20image-20210413221720533.png" alt="image-20210413221720533" style="zoom:33%;" />![image-20210413221733576](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2001%201628506981%201628506981873%20cFKb1n%20image-20210413221733576.png)

![image-20210413221733576](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2001%201628506981%201628506981873%20cFKb1n%20image-20210413221733576.png)

4、选择Incoming References，表示追寻 GC Roots 的源头

点击Show Paths To GC Roots，在弹出界面中选择默认设置即可

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2002%201628506982%201628506982879%20MofMQ7%20image-20210413221756234.png" alt="image-20210413221756234" style="zoom:33%;" />![image-20210413221808937](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2004%201628506984%201628506984183%20uU4Xkw%20image-20210413221808937.png)

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2002%201628506982%201628506982879%20MofMQ7%20image-20210413221756234.png" alt="image-20210413221756234" style="zoom:33%;" />![image-20210413221808937](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2004%201628506984%201628506984183%20uU4Xkw%20image-20210413221808937.png)![image-20210413221817933](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2005%201628506985%201628506985633%206C0j63%20image-20210413221817933.png)

![image-20210413221817933](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2005%201628506985%201628506985633%206C0j63%20image-20210413221817933.png)



##### D5: JProfiler 分析 OOM

> 这里是简单的讲一下，后面篇章会详解

```java
/**
 * -Xms8m -Xmx8m 
 * -XX:+HeapDumpOnOutOfMemoryError  这个参数的意思是当程序出现OOM的时候就会在当前工程目录生成一个dump文件
 */
public class HeapOOM {
    byte[] buffer = new byte[1 * 1024 * 1024];//1MB

    public static void main(String[] args) {
        ArrayList<HeapOOM> list = new ArrayList<>();

        int count = 0;
        try{
            while(true){
                list.add(new HeapOOM());
                count++;
            }
        }catch (Throwable e){
            System.out.println("count = " + count);
            e.printStackTrace();
        }
    }
}
```

程序输出日志

```java
com.atguigu.java.HeapOOM
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid14608.hprof ...
java.lang.OutOfMemoryError: Java heap space
    at com.atguigu.java.HeapOOM.<init>(HeapOOM.java:12)
    at com.atguigu.java.HeapOOM.main(HeapOOM.java:20)
Heap dump file created [7797849 bytes in 0.010 secs]
count = 6
```

打开这个dump文件

1、看这个超大对象

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2006%201628506986%201628506986774%20L6vbm3%20image-20210413221907312.png" alt="image-20210413221907312" style="zoom:33%;" />

2、揪出 main() 线程中出问题的代码

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2007%201628506987%201628506987765%203WOPvL%20image-20210413221927867.png" alt="image-20210413221927867" style="zoom:33%;" />

#### p5: 清除阶段: 标记 - 算法

**垃圾清除阶段**

- 当成功区分出内存中存活对象和死亡对象后，GC接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存。目前在JVM中比较常见的三种垃圾收集算法是

1. 标记-清除算法（Mark-Sweep）
2. 复制算法（Copying）
3. 标记-压缩算法（Mark-Compact）

**背景**

标记-清除算法（Mark-Sweep）是一种非常基础和常见的垃圾收集算法，该算法被J.McCarthy等人在1960年提出并并应用于Lisp语言。

**执行过程**

当堆中的有效内存空间（available memory）被耗尽的时候，就会停止整个程序（也被称为stop the world），然后进行两项工作，第一项则是标记，第二项则是清除

1. 标记：Collector从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的Header中记录为可达对象。
   - 注意：标记的是被引用的对象，也就是可达对象，并非标记的是即将被清除的垃圾对象
2. 清除：Collector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2008%201628506988%201628506988644%20RLuddR%20image-20210413222138145.png" alt="image-20210413222138145" style="zoom:33%;" />

**标记-清除算法的缺点**

1. 标记清除算法的效率不算高
2. 在进行GC的时候，需要停止整个应用程序，用户体验较差
3. 这种方式清理出来的空闲内存是不连续的，产生内碎片，需要维护一个空闲列表

**注意：何为清除？**

这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放（也就是覆盖原有的地址）。

关于空闲列表是在为对象分配内存的时候提过：

1. 如果内存规整
   - 采用指针碰撞的方式进行内存分配
2. 如果内存不规整
   - 虚拟机需要维护一个空闲列表
   - 采用空闲列表分配内存

#### p6: 清除阶段: 复制算法

**背景**

1. 为了解决标记-清除算法在垃圾收集效率方面的缺陷，M.L.Minsky于1963年发表了著名的论文，“使用双存储区的Lisp语言垃圾收集器CA LISP Garbage Collector Algorithm Using Serial Secondary Storage）”。M.L.Minsky在该论文中描述的算法被人们称为复制（Copying）算法，它也被M.L.Minsky本人成功地引入到了Lisp语言的一个实现版本中。

**核心思想**

将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2009%201628506989%201628506989722%202C0Nxn%20image-20210413222351444.png" alt="image-20210413222351444" style="zoom:33%;" />

新生代里面就用到了复制算法，Eden区和S0区存活对象整体复制到S1区

**复制算法的优缺点**

**优点**

1. 没有标记和清除过程，实现简单，运行高效
2. 复制过去以后保证空间的连续性，不会出现“碎片”问题。

**缺点**

1. 此算法的缺点也是很明显的，就是需要两倍的内存空间。
2. 对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对象引用关系，不管是内存占用或者时间开销也不小

**复制算法的应用场景**

1. 如果系统中的垃圾对象很多，复制算法需要复制的存活对象数量并不会太大，效率较高
2. 老年代大量的对象存活，那么复制的对象将会有很多，效率会很低
3. 在新生代，对常规应用的垃圾回收，一次通常可以回收70% - 99% 的内存空间。回收性价比很高。所以现在的商业虚拟机都是用这种收集算法回收新生代。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2010%201628506990%201628506990779%20MYXQQm%20image-20210413222415099.png" alt="image-20210413222415099" style="zoom:33%;" />

#### p7: 清除阶段: 标记压缩算法

**标记-压缩（或标记-整理、Mark - Compact）算法**

**背景**

1. 复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生，但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活对象较多，复制的成本也将很高。因此，**基于老年代垃圾回收的特性，需要使用其他的算法。**
2. 标记-清除算法的确可以应用在老年代中，但是该算法不仅执行效率低下，而且在执行完内存回收后还会产生内存碎片，所以JVM的设计者需要在此基础之上进行改进。标记-压缩（Mark-Compact）算法由此诞生。
3. 1970年前后，G.L.Steele、C.J.Chene和D.s.Wise等研究者发布标记-压缩算法。在许多现代的垃圾收集器中，人们都使用了标记-压缩算法或其改进版本。

**执行过程**

1. 第一阶段和标记清除算法一样，从根节点开始标记所有被引用对象
2. 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放。之后，清理边界外所有的空间。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2011%201628506991%201628506991579%209vrmzC%20image-20210413222511347.png" alt="image-20210413222511347" style="zoom:33%;" />

**标记-压缩算法与标记-清除算法的比较**

1. 标记-压缩算法的最终效果等同于标记-清除算法执行完成后，再进行一次内存碎片整理，因此，也可以把它称为标记-清除-压缩（Mark-Sweep-Compact）算法。
2. 二者的本质差异在于标记-清除算法是一种**非移动式的回收算法**，标记-压缩是**移动式的**。是否移动回收后的存活对象是一项优缺点并存的风险决策。
3. 可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。

**标记-压缩算法的优缺点**

**优点**

1. 消除了标记-清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可。
2. 消除了复制算法当中，内存减半的高额代价。

**缺点**

1. 从效率上来说，标记-整理算法要低于复制算法。
2. 移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址（因为HotSpot虚拟机采用的不是句柄池的方式，而是直接指针）
3. 移动过程中，需要全程暂停用户应用程序。即：STW



#### p8: 垃圾回收算法小结

> **对比三种清除阶段的算法**

1. 效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存。
2. 而为了尽量兼顾上面提到的三个指标，标记-整理算法相对来说更平滑一些，但是效率上不尽如人意，它比复制算法多了一个标记的阶段，比标记-清除多了一个整理内存的阶段。

|              | 标记清除           | 标记整理         | 复制                                  |
| ------------ | ------------------ | ---------------- | ------------------------------------- |
| **速率**     | 中等               | 最慢             | 最快                                  |
| **空间开销** | 少（但会堆积碎片） | 少（不堆积碎片） | 通常需要活对象的2倍空间（不堆积碎片） |
| **移动对象** | 否                 | 是               | 是                                    |



#### p9: 分代收集算法

Q：难道就没有一种最优的算法吗？

A：无，没有最好的算法，只有最合适的算法

**为什么要使用分代收集算法**

1. 前面所有这些算法中，并没有一种算法可以完全替代其他算法，它们都具有自己独特的优势和特点。分代收集算法应运而生。
2. 分代收集算法，是基于这样一个事实：**不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的收集方式，以便提高回收效率。**一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。
3. 在Java程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关:
   - 比如Http请求中的Session对象、线程、Socket连接，这类对象跟业务直接挂钩，因此生命周期比较长。
   - 但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如：String对象，由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收。

**目前几乎所有的GC都采用分代手机算法执行垃圾回收的**

在HotSpot中，基于分代的概念，GC所使用的内存回收算法必须结合年轻代和老年代各自的特点。

1. 年轻代（Young Gen）
   - 年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。
   - 这种情况复制算法的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过hotspot中的两个survivor的设计得到缓解。
2. 老年代（Tenured Gen）
   - 老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。
   - 这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由标记-清除或者是标记-清除与标记-整理的混合实现。
     - Mark阶段的开销与存活对象的数量成正比。
     - Sweep阶段的开销与所管理区域的大小成正相关。
     - Compact阶段的开销与存活对象的数据成正比。
3. 以HotSpot中的CMS回收器为例，CMS是基于Mark-Sweep实现的，对于对象的回收效率很高。对于碎片问题，CMS采用基于Mark-Compact算法的Serial Old回收器作为补偿措施：当内存回收不佳（碎片导致的Concurrent Mode Failure时），将采用Serial Old执行Full GC以达到对老年代内存的整理。
4. 分代的思想被现有的虚拟机广泛使用。几乎所有的垃圾回收器都区分新生代和老年代



#### p10: 增量收集算法和分区算法

- **增量收集算法**

  上述现有的算法，在垃圾回收过程中，应用软件将处于一种Stop the World的状态。在**Stop the World**状态下，应用程序所有的线程都会挂起，暂停一切正常的工作，等待垃圾回收的完成。如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性。为了解决这个问题，即对实时垃圾收集算法的研究直接导致了增量收集（Incremental Collecting）算法的诞生。

  - **增量收集算法基本思想**

    1. 如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。**每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。**
    2. 总的来说，增量收集算法的基础仍是传统的标记-清除和复制算法。增量收集算法通过**对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作**

  - **增量收集算法的缺点**

    使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，**造成系统吞吐量的下降**。

- **分区算法**

  > 主要针对G1收集器来说的

  1. 一般来说，在相同条件下，堆空间越大，一次GC时所需要的时间就越长，有关GC产生的停顿也越长。为了更好地控制GC产生的停顿时间，将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。
  2. 分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间。每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2012%201628506992%201628506992474%20pU5Agp%20image-20210413222930095.png" alt="image-20210413222930095" style="zoom:33%;" />

#### 写在最后

注意，这些只是基本的算法思路，实际GC实现过程要复杂的多，目前还在发展中的前沿GC都是复合算法，并且并行和并发兼备。





#### **大厂面试题**

**蚂蚁金服**

1. 你知道哪几种垃圾回收器，各自的优缺点，重点讲一下CMS和G1？
2. JVM GC算法有哪些，目前的JDK版本采用什么回收算法？
3. G1回收器讲下回收过程GC是什么？为什么要有GC？
4. GC的两种判定方法？CMS收集器与G1收集器的特点

**百度**

1. 说一下GC算法，分代回收说下
2. 垃圾收集策略和算法

**天猫**

1. JVM GC原理，JVM怎么回收内存
2. CMS特点，垃圾回收算法有哪些？各自的优缺点，他们共同的缺点是什么？

**滴滴**

1. Java的垃圾回收器都有哪些，说下G1的应用场景，平时你是如何搭配使用垃圾回收器的

**京东**

1. 你知道哪几种垃圾收集器，各自的优缺点，重点讲下CMS和G1，
2. 包括原理，流程，优缺点。垃圾回收算法的实现原理

**阿里**

1. 讲一讲垃圾回收算法。
2. 什么情况下触发垃圾回收？
3. 如何选择合适的垃圾收集算法？
4. JVM有哪三种垃圾回收器？

**字节跳动**

1. 常见的垃圾回收器算法有哪些，各有什么优劣？
2. System.gc()和Runtime.gc()会做什么事情？
3. Java GC机制？GC Roots有哪些？
4. Java对象的回收方式，回收算法。
5. CMS和G1了解么，CMS解决什么问题，说一下回收的过程。
6. CMS回收停顿了几次，为什么要停顿两次?







## 第十三章 垃圾回收相关概念



### P1: System.gc() 的理解

1. 在默认情况下，通过System.gc()者Runtime.getRuntime().gc() 的调用，**会显式触发Full GC**，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。
2. 然而System.gc()调用附带一个免责声明，无法保证对垃圾收集器的调用(不能确保立即生效)
3. JVM实现者可以通过System.gc() 调用来决定JVM的GC行为。而一般情况下，垃圾回收应该是自动进行的，**无须手动触发，否则就太过于麻烦了。**在一些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之间调用System.gc()

**代码示例：手动执行 GC 操作**

```java
public class SystemGCTest {
    public static void main(String[] args) {
        new SystemGCTest();
        System.gc();//提醒jvm的垃圾回收器执行gc,但是不确定是否马上执行gc
        //与Runtime.getRuntime().gc();的作用一样。

//        System.runFinalization();//强制调用使用引用的对象的finalize()方法
    }
    //如果发生了GC，这个finalize()一定会被调用
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("SystemGCTest 重写了finalize()");
    }
}
```

输出结果不确定：有时候会调用 finalize() 方法，有时候并不会调用

```java
SystemGCTest 重写了finalize()
或
空
```

#### p1: 手动 GC 理解不可达对象的回收行为

```java
//加上参数：  -XX:+PrintGCDetails
public class LocalVarGC {
    public void localvarGC1() {
        byte[] buffer = new byte[10 * 1024 * 1024];//10MB
        System.gc();
    }

    public void localvarGC2() {
        byte[] buffer = new byte[10 * 1024 * 1024];
        buffer = null;
        System.gc();
    }

    public void localvarGC3() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        System.gc();
    }

    public void localvarGC4() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        int value = 10;
        System.gc();
    }

    public void localvarGC5() {
        localvarGC1();
        System.gc();
    }

    public static void main(String[] args) {
        LocalVarGC local = new LocalVarGC();
        //通过在main方法调用这几个方法进行测试
        local.localvarGC1();
    }
}
```

JVM参数：

```
-Xms256m -Xmx256m -XX:+PrintGCDetails -XX:PretenureSizeThreshold=15m
```

1、第四个参数是设置大对象直接进入老年代的阈值，由于我的电脑8G和视频里老师的电脑16G不太一样。我测试的时候10M的数组都是直接进入到了老年代，为了保持一样的效果，我同时设置了堆内存和大对象阈值，尽量和宋红康老师保持一致

2、我也查过了大对象阈值的默认值

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2013%201628506993%201628506993847%20F3k9ij%20image-20210414095456496.png" alt="image-20210414095456496" style="zoom:33%;" />

我不太懂这个默认值为啥是0，我猜测可能是代表什么比例，目前也没有搜到相关的东西。这个不太重要，暂时就没有太深究，希望读者有知道的可以告知我一声。

> 看不懂GC日志请看笔者的 **堆**那篇文章

**1、调用 localvarGC1() 方法**

执行 System.gc() 仅仅是将年轻代的 buffer 数组对象放到了老年代，buffer对象仍然没有回收

```java
[GC (System.gc()) [PSYoungGen: 15492K->10728K(76288K)] 15492K->11000K(251392K), 0.0066473 secs] [Times: user=0.08 sys=0.02, real=0.01 secs] 
[Full GC (System.gc()) [PSYoungGen: 10728K->0K(76288K)] [ParOldGen: 272K->10911K(175104K)] 11000K->10911K(251392K), [Metaspace: 3492K->3492K(1056768K)], 0.0097940 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 76288K, used 655K [0x00000000fab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 65536K, 1% used [0x00000000fab00000,0x00000000faba3ee8,0x00000000feb00000)
  from space 10752K, 0% used [0x00000000feb00000,0x00000000feb00000,0x00000000ff580000)
  to   space 10752K, 0% used [0x00000000ff580000,0x00000000ff580000,0x0000000100000000)
 ParOldGen       total 175104K, used 10911K [0x00000000f0000000, 0x00000000fab00000, 0x00000000fab00000)
  object space 175104K, 6% used [0x00000000f0000000,0x00000000f0aa7d08,0x00000000fab00000)
 Metaspace       used 3498K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
```

**2、调用 localvarGC2() 方法**

由于 buffer 数组对象没有引用指向它，执行 System.gc() 将被回收

```java
[GC (System.gc()) [PSYoungGen: 15492K->808K(76288K)] 15492K->816K(251392K), 0.0294475 secs] [Times: user=0.00 sys=0.00, real=0.04 secs] 
[Full GC (System.gc()) [PSYoungGen: 808K->0K(76288K)] [ParOldGen: 8K->640K(175104K)] 816K->640K(251392K), [Metaspace: 3385K->3385K(1056768K)], 0.0054210 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 76288K, used 1966K [0x00000000fab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 65536K, 3% used [0x00000000fab00000,0x00000000faceb9e0,0x00000000feb00000)
  from space 10752K, 0% used [0x00000000feb00000,0x00000000feb00000,0x00000000ff580000)
  to   space 10752K, 0% used [0x00000000ff580000,0x00000000ff580000,0x0000000100000000)
 ParOldGen       total 175104K, used 640K [0x00000000f0000000, 0x00000000fab00000, 0x00000000fab00000)
  object space 175104K, 0% used [0x00000000f0000000,0x00000000f00a01a8,0x00000000fab00000)
 Metaspace       used 3392K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 379K, capacity 388K, committed 512K, reserved 1048576K
```

**3、调用 localvarGC3() 方法**

虽然出了代码块的作用域，但是 buffer 数组对象并没有被回收

```java
[GC (System.gc()) [PSYoungGen: 15492K->840K(76288K)] 15492K->11088K(251392K), 0.0070281 secs] [Times: user=0.08 sys=0.00, real=0.01 secs] 
[Full GC (System.gc()) [PSYoungGen: 840K->0K(76288K)] [ParOldGen: 10248K->10900K(175104K)] 11088K->10900K(251392K), [Metaspace: 3386K->3386K(1056768K)], 0.0084464 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 76288K, used 1966K [0x00000000fab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 65536K, 3% used [0x00000000fab00000,0x00000000faceb9e0,0x00000000feb00000)
  from space 10752K, 0% used [0x00000000feb00000,0x00000000feb00000,0x00000000ff580000)
  to   space 10752K, 0% used [0x00000000ff580000,0x00000000ff580000,0x0000000100000000)
 ParOldGen       total 175104K, used 10900K [0x00000000f0000000, 0x00000000fab00000, 0x00000000fab00000)
  object space 175104K, 6% used [0x00000000f0000000,0x00000000f0aa52e8,0x00000000fab00000)
 Metaspace       used 3393K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 379K, capacity 388K, committed 512K, reserved 1048576K
```

**原因：**

1、来看看字节码：实例方法局部变量表第一个变量肯定是 this

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2014%201628506994%201628506994934%20g3CcHH%20image-20210414095536048.png" alt="image-20210414095536048" style="zoom:33%;" />

2、你有没有看到，局部变量表的大小是 2。但是局部变量表里只有一个索引为0的啊？那索引为1的是哪个局部变量呢？实际上索引为1的位置是buffer在占用着，执行 System.gc() 时，栈中还有 buffer 变量指向堆中的字节数组，所以没有进行GC

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2017%201628506997%201628506997168%20vKWRaQ%20image-20210414095555754.png" alt="image-20210414095555754" style="zoom:33%;" />

3、那么这种代码块的情况，什么时候会被GC呢？我们来看第四个方法

**4、调用 localvarGC4() 方法**

```java
[GC (System.gc()) [PSYoungGen: 15492K->776K(76288K)] 15492K->784K(251392K), 0.0009430 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 776K->0K(76288K)] [ParOldGen: 8K->646K(175104K)] 784K->646K(251392K), [Metaspace: 3485K->3485K(1056768K)], 0.0065829 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 76288K, used 1966K [0x00000000fab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 65536K, 3% used [0x00000000fab00000,0x00000000faceb9f8,0x00000000feb00000)
  from space 10752K, 0% used [0x00000000feb00000,0x00000000feb00000,0x00000000ff580000)
  to   space 10752K, 0% used [0x00000000ff580000,0x00000000ff580000,0x0000000100000000)
 ParOldGen       total 175104K, used 646K [0x00000000f0000000, 0x00000000fab00000, 0x00000000fab00000)
  object space 175104K, 0% used [0x00000000f0000000,0x00000000f00a1b88,0x00000000fab00000)
 Metaspace       used 3498K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
```

Q：就多定义了一个局部变量 value ，就可以把字节数组回收了呢？

A：局部变量表长度为 2 ，这说明了出了代码块时，buffer 就出了其作用域范围，此时没有为 value 开启新的槽，value 变量直接占据了 buffer 变量的槽（Slot），导致堆中的字节数组没有引用再指向它，执行 System.gc() 时被回收。看，value 位于局部变量表中索引为 1 的位置。value这个局部变量把原本属于buffer的slot给占用了，这样栈上就没有buffer变量指向`new byte[10 * 1024 * 1024]`实例了。

> 这点看不懂的可以看我前面的文章：虚拟机栈 --> Slot的重复利用

 <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2018%201628506998%201628506998205%20lnusqh%20image-20210414095638532.png" alt="image-20210414095638532" style="zoom:33%;" />![image-20210414095650579](https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2019%201628506999%201628506999320%204a74kg%20image-20210414095650579.png)

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2018%201628506998%201628506998205%20lnusqh%20image-20210414095638532.png" alt="image-20210414095638532" style="zoom:33%;" />!

**调用 localvarGC5() 方法**

局部变量除了方法范围就是失效了，堆中的字节数组铁定被回收

```java
[GC (System.gc()) [PSYoungGen: 15492K->840K(76288K)] 15492K->11088K(251392K), 0.0070281 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (System.gc()) [PSYoungGen: 840K->0K(76288K)] [ParOldGen: 10248K->10911K(175104K)] 11088K->10911K(251392K), [Metaspace: 3492K->3492K(1056768K)], 0.0082011 secs] [Times: user=0.03 sys=0.03, real=0.01 secs] 
[GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] 10911K->10911K(251392K), 0.0004440 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] [ParOldGen: 10911K->671K(175104K)] 10911K->671K(251392K), [Metaspace: 3492K->3492K(1056768K)], 0.0108555 secs] [Times: user=0.08 sys=0.02, real=0.01 secs] 
Heap
 PSYoungGen      total 76288K, used 655K [0x00000000fab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 65536K, 1% used [0x00000000fab00000,0x00000000faba3ee8,0x00000000feb00000)
  from space 10752K, 0% used [0x00000000ff580000,0x00000000ff580000,0x0000000100000000)
  to   space 10752K, 0% used [0x00000000feb00000,0x00000000feb00000,0x00000000ff580000)
 ParOldGen       total 175104K, used 671K [0x00000000f0000000, 0x00000000fab00000, 0x00000000fab00000)
  object space 175104K, 0% used [0x00000000f0000000,0x00000000f00a7cf8,0x00000000fab00000)
 Metaspace       used 3499K, capacity 4502K, committed 4864K, reserved 1056768K
  class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
```

### P2: 内存溢出与内存泄漏

#### p1: 内存溢出

1. 内存溢出相对于内存泄漏来说，尽管更容易被理解，但是同样的，内存溢出也是引发程序崩溃的罪魁祸首之一。
2. 由于GC一直在发展，所有一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现OOM的情况。
3. 大多数情况下，GC会进行各种年龄段的垃圾回收，实在不行了就放大招，来一次独占式的Full GC操作，这时候会回收大量的内存，供应用程序继续使用。
4. Javadoc中对OutofMemoryError的解释是，没有空闲内存，并且垃圾收集器也无法提供更多内存。

**内存溢出（OOM）原因分析**

首先说没有空闲内存的情况：说明Java虚拟机的堆内存不够。原因有二：

1. Java虚拟机的堆内存设置不够。
   - 比如：可能存在内存泄漏问题；也很有可能就是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定JVM堆大小或者指定数值偏小。我们可以通过参数-Xms 、-Xmx来调整。
2. 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）
   - 对于老版本的Oracle JDK，因为永久代的大小是有限的，并且JVM对永久代垃圾回收（如，常量池回收、卸载不再需要的类型）非常不积极，所以当我们不断添加新类型的时候，永久代出现OutOfMemoryError也非常多见。尤其是在运行时存在大量动态类型生成的场合；类似intern字符串缓存占用太多空间，也会导致OOM问题。对应的异常信息，会标记出来和永久代相关：“java.lang.OutOfMemoryError:PermGen space"。
   - 随着元数据区的引入，方法区内存已经不再那么窘迫，所以相应的OOM有所改观，出现OOM，异常信息则变成了：“java.lang.OutofMemoryError:Metaspace"。直接内存不足，也会导致OOM。

1. 这里面隐含着一层意思是，在抛出OutofMemoryError之前，通常垃圾收集器会被触发，尽其所能去清理出空间。
   - 例如：在引用机制分析中，涉及到JVM会去尝试**回收软引用指向的对象**等。
   - 在java.nio.Bits.reserveMemory()方法中，我们能清楚的看到，System.gc()会被调用，以清理空间。
2. 当然，也不是在任何情况下垃圾收集器都会被触发的
   - 比如，我们去分配一个超大对象，类似一个超大数组超过堆的最大值，JVM可以判断出垃圾收集并不能解决这个问题，所以直接抛出OutofMemoryError。

#### p2: 内存泄漏

1. 也称作“存储渗漏”。严格来说，**只有对象不会再被程序用到了，但是GC又不能回收他们的情况，才叫内存泄漏。**
2. 但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM，也可以叫做宽泛意义上的“内存泄漏”。
3. 尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现OutofMemory异常，导致程序崩溃。
4. 注意，这里的存储空间并不是指物理内存，而是指虚拟内存大小，这个虚拟内存大小取决于磁盘交换区设定的大小。

**内存泄露官方例子**

左边的图：Java使用可达性分析算法，最上面的数据不可达，就是需要被回收的对象。

右边的图：后期有一些对象不用了，按道理应该断开引用，但是存在一些链没有断开（图示中的Forgotten Reference Memory Leak），从而导致没有办法被回收。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2020%201628507000%201628507000293%20lwnuxY%20image-20210414095834382.png" alt="image-20210414095834382" style="zoom:33%;" />

**常见例子**

1. 单例模式
   - 单例的生命周期和应用程序是一样长的，所以在单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。
2. 一些提供close()的资源未关闭导致内存泄漏
   - 数据库连接 dataSourse.getConnection()，网络连接socket和io连接必须手动close，否则是不能被回收的。

### P3: Stop the World

1. Stop-the-World，简称STW，指的是GC事件发生过程中，会产生应用程序的停顿。**停顿产生时整个应用程序线程都会被暂停，没有任何响应**，有点像卡死的感觉，这个停顿称为STW。
2. 可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿，为什么需要停顿所有 Java 执行线程呢？
   - 分析工作必须在一个能确保一致性的快照中进行
   - 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上
   - **如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证**
3. 被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要减少STW的发生。

1. STW事件和采用哪款GC无关，所有的GC都有这个事件。
2. 哪怕是G1也不能完全避免Stop-the-world情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。
3. STW是JVM在**后台自动发起和自动完成**的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
4. 开发中不要用System.gc() ，这会导致Stop-the-World的发生。

**代码感受 Stop the World**

```java
public class StopTheWorldDemo {
    public static class WorkThread extends Thread {
        List<byte[]> list = new ArrayList<byte[]>();

        public void run() {
            try {
                while (true) {
                    for(int i = 0;i < 1000;i++){
                        byte[] buffer = new byte[1024];
                        list.add(buffer);
                    }

                    if(list.size() > 10000){
                        list.clear();
                        System.gc();//会触发full gc，进而会出现STW事件
                     
                    }
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    public static class PrintThread extends Thread {
        public final long startTime = System.currentTimeMillis();

        public void run() {
            try {
                while (true) {
                    // 每秒打印时间信息
                    long t = System.currentTimeMillis() - startTime;
                    System.out.println(t / 1000 + "." + t % 1000);
                    Thread.sleep(1000);
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        WorkThread w = new WorkThread();
        PrintThread p = new PrintThread();
        w.start();
        p.start();
    }
}
```

关闭工作线程 w ，观察输出：当前时间间隔与上次时间间隔**基本**是每隔1秒打印一次

```java
0.1
1.1
2.2
3.2
4.3
5.3
6.3
7.3

Process finished with exit code -1Copy to clipboardErrorCopied
```

开启工作线程 w ，观察打印输出：当前时间间隔与上次时间间隔相差 1.3s ，可以明显感受到 Stop the World 的存在

```java
0.1
1.4
2.7
3.8
4.12
5.13

Process finished with exit code -1
```

### P4: 垃圾回收的并行与并发

#### p1: 并发的概念

1. 在操作系统中，是指**一个时间段**中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理器上运行
2. 并发不是真正意义上的“同时进行”，只是CPU把一个时间段划分成几个时间片段（时间区间），然后在这几个时间区间之间来回切换。由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2021%201628507001%201628507001646%20d0eot9%20image-20210414100020287.png" alt="image-20210414100020287" style="zoom:33%;" />

#### p2: 并行的概念

1. 当系统有一个以上CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源，可以**同时**进行，我们称之为并行（Parallel）
2. 其实决定并行的因素不是CPU的数量，而是CPU的核心数量，比如一个CPU多个核也可以并行
3. 适合科学计算，后台处理等弱交互场景

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2022%201628507002%201628507002536%20jSqLQ2%20image-20210414100058304.png" alt="image-20210414100058304" style="zoom:33%;" />

> **并发与并行的对比**

1. 并发，指的是多个事情，在同一时间段内同时发生了。
2. 并行，指的是多个事情，在同一时间点上（或者说同一时刻）同时发生了。
3. 并发的多个任务之间是互相抢占资源的。并行的多个任务之间是不互相抢占资源的。
4. 只有在多CPU或者一个CPU多核的情况中，才会发生并行。否则，看似同时发生的事情，其实都是并发执行的。

#### p3: 垃圾回收的并发与并行

1. 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。
   - 如ParNew、Parallel Scavenge、Parallel Old
2. 串行（Serial）
   - 相较于并行的概念，单线程执行。
   - 如果内存不够，则程序暂停，启动JVM垃圾回收器进行垃圾回收（单线程）

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2023%201628507003%201628507003333%20G71fjE%20image-20210414100132093.png" alt="image-20210414100132093" style="zoom:33%;" />

并发和并行，在谈论垃圾收集器的上下文语境中，它们可以解释如下：

1. 并发（Concurrent）：指

   用户线程与垃圾收集线程同时执行

   （但不一定是并行的，可能会交替执行），垃圾回收线程在执行时不会停顿用户程序的运行。

   - 比如用户程序在继续运行，而垃圾收集程序线程运行于另一个CPU上；

2. 典型垃圾回收器：CMS、G1

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2024%201628507004%201628507004121%20fM5CAP%20image-20210414100149258.png" alt="image-20210414100149258" style="zoom:33%;" />

### P5: HotSpot的算法实现细节

#### p1: 根节点枚举

1、固定可作为GC Roots的节点主要在全局性的引用（例如常量或类静态属性）与执行上下文（例如栈帧中的本地变量表）中，尽管目标明确，但查找过程要做到高效并非一件容易的事情，现在Java应用越做越庞大，光是方法区的大小就常有数百上千兆，里面的类、常量等更是恒河沙数，若要逐个检查以这里为起源的引用肯定得消耗不少时间。

2、迄今为止，**所有收集器在根节点枚举这一步骤时都是必须暂停用户线程的**，因此毫无疑问根节点 枚举与之前提及的整理内存碎片一样会面临相似的“Stop The World”的困扰。现在可达性分析算法耗时 最长的查找引用链的过程已经可以做到与用户线程一起并发，**但根节点枚举始终还 是必须在一个能保障一致性的快照中才得以进行**——这里“一致性”的意思是整个枚举期间执行子系统 看起来就像被冻结在某个时间点上，不会出现分析过程中，根节点集合的对象引用关系还在不断变化 的情况，若这点不能满足的话，分析结果准确性也就无法保证。这是导致垃圾收集过程必须停顿所有 用户线程的其中一个重要原因，即使是号称停顿时间可控，或者（几乎）不会发生停顿的CMS、G1、 ZGC等收集器，枚举根节点时也是必须要停顿的。

3、由于目前主流Java虚拟机使用的都是**准确式垃圾收集**，所以当用户线程停顿下来之后，其实并不需要一个不漏地检查完所有 执行上下文和全局的引用位置，虚拟机应当是有办法直接得到哪些地方存放着对象引用的。在HotSpot 的解决方案里，是使用一组称为**OopMap的数据结构**来达到这个目的。一旦类加载动作完成的时候， HotSpot就会把对象内什么偏移量上是什么类型的数据计算出来，在即时编译过程中，也 会在特定的位置记录下栈里和寄存器里哪些位置是引用。这样收集器在扫描时就可以直接得知这些信 息了，**并不需要真正一个不漏地从方法区等GC Roots开始查找**。

4、Exact VM因它使用**准确式内存管理**（Exact Memory Management，也可以叫Non-Con- servative/Accurate Memory Management）而得名。准确式内存管理是指虚拟机可以知道内存中某个位 置的数据具体是什么类型。譬如内存中有一个32bit的整数123456，虚拟机将有能力分辨出它到底是一 个指向了123456的内存地址的引用类型还是一个数值为123456的整数，准确分辨出哪些内存是引用类 型，这也是在垃圾收集时准确判断堆上的数据是否还可能被使用的前提。【**这个不是特别重要，了解一下即可**】

> 常考面试：**在OopMap的协助下，HotSpot可以快速准确地完成GC Roots枚举**

#### p2: 安全点与安全区域

**安全点（Safepoint）**

1. 程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为“安全点（Safepoint）”。
2. Safe Point的选择很重要，**如果太少可能导致GC等待的时间太长，如果太频繁可能导致运行时的性能问题**。大部分指令的执行时间都非常短暂，通常会根据“**是否具有让程序长时间执行的特征**”为标准。比如：选择一些执行时间较长的指令作为Safe Point，**如方法调用、循环跳转和异常跳转等**。

**如何在GC发生时，检查所有线程都跑到最近的安全点停顿下来呢？**

1. 抢先式中断：（目前没有虚拟机采用了）首先中断所有线程。如果还有线程不在安全点，就恢复线程，让线程跑到安全点。
2. 主动式中断：设置一个中断标志，各个线程运行到Safe Point的时候**主动轮询**这个标志，如果中断标志为真，则将自己进行中断挂起。

**安全区域（Safe Region）**

1. Safepoint 机制保证了程序执行时，在不太长的时间内就会遇到可进入GC的Safepoint。但是，程序“不执行”的时候呢？
2. 例如线程处于Sleep状态或Blocked 状态，这时候线程无法响应JVM的中断请求，“走”到安全点去中断挂起，JVM也不太可能等待线程被唤醒。对于这种情况，就需要安全区域（Safe Region）来解决。
3. **安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始GC都是安全的**。我们也可以把Safe Region看做是被扩展了的Safepoint。

**安全区域的执行流程**

1. 当线程运行到Safe Region的代码时，首先标识已经进入了Safe Region，如果这段时间内发生GC，JVM会忽略标识为Safe Region状态的线程
2. 当线程即将离开Safe Region时，会检查JVM是否已经完成根节点枚举（即GC Roots的枚举），如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开Safe Region的信号为止；

#### p3: 记忆集与卡表

- **什么是跨代引用?**

1、一般的垃圾回收算法至少会划分出两个年代，年轻代和老年代。但是单纯的分代理论在垃圾回收的时候存在一个巨大的缺陷：为了找到年轻代中的存活对象，却不得不遍历整个老年代，反过来也是一样的。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2024%201628507004%201628507004887%20e1o0e4%20image-20210414100314386.png" alt="image-20210414100314386" style="zoom:33%;" />

2、如果我们从年轻代开始遍历，那么可以断定N, S, P, Q都是存活对象。但是，V却不会被认为是存活对象，其占据的内存会被回收了。这就是一个惊天的大漏洞！因为U本身是老年代对象，而且有外部引用指向它，也就是说U是存活对象，而U指向了V，也就是说V也应该是存活对象才是！而这都是因为我们只遍历年轻代对象！

3、所以，为了解决这种跨代引用的问题，最笨的办法就是遍历老年代的对象，找出这些跨代引用来。这种方案存在极大的性能浪费。因为从两个分代假说里面，其实隐含了一个推论：跨代引用是极少的。也就是为了找出那么一点点跨代引用，我们却得遍历整个老年代！从上图来说，很显然的是，我们根本不必遍历R。

4、因此，为了避免这种遍历老年代的性能开销，通常的分代垃圾回收器会引入一种称为**记忆集**的技术。**简单来说，记忆集就是用来记录跨代引用的表。**

- **记忆集与卡表**

1、为解决对象跨代引用所带来的问题，垃圾收集器在新生代中建 立了名为**记忆集（Remembered Set）的数据结构**，用以避免把整个老年代加进GC Roots扫描范围。事实上并不只是新生代、老年代之间才有跨代引用的问题，所有涉及部分区域收集（Partial GC）行为的 垃圾收集器，典型的如G1、ZGC和Shenandoah收集器，都会面临相同的问题，因此我们有必要进一步 理清记忆集的原理和实现方式，以便在后续章节里介绍几款最新的收集器相关知识时能更好地理解。

2、记忆集是一种用于记录**从非收集区域指向收集区域的指针集合的抽象数据结构**。如果我们不考虑效率和成本的话，最简单的实现可以用非收集区域中所有含跨代引用的对象数组来实现这个数据结构。

> 比如说我们有老年代（非收集区域）和年轻代（收集区域）的对象之间有一条引用链

3、这种记录全部含跨代引用对象的实现方案，无论是空间占用还是维护成本都相当高昂。而在垃圾 收集的场景中，收集器只需要通过记忆集判断出某一块非收集区域是否存在有指向了收集区域的指针 就可以了，并不需要了解这些跨代指针的全部细节。那设计者在实现记忆集的时候，便可以选择更为 粗犷的记录粒度来节省记忆集的存储和维护成本，下面列举了一些可供选择（当然也可以选择这个范 围以外的）的记录精度：

- 字长精度：每个记录精确到一个机器字长（就是处理器的寻址位数，如常见的32位或64位，这个 精度决定了机器访问物理内存地址的指针长度），该字包含跨代指针。
- 对象精度：每个记录精确到一个对象，该对象里有字段含有跨代指针。
- 卡精度：每个记录精确到一块内存区域，该区域内有对象含有跨代指针。

4、其中，第三种“卡精度”所指的是用一种称为“卡表”（Card Table）的方式去实现记忆集，这也是 目前最常用的一种记忆集实现形式，一些资料中甚至直接把它和记忆集混为一谈。前面定义中提到记 忆集其实是一种“抽象”的数据结构，抽象的意思是只定义了记忆集的行为意图，并没有定义其行为的 具体实现。卡表就是记忆集的一种具体实现，它定义了记忆集的记录精度、与堆内存的映射关系等。 关于卡表与记忆集的关系，读者不妨按照Java语言中HashMap与Map的关系来类比理解。 卡表最简单的形式可以只是一个字节数组，而HotSpot虚拟机确实也是这样做的

> 读者只需要知道有这个东西，面试的时候能说出来，再细致一点的就需要看周志明老师的第三版书了

### P6: 再谈引用概述

1. 我们希望能描述这样一类对象：当内存空间还足够时，则能保留在内存中；如果内存空间在进行垃圾收集后还是很紧张，则可以抛弃这些对象。
2. 既偏门又非常高频的面试题：强引用、软引用、弱引用、虚引用有什么区别？具体使用场景是什么？
3. 在JDK1.2版之后，Java对引用的概念进行了扩充，将引用分为：
   - 强引用（Strong Reference）
   - 软引用（Soft Reference）
   - 弱引用（Weak Reference）
   - 虚引用（Phantom Reference）
4. 这4种引用强度依次逐渐减弱。除强引用外，其他3种引用均可以在java.lang.ref包中找到它们的身影。如下图，显示了这3种引用类型对应的类，开发人员可以在应用程序中直接使用它们。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2025%201628507005%201628507005652%20LKtpfB%20image-20210414100441940.png" alt="image-20210414100441940" style="zoom:33%;" />

Reference子类中只有终结器引用是包内可见的，其他3种引用类型均为public，可以在应用程序中直接使用

1. 强引用（StrongReference）：最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似“`object obj=new Object()`”这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。宁可报OOM，也不会GC强引用
2. 软引用（SoftReference）：在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
3. 弱引用（WeakReference）：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
4. 虚引用（PhantomReference）：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

### p7: 再谈引用：强引用

1. 在Java程序中，最常见的引用类型是强引用（普通系统99%以上都是强引用），也就是我们最常见的普通对象引用，**也是默认的引用类型**。
2. 当在Java语言中使用new操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。
3. **只要强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象。**只要强引用的对象是可达的，jvm宁可报OOM，也不会回收强引用。
4. 对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。
5. 相对的，软引用、弱引用和虚引用的对象是软可触及、弱可触及和虚可触及的，在一定条件下，都是可以被回收的。所以，强引用是造成Java内存泄漏的主要原因之一。

**强引用代码举例**

```java
public class StrongReferenceTest {
    public static void main(String[] args) {
        StringBuffer str = new StringBuffer ("Hello,尚硅谷");
        StringBuffer str1 = str;

        str = null;
        System.gc();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(str1);
    }
}
```

输出

```java
Hello,尚硅谷
```

局部变量str指向stringBuffer实例所在堆空间，通过str可以操作该实例，那么str就是stringBuffer实例的强引用对应内存结构：

```java
StringBuffer str = new StringBuffer("hello,尚硅谷");
```

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-4/2021%2008%2009%2019%2003%2026%201628507006%201628507006650%20dG5OSA%20image-20210414100527620.png" alt="image-20210414100527620" style="zoom:33%;" />

**总结**

本例中的两个引用，都是强引用，强引用具备以下特点：

1. 强引用可以直接访问目标对象。
2. 强引用所指向的对象在任何时候都不会被系统回收，虚拟机宁愿抛出OOM异常，也不会回收强引用所指向对象。
3. 强引用可能导致内存泄漏。

### P8: 再谈引用：软引用

**软引用（Soft Reference）：内存不足即回收**

1. 软引用是用来描述一些还有用，但非必需的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。注意，这里的第一次回收是不可达的对象
2. 软引用通常用来实现内存敏感的缓存。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
3. 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（Reference Queue）。
4. 类似弱引用，只不过Java虚拟机会尽量让软引用的存活时间长一些，迫不得已才清理。
5. 一句话概括：当内存足够时，不会回收软引用可达的对象。内存不够时，会回收软引用的可达对象

在JDK1.2版之后提供了SoftReference类来实现软引用

```java
Object obj = new Object();// 声明强引用
SoftReference<Object> sf = new SoftReference<>(obj);
obj = null; //销毁强引用
```

**软引用代码举例**

代码

```java
public class SoftReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //创建对象，建立软引用
//        SoftReference<User> userSoftRef = new SoftReference<User>(new User(1, "songhk"));
        //上面的一行代码，等价于如下的三行代码
        User u1 = new User(1,"songhk");
        SoftReference<User> userSoftRef = new SoftReference<User>(u1);
        u1 = null;//取消强引用


        //从软引用中重新获得强引用对象
        System.out.println(userSoftRef.get());

        System.out.println("---目前内存还不紧张---");
        System.gc();
        System.out.println("After GC:");
//        //垃圾回收之后获得软引用中的对象
        System.out.println(userSoftRef.get());//由于堆空间内存足够，所有不会回收软引用的可达对象。
        System.out.println("---下面开始内存紧张了---");
        try {
            //让系统认为内存资源紧张、不够
//            byte[] b = new byte[1024 * 1024 * 7];
            byte[] b = new byte[1024 * 7168 - 635 * 1024];
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            //再次从软引用中获取数据
            System.out.println(userSoftRef.get());//在报OOM之前，垃圾回收器会回收软引用的可达对象。
        }
    }
}

```

JVM参数

```
-Xms10m -Xmx10m
```

在 JVM 内存不足时，会清理软引用对象

输出结果：

```java
[id=1, name=songhk] 
---目前内存还不紧张---
After GC:
[id=1, name=songhk] 
---下面开始内存紧张了---
null
java.lang.OutOfMemoryError: Java heap space
    at com.atguigu.java1.SoftReferenceTest.main(SoftReferenceTest.java:48)

Process finished with exit code 0
```

### P9: 再谈引用：弱引用

> **弱引用（Weak Reference）发现即回收**

1. 弱引用也是用来描述那些非必需对象，**只被弱引用关联的对象只能生存到下一次垃圾收集发生为止。在系统GC时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象**。
2. 但是，由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。
3. 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。
4. 软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。

在JDK1.2版之后提供了WeakReference类来实现弱引用

```java
// 声明强引用
Object obj = new Object();
WeakReference<Object> sf = new WeakReference<>(obj);
obj = null; //销毁强引用
```

弱引用对象与软引用对象的最大不同就在于，当GC在进行回收时，需要通过算法检查是否回收软引用对象，而对于弱引用对象，GC总是进行回收。弱引用对象更容易、更快被GC回收。

**面试题：你开发中使用过WeakHashMap吗？**

**弱引用代码举例**

```java
public class WeakReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //构造了弱引用
        WeakReference<User> userWeakRef = new WeakReference<User>(new User(1, "songhk"));
        //从弱引用中重新获取对象
        System.out.println(userWeakRef.get());

        System.gc();
        // 不管当前内存空间足够与否，都会回收它的内存
        System.out.println("After GC:");
        //重新尝试从弱引用中获取对象
        System.out.println(userWeakRef.get());
    }
}
```

执行垃圾回收后，软引用对象必定被清除

```java
[id=1, name=songhk] 
After GC:
null

Process finished with exit code 0
```

### P10: 再谈引用：虚引用

**虚引用（Phantom Reference）：对象回收跟踪**

1. 也称为“幽灵引用”或者“幻影引用”，是所有引用类型中最弱的一个
2. 一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收。
3. 它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的get()方法取得对象时，总是null 。**即通过虚引用无法获取到我们的数据**
4. **为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程。比如：能在这个对象被收集器回收时收到一个系统通知。**
5. 虚引用必须和引用队列一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
6. 由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虚引用中执行和记录。

在JDK1.2版之后提供了PhantomReference类来实现虚引用。

```java
// 声明强引用
Object obj = new Object();
// 声明引用队列
ReferenceQueue phantomQueue = new ReferenceQueue();
// 声明虚引用（还需要传入引用队列）
PhantomReference<Object> sf = new PhantomReference<>(obj, phantomQueue);
obj = null;
```

**虚引用代码示例**

```java
public class PhantomReferenceTest {
    public static PhantomReferenceTest obj;//当前类对象的声明
    static ReferenceQueue<PhantomReferenceTest> phantomQueue = null;//引用队列

    public static class CheckRefQueue extends Thread {
        @Override
        public void run() {
            while (true) {
                if (phantomQueue != null) {
                    PhantomReference<PhantomReferenceTest> objt = null;
                    try {
                        objt = (PhantomReference<PhantomReferenceTest>) phantomQueue.remove();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if (objt != null) {
                        System.out.println("追踪垃圾回收过程：PhantomReferenceTest实例被GC了");
                    }
                }
            }
        }
    }

    @Override
    protected void finalize() throws Throwable { //finalize()方法只能被调用一次！
        super.finalize();
        System.out.println("调用当前类的finalize()方法");
        obj = this;
    }

    public static void main(String[] args) {
        Thread t = new CheckRefQueue();
        t.setDaemon(true);//设置为守护线程：当程序中没有非守护线程时，守护线程也就执行结束。
        t.start();

        phantomQueue = new ReferenceQueue<PhantomReferenceTest>();
        obj = new PhantomReferenceTest();
        //构造了 PhantomReferenceTest 对象的虚引用，并指定了引用队列
        PhantomReference<PhantomReferenceTest> phantomRef = new PhantomReference<PhantomReferenceTest>(obj, phantomQueue);

        try {
            //不可获取虚引用中的对象
            System.out.println(phantomRef.get());
            System.out.println("第 1 次 gc");
            //将强引用去除
            obj = null;
            //第一次进行GC,由于对象可复活，GC无法回收该对象
            System.gc();
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用");
            }
            System.out.println("第 2 次 gc");
            obj = null;
            System.gc(); //一旦将obj对象回收，就会将此虚引用存放到引用队列中。
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

1、第一次尝试获取虚引用的值，发现无法获取的，这是因为虚引用是无法直接获取对象的值，然后进行第一次GC，因为会调用finalize方法，将对象复活了，所以对象没有被回收

2、但是调用第二次GC操作的时候，因为finalize方法只能执行一次，所以就触发了GC操作，将对象回收了，同时将会触发第二个操作就是将待回收的对象存入到引用队列中。

输出结果：

```java
null
第 1 次 gc
调用当前类的finalize()方法
obj 可用
第 2 次 gc
追踪垃圾回收过程：PhantomReferenceTest实例被GC了
obj 是 null

Process finished with exit code 0
```

### P11: 再谈引用：终结器引用（了解）

1. 它用于实现对象的finalize() 方法，也可以称为终结器引用
2. 无需手动编码，其内部配合引用队列使用
3. 在GC时，终结器引用入队。由Finalizer线程通过终结器引用找到被引用对象调用它的finalize()方法，第二次GC时才回收被引用的对象