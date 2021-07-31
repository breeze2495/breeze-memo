

## 第十章: 执行引擎

### P1: 概述

- **概述**

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413105707853.png" alt="image-20210413105707853" style="zoom:30%;float:left" />

  - 执行引擎是Java虚拟机核心的组成部分之一。
  - “虚拟机”是一个相对于“物理机”的概念，这两种机器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而**虚拟机的执行引擎则是由软件自行实现的**，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，**能够执行那些不被硬件直接支持的指令集格式**。
  - JVM的主要任务是负责**装载字节码到其内部**，但字节码并不能够直接运行在操作系统之上，因为字节码指令并非等价于本地机器指令，它内部包含的仅仅只是一些能够被JVM所识别的字节码指令、符号表，以及其他辅助信息。
  - 那么，如果想要让一个Java程序运行起来，执行引擎（Execution Engine）的任务就是**将字节码指令解释/编译为对应平台上的本地机器指令才可以**。简单来说，JVM中的执行引擎充当了将高级语言翻译为机器语言的译者。

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413110122850.png" alt="image-20210413110122850" style="zoom:33%;float:left" />

  1、前端编译：从Java程序员-字节码文件的这个过程叫前端编译

  2、执行引擎这里有两种行为：一种是解释执行，一种是编译执行（这里的是后端编译）。

- **执行引擎工作过程**

  1. 执行引擎在执行的过程中究竟需要执行什么样的字节码指令完全依赖于PC寄存器。
  2. 每当执行完一项指令操作后，PC寄存器就会更新下一条需要被执行的指令地址。
  3. 当然方法在执行的过程中，执行引擎有可能会通过存储在局部变量表中的对象引用准确定位到存储在Java堆区中的对象实例信息，以及通过对象头中的元数据指针定位到目标对象的类型信息。
  4. 从外观上来看，所有的Java虚拟机的执行引擎输入、处理、输出都是一致的：输入的是字节码二进制流，处理过程是字节码解析执行、即时编译的等效过程，输出的是执行过程。

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413110707069.png" alt="image-20210413110707069" style="zoom:33%;float:left" />



### P2: JAVA代码编译和执行过程

- **解释执行和即使编译**

  大部分的程序代码转换成物理机的目标代码或虚拟机能执行的指令集之前，都需要经过下图中的各个步骤：

  1. 前面橙色部分是编译生成生成字节码文件的过程（javac编译器来完成，也就是前端编译器），和JVM没有关系。
  2. 后面绿色（解释执行）和蓝色（即时编译）才是JVM需要考虑的过程

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413111047704.png" alt="image-20210413111047704" style="zoom:33%;float:left" />

  3. java编译器(前端编译器) 流程图:

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413111206352.png" alt="image-20210413111206352" style="zoom:33%;float:left" />

  4. Java字节码的执行时由JVM执行引擎来完成:

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413111335975.png" alt="image-20210413111335975" style="zoom:33%;float:left" />



- **解释器和编译器**

  - 解释器：当Java虚拟机启动时会根据预定义的规范对字节码采用**逐行**解释的方式**执行**，将每条字节码文件中的内容“翻译”为对应平台的本地机器指令执行。
  - JIT（Just In Time Compiler）编译器：就是虚拟机将源代码**一次性直接**编译成和本地机器平台相关的机器语言，**但并不是马上执行**。

  **为什么Java是半编译半解释型语言？**

  1. JDK1.0时代，将Java语言定位为“解释执行”还是比较准确的。再后来，Java也发展出可以直接生成本地代码的编译器。
  2. 现在JVM在执行Java代码的时候，通常都会将**解释执行与编译执行二者结合起来进行**。
  3. JIT编译器将字节码翻译成本地代码后，就可以做一个缓存操作，存储在方法区的JIT 代码缓存中（执行效率更高了），并且在翻译成本地代码的过程中可以做优化。

- **总结**

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413111602933.png" alt="image-20210413111602933" style="zoom:33%;float:left" />



### P3: 机器码/指令/汇编语言

- **机器码**
  - 各种用二进制编码方式表示的指令，叫做机器指令码。开始，人们就用它采编写程序，这就是机器语言。
  - 机器语言虽然能够被计算机理解和接受，但和人们的语言差别太大，不易被人们理解和记忆，并且用它编程容易出差错。
  - 用它编写的程序一经输入计算机，CPU直接读取运行，因此和其他语言编的程序相比，执行速度最快。
  - 机器指令与CPU紧密相关，所以不同种类的CPU所对应的机器指令也就不同。

- **指令和指令集**

  - **指令**

  1. 由于机器码是由0和1组成的二进制序列，可读性实在太差，于是人们发明了指令。

  2. 指令就是把机器码中特定的0和1序列，简化成对应的指令（一般为英文简写，如mov，inc等），可读性稍好

  3. 由于不同的硬件平台，执行同一个操作，对应的机器码可能不同，所以不同的硬件平台的同一种指令（比如mov），对应的机器码也可能不同。

     

  - **指令集**

  不同的硬件平台，各自支持的指令，是有差别的。因此每个平台所支持的指令，称之为对应平台的指令集。如 :

  1. x86指令集，对应的是x86架构的平台
  2. ARM指令集，对应的是ARM架构的平台

- **汇编语言**
  1. 由于指令的可读性还是太差，于是人们又发明了汇编语言。
  2. 在汇编语言中，用助记符（Mnemonics）代替机器指令的操作码，用地址符号（Symbol）或标号（Label）代替指令或操作数的地址。
  3. 在不同的硬件平台，汇编语言对应着不同的机器语言指令集，通过汇编过程转换成机器指令。
  4. 由于计算机只认识指令码，所以用汇编语言编写的程序还必须翻译（汇编）成机器指令码，计算机才能识别和执行。

- **高级语言**

  1. 为了使计算机用户编程序更容易些，后来就出现了各种高级计算机语言。高级语言比机器语言、汇编语言更接近人的语言
  2. 当计算机执行高级语言编写的程序时，仍然需要把程序解释和编译成机器的指令码。完成这个过程的程序就叫做解释程序或编译程序。

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413112656525.png" alt="image-20210413112656525" style="zoom:33%;float:left" />



- **字节码**
  - 字节码是一种中间状态（中间码）的二进制代码（文件），它比机器码更抽象，需要直译器转译后才能成为机器码
  - 字节码主要为了实现特定软件运行和软件环境、与硬件环境无关。
  - 字节码的实现方式是通过编译器和虚拟机器。编译器将源码编译成字节码，特定平台上的虚拟机器将字节码转译为可以直接执行的指令。
  - 字节码典型的应用为：Java bytecode



- **C/C++源程序执行过程**

  **编译过程又可以分成两个阶段：编译和汇编。**

  1. 编译过程：是读取源程序（字符流），对之进行词法和语法的分析，将高级语言指令转换为功能等效的汇编代码
  2. 汇编过程：实际上指把汇编语言代码翻译成目标机器指令的过程。

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413142436940.png" alt="image-20210413142436940" style="zoom:33%;float:left" />



### P4: 解释器

#### p1: 为什么要有解释器

