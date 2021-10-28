# jvm

## 第六章: 堆

### P1: 概述

- **堆与进程**

  - 堆针对一个JVM进程来说是唯一的。也就是**一个进程只有一个JVM实例**，一个JVM实例中就有一个运行时数据区，一个运行时数据区只有一个堆和一个方法区。	
  - 但是**进程包含多个线程，他们是共享同一堆空间的**。

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2035%201628506775%201628506775656%20NB9180%20image-20210329114244343.png" alt="image-20210329114244343" style="zoom:33%" />

  - 一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域。
  - Java堆区在JVM启动的时候即被创建，其空间大小也就确定了，堆是JVM管理的最大一块内存空间，并且堆内存的大小是可以调节的。
  - 《Java虚拟机规范》规定，堆可以处于**物理上不连续**的内存空间中，但在逻**辑上它应该被视为连续**的。
  - 所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区（Thread Local Allocation Buffer，**TLAB**）。
  - 《Java虚拟机规范》中对Java堆的描述是：**所有的对象实例以及数组都应当在运行时分配在堆上**。（The heap is the run-time data area from which memory for all class instances and arrays is allocated）
    - 从实际使用角度看：“几乎”所有的对象实例都在堆分配内存，但并非全部。因为还有一些对象是在栈上分配的（逃逸分析，标量替换）
  - 数组和对象可能永远不会存储在栈上（**不一定**），因为栈帧中保存**引用**，这个引用指向对象或者数组在堆中的位置。
  - 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。
    - 也就是触发了GC的时候，才会进行回收
    - 如果堆中对象马上被回收，那么用户线程就会受到影响，因为有stop the word
  - 堆，是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域。

- **堆内存细分**

  现代垃圾收集器大部分都基于分代收集理论设计，堆空间细分为：

  - Java7 及之前堆内存逻辑上分为三部分：新生区+养老区+永久区

    <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2036%201628506776%201628506776638%20CSDW7z%20image-20210329114811713.png" alt="image-20210329114811713" style="zoom:25%" />

  - Java 8及之后堆内存逻辑上分为三部分：新生区+养老区+元空间	

    - Young Generation Space 新生区，又被划分为Eden区和Survivor区
    - Old generation space 养老区
    - Meta Space 元空间 Meta

    

### P2: JVisualVM可视化查看堆内存

### P3: 设置堆内存大小与OOM

- **设置堆内存**
- **OOM**

### P4: 新生代与老年代

- 存储在JVM中的Java对象可以被划分为两类:

  - 一类是生命周期较短的瞬时对象,这类对象的创建和消亡非常迅速
  - 另一类对象生命周期却非常长,在某些极端的情况下与JVM生命周期一致

- Java堆取进一步划分为 **新生代** 与 **老年代**

  - **新生代**
    - Eden空间
    - Survivor0 和 Suvivor1 空间 (也称from,to区)

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2037%201628506777%201628506777542%20hYKEst%20image-20210329120313545.png" alt="image-20210329120313545" style="zoom:33%" />

  

  - **参数设置**

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2038%201628506778%201628506778327%20GXTwXQ%20image-20210329144425626.png" alt="image-20210329144425626" style="zoom:30%" />

  - 配置新生代与老年代在堆结构的占比
    - 默认**-XX:NewRatio**=2，表示新生代占1，老年代占2，新生代占整个堆的1/3
    - 可以修改**-XX:NewRatio**=4，表示新生代占1，老年代占4，新生代占整个堆的1/5

  1. 在HotSpot中，Eden空间和另外两个survivor空间缺省所占的比例是8 : 1 : 1，
  2. 当然开发人员可以通过选项**-XX:SurvivorRatio**调整这个空间比例。比如-XX:SurvivorRatio=8
  3. 几乎所有的Java对象都是在Eden区被new出来的。
  4. 绝大部分的Java对象的销毁都在新生代进行了（有些大的对象在Eden区无法存储时候，将直接进入老年代），IBM公司的专门研究表明，新生代中80%的对象都是“朝生夕死”的。
  5. 可以使用选项"-Xmn"设置新生代最大内存大小，但这个参数一般使用默认值就可以了。

### P5: 对象分配过程

`为新对象分配内存是一件非常严谨和复杂的任务，JVM的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。`