1. JVM设计者们的初衷仅仅只是单纯地为了满足Java程序实现跨平台特性，因此避免采用静态编译的方式由高级语言直接生成本地机器指令，从而诞生了实现解释器在运行时采用逐行解释字节码执行程序的想法（也就是产生了一个中间产品**字节码**）。
2. 解释器真正意义上所承担的角色就是一个运行时“翻译者”，将字节码文件中的内容“翻译”为对应平台的本地机器指令执行。
3. 当一条字节码指令被解释执行完成后，接着再根据PC寄存器中记录的下一条需要被执行的字节码指令执行解释操作。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413142755974.png" alt="image-20210413142755974" style="zoom:33%;float:left" />

#### p2: 解释器的分类

1. 在Java的发展历史里，一共有两套解释执行器，即古老的**字节码解释器**,现在普遍使用的**模板解释器**。
   - 字节码解释器在执行时通过纯软件代码模拟字节码的执行，效率非常低下。
   - 而模板解释器将每一条字节码和一个模板函数相关联，模板函数中直接产生这条字节码执行时的机器码，从而很大程度上提高了解释器的性能。
2. 在HotSpot VM中，解释器主要由Interpreter模块和Code模块构成。
   - Interpreter模块：实现了解释器的核心功能
   - Code模块：用于管理HotSpot VM在运行时生成的本地机器指令

#### p3: 解释器的现状

1. 由于解释器在设计和实现上非常简单，因此除了Java语言之外，还有许多高级语言同样也是基于解释器执行的，比如Python、Perl、Ruby等。但是在今天，基于解释器执行已经沦落为低效的代名词，并且时常被一些C/C++程序员所调侃。
2. 为了解决这个问题，JVM平台支持一种叫作即时编译的技术。**即时编译的目的是避免函数被解释执行，而是将整个函数体编译成为机器码，每次函数执行时，只执行编译后的机器码即可，这种方式可以使执行效率大幅度提升。**
3. 不过无论如何，基于解释器的执行模式仍然为中间语言的发展做出了不可磨灭的贡献。



### P5: JIT编译器

#### p1: Java代码执行的分类

1. 第一种是将源代码编译成字节码文件，然后在运行时通过解释器将字节码文件转为机器码执行
2. 第二种是编译执行（直接编译成机器码）。现代虚拟机为了提高执行效率，会使用即时编译技术（JIT，Just In Time）将方法编译成机器码后再执行

1. HotSpot VM是目前市面上高性能虚拟机的代表作之一。**它采用解释器与即时编译器并存的架构**。在Java虚拟机运行时，解释器和即时编译器能够相互协作，各自取长补短，尽力去选择最合适的方式来权衡编译本地代码的时间和直接解释执行代码的时间。
2. 在今天，Java程序的运行性能早已脱胎换骨，已经达到了可以和C/C++ 程序一较高下的地步。



#### p2: 为什么还需要解释器?

1. 有些开发人员会感觉到诧异，既然HotSpot VM中已经内置JIT编译器了，那么为什么还需要再使用解释器来“拖累”程序的执行性能呢？比如JRockit VM内部就不包含解释器，字节码全部都依靠即时编译器编译后执行。
2. JRockit虚拟机是砍掉了解释器，也就是只采用及时编译器。那是因为JRockit只部署在服务器上，一般已经有时间让他进行指令编译的过程了，对于响应来说要求不高，等及时编译器的编译完成后，就会提供更好的性能

**首先明确两点：**

1. 当程序启动后，解释器可以马上发挥作用，**响应速度快**，省去编译的时间，立即执行。
2. 编译器要想发挥作用，把代码编译成本地代码，**需要一定的执行时间**，但编译为本地代码后，执行效率高。

**所以：**

1. 尽管JRockit VM中程序的执行性能会非常高效，但程序在启动时必然需要花费更长的时间来进行编译。对于服务端应用来说，启动时间并非是关注重点，但对于那些看中启动时间的应用场景而言，或许就需要采用解释器与即时编译器并存的架构来换取一个平衡点。
2. 在此模式下，在Java虚拟器启动时，解释器可以首先发挥作用，而不必等待即时编译器全部编译完成后再执行，这样可以省去许多不必要的编译时间。随着时间的推移，编译器发挥作用，把越来越多的代码编译成本地代码，获得更高的执行效率。
3. 同时，解释执行在编译器进行激进优化不成立的时候，作为编译器的“逃生门”（后备方案）。



#### p3: 案例

- 当虚拟机启动的时候，解释器可以首先发挥作用，而不必等待即时编译器全部编译完成再执行，这样可以省去许多不必要的编译时间。随着程序运行时间的推移，即时编译器逐渐发挥作用，根据热点探测功能，将有价值的字节码编译为本地机器指令，以换取更高的程序执行效率。

1. 注意解释执行与编译执行在线上环境微妙的辩证关系。**机器在热机状态（已经运行了一段时间叫热机状态）可以承受的负载要大于冷机状态（刚启动的时候叫冷机状态）**。如果以热机状态时的流量进行切流，可能使处于冷机状态的服务器因无法承载流量而假死。
2. 在生产环境发布过程中，以分批的方式进行发布，根据机器数量划分成多个批次，每个批次的机器数至多占到整个集群的1/8。曾经有这样的故障案例：某程序员在发布平台进行分批发布，在输入发布总批数时，误填写成分为两批发布。如果是热机状态，在正常情况下一半的机器可以勉强承载流量，但由于刚启动的JVM均是解释执行，还没有进行热点代码统计和JIT动态编译，导致机器启动之后，当前1/2发布成功的服务器马上全部宕机，此故障说明了JIT的存在。—**阿里团队**

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413144316347.png" alt="image-20210413144316347" style="zoom:33%;float:left" />

```java
public class JITTest {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();

        for (int i = 0; i < 1000; i++) {
            list.add("让天下没有难学的技术");

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
```

通过 JVisualVM 查看 JIT 编译器执行的编译次数

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413144441319.png" alt="image-20210413144441319" style="zoom:33%;float:left" />



#### p4: JIT编译器相关概念

- Java 语言的“编译期”其实是一段“不确定”的操作过程，
  1. 因为它可能是指一个前端编译器（其实叫“编译器的前端”更准确一些）把.java文件转变成.class文件的过程。
  2. 也可能是指虚拟机的后端运行期编译器（JIT编译器，Just In Time Compiler）把字节码转变成机器码的过程。
  3. 还可能是指使用静态提前编译器（AOT编译器，Ahead of Time Compiler）直接把.java文件编译成本地机器代码的过程。（可能是后续发展的趋势）                   

**典型的编译器：**

1. 前端编译器：Sun的javac、Eclipse JDT中的增量式编译器（ECJ）。
2. JIT编译器：HotSpot VM的C1、C2编译器。
3. AOT 编译器：GNU Compiler for the Java（GCJ）、Excelsior JET。



#### p5: 热点代码及探测方式

1. 当然是否需要启动JIT编译器将字节码直接编译为对应平台的本地机器指令，则需要根据代码被调用**执行的频率**而定。
2. 关于那些需要被编译为本地代码的字节码，也被称之为**“热点代码”**，JIT编译器在运行时会针对那些频繁被调用的“热点代码”做出**深度优化**，将其直接编译为对应平台的本地机器指令，以此提升Java程序的执行性能。
3. 一个被多次调用的方法，或者是一个方法体内部循环次数较多的循环体都可以被称之为“热点代码”，因此都可以通过JIT编译器编译为本地机器指令。由于这种编译方式发生在方法的执行过程中，因此也被称之为栈上替换，或简称为OSR (On StackReplacement)编译。
4. 一个方法究竟要被调用多少次，或者一个循环体究竟需要执行多少次循环才可以达到这个标准？必然需要一个明确的阈值，JIT编译器才会将这些“热点代码”编译为本地机器指令执行。这里主要依靠热点探测功能。
5. **目前HotSpot VM所采用的热点探测方式是基于计数器的热点探测**。
6. 采用基于计数器的热点探测，HotSpot VM将会为每一个方法都建立2个不同类型的计数器，分别为方法调用计数器（Invocation Counter）和回边计数器（Back Edge Counter）。
   1. 方法调用计数器用于统计方法的调用次数
   2. 回边计数器则用于统计循环体执行的循环次数

##### D1: 方法调用计数器

1. 这个计数器就用于统计方法被调用的次数，它的默认阀值在Client模式下是1500次，在Server模式下是10000次。超过这个阈值，就会触发JIT编译。
2. 这个阀值可以通过虚拟机参数 -XX:CompileThreshold 来人为设定。
3. 当一个方法被调用时，会先检查该方法是否存在被JIT编译过的版本
   - 如果存在，则优先使用编译后的本地代码来执行
   - 如果不存在已被编译过的版本，则将此方法的调用计数器值加1，然后判断方法调用计数器与回边计数器值之和是否超过方法调用计数器的阀值。
     - 如果已超过阈值，那么将会向即时编译器提交一个该方法的代码编译请求。
     - 如果未超过阈值，则使用解释器对字节码文件解释执行

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413145546237.png" alt="image-20210413145546237" style="zoom:33%;float:left" />



##### D2: 热度衰减

1. 如果不做任何设置，方法调用计数器统计的并不是方法被调用的绝对次数，而是一个相对的执行频率，即**一段时间之内方法被调用的次数**。当超过一定的时间限度，如果方法的调用次数仍然不足以让它提交给即时编译器编译，那这个方法的调用计数器就会被减少一半，这个过程称为方法调用计数器热度的衰减（Counter Decay），而这段时间就称为此方法统计的**半衰周期**（Counter Half Life Time）（半衰周期是化学中的概念，比如出土的文物通过查看C60来获得文物的年龄）
2. 进行热度衰减的动作是在虚拟机进行垃圾收集时顺便进行的，可以使用虚拟机参数 -XX:-UseCounterDecay 来关闭热度衰减，让方法计数器统计方法调用的绝对次数，这样的话，只要系统运行时间足够长，绝大部分方法都会被编译成本地代码。
3. 另外，可以使用-XX:CounterHalfLifeTime参数设置半衰周期的时间，单位是秒。



##### D3: 回边计数器

`回边: 循环边界往回跳转`

它的作用是**统计一个方法中循环体代码执行的次数**，在字节码中遇到控制流向后跳转的指令称为“回边”（Back Edge）。显然，建立回边计数器统计的目的就是为了触发OSR编译。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413145931459.png" alt="image-20210413145931459" style="zoom:33%;float:left" />



#### p6: HopSpotVM可以设置程序执行方法

缺省情况下HotSpot VM是采用解释器与即时编译器并存的架构，当然开发人员可以根据具体的应用场景，通过命令显式地为Java虚拟机指定在运行时到底是完全采用解释器执行，还是完全采用即时编译器执行。如下所示：

1. -Xint：完全采用解释器模式执行程序；
2. -Xcomp：完全采用即时编译器模式执行程序。如果即时编译出现问题，解释器会介入执行
3. -Xmixed：采用解释器+即时编译器的混合模式共同执行程序。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413150303303.png" alt="image-20210413150303303" style="zoom:33%;float:left" />

```java
/**
 * 测试解释器模式和JIT编译模式
 *  -Xint  : 6520ms
 *  -Xcomp : 950ms
 *  -Xmixed : 936ms
 */
public class IntCompTest {
    public static void main(String[] args) {

        long start = System.currentTimeMillis();

        testPrimeNumber(1000000);

        long end = System.currentTimeMillis();

        System.out.println("花费的时间为：" + (end - start));

    }

    public static void testPrimeNumber(int count){
        for (int i = 0; i < count; i++) {
            //计算100以内的质数
            label:for(int j = 2;j <= 100;j++){
                for(int k = 2;k <= Math.sqrt(j);k++){
                    if(j % k == 0){
                        continue label;
                    }
                }
                //System.out.println(j);
            }

        }
    }
}
```



#### p7: HotSpotVM JIT分类

在HotSpot VM中内嵌有两个JIT编译器，分别为Client Compiler和Server Compiler，但大多数情况下我们简称为C1编译器 和 C2编译器。开发人员可以通过如下命令显式指定Java虚拟机在运行时到底使用哪一种即时编译器，如下所示：

1. -client：指定Java虚拟机运行在Client模式下，并使用C1编译器；
   - C1编译器会对字节码进行简单和可靠的优化，耗时短，以达到更快的编译速度。
2. -server：指定Java虚拟机运行在server模式下，并使用C2编译器。
   - C2进行耗时较长的优化，以及激进优化，但优化的代码执行效率更高。（使用C++）



#### p8: C1和C2编译器不同的优化策略

1. 在不同的编译器上有不同的优化策略，C1编译器上主要有方法内联，去虚拟化、元余消除。
   - 方法内联：多个方法调用,执行时要经历多次参数传递,返回值传递以及跳转.方法内联将调用的方法直接植入到当前方法中，这样可以减少栈帧的生成，减少参数传递以及跳转过程; 
     - 1.去除方法调用的成本
     - 2.为其它优化建立良好的基础
   - 去虚拟化：在装载class文件后,进行类层次的分析,如果发现类中的方法只提供了一个实现类,那么就可对调用此方法的代码进行方法内联;
   - 冗余消除：在运行期间把一些不会执行的代码折叠掉
2. C2的优化主要是在全局层面，逃逸分析是优化的基础。基于逃逸分析在C2上有如下几种优化：
   - 标量替换：用标量值代替聚合对象的属性值
   - 栈上分配：对于未逃逸的对象分配对象在栈而不是堆
   - 同步消除：清除同步操作，通常指synchronized

> 也就是说之前的逃逸分析，只有在C2（server模式下）才会触发。那是否说明C1就用不了了？



#### p9: 分层编译策略

1. 分层编译（Tiered Compilation）策略：程序解释执行（不开启性能监控）可以触发C1编译，将字节码编译成机器码，可以进行简单优化，也可以加上性能监控，C2编译会根据性能监控信息进行激进优化。
2. 不过在Java7版本之后，一旦开发人员在程序中显式指定命令“-server"时，默认将会开启分层编译策略，由C1编译器和C2编译器相互协作共同来执行编译任务。