- **具体过程 (一般情况 **

  1.我们创建的对象,一般是放在Eden区.当Eden区满了之后,就会触发GC操作,一般为YGC/minor GC操作

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2039%201628506779%201628506779136%20Hs8ooT%20image-20210329145506412.png" alt="image-20210329145506412" style="zoom:30%" />

  

  2.在进行了一次GC之后,红色的对象被回收,绿色的对象还被占用着,存放至 Survivor 0(From)区.同时为每个对象设置一个年龄计数器,经过一次回收之后还存在的对象,年龄加1;

  

  3.同时Eden区继续存放对象,当Eden区再次存满的时候,会再次出发GC,此时会把Eden和Survivor From中的对象进行一次垃圾收集,把存活的对象放到Survivor 1(To)区,年龄加1;

  [注]:	再次进行GC时,会将某一个存在对象的Survivor from区中的对象移到另一个Survivor To区,因此From To不固定指定

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2039%201628506779%201628506779782%20W0N4aH%20image-20210329150516986.png" alt="image-20210329150516986" style="zoom:30%" />

  

  4.我们继续不断的进行对象生成和垃圾回收,当Survivor中的对象年龄达到阈值15时,将会触发一次Promotion(晋升)操作,将该对象晋升至老年代当中

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2040%201628506780%201628506780494%20DbiEXT%20image-20210329150754205.png" alt="image-20210329150754205" style="zoom:30%" />



- **关于垃圾回收**

- 频繁在新生区收集，很少在养老区收集，几乎不在永久区/元空间收集。

- **特殊情况**

  1. 如果来了一个新对象，先看看 Eden 是否放的下？
     - 如果 Eden 放得下，则直接放到 Eden 区
     - 如果 Eden 放不下，则触发 YGC ，执行垃圾回收，看看还能不能放下？
  2. 将对象放到老年区又有两种情况：
     - 如果 Eden 执行了 YGC 还是无法放不下该对象，那没得办法，只能说明是超大对象，只能直接放到老年代
     - 那万一老年代都放不下，则先触发FullGC ，再看看能不能放下，放得下最好，但如果还是放不下，那只能报 OOM
  3. 如果 Eden 区满了，将对象往幸存区拷贝时，发现幸存区放不下啦，那只能便宜了某些新对象，让他们直接晋升至老年区

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2042%201628506782%201628506782218%20kIhZIR%20image-20210329151029749.png" alt="image-20210329151029749" style="zoom:30%" />



### P6: 常用调优工具

- JDK命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- Visual VM（实时监控，推荐）
- Jprofiler（IDEA插件）
- Java Flight Recorder（实时监控）
- GCViewer
- GCEasy



### P7: GC分类

1. 我们都知道，JVM的调优的一个环节，也就是垃圾收集，我们需要尽量的避免垃圾回收，因为在垃圾回收的过程中，容易出现STW（Stop the World）的问题，**而 Major GC 和 Full GC出现STW的时间，是Minor GC的10倍以上**
2. JVM在进行GC时，并非每次都对上面三个内存区域一起回收的，大部分时候回收的都是指新生代。针对Hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（FullGC）

- 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
  - **新生代收集**（Minor GC/Young GC）：只是新生代（Eden，s0，s1）的垃圾收集
  - **老年代收集**（Major GC/Old GC）：只是老年代的圾收集。
  - 目前，只有CMS GC会有单独收集老年代的行为。
  - 注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。目前，只有G1 GC会有这种行为
- **整堆收集**（Full GC）：收集整个java堆和方法区的垃圾收集。

> 由于历史原因,外界各种解读,majorGC和Full GC有些混淆

#### p1: Young GC (Minor GC

**年轻代 GC（Minor GC）触发机制**

1. 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden代满。Survivor满不会主动引发GC，在Eden区满的时候，会顺带触发s0区的GC，也就是被动触发GC（每次Minor GC会清理年轻代的内存）
2. 因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。这一定义既清晰又易于理解。
3. Minor GC会引发STW（Stop The World），暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2043%201628506783%201628506783524%20k6bUMV%20image-20210330103613539.png" alt="image-20210330103613539" style="zoom:28%" />



#### p2: Major / Full GC

**老年代GC（MajorGC）触发机制**

1. 指发生在老年代的GC，对象从老年代消失时，我们说 “Major Gc” 或 “Full GC” 发生了
2. 出现了MajorGc，经常会伴随至少一次的Minor GC。（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行MajorGC的策略选择过程）
   - 也就是在老年代空间不足时，会先尝试触发Minor GC，如果之后空间还不足，则触发Major GC
3. Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
4. 如果Major GC后，内存还不足，就报OOM了

**Full GC 触发机制**

**触发Full GC执行的情况有如下五种：**

1. 调用System.gc()时，系统建议执行FullGC，但是不必然执行
2. 老年代空间不足                                                                                                    
3. 方法区空间不足
4. 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
5. 由Eden区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

说明：Full GC 是开发或调优中尽量要避免的。这样STW时间会短一些



### P8: 堆空间分代思想

为什么要把Java堆分代？不分代就不能正常工作了吗？经研究，不同对象的生命周期不同。70%-99%的对象是临时对象。

- 新生代：有Eden、两块大小相同的survivor（又称为from/to或s0/s1）构成，to总为空。
- 老年代：存放新生代中经历多次GC仍然存活的对象。

其实不分代完全可以，**分代的唯一理由就是优化GC性能**。

- 如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。GC的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描。（性能低）

- 而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。（多回收新生代，少回收老年代，性能会提高很多）



### P9: 对象内存分配策略

1. 如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1。
2. 对象在Survivor区中每熬过一次MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代
3. 对象晋升老年代的年龄阀值，可以通过选项**-XX:MaxTenuringThreshold**来设置

**针对不同年龄段的对象分配原则如下所示：**

1. **优先分配到Eden**：开发中比较长的字符串或者数组，会直接存在老年代，但是因为新创建的对象都是朝生夕死的，所以这个大对象可能也很快被回收，但是因为老年代触发Major GC的次数比 Minor GC要更少，因此可能回收起来就会比较慢
2. **大对象直接分配到老年代**：尽量避免程序中出现过多的大对象
3. **长期存活的对象分配到老年代**
4. **动态对象年龄判断**：如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。
5. **空间分配担保**： -XX:HandlePromotionFailure 。



### P10: TLAB为对象分配内存

**为什么有 TLAB** (保证线程安全)

1. 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
2. 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
3. 为避免多个线程操作同一地址，需要使用**加锁等机制**，进而影响分配速度。

**什么是TLAB**

`TLAB（Thread Local Allocation Buffer）`

1. 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，**JVM为每个线程分配了一个私有缓存区域，它包含在Eden空间内**。

2. 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为**快速分配策略**。

3. 据我所知所有OpenJDK衍生出来的JVM都提供了TLAB的设计。

   <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2044%201628506784%201628506784510%20yX78yL%20image-20210330104846839.png" alt="image-20210330104846839" style="zoom:20%" />

- 每一个线程都有一个 TLAB空间
- 当一个线程的TLAB存满时,可以使用公共区域(蓝色)

**TLAB补充**

1. 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但**JVM确实是将TLAB作为内存分配的首选**。
2. 在程序中，开发人员可以通过选项“**-XX:UseTLAB**”设置是否开启TLAB空间。
3. 默认情况下，TLAB空间的内存非常小，仅占有整个Eden空间的1%，当然我们可以通过选项“**-XX:TLABWasteTargetPercent**”设置TLAB空间所占用Eden空间的百分比大小。
4. 一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过**使用加锁机制确保数据操作的原子性**，从而直接在Eden空间中分配内存。

> 1、哪个线程要分配内存，就在哪个线程的本地缓冲区中分配，只有本地缓冲区用完 了，**分配新的缓存区时才需要同步锁定** ----这是《深入理解JVM》--第三版里说的
>
> 2、和这里讲的有点不同。我猜测说的意思是某一次分配，如果TLAB用完了，那么**这一次**先在Eden区直接分配。空闲下来后再加锁分配新的TLAB（TLAB内存较大，分配时间应该较长）

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2045%201628506785%201628506785336%2000W26j%20image-20210330105333272.png" alt="image-20210330105333272" style="zoom:33%;" />



### P11: 堆空间参数设置

#### p1: 常用参数设置

```java
/**
 * 测试堆空间常用的jvm参数：
 * -XX:+PrintFlagsInitial : 查看所有的参数的默认初始值
 * -XX:+PrintFlagsFinal  ：查看所有的参数的最终值（可能会存在修改，不再是初始值）
 * 具体查看某个参数的指令： jps：查看当前运行中的进程
 *                      jinfo -flag SurvivorRatio 进程id
 *
 * -Xms：初始堆空间内存 （默认为物理内存的1/64）
 * -Xmx：最大堆空间内存（默认为物理内存的1/4）
 * -Xmn：设置新生代的大小。(初始值及最大值)
 * -XX:NewRatio：配置新生代与老年代在堆结构的占比
 * -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例
 * -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄
 * -XX:+PrintGCDetails：输出详细的GC处理日志
 * 打印gc简要信息：① -XX:+PrintGC   ② -verbose:gc
 * -XX:HandlePromotionFailure：是否设置空间分配担保
 */
```



#### p2: 空间分配担保

1、在发生Minor GC之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间。

- 如果大于，则此次Minor GC是安全的
- 如果小于，则虚拟机会查看**-XX:HandlePromotionFailure**设置值是否允担保失败。
  - 如果HandlePromotionFailure=true，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小。
    - 如果大于，则尝试进行一次Minor GC，但这次Minor GC依然是有风险的；
    - 如果小于，则进行一次Full GC。
  - 如果HandlePromotionFailure=false，则进行一次Full GC。

**历史版本**

1. 在JDK6 Update 24之后，HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，观察openJDK中的源码变化，虽然源码中还定义了HandlePromotionFailure参数，但是在代码中已经不会再使用它。
2. JDK6 Update 24之后的规则变为**只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC**，否则将进行Full GC。即 HandlePromotionFailure=true



### P12: 堆是分配对象的唯一选择吗

**在《深入理解Java虚拟机》中关于Java堆内存有这样一段描述：**

1. 随着JIT编译期的发展与**逃逸分析技术**逐渐成熟，**栈上分配、标量替换**优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。
2. 在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是**如果经过逃逸分析（Escape Analysis）后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配**。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。
3. 此外，前面提到的基于OpenJDK深度定制的TaoBao VM，其中创新的GCIH（GC invisible heap）技术实现off-heap，将生命周期较长的Java对象从heap中移至heap外，并且GC不能管理GCIH内部的Java对象，以此达到降低GC的回收频率和提升GC的回收效率的目的。



#### p1: 逃逸分析

1. 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
2. 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
3. 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。
4. 逃逸分析的基本行为就是分析对象动态作用域：
   - 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
   - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。

**逃逸分析举例**

1、没有发生逃逸的对象，则可以分配到栈（无线程安全问题）上，随着方法执行的结束，栈空间就被移除（也就无需GC）

```java
public void my_method() {
    V v = new V();
    // use v
    // ....
    v = null;
}
```

2、下面代码中的 StringBuffer sb 发生了逃逸，不能在栈上分配

```java
public static StringBuffer createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
```

3、如果想要StringBuffer sb不发生逃逸，可以这样写

```java
public static String createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
```

```java
/**
 * 逃逸分析
 *
 *  如何快速的判断是否发生了逃逸分析，大家就看new的对象实体是否有可能在方法外被调用。
 */
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /*
    方法返回EscapeAnalysis对象，发生逃逸
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /*
    为成员属性赋值，发生逃逸
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    //思考：如果当前的obj引用声明为static的？仍然会发生逃逸。

    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
        //getInstance().xxx()同样会发生逃逸
    }
}
```

**逃逸分析参数设置**

1. 在JDK 1.7 版本之后，HotSpot中默认就已经开启了逃逸分析
2. 如果使用的是较早的版本，开发人员则可以通过：
   - 选项“-XX:+DoEscapeAnalysis"显式开启逃逸分析
   - 通过选项“-XX:+PrintEscapeAnalysis"查看逃逸分析的筛选结果

**总结**

开发中能使用局部变量的，就不要使用在方法外定义。



#### p2: 代码优化

使用逃逸分析，编译器可以对代码做如下优化：

1. **栈上分配**：将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会发生逃逸，对象可能是栈上分配的候选，而不是堆上分配
2. **同步省略**：如果一个对象被发现只有一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
3. **分离对象或标量替换**：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

##### D1: 栈上分配

1. JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了。
2. 常见的栈上分配的场景：在逃逸分析中，已经说明了，分别是给成员变量赋值、方法返回值、实例引用传递。

```java
/**
 * 栈上分配测试
 * 关闭逃逸分析
 * -Xmx128m -Xms128m -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();//未发生逃逸
    }

    static class User {

    }
}

// 输出结果  发生了GC
[GC (Allocation Failure) [PSYoungGen: 33280K->808K(38400K)] 33280K->816K(125952K), 0.0483350 secs] [Times: user=0.00 sys=0.00, real=0.06 secs] 
[GC (Allocation Failure) [PSYoungGen: 34088K->808K(38400K)] 34096K->816K(125952K), 0.0008411 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 34088K->792K(38400K)] 34096K->800K(125952K), 0.0008427 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 34072K->808K(38400K)] 34080K->816K(125952K), 0.0012223 secs] [Times: user=0.08 sys=0.00, real=0.00 secs] 
花费的时间为： 114 ms

  
// 开启逃逸分析
// -Xmx128m -Xms128m -XX:+DoEscapeAnalysis -XX:+PrintGCDetails
// 没有发生GC 
花费的时间为： 5 ms
  
```



##### D2: 同步省略

1. 线程同步的代价是相当高的，同步的后果是降低并发性和性能。
2. 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来**判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程**。
3. 如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个**取消同步的过程就叫同步省略，也叫锁消除**。也称为同步消除

```java
✨ public void f() {
    Object hollis = new Object();
    synchronized(hollis) {
        System.out.println(hollis);
    }
}

//代码中对hollis这个对象加锁，但是hollis对象的生命周期只在f()方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化

//优化成：
public void f() {
    Object hellis = new Object();
    System.out.println(hellis);
}

// ✨代码的字节码分析
 0 new #2 <java/lang/Object>
 3 dup
 4 invokespecial #1 <java/lang/Object.<init>>
 7 astore_1
 8 aload_1
 9 dup
10 astore_2
11 monitorenter 🔑
12 getstatic #3 <java/lang/System.out>
15 aload_1
16 invokevirtual #4 <java/io/PrintStream.println>
19 aload_2
20 monitorexit 🔑
21 goto 29 (+8)
24 astore_3
25 aload_2
26 monitorexit
27 aload_3
28 athrow
29 return

```

注意：字节码文件中并没有进行优化，可以看到加锁和释放锁的操作依然存在，**同步省略操作是在解释运行时发生的**



##### D3: 标量替换

**分离对象或标量替换**

1. 标量（scalar）是指一个无法再分解成更小的数据的数据。Java中的原始数据类型就是标量。
2. 相对的，那些还可以分解的数据叫做聚合量（Aggregate），Java中的对象就是聚合量，因为他可以分解成其他聚合量和标量。
3. 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是标量替换。

```java
public static void main(String args[]) {
    alloc();
}
private static void alloc() {
    Point point = new Point(1,2);
    System.out.println("point.x" + point.x + ";point.y" + point.y);
}
class Point {
    private int x;
    private int y;
}

//以上代码经过标量替换
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println("point.x = " + x + "; point.y=" + y);
}

/**
1.可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个聚合量了。
2.那么标量替换有什么好处呢？就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。
3.标量替换为栈上分配提供了很好的基础。
*/



// 标量替换参数设置
参数 -XX:+ElimilnateAllocations：开启了标量替换（默认打开），允许将对象打散分配在栈上。

/**
 * 标量替换测试
 *  -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations
 */
public class ScalarReplace {
    public static class User {
        public int id;
        public String name;
    }

    public static void alloc() {
        User u = new User();//未发生逃逸
        u.id = 5;
        u.name = "www.atguigu.com";
    }

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
    }
}
//输出结果 
[GC (Allocation Failure)  25600K->880K(98304K), 0.0012658 secs]
[GC (Allocation Failure)  26480K->832K(98304K), 0.0012124 secs]
[GC (Allocation Failure)  26432K->784K(98304K), 0.0009719 secs]
[GC (Allocation Failure)  26384K->832K(98304K), 0.0009071 secs]
[GC (Allocation Failure)  26432K->768K(98304K), 0.0010643 secs]
[GC (Allocation Failure)  26368K->824K(101376K), 0.0012354 secs]
[GC (Allocation Failure)  32568K->712K(100864K), 0.0011291 secs]
[GC (Allocation Failure)  32456K->712K(100864K), 0.0006368 secs]
花费的时间为： 99 ms
  

// 开启标量替换 
// -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:+EliminateAllocations
花费的时间为： 6 ms

//上述代码在主函数中调用了1亿次alloc()方法，进行对象创建由于User对象实例需要占据约16字节的空间，因此累计分配空间达到将近1.5GB。如果堆空间小于这个值，就必然会发生GC。使用如下参数运行上述代码：
-server -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:+EliminateAllocations

这里设置参数如下：

参数 -server：启动Server模式，因为在server模式下，才可以启用逃逸分析。
参数 -XX:+DoEscapeAnalysis：启用逃逸分析
参数 -Xmx10m：指定了堆空间最大为10MB
参数 -XX:+PrintGC：将打印GC日志。
参数 -XX:+EliminateAllocations：开启了标量替换（默认打开），允许将对象打散分配在栈上，比如对象拥有id和name两个字段，那么这两个字段将会被视为两个独立的局部变量进行分配  
```



#### p3: 逃逸分析的不足

1. 关于逃逸分析的论文在1999年就已经发表了，但直到JDK1.6才有实现，而且这项技术到如今也并不是十分成熟的。
2. 其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。
3. 一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
4. 虽然这项技术并不十分成熟，但是它也是即时编译器优化技术中一个十分重要的手段。
5. 注意到有一些观点，认为通过逃逸分析，JVM会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于JVM设计者的选择。据我所知，**Oracle Hotspot JVM中并未这么做**（刚刚演示的效果，是因为HotSpot实现了标量替换），这一点在逃逸分析相关的文档里已经说明，**所以可以明确在HotSpot虚拟机上，所有的对象实例都是创建在堆上**。
6. 目前很多书籍还是基于JDK7以前的版本，JDK已经发生了很大变化，intern字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是**intern字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配**，**所以这一点同样符合前面一点的结论：对象实例都是分配在堆上**。

> **结论:** 对象实例都是分配在堆上



### P13: 小结

1. 年轻代是对象的诞生、成长、消亡的区域，一个对象在这里产生、应用，最后被垃圾回收器收集、结束生命。
2. 老年代放置长生命周期的对象，通常都是从Survivor区域筛选拷贝过来的Java对象。
3. 当然，也有特殊情况，我们知道普通的对象可能会被分配在TLAB上；
4. 如果对象较大，无法分配在 TLAB 上，则JVM会试图直接分配在Eden其他位置上；
5. 如果对象太大，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代。
6. 当GC只发生在年轻代中，回收年轻代对象的行为被称为Minor GC。
7. 当GC发生在老年代时则被称为Major GC或者Full GC。
8. 一般的，Minor GC的发生频率要比Major GC高很多，即老年代中垃圾回收发生的频率将大大低于年轻代。



## 第七章: 方法区

### P1: 栈,堆,方法区的交互关系

ThreadLocal：如何保证多个线程在并发环境下的安全性？典型场景就是数据库连接管理，以及会话管理。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2046%201628506786%201628506786201%20QiJBFU%20image-20210401154330032.png" alt="image-20210401154330032" style="zoom:36%" />

**举例:**

`下面涉及了对象的访问定位`

1. Person 类的 .class 信息存放在方法区中
2. person 变量存放在 Java 栈的局部变量表中
3. 真正的 person 对象存放在 Java 堆中
4. 在 person 对象中，有个指针指向方法区中的 person 类型数据，表明这个 person 对象是用方法区中的 Person 类 new 出来的

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2047%201628506787%201628506787429%20oIUF8q%20image-20210401154532358.png" alt="image-20210401154532358" style="zoom:33%" />



### P2: 方法区的理解

#### p1: 方法区在哪?

1. 《Java虚拟机规范》中明确说明：尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。但对于HotSpotJVM而言，方法区还有一个别名叫做Non-Heap（非堆），目的就是要和堆分开。
2. 所以，**方法区可以看作是一块独立于Java堆的内存空间**。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2048%201628506788%201628506788334%20O2VDcW%20image-20210401154900412.png" alt="image-20210401154900412" style="zoom:25%" />



#### p2: 方法区的基本理解

`方法区主要存放的是 Class，而堆中主要存放的是实例化的对象`

1. 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。多个线程同时加载统一个类时，只能有一个线程能加载该类，其他线程只能等等待该线程加载完毕，然后直接使用该类，即类只能加载一次。

2. 方法区在JVM启动的时候被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。

3. 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。

4. 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：

   ```java
   java.lang.OutofMemoryError:PermGen space
   ```

   或者

   ```java
   java.lang.OutOfMemoryError:Metaspace
   ```

   - 加载大量的第三方的jar包
   - Tomcat部署的工程过多（30~50个）
   - 大量动态的生成反射类

5. 关闭JVM就会释放这个区域的内存。

 🐴代码举例

```java
public class MethodAreaDemo {
    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("end...");
    }
}
```

很简单的一段代码,却加载了1600多个类

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2049%201628506789%201628506789148%20ZvolQx%20image-20210401155458705.png" alt="image-20210401155458705" style="zoom:33%" />



#### p3: HotSpot方法区演进

1. 在 JDK7 及以前，习惯上把方法区，称为永久代。JDK8开始，使用元空间取代了永久代。我们可以将方法区类比为Java中的接口，将永久代或元空间类比为Java中具体的实现类
2. 本质上，方法区和永久代并不等价。仅是对Hotspot而言的可以看作等价。《Java虚拟机规范》对如何实现方法区，不做统一要求。例如：BEAJRockit / IBM J9 中不存在永久代的概念。
   - 现在来看，当年使用永久代，不是好的idea。导致Java程序更容易OOm（超过-XX:MaxPermsize上限）
3. 而到了JDK8，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Metaspace）来代替
4. 元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：**元空间不在虚拟机设置的内存中，而是使用本地内存**。
5. 永久代、元空间二者并不只是名字变了，内部结构也调整了
6. 根据《Java虚拟机规范》的规定，如果方法区无法满足新的内存分配需求时，将抛出OOM异常

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2050%201628506790%201628506790273%20UJc5pM%20image-20210401155751108.png" alt="image-20210401155751108" style="zoom:33%" />



### P3: 设置方法区大小与OOM

`方法区的大小不必是固定的，JVM可以根据应用的需要动态调整。`

#### p1: JDK7及以前(永久代)

1. 通过-XX:Permsize来设置永久代初始分配空间。默认值是20.75M
2. -XX:MaxPermsize来设定永久代最大可分配空间。32位机器默认是64M，64位机器模式是82M
3. 当JVM加载的类信息容量超过了这个值，会报异常OutofMemoryError:PermGen space。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2051%201628506791%201628506791281%20bFW1Mk%20image-20210401155945982.png" alt="image-20210401155945982" style="zoom:33%" />



#### p2: JDK8及以后(元空间)

1. 元数据区大小可以使用参数 **-XX:MetaspaceSize** 和 **-XX:MaxMetaspaceSize** 指定
2. 默认值依赖于平台，Windows下，-XX:MetaspaceSize 约为21M，-XX:MaxMetaspaceSize的值是-1，即没有限制。
3. 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据区发生溢出，虚拟机一样会抛出异常OutOfMemoryError:Metaspace
4. -XX:MetaspaceSize：设置初始的元空间大小。对于一个 64位 的服务器端 JVM 来说，其默认的 -XX:MetaspaceSize值为21MB。这就是初始的高水位线，一旦触及这个水位线，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置。新的高水位线的值取决于GC后释放了多少元空间。如果释放的空间不足，那么在不超过MaxMetaspaceSize时，适当提高该值。如果释放空间过多，则适当降低该值。
5. 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到Full GC多次调用。为了避免频繁地GC，建议将-XX:MetaspaceSize设置为一个相对较高的值。



#### p3: 方法区OOM

**🐴代码举例**

OOMTest 类继承 ClassLoader 类，获得 defineClass() 方法，可自己进行类的加载

```java
/**
 * jdk6/7中：
 * -XX:PermSize=10m -XX:MaxPermSize=10m
 *
 * jdk8中：
 * -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
 *
 */
public class OOMTest extends ClassLoader {
    public static void main(String[] args) {
        int j = 0;
        try {
            OOMTest test = new OOMTest();
            for (int i = 0; i < 10000; i++) {
                //创建ClassWriter对象，用于生成类的二进制字节码
                ClassWriter classWriter = new ClassWriter(0);
                //指明版本号，修饰符，类名，包名，父类，接口
                classWriter.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                //返回byte[]
                byte[] code = classWriter.toByteArray();
                //类的加载
                test.defineClass("Class" + i, code, 0, code.length);//Class对象
                j++;
            }
        } finally {
            System.out.println(j);
        }
    }
}

//不设置元空间的上限 使用默认的Jvm默认参数 
输出: 10000
//设置元空间的上限
  -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m

输出: 8531
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
    at com.atguigu.java.OOMTest.main(OOMTest.java:29)
```



#### p4: 如何解决OOM

1. 要解决OOM异常或heap space的异常，一般的手段是首先通过内存映像分析工具（如Ec1ipse Memory Analyzer）对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）
2. **内存泄漏**就是有大量的引用指向某些对象，但是这些对象以后不会使用了，但是因为它们还和GC ROOT有关联，所以导致以后这些对象也不会被回收，这就是内存泄漏的问题
3. 如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链。于是就能找到泄漏对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GC Roots引用链的信息，就可以比较准确地定位出泄漏代码的位置。
4. 如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。



### P4: 方法区的内部结构

#### p1: 方法区存储什么?

- **概念**

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2052%201628506792%201628506792268%20ut2i54%20image-20210401221106362.png" alt="image-20210401221106362" style="zoom:30%" />

  

  《深入理解Java虚拟机》书中对方法区（Method Area）存储内容描述如下：它用于存储已被虚拟机加载的**类型信息、常量、静态变量、即时编译器编译后的代码缓存**等。

  

  <img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2053%201628506793%201628506793282%20xMQbY5%20image-20210401222000718.png" alt="image-20210401222000718" style="zoom:33%" />

  - **类型信息**

    `对每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：`

    1. 这个类型的完整有效名称（全名=包名.类名）
    2. 这个类型直接父类的完整有效名（对于interface或是java.lang.Object，都没有父类）
    3. 这个类型的修饰符（public，abstract，final的某个子集）
    4. 这个类型直接接口的一个有序列表

  - **域信息**

    `也就是常说的成员变量,域信息是比较官方的称呼`

    1. JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
    2. 域的相关信息包括：域名称，域类型，域修饰符（public，private，protected，static，final，volatile，transient的某个子集）

  - **方法信息**

    `JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序：`

    1. 方法名称
    2. 方法的返回类型（包括 void 返回类型），void 在 Java 中对应的为 void.class
    3. 方法参数的数量和类型（按顺序）
    4. 方法的修饰符（public，private，protected，static，final，synchronized，native，abstract的一个子集）
    5. 方法的字节码（bytecodes）、操作数栈、局部变量表及大小（abstract和native方法除外）
    6. 异常表（abstract和native方法除外），异常表记录每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

  **🐴代码举例**

  ```java
  **
   * 测试方法区的内部构成
   */
  public class MethodInnerStrucTest extends Object implements Comparable<String>,Serializable {
      //属性
      public int num = 10;
      private static String str = "测试方法的内部结构";
      //构造器
      //方法
      public void test1(){
          int count = 20;
          System.out.println("count = " + count);
      }
      public static int test2(int cal){
          int result = 0;
          try {
              int value = 30;
              result = value / cal;
          } catch (Exception e) {
              e.printStackTrace();
          }
          return result;
      }
  
      @Override
      public int compareTo(String o) {
          return 0;
      }
  }
  ```

  ​	**反编译**

  ```java
  javap -v -p MethodInnerStrucTest.class > test.txt
  //反编译字节码文件，并输出值文本文件中，便于查看。参数 -p 确保能查看 private 权限类型的字段或方法  
  ```

   	**字节码**

  ```java
  Classfile /F:/IDEAWorkSpaceSourceCode/JVMDemo/out/production/chapter09/com/atguigu/java/MethodInnerStrucTest.class
    Last modified 2020-11-13; size 1626 bytes
    MD5 checksum 0d0fcb54854d4ce183063df985141ad0
    Compiled from "MethodInnerStrucTest.java"
  //类型信息      
  public class com.atguigu.java.MethodInnerStrucTest extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
    minor version: 0
    major version: 52
    flags: ACC_PUBLIC, ACC_SUPER
  Constant pool:
     #1 = Methodref          #18.#52        // java/lang/Object."<init>":()V
     #2 = Fieldref           #17.#53        // com/atguigu/java/MethodInnerStrucTest.num:I
     #3 = Fieldref           #54.#55        // java/lang/System.out:Ljava/io/PrintStream;
     #4 = Class              #56            // java/lang/StringBuilder
     #5 = Methodref          #4.#52         // java/lang/StringBuilder."<init>":()V
     #6 = String             #57            // count =
     #7 = Methodref          #4.#58         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
     #8 = Methodref          #4.#59         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
     #9 = Methodref          #4.#60         // java/lang/StringBuilder.toString:()Ljava/lang/String;
    #10 = Methodref          #61.#62        // java/io/PrintStream.println:(Ljava/lang/String;)V
    #11 = Class              #63            // java/lang/Exception
    #12 = Methodref          #11.#64        // java/lang/Exception.printStackTrace:()V
    #13 = Class              #65            // java/lang/String
    #14 = Methodref          #17.#66        // com/atguigu/java/MethodInnerStrucTest.compareTo:(Ljava/lang/String;)I
    #15 = String             #67            // 测试方法的内部结构
    #16 = Fieldref           #17.#68        // com/atguigu/java/MethodInnerStrucTest.str:Ljava/lang/String;
    #17 = Class              #69            // com/atguigu/java/MethodInnerStrucTest
    #18 = Class              #70            // java/lang/Object
    #19 = Class              #71            // java/lang/Comparable
    #20 = Class              #72            // java/io/Serializable
    #21 = Utf8               num
    #22 = Utf8               I
    #23 = Utf8               str
    #24 = Utf8               Ljava/lang/String;
    #25 = Utf8               <init>
    #26 = Utf8               ()V
    #27 = Utf8               Code
    #28 = Utf8               LineNumberTable
    #29 = Utf8               LocalVariableTable
    #30 = Utf8               this
    #31 = Utf8               Lcom/atguigu/java/MethodInnerStrucTest;
    #32 = Utf8               test1
    #33 = Utf8               count
    #34 = Utf8               test2
    #35 = Utf8               (I)I
    #36 = Utf8               value
    #37 = Utf8               e
    #38 = Utf8               Ljava/lang/Exception;
    #39 = Utf8               cal
    #40 = Utf8               result
    #41 = Utf8               StackMapTable
    #42 = Class              #63            // java/lang/Exception
    #43 = Utf8               compareTo
    #44 = Utf8               (Ljava/lang/String;)I
    #45 = Utf8               o
    #46 = Utf8               (Ljava/lang/Object;)I
    #47 = Utf8               <clinit>
    #48 = Utf8               Signature
    #49 = Utf8               Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
    #50 = Utf8               SourceFile
    #51 = Utf8               MethodInnerStrucTest.java
    #52 = NameAndType        #25:#26        // "<init>":()V
    #53 = NameAndType        #21:#22        // num:I
    #54 = Class              #73            // java/lang/System
    #55 = NameAndType        #74:#75        // out:Ljava/io/PrintStream;
    #56 = Utf8               java/lang/StringBuilder
    #57 = Utf8               count =
    #58 = NameAndType        #76:#77        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
    #59 = NameAndType        #76:#78        // append:(I)Ljava/lang/StringBuilder;
    #60 = NameAndType        #79:#80        // toString:()Ljava/lang/String;
    #61 = Class              #81            // java/io/PrintStream
    #62 = NameAndType        #82:#83        // println:(Ljava/lang/String;)V
    #63 = Utf8               java/lang/Exception
    #64 = NameAndType        #84:#26        // printStackTrace:()V
    #65 = Utf8               java/lang/String
    #66 = NameAndType        #43:#44        // compareTo:(Ljava/lang/String;)I
    #67 = Utf8               测试方法的内部结构
    #68 = NameAndType        #23:#24        // str:Ljava/lang/String;
    #69 = Utf8               com/atguigu/java/MethodInnerStrucTest
    #70 = Utf8               java/lang/Object
    #71 = Utf8               java/lang/Comparable
    #72 = Utf8               java/io/Serializable
    #73 = Utf8               java/lang/System
    #74 = Utf8               out
    #75 = Utf8               Ljava/io/PrintStream;
    #76 = Utf8               append
    #77 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
    #78 = Utf8               (I)Ljava/lang/StringBuilder;
    #79 = Utf8               toString
    #80 = Utf8               ()Ljava/lang/String;
    #81 = Utf8               java/io/PrintStream
    #82 = Utf8               println
    #83 = Utf8               (Ljava/lang/String;)V
    #84 = Utf8               printStackTrace
  {
  //域信息
    public int num;
      descriptor: I
      flags: ACC_PUBLIC
  
    private static java.lang.String str;
      descriptor: Ljava/lang/String;
      flags: ACC_PRIVATE, ACC_STATIC
  
    //方法信息
    public com.atguigu.java.MethodInnerStrucTest();
      descriptor: ()V
      flags: ACC_PUBLIC
      Code:
        stack=2, locals=1, args_size=1
           0: aload_0
           1: invokespecial #1                  // Method java/lang/Object."<init>":()V
           4: aload_0
           5: bipush        10
           7: putfield      #2                  // Field num:I
          10: return
        LineNumberTable:
          line 10: 0
          line 12: 4
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              0      11     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
  
    public void test1();
      descriptor: ()V
      flags: ACC_PUBLIC
      Code:
        stack=3, locals=2, args_size=1
           0: bipush        20
           2: istore_1
           3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
           6: new           #4                  // class java/lang/StringBuilder
           9: dup
          10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
          13: ldc           #6                  // String count =
          15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
          18: iload_1
          19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
          22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
          25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
          28: return
        LineNumberTable:
          line 17: 0
          line 18: 3
          line 19: 28
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              0      29     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
              3      26     1 count   I
  
    public static int test2(int);
      descriptor: (I)I
      flags: ACC_PUBLIC, ACC_STATIC
      Code:
        stack=2, locals=3, args_size=1
           0: iconst_0
           1: istore_1
           2: bipush        30
           4: istore_2
           5: iload_2
           6: iload_0
           7: idiv
           8: istore_1
           9: goto          17
          12: astore_2
          13: aload_2
          14: invokevirtual #12                 // Method java/lang/Exception.printStackTrace:()V
          17: iload_1
          18: ireturn
        Exception table:
           from    to  target type
               2     9    12   Class java/lang/Exception
        LineNumberTable:
          line 21: 0
          line 23: 2
          line 24: 5
          line 27: 9
          line 25: 12
          line 26: 13
          line 28: 17
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              5       4     2 value   I
             13       4     2     e   Ljava/lang/Exception;
              0      19     0   cal   I
              2      17     1 result   I
        StackMapTable: number_of_entries = 2
          frame_type = 255 /* full_frame */
            offset_delta = 12
            locals = [ int, int ]
            stack = [ class java/lang/Exception ]
          frame_type = 4 /* same */
  
    public int compareTo(java.lang.String);
      descriptor: (Ljava/lang/String;)I
      flags: ACC_PUBLIC
      Code:
        stack=1, locals=2, args_size=2
           0: iconst_0
           1: ireturn
        LineNumberTable:
          line 33: 0
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              0       2     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
              0       2     1     o   Ljava/lang/String;
  
    public int compareTo(java.lang.Object);
      descriptor: (Ljava/lang/Object;)I
      flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
      Code:
        stack=2, locals=2, args_size=2
           0: aload_0
           1: aload_1
           2: checkcast     #13                 // class java/lang/String
           5: invokevirtual #14                 // Method compareTo:(Ljava/lang/String;)I
           8: ireturn
        LineNumberTable:
          line 10: 0
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              0       9     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
  
    static {};
      descriptor: ()V
      flags: ACC_STATIC
      Code:
        stack=1, locals=0, args_size=0
           0: ldc           #15                 // String 测试方法的内部结构
           2: putstatic     #16                 // Field str:Ljava/lang/String;
           5: return
        LineNumberTable:
          line 13: 0
  }
  Signature: #49                          // Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
  SourceFile: "MethodInnerStrucTest.java"
  ```

  ```java
  //类型信息
  在运行时方法区中，类信息中记录了哪个加载器加载了该类，同时类加载器也记录了它加载了哪些类
  public class com.atguigu.java.MethodInnerStrucTest extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
  
  //域信息
  1.descriptor: I 表示字段类型为 Integer
  2.flags: ACC_PUBLIC 表示字段权限修饰符为 public
  
    public int num;
      descriptor: I
      flags: ACC_PUBLIC
  
    private static java.lang.String str;
      descriptor: Ljava/lang/String;
      flags: ACC_PRIVATE, ACC_STATIC
  
  //方法信息
    descriptor: ()V 表示方法返回值类型为 void
  	flags: ACC_PUBLIC 表示方法权限修饰符为 public	
  	stack=3 表示操作数栈深度为 3
  	locals=2 表示局部变量个数为 2 个（实力方法包含 this）
  	test1() 方法虽然没有参数，但是其 args_size=1 ，这时因为将 this 作为了参数
      
  public void test1();
      descriptor: ()V
      flags: ACC_PUBLIC
      Code:
        stack=3, locals=2, args_size=1
           0: bipush        20
           2: istore_1
           3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
           6: new           #4                  // class java/lang/StringBuilder
           9: dup
          10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
          13: ldc           #6                  // String count =
          15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
          18: iload_1
          19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
          22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
          25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
          28: return
        LineNumberTable:
          line 17: 0
          line 18: 3
          line 19: 28
        LocalVariableTable:
          Start  Length  Slot  Name   Signature
              0      29     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
              3      26     1 count   I
  ```

  

#### p2: non-final 类型的类变量

1. 静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分
2. 类变量被类的所有实例共享，即使没有类实例时，你也可以访问它

**🐴代码举例**

1. 如下代码所示，即使我们把order设置为null，也不会出现空指针异常
2. 这更加表明了 static 类型的字段和方法随着类的加载而加载，并不属于特定的类实例

```java
public class MethodAreaTest {
    public static void main(String[] args) {
        Order order = null;
        order.hello();
        System.out.println(order.count);
    }
}

class Order {
    public static int count = 1;
    public static final int number = 2;


    public static void hello() {
        System.out.println("hello!");
    }
}

// 输出结果 
hello!
1
```



**全局常量: static final**

1. 全局常量就是使用 static final 进行修饰
2. 被声明为final的类变量的处理方法则不同，每个全局常量在编译的时候就会被分配了。

查看上面代码，这部分的字节码指令

```java
class Order {
    public static int count = 1;
    public static final int number = 2;
    ...
}    
```

```java
public static int count;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC

  public static final int number;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 2
```

可以发现 staitc和final同时修饰的number 的值在编译上的时候已经写死在字节码文件中了。



#### p3: 运行时常量池

**常量池与运行时常量池**

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2054%201628506794%201628506794172%20to00Ta%20image-20210407120943139.png" alt="image-20210407120943139" style="zoom:30%" />

1. 方法区，内部包含了运行时常量池
2. 字节码文件，内部包含了常量池。（之前的字节码文件中已经看到了很多Constant pool的东西，这个就是常量池）
3. 要弄清楚方法区，需要理解清楚ClassFile，因为加载类的信息都在方法区。
4. 要弄清楚方法区的运行时常量池，需要理解清楚ClassFile中的常量池。



##### D1: 常量池

1. 一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述符信息外。还包含一项信息就是**常量池表**（**Constant Pool Table**），包括各种字面量和对类型、域和方法的符号引用。
2. 字面量： 10 ， “我是某某”这种数字和字符串都是字面量

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2055%201628506795%201628506795329%20i8V8I3%20image-20210407121144825.png" alt="image-20210407121144825" style="zoom:30%" />

- **为什么需要常量池？**

  一个java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池。这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池，之前有介绍

  比如：如下的代码：

  ```java
  public class SimpleClass {
      public void sayHello() {
          System.out.println("hello");
      }
  }
  ```

  1. 虽然上述代码只有194字节，但是里面却使用了String、System、PrintStream及Object等结构。
  2. 比如说我们这个文件中有6个地方用到了"hello"这个字符串，如果不用常量池，就需要在6个地方全写一遍，造成臃肿。我们可以将"hello"等所需用到的结构信息记录在常量池中，并通过**引用的方式**，来加载、调用所需的结构
  3. 这里的代码量其实很少了，如果代码多的话，引用的结构将会更多，这里就需要用到常量池了。

- **常量池中有啥？**

  1. 数量值
  2. 字符串值
  3. 类引用
  4. 字段引用
  5. 方法引用

  MethodInnerStrucTest 的 test1方法的字节码	

  ```java
   0 bipush 20
   2 istore_1
   3 getstatic #3 <java/lang/System.out>
   6 new #4 <java/lang/StringBuilder>
   9 dup
  10 invokespecial #5 <java/lang/StringBuilder.<init>>
  13 ldc #6 <count = >
  15 invokevirtual #7 <java/lang/StringBuilder.append>
  18 iload_1
  19 invokevirtual #8 <java/lang/StringBuilder.append>
  22 invokevirtual #9 <java/lang/StringBuilder.toString>
  25 invokevirtual #10 <java/io/PrintStream.println>
  28 return
     
  // #3，#5等等这些带# 的，都是引用了常量池。
  ```

- **常量池总结**

  常量池、可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型。



##### D2: 运行时常量池

1. 运行时常量池（Runtime Constant Pool）是方法区的一部分。
2. 常量池表（Constant Pool Table）是Class字节码文件的一部分，用于存放编译期生成的各种字面量与符号引用，**这部分内容将在类加载后存放到方法区的运行时常量池中**。（运行时常量池就是常量池在程序运行时的称呼）
3. 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
4. JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的。
5. 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。**此时不再是常量池中的符号地址了，这里换为真实地址**。

- 运行时常量池，相对于Class文件常量池的另一重要特征是：**具备动态性**。

1. 运行时常量池类似于传统编程语言中的符号表（symbol table），但是它所包含的数据却比符号表要更加丰富一些。
2. 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会抛OutofMemoryError异常。



#### p4: 方法区使用举例

**🐴代码举例**

```java
public class MethodAreaDemo {
    public static void main(String[] args) {
        int x = 500;
        int y = 100;
        int a = x / y;
        int b = 50;
        System.out.println(a + b);
    }
}
```

**字节码:**

```java
public class com.atguigu.java1.MethodAreaDemo
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #25.#26        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #27.#28        // java/io/PrintStream.println:(I)V
   #4 = Class              #29            // com/atguigu/java1/MethodAreaDemo
   #5 = Class              #30            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               LocalVariableTable
  #11 = Utf8               this
  #12 = Utf8               Lcom/atguigu/java1/MethodAreaDemo;
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               args
  #16 = Utf8               [Ljava/lang/String;
  #17 = Utf8               x
  #18 = Utf8               I
  #19 = Utf8               y
  #20 = Utf8               a
  #21 = Utf8               b
  #22 = Utf8               SourceFile
  #23 = Utf8               MethodAreaDemo.java
  #24 = NameAndType        #6:#7          // "<init>":()V
  #25 = Class              #31            // java/lang/System
  #26 = NameAndType        #32:#33        // out:Ljava/io/PrintStream;
  #27 = Class              #34            // java/io/PrintStream
  #28 = NameAndType        #35:#36        // println:(I)V
  #29 = Utf8               com/atguigu/java1/MethodAreaDemo
  #30 = Utf8               java/lang/Object
  #31 = Utf8               java/lang/System
  #32 = Utf8               out
  #33 = Utf8               Ljava/io/PrintStream;
  #34 = Utf8               java/io/PrintStream
  #35 = Utf8               println
  #36 = Utf8               (I)V
{
  public com.atguigu.java1.MethodAreaDemo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/atguigu/java1/MethodAreaDemo;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=5, args_size=1
         0: sipush        500
         3: istore_1
         4: bipush        100
         6: istore_2
         7: iload_1
         8: iload_2
         9: idiv
        10: istore_3
        11: bipush        50
        13: istore        4
        15: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        18: iload_3
        19: iload         4
        21: iadd
        22: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        25: return
      LineNumberTable:
        line 9: 0
        line 10: 4
        line 11: 7
        line 12: 11
        line 13: 15
        line 14: 25
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      26     0  args   [Ljava/lang/String;
            4      22     1     x   I
            7      19     2     y   I
           11      15     3     a   I
           15      11     4     b   I
}
SourceFile: "MethodAreaDemo.java"
```

**图解:**

1.初始状态

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2056%201628506796%201628506796330%208gwpLF%20image-20210407122150970.png" alt="image-20210407122150970" style="zoom:30%;" />

2.首先将操作数500压入操作数栈中

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2057%201628506797%201628506797217%20jLdfQq%20image-20210407122233831.png" alt="image-20210407122233831" style="zoom:30%;" />

3.然后操作数500从操作数栈中取出,存储到局部变量表中索引为1的位置

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2018%2059%2059%201628506799%201628506799016%20w3oSk9%20image-20210407135920599.png" alt="image-20210407135920599" style="zoom:30%;" />

4.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2000%201628506800%201628506800175%20XrxoXv%20image-20210407135950264.png" alt="image-20210407135950264" style="zoom:30%;" />

5.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2001%201628506801%201628506801017%2048lrXV%20image-20210407140012099.png" alt="image-20210407140012099" style="zoom:30%;" />

6.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2001%201628506801%201628506801960%20OqoNmn%20image-20210407140040509.png" alt="image-20210407140040509" style="zoom:30%;" />

7.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2002%201628506802%201628506802884%20qeWU5s%20image-20210407140105963.png" alt="image-20210407140105963" style="zoom:30%;" />

8.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2003%201628506803%201628506803813%20DrthGr%20image-20210407140127869.png" alt="image-20210407140127869" style="zoom:30%;" />

9.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2005%201628506805%201628506805051%20Pxz9E9%20image-20210407140149264.png" alt="image-20210407140149264" style="zoom:30%;" />

10.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2005%201628506805%201628506805904%207AzYn0%20image-20210407140215365.png" alt="image-20210407140215365" style="zoom:30%;" />

11.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2006%201628506806%201628506806991%20t0vWEr%20image-20210407140256632.png" alt="image-20210407140256632" style="zoom:30%;" />

12.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2007%201628506807%201628506807825%20upVYeY%20image-20210407140314569.png" alt="image-20210407140314569" style="zoom:30%;" />

13.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2008%201628506808%201628506808637%20RKXTgX%20image-20210407140354063.png" alt="image-20210407140354063" style="zoom:30%;" />

14.执行加法运算后，将计算结果放在操作数栈顶

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2009%201628506809%201628506809979%200jSsv6%20image-20210407140445783.png" alt="image-20210407140445783" style="zoom:30%;" />

15.打印

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2010%201628506810%201628506810862%20YAfy4W%20image-20210407140519139.png" alt="image-20210407140519139" style="zoom:30%;" />

16.

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2011%201628506811%201628506811767%20NhKh09%20image-20210407140540271.png" alt="image-20210407140540271" style="zoom:30%;" />

**符号引用 --> 直接引用**

1. 上面代码调用 System.out.println() 方法时，首先需要看看 System 类有没有加载，再看看 PrintStream 类有没有加载
2. 如果没有加载，则执行加载，执行时，将常量池中的符号引用（字面量）转换为运行时常量池的直接引用（真正的地址值）



### P5: 方法区演进细节

#### p1: 永久代演进过程

1.首先明确:只有HotSpot才有永久代.BEA JRockit、IBMJ9等来说，是不存在永久代的概念的。原则上如何实现方法区属于虚拟机实现细节，不受《Java虚拟机规范》管束，并不要求统一

2.HotSpot中方法区的变化:

| JDK1.6及以前 | 有永久代（permanent generation），静态变量存储在永久代上     |
| ------------ | ------------------------------------------------------------ |
| JDK1.7       | 有永久代，但已经逐步 “去永久代”，**字符串常量池，静态变量移除，保存在堆中** |
| JDK1.8       | 无永久代，类型信息，字段，方法，常量保存在本地内存的元空间，但字符串常量池、静态变量仍然在堆中。 |

**JDK6**

`方法区有永久代实现,使用JVM虚拟机内存`

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2012%201628506812%201628506812668%20XIU1Qm%20image-20210407155159519.png" alt="image-20210407155159519" style="zoom:33%" />

**JDK7**

`方法区有永久代实现,使用JVM虚拟机内存`

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2013%201628506813%201628506813626%20Sr9I86%20image-20210407155344436.png" alt="image-20210407155344436" style="zoom:30%" />

**JDK8**

`方法区由元空间实现,使用物理机本地内存`

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2014%201628506814%201628506814432%209UcmfT%20image-20210407155551508.png" alt="image-20210407155551508" style="zoom:30%" />



#### p2:  永久代为什么要被元空间替代?

> **官方文档**：http://openjdk.java.net/jeps/122 (官方说要整合HotSpot和JRockit,因为JRockit不用,所以就不用....)

1. 随着Java8的到来，HotSpot VM中再也见不到永久代了。但是这并不意味着类的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（Metaspace）。
2. 由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。
3. 这项改动是很有必要的，原因有：
   1. **为永久代设置空间大小是很难确定的**。在某些场景下，如果动态加载类过多，容易产生Perm区的OOM。比如某个实际Web工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。`Exception in thread 'dubbo client x.x connector' java.lang.OutOfMemoryError:PermGen space`而元空间和永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。 因此，默认情况下，元空间的大小仅受本地内存限制。
   2. **对永久代进行调优是很困难的**。方法区的垃圾收集主要回收两部分内容：常量池中废弃的常量和不再用的类型，方法区的调优主要是为了降低Full GC
      1. 有些人认为方法区（如HotSpot虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如JDK11时期的ZGC收集器就不支持类卸载）。
      2. 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻**。但是这部分区域的回收有时又确实是必要的。以前Sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏。

#### p3: 字符串常量池

**字符串常量池 StringTable 为什么要调整位置？**

- JDK7中将StringTable放到了堆空间中。因为永久代的回收效率很低，在Full GC的时候才会执行永久代的垃圾回收，而Full GC是老年代的空间不足、永久代不足时才会触发。
- 这就导致StringTable回收效率不高，而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

#### p4: 静态变量放在哪里?

##### D1: 对象实体放在哪里?

```java
/**
 * 结论：
 * 1、静态引用对应的对象实体(也就是这个new byte[1024 * 1024 * 100])始终都存在堆空间，
 * 2、只是那个变量(相当于下面的arr变量名)在JDK6,JDK7,JDK8存放位置中有所变化
 *
 * jdk7：
 * -Xms200m -Xmx200m -XX:PermSize=300m -XX:MaxPermSize=300m -XX:+PrintGCDetails
 * jdk 8：
 * -Xms200m -Xmx200m -XX:MetaspaceSize=300m -XX:MaxMetaspaceSize=300m -XX:+PrintGCDetails
 */
public class StaticFieldTest {
    private static byte[] arr = new byte[1024 * 1024 * 100];//100MB

    public static void main(String[] args) {
        System.out.println(StaticFieldTest.arr);
    }
}
```

jdk6环境下

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2015%201628506815%201628506815193%20bUrA8s%20image-20210407160907622.png" alt="image-20210407160907622" style="zoom:30%" />

jdk7环境下

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2016%201628506816%201628506816246%20CKace4%20image-20210407160941607.png" alt="image-20210407160941607" style="zoom:33%" />

jdk8环境下

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2017%201628506817%201628506817331%20q9cSLw%20image-20210407161021949.png" alt="image-20210407161021949" style="zoom:30%" />



##### D2: 变量(名)存放在哪里?

`这个问题需要用JHSDB工具来进行分析，这个工具是JDK9开始自带的(JDK9以前没有)，在bin目录下可以找到`

```java
package com.atguigu.java1;

/**
 * 《深入理解Java虚拟机》中的案例：
 * staticObj、instanceObj、localObj存放在哪里？
 */
public class StaticObjTest {
    static class Test {
        static ObjectHolder staticObj = new ObjectHolder();
        ObjectHolder instanceObj = new ObjectHolder();

        void foo() {
            ObjectHolder localObj = new ObjectHolder();
            System.out.println("done");
        }
    }

    private static class ObjectHolder {
    }

    public static void main(String[] args) {
        Test test = new StaticObjTest.Test();
        test.foo();
    }
}
```

**JDK6环境下**

1、staticObj随着Test的类型信息存放在方法区

2、instanceObj随着Test的对象实例存放在Java堆

3、localObject则是存放在foo()方法栈帧的局部变量表中。

4、测试发现：三个对象的数据在内存中的地址都落在Eden区范围内，所以结论：**只要是对象实例必然会在Java堆中分配**。

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2018%201628506818%201628506818550%20mdMbDM%20image-20210407161546883.png" alt="image-20210407161546883" style="zoom:33%" />

> 1、0x00007f32c7800000(Eden区的起始地址) ---- 0x00007f32c7b50000(Eden区的终止地址)
>
> 2、可以发现三个变量都在这个范围内
>
> 3、所以可以得到上面结论

5、接着，找到了一个引用该staticObj对象的地方，是在一个java.lang.Class的实例里，并且给出了这个实例的地址，通过Inspector查看该对象实例，可以清楚看到这确实是一个java.lang.Class类型的对象实例，里面有一个名为staticobj的实例字段：

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2019%201628506819%201628506819140%20D9XNjD%20image-20210407162013900.png" alt="image-20210407162013900" style="zoom:33%" />

从《Java虚拟机规范》所定义的概念模型来看，所有Class相关的信息都应该存放在方法区之中，但方法区该如何实现，《Java虚拟机规范》并未做出规定，这就成了一件允许不同虚拟机自己灵活把握的事情。JDK7及其以后版本的HotSpot虚拟机选择把静态变量与类型在Java语言一端的映射Class对象存放在一起，**存储于Java堆之中**，从我们的实验中也明确验证了这一点

### P6: 方法区的垃圾回收

1. 有些人认为方法区（如Hotspot虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区**类型卸载**的收集器存在（如JDK11时期的ZGC收集器就不支持类卸载）。
2. 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏。
3. 方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**。

1. 先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。字面量比较接近Java语言层次的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：
   - 类和接口的全限定名
   - 字段的名称和描述符
   - 方法的名称和描述符
2. HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。
3. 回收废弃常量与回收Java堆中的对象非常类似。（关于常量的回收比较简单，重点是类的回收）

下面也称作**类卸载**

1、判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
- 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

2、Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了`-Xnoclassgc`参数进行控制，还可以使用`-verbose:class` 以及 `-XX：+TraceClass-Loading`、`-XX：+TraceClassUnLoading`查看类加载和卸载信息

3、在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。



## 第八章: 对象实例化内存布局与访问定位

### P1: 对象的实例化

**相关面试题**

美团：

1. 对象在`JVM`中是怎么存储的？
2. 对象头信息里面有哪些东西？

蚂蚁金服：

二面：`java`对象头里有什么

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2020%201628506820%201628506820662%20bs3dgd%20image-20210408201400803.png" alt="image-20210408201400803" style="zoom:33%" />



#### p1: 对象创建的方式

1. new：最常见的方式、单例类中调用getInstance的静态类方法，XXXFactory的静态方法
2. Class的newInstance方法：在JDK9里面被标记为过时的方法，因为只能调用空参构造器，并且权限必须为 public
3. Constructor的newInstance(Xxxx)：反射的方式，可以调用空参的，或者带参的构造器
4. 使用clone()：不调用任何的构造器，要求当前的类需要实现Cloneable接口中的clone方法
5. 使用序列化：从文件中，从网络中获取一个对象的二进制流，序列化一般用于Socket的网络传输
6. 第三方库 Objenesis



#### p2: 对象创建的步骤

从字节码看待对象的创建过程

```java
public class ObjectTest {
    public static void main(String[] args) {
        Object obj = new Object();
    }
}
```

```java
 public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class java/lang/Object
         3: dup           
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: astore_1
         8: return
      LineNumberTable:
        line 9: 0
        line 10: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            8       1     1   obj   Ljava/lang/Object;
}
```

1. **判断对象对应的类是否加载,链接,初始化**
   - 虚拟机遇到一条new指令，首先去检查这个指令的参数能否在Metaspace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载，解析和初始化。（即判断类元信息是否存在）。
   - 如果该类没有加载，那么在双亲委派模式下，使用当前类加载器以ClassLoader + 包名 + 类名为key进行查找对应的.class文件，如果没有找到文件，则抛出ClassNotFoundException异常，如果找到，则进行类加载，并生成对应的Class对象。
2. **为对象分配内存**
   - 首先计算对象占用空间的大小，接着在堆中划分一块内存给新对象。如果实例成员变量是引用变量，仅分配引用变量空间即可，即4个字节大小
   - 如果<u>**内存规整**</u>：采用指针碰撞分配内存
     - 如果内存是规整的，那么虚拟机将采用的是指**针碰撞法（Bump The Point）**来为对象分配内存。
     - 意思是所有用过的内存在一边，空闲的内存放另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针往空闲内存那边挪动一段与对象大小相等的距离罢了。
     - 如果垃圾收集器选择的是Serial ，ParNew这种基于压缩算法的，虚拟机采用这种分配方式。一般使用带Compact（整理）过程的收集器时，使用指针碰撞。
     - 标记压缩（整理）算法会整理内存碎片，堆内存一存对象，另一边为空闲区域
   - 如果<u>**内存不规整**</u>
     - 如果内存不是规整的，已使用的内存和未使用的内存相互交错，那么虚拟机将采用的是空闲列表来为对象分配内存。
     - 意思是虚拟机维护了一个列表，记录上哪些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。这种分配方式成为了 “**空闲列表（Free List）**”
     - 选择哪种分配方式由Java堆是否规整所决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定
     - 标记清除算法清理过后的堆内存，就会存在很多内存碎片。
3. **处理并发问题**
   - 采用CAS+失败重试保证更新的原子性
   - 每个线程预先分配TLAB - 通过设置 -XX:+UseTLAB参数来设置（区域加锁机制）
   - 在Eden区给每个线程分配一块区域
4. **初始化分配到的空间**
   - 所有属性设置默认值，保证对象实例字段在不赋值可以直接使用
   - 给对象属性赋值的顺序：
     - 属性的默认值初始化
     - 显示初始化/代码块初始化（并列关系，谁先谁后看代码编写的顺序）
     - 构造器初始化
5. **设置对象的对象头**
   - 将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。
6. **执行<init>方法进行初始化**
   - 在Java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量
   - 因此一般来说（由字节码中跟随invokespecial指令所决定），new指令之后会接着就是执行init方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完成创建出来。

> **从字节码角度看 init 方法**

```java
/**
 * 测试对象实例化的过程
 *  ① 加载类元信息 - ② 为对象分配内存 - ③ 处理并发问题  - ④ 属性的默认初始化（零值初始化）
 *  - ⑤ 设置对象头的信息 - ⑥ 属性的显式初始化、代码块中初始化、构造器中初始化
 *
 *
 *  给对象的属性赋值的操作：
 *  ① 属性的默认初始化 - ② 显式初始化 / ③ 代码块中初始化 - ④ 构造器中初始化
 */

public class Customer{
    int id = 1001;
    String name;
    Account acct;

    {
        name = "匿名客户";
    }
    public Customer(){
        acct = new Account();
    }

}
class Account{

}
```

**Customer类的字节码**

```java
 0 aload_0
 1 invokespecial #1 <java/lang/Object.<init>>
 4 aload_0
 5 sipush 1001
 8 putfield #2 <com/atguigu/java/Customer.id>
11 aload_0
12 ldc #3 <匿名客户>
14 putfield #4 <com/atguigu/java/Customer.name>
17 aload_0
18 new #5 <com/atguigu/java/Account>
21 dup
22 invokespecial #6 <com/atguigu/java/Account.<init>>
25 putfield #7 <com/atguigu/java/Customer.acct>
28 returnCopy to clipboardErrorCopied
```

- init() 方法的字节码指令：
  - 属性的默认值初始化：`id = 1001;`
  - 显示初始化/代码块初始化：`name = "匿名客户";`
  - 构造器初始化：`acct = new Account();`



### P2: 对象的内存布局

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2021%201628506821%201628506821730%203qpmeR%20image-20210409095334955.png" alt="image-20210409095334955" style="zoom:50%" />

> 内存布局总结

```java
public class Customer{
    int id = 1001;
    String name;
    Account acct;

    {
        name = "匿名客户";
    }
    public Customer(){
        acct = new Account();
    }
    public static void main(String[] args) {
        Customer cust = new Customer();
    }
}
class Account{

}
```

**图解内存布局**

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2022%201628506822%201628506822733%20Iat6GK%20image-20210409134508710.png" alt="image-20210409134508710" style="zoom:50%;" />



### P3: 对象的访问定位

`JVM是如何通过栈帧中的对象引用访问到其内部的对象实例呢？`

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2023%201628506823%201628506823746%20u85ImD%20image-20210409135008303.png" alt="image-20210409135008303" style="zoom:33%" />

定位，通过栈上reference访问

**对象的两种访问方式：句柄访问和直接指针**

**1、句柄访问**

1. 缺点：在堆空间中开辟了一块空间作为句柄池，句柄池本身也会占用空间；通过两次指针访问才能访问到堆中的对象，效率低
2. 优点：reference中存储稳定句柄地址，对象被移动（垃圾收集时移动对象很普遍）时只会改变句柄中实例数据指针即可，reference本身不需要被修改

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2024%201628506824%201628506824907%20vjocFF%20image-20210409135059087.png" alt="image-20210409135059087" style="zoom:33%" />

**2、直接指针（HotSpot采用）**

1. 优点：直接指针是局部变量表中的引用，直接指向堆中的实例，在对象实例中有类型指针，指向的是方法区中的对象类型数据
2. 缺点：对象被移动（垃圾收集时移动对象很普遍）时需要修改 reference 的值

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2025%201628506825%201628506825850%20IXXpmA%20image-20210409135127247.png" alt="image-20210409135127247" style="zoom:33%" />



## 第九章: 直接内存

### P1: 概述

- 不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存区域。
- 直接内存是在Java堆外的、直接向系统申请的内存区间。
- 来源于NIO，通过存在堆中的DirectByteBuffer操作Native内存
- 通常，访问直接内存的速度会优于Java堆。即读写性能高。
- 因此出于性能考虑，读写频繁的场合可能会考虑使用直接内存。
- Java的NIO库允许Java程序使用直接内存，用于数据缓冲区

```java
/**
 *  IO                  NIO (New IO / Non-Blocking IO)
 *  byte[] / char[]     Buffer
 *  Stream              Channel
 *
 * 查看直接内存的占用与释放
 */
public class BufferTest {
    private static final int BUFFER = 1024 * 1024 * 1024;//1GB

    public static void main(String[] args){
        //直接分配本地内存空间
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(BUFFER);
        System.out.println("直接内存分配完毕，请求指示！");

        Scanner scanner = new Scanner(System.in);
        scanner.next();

        System.out.println("直接内存开始释放！");
        byteBuffer = null;
        System.gc();
        scanner.next();
    }
}
```

直接占用了 1G 的本地内存

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2026%201628506826%201628506826808%20vB0kTF%20image-20210413104417822.png" alt="image-20210413104417822" style="zoom:33%" />

### P2: BIO 和 NIO

**非直接缓存区 (BIO)**

原来采用BIO的架构，在读写本地文件时，我们需要从用户态切换成内核态

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2028%201628506828%201628506828009%205qMb8L%20image-20210413104606263.png" alt="image-20210413104606263" style="zoom:33%" />



**直接缓冲区 (NIO)**

NIO直接操作物理磁盘,省去了中间过程

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2029%201628506829%201628506829072%20v7s7TD%20image-20210413104925285.png" alt="image-20210413104925285" style="zoom:33%" />



### P3: 直接内存与OOM

1. 直接内存也可能导致OutofMemoryError异常
2. 由于直接内存在Java堆外，因此它的大小不会直接受限于-Xmx指定的最大堆大小，但是系统内存是有限的，Java堆和直接内存的总和依然受限于操作系统能给出的最大内存。
3. 直接内存的缺点为：
   - 分配回收成本较高
   - 不受JVM内存回收管理
4. 直接内存大小可以通过MaxDirectMemorySize设置
5. 如果不指定，默认与堆的最大值-Xmx参数值一致

```java
/**
 * 本地内存的OOM:  OutOfMemoryError: Direct buffer memory
 *
 */
public class BufferTest2 {
    private static final int BUFFER = 1024 * 1024 * 20;//20MB

    public static void main(String[] args) {
        ArrayList<ByteBuffer> list = new ArrayList<>();

        int count = 0;
        try {
            while(true){
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect(BUFFER);
                list.add(byteBuffer);
                count++;
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        } finally {
            System.out.println(count);
        }


    }
}


Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
    at java.nio.Bits.reserveMemory(Bits.java:694)
    at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
    at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:311)
    at com.atguigu.java.BufferTest2.main(BufferTest2.java:21)

```

<img src="https://gitee.com/breeze1002/upic/raw/master/JVM/JVM-2/2021%2008%2009%2019%2000%2030%201628506830%201628506830011%20aUE681%20image-20210413105321032.png" alt="image-20210413105321032" style="zoom:33%" />





## 常见面试题

1. 百度
   - 三面：说一下JVM内存模型吧，有哪些区？分别干什么的？
2. 蚂蚁金服：
   - Java8的内存分代改进
   - JVM内存分哪几个区，每个区的作用是什么？
   - 一面：JVM内存分布/内存结构？栈和堆的区别？堆的结构？为什么两个survivor区？
   - 二面：Eden和survior的比例分配
3. 小米：
   - jvm内存分区，为什么要有新生代和老年代
4. 字节跳动：
   - 二面：Java的内存分区
   - 二面：讲讲vm运行时数据库区
   - 什么时候对象会进入老年代？
5. 京东：
   - JVM的内存结构，Eden和Survivor比例。
   - JVM内存为什么要分成新生代，老年代，持久代。新生代中为什么要分为Eden和survivor。
6. 天猫：
   - 一面：Jvm内存模型以及分区，需要详细到每个区放什么。
   - 一面：JVM的内存模型，Java8做了什么改
7. 拼多多：
   - JVM内存分哪几个区，每个区的作用是什么？
8. 美团：
   - java内存分配
   - jvm的永久代中会发生垃圾回收吗？
   - 一面：jvm内存分区，为什么要有新生代和老年代？