1. 一般来讲，JIT编译出来的机器码性能比解释器解释执行的性能高
2. C2编译器启动时长比C1慢，系统稳定执行以后，C2编译器执行速度远快于C1编译器

##### D1: Graal 编译器

- 自JDK10起，HotSpot又加入了一个全新的即时编译器：Graal编译器

- 编译效果短短几年时间就追平了G2编译器，未来可期（对应还出现了Graal虚拟机，是有可能替代Hotspot的虚拟机的）

- 目前，带着实验状态标签，需要使用开关参数去激活才能使用

  -XX:+UnlockExperimentalvMOptions -XX:+UseJVMCICompiler

##### D2: AOT编译器

1. jdk9引入了AoT编译器（静态提前编译器，Ahead of Time Compiler）

2. Java 9引入了实验性AOT编译工具jaotc。它借助了Graal编译器，将所输入的Java类文件转换为机器码，并存放至生成的动态共享库之中。

3. 所谓AOT编译，是与即时编译相对立的一个概念。我们知道，即时编译指的是**在程序的运行过程中**，将字节码转换为可在硬件上直接运行的机器码，并部署至托管环境中的过程。而AOT编译指的则是，**在程序运行之前**，便将字节码转换为机器码的过程。

   .java -> .class -> (使用jaotc) -> .so

**AOT编译器编译器的优缺点**

**最大的好处：**

1. Java虚拟机加载已经预编译成二进制库，可以直接执行。
2. 不必等待即时编译器的预热，减少Java应用给人带来“第一次运行慢” 的不良体验

**缺点：**

1. 破坏了 java “ 一次编译，到处运行”，必须为每个不同的硬件，OS编译对应的发行包
2. 降低了Java链接过程的动态性，加载的代码在编译器就必须全部已知。
3. 还需要继续优化中，最初只支持Linux X64 java base



## 第十一章: StringTable（字符串常量池)

### P1: String的基本特性

1. String：字符串，使用一对 “” 引起来表示

```JAVA
 String s1 = "atguigu" ;               // 字面量的定义方式
 String s2 =  new String("hello");     // new 对象的方式Copy to clipboardErrorCopied
```

2. String被声明为final的，不可被继承
3. String实现了Serializable接口：表示字符串是支持序列化的。实现了Comparable接口：表示String可以比较大小
4. String在jdk8及以前内部定义了`final char value[]`用于存储字符串数据。JDK9时改为`byte[]`

### P2: 为什么 JDK9 改变了String的结构

> **官方文档**：http://openjdk.java.net/jeps/254

- **为什么改为 byte[] 存储？**
  1. String类的当前实现将字符存储在char数组中，每个字符使用两个字节(16位)。
  2. 从许多不同的应用程序收集的数据表明，字符串是堆使用的主要组成部分，而且大多数字符串对象只包含拉丁字符（Latin-1）。这些字符只需要一个字节的存储空间，因此这些字符串对象的内部char数组中有一半的空间将不会使用，产生了大量浪费。
  3. 之前 String 类使用 UTF-16 的 char[] 数组存储，现在改为 byte[] 数组 外加一个编码标识存储。该编码表示如果你的字符是ISO-8859-1或者Latin-1，那么只需要一个字节存。如果你是其它字符集，比如UTF-8，你仍然用两个字节存
  4. 结论：String再也不用char[] 来存储了，改成了byte [] 加上编码标记，节约了一些空间
  5. 同时基于String的数据结构，例如StringBuffer和StringBuilder也同样做了修改

```java
// 之前
private final char value[];
// 之后
private final byte[] ;
```

#### p1: String 的基本特性

- String：代表不可变的字符序列。简称：不可变性。

1. 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
2. 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
3. 当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

- 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。

**当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值**

代码

```java
    @Test
    public void test1() {
        ①String s1 = "abc";//字面量定义的方式，"abc"存储在字符串常量池中
        String s2 = "abc";
        ②s1 = "hello";

        System.out.println(s1 == s2);//判断地址：①true  --> ②false

        System.out.println(s1);//
        System.out.println(s2);//abc

    }
```

字节码指令

- 取字符串 “abc” 时，使用的是同一个符号引用：#2
- 取字符串 “hello” 时，使用的是另一个符号引用：#3

**当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值**

```java
    @Test
    public void test2() {
        String s1 = "abc";
        String s2 = "abc";
        s2 += "def";
        System.out.println(s2);//abcdef
        System.out.println(s1);//abc
    }
```

**当调用string的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值**

```java
@Test
public void test3() {
    String s1 = "abc";
    String s2 = s1.replace('a', 'm');
    System.out.println(s1);//abc
    System.out.println(s2);//mbc
}
```

**一道笔试题**

```java
public class StringExer {
    String str = new String("good");
    char[] ch = {'t', 'e', 's', 't'};

    public void change(String str, char ch[]) {
        str = "test ok";
        ch[0] = 'b';
    }

    public static void main(String[] args) {
        StringExer ex = new StringExer();
        ex.change(ex.str, ex.ch);
        System.out.println(ex.str);//good
        System.out.println(ex.ch);//best
    }
}
```

str 的内容并没有变：“test ok” 位于字符串常量池中的另一个区域（地址），进行赋值操作并没有修改原来 str 指向的引用的内容

#### p2: String 的底层结构

**字符串常量池是不会存储相同内容的字符串的**

1. String的String Pool（字符串常量池）是一个固定大小的Hashtable，默认值大小长度是1009。如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern()方法时性能会大幅下降。
2. 使用-XX:StringTablesize可设置StringTable的长度
3. 在JDK6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快，StringTablesize设置没有要求
4. 在JDK7中，StringTable的长度默认值是60013，StringTablesize设置没有要求
5. 在JDK8中，StringTable的长度默认值是60013，StringTable可以设置的最小值为1009

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413202213040.png" alt="image-20210413202213040" style="zoom:33%;float:left" />

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413202243296.png" alt="image-20210413202243296" style="zoom:50%;" />

**测试不同 StringTable 长度下，程序的性能**

代码

```java
/**
 * 产生10万个长度不超过10的字符串，包含a-z,A-Z
 */
public class GenerateString {
    public static void main(String[] args) throws IOException {
        FileWriter fw =  new FileWriter("words.txt");

        for (int i = 0; i < 100000; i++) {
            //1 - 10
           int length = (int)(Math.random() * (10 - 1 + 1) + 1);
            fw.write(getString(length) + "\n");
        }

        fw.close();
    }

    public static String getString(int length){
        String str = "";
        for (int i = 0; i < length; i++) {
            //65 - 90, 97-122
            int num = (int)(Math.random() * (90 - 65 + 1) + 65) + (int)(Math.random() * 2) * 32;
            str += (char)num;
        }
        return str;
    }
}

public class StringTest2 {
    public static void main(String[] args) {

        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader("words.txt"));
            long start = System.currentTimeMillis();
            String data;
            while((data = br.readLine()) != null){
                data.intern(); //如果字符串常量池中没有对应data的字符串的话，则在常量池中生成
            }

            long end = System.currentTimeMillis();

            System.out.println("花费的时间为：" + (end - start));//1009:143ms  100009:47ms
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
}
```

- -XX:StringTableSize=1009 ：程序耗时 143ms
- -XX:StringTableSize=100009 ：程序耗时 47ms

### P3: String 的内存分配

1. 在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念。
2. 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种。
   - 直接使用双引号声明出来的String对象会直接存储在常量池中。比如：`String info="atguigu.com";`
   - 如果不是用双引号声明的String对象，可以使用String提供的intern()方法。这个后面重点谈

1. Java 6及以前，字符串常量池存放在永久代
2. Java 7中 Oracle的工程师对字符串池的逻辑做了很大的改变，即将字符串常量池的位置调整到Java堆内
   - 所有的字符串都保存在堆（Heap）中，和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
   - 字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在Java 7中使用String.intern()。
3. Java8元空间，字符串常量在堆

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413202334713.png" alt="image-20210413202334713" style="zoom:33%;float:left" />

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413202405216.png" alt="image-20210413202405216" style="zoom:33%;float:left" />



#### p1: StringTable为什么要调整?

> **官方文档**:https://www.oracle.com/java/technologies/javase/jdk7-relnotes.html#jdk7changes

1. 为什么要调整位置？
   - 永久代的默认空间大小比较小
   - 永久代垃圾回收频率低，大量的字符串无法及时回收，容易进行Full GC产生STW或者容易产生OOM：PermGen Space
   - 堆中空间足够大，字符串可被及时回收
2. 在JDK 7中，interned字符串不再在Java堆的永久代中分配，而是在Java堆的主要部分（称为年轻代和年老代）中分配，与应用程序创建的其他对象一起分配。
3. 此更改将导致驻留在主Java堆中的数据更多，驻留在永久生成中的数据更少，因此可能需要调整堆大小。

**代码示例**

```java
/**
 * jdk6中：
 * -XX:PermSize=6m -XX:MaxPermSize=6m -Xms6m -Xmx6m
 *
 * jdk8中：
 * -XX:MetaspaceSize=6m -XX:MaxMetaspaceSize=6m -Xms6m -Xmx6m
 */
public class StringTest3 {
    public static void main(String[] args) {
        //使用Set保持着常量池引用，避免full gc回收常量池行为
        Set<String> set = new HashSet<String>();
        //在short可以取值的范围内足以让6MB的PermSize或heap产生OOM了。
        short i = 0;
        while(true){
            set.add(String.valueOf(i++).intern());
        }
    }
}
```

输出结果：我真没骗你，字符串真的在堆中（JDK8）

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
    at java.util.HashMap.resize(HashMap.java:703)
    at java.util.HashMap.putVal(HashMap.java:662)
    at java.util.HashMap.put(HashMap.java:611)
    at java.util.HashSet.add(HashSet.java:219)
    at com.atguigu.java.StringTest3.main(StringTest3.java:22)

Process finished with exit code 1
```

### P4: String 的基本操作

Java语言规范里要求完全相同的字符串字面量，应该包含同样的Unicode字符序列（包含同一份码点序列的常量），并且必须是指向同一个String类实例。

#### p1: 举例1

```java
public class StringTest4 {
    public static void main(String[] args) {
        System.out.println();//2293
        System.out.println("1");//2294
        System.out.println("2");
        System.out.println("3");
        System.out.println("4");
        System.out.println("5");
        System.out.println("6");
        System.out.println("7");
        System.out.println("8");
        System.out.println("9");
        System.out.println("10");//2303
        //如下的字符串"1" 到 "10"不会再次加载
        System.out.println("1");//2304
        System.out.println("2");//2304
        System.out.println("3");
        System.out.println("4");
        System.out.println("5");
        System.out.println("6");
        System.out.println("7");
        System.out.println("8");
        System.out.println("9");
        System.out.println("10");//2304
    }
}
```

分析字符串常量池的变化

1、程序启动时已经加载了 2293 个字符串常量

![image-20210413202642869](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202642869.png)

2、加载了一个换行符（println），所以多了一个

![image-20210413202708824](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202708824.png)

3、加载了字符串常量 “1”~“9”

![image-20210413202718725](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202718725.png)

4、加载字符串常量 “10”

![image-20210413202730179](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202730179.png)

5、之后的字符串"1" 到 "10"不会再次加载

![image-20210413202739624](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202739624.png)

#### p2: 举例2

```java
//官方示例代码
class Memory {
    public static void main(String[] args) {//line 1
        int i = 1;//line 2
        Object obj = new Object();//line 3
        Memory mem = new Memory();//line 4
        mem.foo(obj);//line 5
    }//line 9

    private void foo(Object param) {//line 6
        String str = param.toString();//line 7
        System.out.println(str);
    }//line 8
}
```

分析运行时内存（foo() 方法是实例方法，其实图中少了一个 this 局部变量）

![image-20210413202815365](/Users/breeze/Library/Application Support/typora-user-images/image-20210413202815365.png)

### P5: 字符串拼接操作

#### p1: 先说结论

1. 常量与常量的拼接结果在常量池，原理是编译期优化
2. 常量池中不会存在相同内容的变量
3. 拼接前后，只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder
4. 如果拼接的结果调用intern()方法，根据该字符串是否在常量池中存在，分为：
   - 如果存在，则返回字符串在常量池中的地址
   - 如果字符串常量池中不存在该字符串，则在常量池中创建一份，并返回此对象的地址

**1、常量与常量的拼接结果在常量池，原理是编译期优化**

代码

```java
@Test
    public void test1(){
        String s1 = "a" + "b" + "c";//编译期优化：等同于"abc"
        String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
        /*
         * 最终.java编译成.class,再执行.class
         * String s1 = "abc";
         * String s2 = "abc"
         */
        System.out.println(s1 == s2); //true
        System.out.println(s1.equals(s2)); //true
    }
```

从字节码指令看出：编译器做了优化，将 “a” + “b” + “c” 优化成了 “abc”

```java
0 ldc #2 <abc>
2 astore_1
3 ldc #2 <abc>
5 astore_2
6 getstatic #3 <java/lang/System.out>
9 aload_1
10 aload_2
11 if_acmpne 18 (+7)
14 iconst_1
15 goto 19 (+4)
18 iconst_0
19 invokevirtual #4 <java/io/PrintStream.println>
22 getstatic #3 <java/lang/System.out>
25 aload_1
26 aload_2
27 invokevirtual #5 <java/lang/String.equals>
30 invokevirtual #4 <java/io/PrintStream.println>
33 returnCopy to clipboardErrorCopied
```

IDEA 反编译 class 文件后，来看这个问题

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413202944570.png" alt="image-20210413202944570" style="zoom:33%;float:left" />

**2、拼接前后，只要其中有一个是变量，结果就在堆中**

**调用 intern() 方法，则主动将字符串对象存入字符串常量池中，并将其地址返回**

```java
@Test
    public void test2(){
        String s1 = "javaEE";
        String s2 = "hadoop";

        String s3 = "javaEEhadoop";
        String s4 = "javaEE" + "hadoop";//编译期优化
        //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，具体的内容为拼接的结果：javaEEhadoop
        String s5 = s1 + "hadoop";
        String s6 = "javaEE" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4);//true
        System.out.println(s3 == s5);//false
        System.out.println(s3 == s6);//false
        System.out.println(s3 == s7);//false
        System.out.println(s5 == s6);//false
        System.out.println(s5 == s7);//false
        System.out.println(s6 == s7);//false
        //intern():判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址；
        //如果字符串常量池中不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回次对象的地址。
        String s8 = s6.intern();
        System.out.println(s3 == s8);//true
    }
```

从字节码角度来看：拼接前后有变量，都会使用到 StringBuilder 类

```java
0 ldc #6 <javaEE>
2 astore_1
3 ldc #7 <hadoop>
5 astore_2
6 ldc #8 <javaEEhadoop>
8 astore_3
9 ldc #8 <javaEEhadoop>
11 astore 4
13 new #9 <java/lang/StringBuilder>
16 dup
17 invokespecial #10 <java/lang/StringBuilder.<init>>
20 aload_1
21 invokevirtual #11 <java/lang/StringBuilder.append>
24 ldc #7 <hadoop>
26 invokevirtual #11 <java/lang/StringBuilder.append>
29 invokevirtual #12 <java/lang/StringBuilder.toString>
32 astore 5
34 new #9 <java/lang/StringBuilder>
37 dup
38 invokespecial #10 <java/lang/StringBuilder.<init>>
41 ldc #6 <javaEE>
43 invokevirtual #11 <java/lang/StringBuilder.append>
46 aload_2
47 invokevirtual #11 <java/lang/StringBuilder.append>
50 invokevirtual #12 <java/lang/StringBuilder.toString>
53 astore 6
55 new #9 <java/lang/StringBuilder>
58 dup
59 invokespecial #10 <java/lang/StringBuilder.<init>>
62 aload_1
63 invokevirtual #11 <java/lang/StringBuilder.append>
66 aload_2
67 invokevirtual #11 <java/lang/StringBuilder.append>
70 invokevirtual #12 <java/lang/StringBuilder.toString>
73 astore 7
75 getstatic #3 <java/lang/System.out>
78 aload_3
79 aload 4
81 if_acmpne 88 (+7)
84 iconst_1
85 goto 89 (+4)
88 iconst_0
89 invokevirtual #4 <java/io/PrintStream.println>
92 getstatic #3 <java/lang/System.out>
95 aload_3
96 aload 5
98 if_acmpne 105 (+7)
101 iconst_1
102 goto 106 (+4)
105 iconst_0
106 invokevirtual #4 <java/io/PrintStream.println>
109 getstatic #3 <java/lang/System.out>
112 aload_3
113 aload 6
115 if_acmpne 122 (+7)
118 iconst_1
119 goto 123 (+4)
122 iconst_0
123 invokevirtual #4 <java/io/PrintStream.println>
126 getstatic #3 <java/lang/System.out>
129 aload_3
130 aload 7
132 if_acmpne 139 (+7)
135 iconst_1
136 goto 140 (+4)
139 iconst_0
140 invokevirtual #4 <java/io/PrintStream.println>
143 getstatic #3 <java/lang/System.out>
146 aload 5
148 aload 6
150 if_acmpne 157 (+7)
153 iconst_1
154 goto 158 (+4)
157 iconst_0
158 invokevirtual #4 <java/io/PrintStream.println>
161 getstatic #3 <java/lang/System.out>
164 aload 5
166 aload 7
168 if_acmpne 175 (+7)
171 iconst_1
172 goto 176 (+4)
175 iconst_0
176 invokevirtual #4 <java/io/PrintStream.println>
179 getstatic #3 <java/lang/System.out>
182 aload 6
184 aload 7
186 if_acmpne 193 (+7)
189 iconst_1
190 goto 194 (+4)
193 iconst_0
194 invokevirtual #4 <java/io/PrintStream.println>
197 aload 6
199 invokevirtual #13 <java/lang/String.intern>
202 astore 8
204 getstatic #3 <java/lang/System.out>
207 aload_3
208 aload 8
210 if_acmpne 217 (+7)
213 iconst_1
214 goto 218 (+4)
217 iconst_0
218 invokevirtual #4 <java/io/PrintStream.println>
221 return
```

#### p2: 字符串拼接的底层细节

**举例1**

```java
    @Test
    public void test3(){
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        /*
        如下的s1 + s2 的执行细节：(变量s是我临时定义的）
        ① StringBuilder s = new StringBuilder();
        ② s.append("a")
        ③ s.append("b")
        ④ s.toString()  --> 约等于 new String("ab")，但不等价

        补充：在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer
         */
        String s4 = s1 + s2;//
        System.out.println(s3 == s4);//false
    }
```

字节码指令

```java
0 ldc #14 <a>
2 astore_1
3 ldc #15 <b>
5 astore_2
6 ldc #16 <ab>
8 astore_3
9 new #9 <java/lang/StringBuilder>
12 dup
13 invokespecial #10 <java/lang/StringBuilder.<init>>
16 aload_1
17 invokevirtual #11 <java/lang/StringBuilder.append>
20 aload_2
21 invokevirtual #11 <java/lang/StringBuilder.append>
24 invokevirtual #12 <java/lang/StringBuilder.toString>
27 astore 4
29 getstatic #3 <java/lang/System.out>
32 aload_3
33 aload 4
35 if_acmpne 42 (+7)
38 iconst_1
39 goto 43 (+4)
42 iconst_0
43 invokevirtual #4 <java/io/PrintStream.println>
46 return
```

**举例2**

```java
/*
    1. 字符串拼接操作不一定使用的是StringBuilder!
       如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译期优化，即非StringBuilder的方式。
    2. 针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用上final的时候建议使用上。
     */
    @Test
    public void test4(){
        final String s1 = "a";
        final String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;
        System.out.println(s3 == s4);//true
    }
```

从字节码角度来看：为变量 s4 赋值时，直接使用 #16 符号引用，即字符串常量 “ab”

```java
0 ldc #14 <a>
2 astore_1
3 ldc #15 <b>
5 astore_2
6 ldc #16 <ab>
8 astore_3
9 ldc #16 <ab>
11 astore 4
13 getstatic #3 <java/lang/System.out>
16 aload_3
17 aload 4
19 if_acmpne 26 (+7)
22 iconst_1
23 goto 27 (+4)
26 iconst_0
27 invokevirtual #4 <java/io/PrintStream.println>
30 returnCopy to clipboardErrorCopied
```

**拼接操作与 append 操作的效率对比**

```java
    @Test
    public void test6(){

        long start = System.currentTimeMillis();

//        method1(100000);//4014
        method2(100000);//7

        long end = System.currentTimeMillis();

        System.out.println("花费的时间为：" + (end - start));
    }

    public void method1(int highLevel){
        String src = "";
        for(int i = 0;i < highLevel;i++){
            src = src + "a";//每次循环都会创建一个StringBuilder、String
        }
//        System.out.println(src);

    }

    public void method2(int highLevel){
        //只需要创建一个StringBuilder
        StringBuilder src = new StringBuilder();
        for (int i = 0; i < highLevel; i++) {
            src.append("a");
        }
//        System.out.println(src);
    }Copy to clipboardErrorCopied
```

1. 体会执行效率：通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
2. 原因：
   1. StringBuilder的append()的方式：
      - 自始至终中只创建过一个StringBuilder的对象
   2. 使用String的字符串拼接方式：
      - 创建过多个StringBuilder和String（调的toString方法）的对象，内存占用更大；
      - 如果进行GC，需要花费额外的时间（在拼接的过程中产生的一些中间字符串可能永远也用不到，会产生大量垃圾字符串）。
3. 改进的空间：
   - 在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下，建议使用构造器实例化：
   - `StringBuilder s = new StringBuilder(highLevel); //new char[highLevel]`
   - 这样可以避免频繁扩容

### P6: intern() 的使用

#### p1: intern() 方法的说明

```java
public native String intern();
```

1. intern是一个native方法，调用的是底层C的方法

2. 字符串常量池池最初是空的，由String类私有地维护。在调用intern方法时，如果池中已经包含了由equals(object)方法确定的与该字符串内容相等的字符串，则返回池中的字符串地址。否则，该字符串对象将被添加到池中，并返回对该字符串对象的地址。（这是源码里的大概翻译）

3. 如果不是用双引号声明的String对象，可以使用String提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。比如：

   ```markup
    String myInfo = new string("I love atguigu").intern();
   ```

4. 也就是说，如果在任意字符串上调用String.intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true

   ```markup
    ("a"+"b"+"c").intern()=="abc"
   ```

5. 通俗点讲，Interned String就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池（String Intern Pool）

#### p2: new String() 的说明

- ##### new String(“ab”)会创建几个对象？

```java
/**
 * 题目：
 * new String("ab")会创建几个对象？看字节码，就知道是两个。
 *     一个对象是：new关键字在堆空间创建的
 *     另一个对象是：字符串常量池中的对象"ab"。 字节码指令：ldc
 *
 */
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}
```

字节码指令

```java
0 new #2 <java/lang/String>
3 dup
4 ldc #3 <ab>
6 invokespecial #4 <java/lang/String.<init>>
9 astore_1
10 return
```

`0 new #2 <java/lang/String>`：在堆中创建了一个 String 对象

`4 ldc #3 <ab>` ：在字符串常量池中放入 “ab”（如果之前字符串常量池中没有 “ab” 的话）

- ##### new String(“a”) + new String(“b”) 会创建几个对象？

代码

```java
/**
 * 思考：
 * new String("a") + new String("b")呢？
 *  对象1：new StringBuilder()
 *  对象2： new String("a")
 *  对象3： 常量池中的"a"
 *  对象4： new String("b")
 *  对象5： 常量池中的"b"
 *
 *  深入剖析： StringBuilder的toString():
 *      对象6 ：new String("ab")
 *       强调一下，toString()的调用，在字符串常量池中，没有生成"ab"
 *
 */
public class StringNewTest {
    public static void main(String[] args) {

        String str = new String("a") + new String("b");
    }
}
```

字节码指令

```java
0 new #2 <java/lang/StringBuilder>
3 dup
4 invokespecial #3 <java/lang/StringBuilder.<init>>
7 new #4 <java/lang/String>
10 dup
11 ldc #5 <a>
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>
22 dup
23 ldc #8 <b>
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>
34 astore_1
35 return
```

**答案是4个或5个或6个**

字节码指令分析：

1. `0 new #2 <java/lang/StringBuilder>` ：拼接字符串会创建一个 StringBuilder 对象
2. `7 new #4 <java/lang/String>` ：创建 String 对象，对应于 new String(“a”)
3. `11 ldc #5 <a>` ：在字符串常量池中放入 “a”（如果之前字符串常量池中没有 “a” 的话）
4. `19 new #4 <java/lang/String>` ：创建 String 对象，对应于 new String(“b”)
5. `23 ldc #8 <b>` ：在字符串常量池中放入 “b”（如果之前字符串常量池中没有 “b” 的话）
6. `31 invokevirtual #9 <java/lang/StringBuilder.toString>` ：调用 StringBuilder 的 toString() 方法，会生成一个 String 对象

![image-20210413203445759](/Users/breeze/Library/Application Support/typora-user-images/image-20210413203445759.png)

#### p3: 有点难的面试题

```java
**
 * 如何保证变量s指向的是字符串常量池中的数据呢？
 * 有两种方式：
 * 方式一： String s = "shkstart";//字面量定义的方式
 * 方式二： 调用intern()
 *         String s = new String("shkstart").intern();
 *         String s = new StringBuilder("shkstart").toString().intern();
 *
 */
public class StringIntern {
    public static void main(String[] args) {

        String s = new String("1");
        s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
        String s2 = "1";
        System.out.println(s == s2);//jdk6：false   jdk7/8：false
        
        /*
         1、s3变量记录的地址为：new String("11")
         2、经过上面的分析，我们已经知道执行完pos_1的代码，在堆中有了一个new String("11")
         这样的String对象。但是在字符串常量池中没有"11"
         3、接着执行s3.intern()，在字符串常量池中生成"11"
           3-1、在JDK6的版本中，字符串常量池还在永久代，所以直接在永久代生成"11",也就有了新的地址
           3-2、而在JDK7的后续版本中，字符串常量池被移动到了堆中，此时堆里已经有new String（"11"）了
           出于节省空间的目的，直接将堆中的那个字符串的引用地址储存在字符串常量池中。没错，字符串常量池
           中存的是new String（"11"）在堆中的地址
         4、所以在JDK7后续版本中，s3和s4指向的完全是同一个地址。
         */
        String s3 = new String("1") + new String("1");//pos_1
        s3.intern();
        
        String s4 = "11";//s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址
        System.out.println(s3 == s4);//jdk6：false  jdk7/8：true
    }


}
```

解释的已经比较清楚了，下面看一下内存图

**内存分析**

JDK6 ：正常眼光判断即可

- new String() 即在堆中
- str.intern() 则把字符串放入常量池中

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413203536346.png" alt="image-20210413203536346" style="zoom:26%;float:left" />

JDK7及后续版本，**注意大坑**

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413203613868.png" alt="image-20210413203613868" style="zoom:26%;float:left" />

#### p4: 面试题的拓展

```java
/**
 * StringIntern.java中练习的拓展：
 *
 */
public class StringIntern1 {
    public static void main(String[] args) {
        //执行完下一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        String s3 = new String("1") + new String("1");//new String("11")
        //在字符串常量池中生成对象"11"，代码顺序换一下，实打实的在字符串常量池里有一个"11"对象
        String s4 = "11";  
        String s5 = s3.intern();

        // s3 是堆中的 "ab" ，s4 是字符串常量池中的 "ab"
        System.out.println(s3 == s4);//false

        // s5 是从字符串常量池中取回来的引用，当然和 s4 相等
        System.out.println(s5 == s4);//true
    }
}Copy to clipboardErrorCopied
```

#### p5: intern() 方法的练习

**练习 1**

```java
public class StringExer1 {
    public static void main(String[] args) {
        String x = "ab";
        String s = new String("a") + new String("b");//new String("ab")
        //在上一行代码执行完以后，字符串常量池中并没有"ab"
        /*
        1、jdk6中：在字符串常量池（此时在永久代）中创建一个字符串"ab"
        2、jdk8中：字符串常量池（此时在堆中）中没有创建字符串"ab",而是创建一个引用，指向new String("ab")，          将此引用返回
        3、详解看上面
        */
        String s2 = s.intern();

        System.out.println(s2 == "ab");//jdk6:true  jdk8:true
        System.out.println(s == "ab");//jdk6:false  jdk8:true
    }
}Copy to clipboardErrorCopied
```

**JDK6**

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413203736499.png" alt="image-20210413203736499" style="zoom:33%;float:left" />

**JDK7/8**

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413203756940.png" alt="image-20210413203756940" style="zoom:33%;float:left" />

**练习2**

```java
public class StringExer1 {
    public static void main(String[] args) {
        //加一行这个
        String x = "ab";
        String s = new String("a") + new String("b");//new String("ab")

        String s2 = s.intern();

        System.out.println(s2 == "ab");//jdk6:true  jdk8:true
        System.out.println(s == "ab");//jdk6:false  jdk8:true
    }
}Copy to clipboardErrorCopied
```

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413203831896.png" alt="image-20210413203831896" style="zoom:33%;float:left" />

**练习3**

```java
public class StringExer2 {
    public static void main(String[] args) {
        String s1 = new String("ab");//执行完以后，会在字符串常量池中会生成"ab"

        s1.intern();
        String s2 = "ab";
        System.out.println(s1 == s2);//false
    }
}Copy to clipboardErrorCopied
```

**验证**

```java
public class StringExer2 {
    // 对象内存地址可以使用System.identityHashCode(object)方法获取
    public static void main(String[] args) {
        String s1 = new String("a") + new String("b");//执行完以后，不会在字符串常量池中会生成"ab"
        System.out.println(System.identityHashCode(s1));
        s1.intern();
        System.out.println(System.identityHashCode(s1));
        String s2 = "ab";
        System.out.println(System.identityHashCode(s2));
        System.out.println(s1 == s2); // true
    }
}Copy to clipboardErrorCopied
```

输出结果：

```java
1836019240
1836019240
1836019240
trueCopy to clipboardErrorCopied
```

#### p6: intern() 的效率测试（空间角度）

```java
/**
 * 使用intern()测试执行效率：空间使用上
 *
 * 结论：对于程序中大量存在存在的字符串，尤其其中存在很多重复字符串时，使用intern()可以节省内存空间。
 *
 */
public class StringIntern2 {
    static final int MAX_COUNT = 1000 * 10000;
    static final String[] arr = new String[MAX_COUNT];

    public static void main(String[] args) {
        Integer[] data = new Integer[]{1,2,3,4,5,6,7,8,9,10};

        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
//            arr[i] = new String(String.valueOf(data[i % data.length]));
            arr[i] = new String(String.valueOf(data[i % data.length])).intern();

        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为：" + (end - start));

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.gc();
    }
}
```

1、直接 new String ：由于每个 String 对象都是 new 出来的，所以程序需要维护大量存放在堆空间中的 String 实例，程序内存占用也会变高

```java
arr[i] = new String(String.valueOf(data[i % data.length]));
```

 ![image-20210413203950094](/Users/breeze/Library/Application Support/typora-user-images/image-20210413203950094.png)

![image-20210413204024590](/Users/breeze/Library/Application Support/typora-user-images/image-20210413204024590.png)

2、使用 intern() 方法：由于数组中字符串的引用都指向字符串常量池中的字符串，所以程序需要维护的 String 对象更少，内存占用也更低

```java
//调用了intern()方法使用了字符串常量池里的字符串，那么前面堆里的字符串便会被GC掉，这也是intern省内存的关键原因
arr[i] = new String(String.valueOf(data[i % data.length])).intern();
```

 ![image-20210413204044645](/Users/breeze/Library/Application Support/typora-user-images/image-20210413204044645.png)

![image-20210413204059871](/Users/breeze/Library/Application Support/typora-user-images/image-20210413204059871.png)

**结论**：

1. 对于程序中大量使用存在的字符串时，尤其存在很多已经重复的字符串时，使用intern()方法能够节省很大的内存空间。
2. 大的网站平台，需要内存中存储大量的字符串。比如社交网站，很多人都存储：北京市、海淀区等信息。这时候如果字符串都调用intern() 方法，就会很明显降低内存的大小。

### P7: StringTable 的垃圾回收

```java
/**
 * String的垃圾回收:
 * -Xms15m -Xmx15m -XX:+PrintStringTableStatistics -XX:+PrintGCDetails
 */
public class StringGCTest {
    public static void main(String[] args) {
        for (int j = 0; j < 100000; j++) {
            String.valueOf(j).intern();
        }
    }
}Copy to clipboardErrorCopied
```

输出结果：

- 在 PSYoungGen 区发生了垃圾回收
- Number of entries 和 Number of literals 明显没有 100000
- 以上两点均说明 StringTable 区发生了垃圾回收

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413204201010.png" alt="image-20210413204201010" style="zoom: 33%; float: left;" />![image-20210413204227048](/Users/breeze/Library/Application Support/typora-user-images/image-20210413204227048.png)

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210413204301529.png" alt="image-20210413204301529" style="zoom:33%;float:left" />

### P8: G1 中的 String 去重操作

> **官方文档**：http://openjdk.java.net/jeps/192

暂时了解一下，后面会详解垃圾回收器

**String去重操作的背景**

> 注意不是字符串常量池的去重操作，字符串常量池本身就没有重复的

1. 背景：对许多Java应用（有大的也有小的）做的测试得出以下结果：
   - 堆存活数据集合里面String对象占了25%
   - 堆存活数据集合里面重复的String对象有13.5%
   - String对象的平均长度是45
2. 许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是String对象。更进一步，这里面差不多一半String对象是重复的，重复的意思是说：`str1.equals(str2)= true`。堆上存在重复的String对象必然是一种内存的浪费。这个项目将在G1垃圾收集器中实现自动持续对重复的String对象进行去重，这样就能避免浪费内存。

**String 去重的的实现**

1. 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的String对象。
2. 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象。
3. 使用一个Hashtable来记录所有的被String对象使用的不重复的char数组。当去重的时候，会查这个Hashtable，来看堆上是否已经存在一个一模一样的char数组。
4. 如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
5. 如果查找失败，char数组会被插入到Hashtable，这样以后的时候就可以共享这个数组了。

**命令行选项**

1. UseStringDeduplication(bool) ：开启String去重，默认是不开启的，需要手动开启。
2. PrintStringDeduplicationStatistics(bool) ：打印详细的去重统计信息
3. stringDeduplicationAgeThreshold(uintx) ：达到这个年龄的String对象被认为是去重的候选对象











