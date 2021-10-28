### P1:  泛型

* 概述:
    * 泛型本质是参数化类型,解决不确定对象具体类型的问题.
    * 泛型在定义处只具备执行Object方法的能力.
* 优点:
    * 类型安全,放置什么类型,取出来就是什么类型,不存在ClassCastException类型转换异常;
    * 提升可读性,编码阶段就显式地知道泛型集合,泛型方法等处理的对象类型.
    * 代码重用,合并了用类型的处理代码;
* **泛型擦除:**
    * 泛型用于编译阶段,编译后的字节码文件不包含泛型类型信息,因为虚拟机没有泛型类型对象,所有对象都属于普通类.
        * 例如:List<Object>或List<String>,在编译后都会变成list;
    * 定义一个泛型类型,会自动提供一个对应原始类型,类型变量会被擦除;
        * 如果没有限定类型就会替换为Object,如果有限定类型就会替换为第一个限定类型;
            * 例如:<T extends A & B> 会使用A类型替换T.



### P2:  反射

* **概述:(什么是反射?)**
  * 是指在运行状态中,对于任意一个类,都能够**知道**这个类的所有属性和方法;对于任意一个对象,都能够**调用**它的任意一个属性和方法.通俗的讲,就是在得到class对象之后,反向获取该对象的各种信息
  * 前提条件:想要解剖一个类,必须要先获取到该类的字节码文件对象.
* **哪里用到了反射?**
  * 框架中:
  
    Spring mybatis 框架中使用了大量的动态代理,而动态代理的实现也依赖反射
* **优缺点:**
  * 优点:提高了程序的灵活性和扩展性,降低耦合性,提高自适应能力;
  * 缺点:
    * 性能问题:
      * 反射基本上是一种解释操作,用于字段和方法接入时要远慢于直接代码.因此反射机制主要用于在灵活性和扩展性要求很高的系统框架上,普通的程序不建议使用.
    * 反射会模糊程序的内部逻辑:
      * 程序员希望能在源代码中看到程序的逻辑,反射则绕过了源代码,因而会带来维护的问题,反射代码比相应的直接直接代码更复杂.

**6）如何使用java的反射?**

    * 通过一个全限类名创建一个对象
        * Class.forName(“全限类名”); 例如：com.mysql.jdbc.Driver Driver类已经被加载到 jvm中，并且完成了类的初始化工作就行了
        * 类名.class; 获取Class<？> clz 对象
        * 对象.getClass();
    * 获取构造器对象，通过构造器new出一个对象
        * Clazz.getConstructor([String.class]);
        * Con.newInstance([参数]);
    * 通过class对象创建一个实例对象（就相当与new类名（）无参构造器)
        * Clazz.newInstance();
    * 通过class对象获得一个属性对象
        * Field c=clz.getFields()：获得某个类的所有的公共（public）的字段，包括父类中的字段。
        * Field c=clz.getDeclaredFields()：获得某个类的所有声明的字段，即包括public、private和proteced，但是不包括父类的申明字段 e.
    * 通过class对象获得一个方法对象
        * Clazz.getMethod(“方法名”,class……parameaType);（只能获取公共的）
        * Clazz.getDeclareMethod(“方法名”);（获取任意修饰的方法，不能执行私有）
        * M.setAccessible(true);（让私有的方法可以执行）
    * f. 让方法执行
        * Method.invoke(obj实例对象,obj可变参数);-----（是有返回值的）



### P3: java集合框架

#### 3.1: 概述

`除了以Map结尾的类之外,其他类都实现了Collection接口`

<img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2019%201635391579%201635391579068%20qIckG9%20image-20210804112012458.png" alt="image-20210804112012458" style="zoom:30%;float:left" />

- **List , Set , Map三者区别**
  - **List:** 存储的元素有序可重复
  - **Set:** 无序不可重复
  - **Map:** 键值对存储; key无序不可重复,value无序可重复

  

#### 3.2:  Collection接口之 List

##### 3.2.1: ArrayList与Vector的区别

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；
- `Vector` 是 `List` 的古老实现类，底层使用`Object[ ]` 存储，线程安全的。



##### 3.2.2: ArrayList和LinkedList的区别

|                  | ArrayList                                      | LinkedList                                                   |
| ---------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| **是否线程安全** | 否                                             | 否                                                           |
| **底层数据结构** | Object[]                                       | 双向链表(1.6及以前循环双向链表,1.7开始取消了循环)            |
| **增删改查**     | 增删慢,查改快                                  | 增删快,查改慢                                                |
| **随机访问**     | 支持                                           | 不支持                                                       |
| **内存空间占用** | 空间浪费体现在list列表结尾会预留一定的容量空间 | 空间浪费体现在每个元素都需要消耗比arrayList更多的空间(存放直接后继和直接前驱以及数据) |



##### 3.2.3: ArrayList核心源码

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;


public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

     //用于默认大小空实例的共享空数组实例。
      //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList 所包含的元素个数
     */
    private int size;

    /**
     * 带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //如果传入的参数大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果传入的参数等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //其他情况，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *默认无参构造函数
     *DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //将指定集合转换为数组
        elementData = c.toArray();
        //如果elementData数组的长度不为0
        if ((size = elementData.length) != 0) {
            // 如果elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组所以加上下面的语句用于判断）
            if (elementData.getClass() != Object[].class)
                //将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 其他情况，用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 修改这个ArrayList实例的容量是列表的当前大小。 应用程序可以使用此操作来最小化ArrayList实例的存储。
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
//那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        //如果是true，minExpand的值为0，如果是false,minExpand的值为10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //如果最小容量大于已有的最大容量
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取“默认的容量”和“传入参数”两者之间的最大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
  //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量，
        //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
        //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     *返回此列表中的元素数。
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回 true 。
     */
    public boolean isEmpty() {
        //注意=和==的区别
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true 。
     */
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。）
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）;
     *返回的数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返回其中。
     *否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。
     *如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null 。
     *（这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。）
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }

    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此列表中的指定位置插入指定的元素。
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
      //从列表中删除的元素
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
     *返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从列表中删除所有元素。
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除所有索引为fromIndex （含）和toIndex之间的元素。
     *将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合中包含的所有元素。
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。
     *返回的迭代器是fail-fast 。
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

```



##### 3.2.4: ArrayList扩容机制

1. **先从 ArrayList构造方法说起**

   `JDK8三种方式`

   ```java
      /**
        * 默认初始容量大小
        */
       private static final int DEFAULT_CAPACITY = 10;
   
   
       private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
   
       /**
        *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
        */
       public ArrayList() {
           this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
       }
   
       /**
        * 带初始容量参数的构造函数。（用户自己指定容量）
        */
       public ArrayList(int initialCapacity) {
           if (initialCapacity > 0) {//初始容量大于0
               //创建initialCapacity大小的数组
               this.elementData = new Object[initialCapacity];
           } else if (initialCapacity == 0) {//初始容量等于0
               //创建空数组
               this.elementData = EMPTY_ELEMENTDATA;
           } else {//初始容量小于0，抛出异常
               throw new IllegalArgumentException("Illegal Capacity: "+
                                                  initialCapacity);
           }
       }
   
   
      /**
       *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
       *如果指定的集合为null，throws NullPointerException。
       */
        public ArrayList(Collection<? extends E> c) {
           elementData = c.toArray();
           if ((size = elementData.length) != 0) {
               // c.toArray might (incorrectly) not return Object[] (see 6260652)
               if (elementData.getClass() != Object[].class)
                   elementData = Arrays.copyOf(elementData, size, Object[].class);
           } else {
               // replace with empty array.
               this.elementData = EMPTY_ELEMENTDATA;
           }
       }
   
   ```

   细心的同学一定会发现 ：**以无参数构造方法创建 `ArrayList` 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。** 下面在我们分析 ArrayList 扩容时会讲到这一点内容！

   > 补充：JDK6 new 无参构造的 `ArrayList` 对象时，直接创建了长度是 10 的 `Object[]` 数组 elementData 。



2. **add() 方法**

```java
    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
   //添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

```

> **注意** ：JDK11 移除了 `ensureCapacityInternal()` 和 `ensureExplicitCapacity()` 方法

3. **ensureCapacityInternal()方法**

```java
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

```

**当 要 add 进第 1 个元素时，minCapacity 为 1，在 Math.max()方法比较后，minCapacity 为 10。**

> 此处和后续 JDK8 代码格式化略有不同，核心代码基本一样。

4. **ensureCapacityInternal()**

   ```java
     //判断是否需要扩容
       private void ensureExplicitCapacity(int minCapacity) {
           modCount++;
   
           // overflow-conscious code
           if (minCapacity - elementData.length > 0)
               //调用grow方法进行扩容，调用此方法代表已经开始扩容了
               grow(minCapacity);
       }
   
   ```

   我们来仔细分析一下：

   - 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法。
   - 当 add 第 2 个元素时，minCapacity 为 2，此时 e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
   - 添加第 3、4···到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

   直到添加第 11 个元素，minCapacity(为 11)比 elementData.length（为 10）要大。进入 grow 方法进行扩容。



5. **grow()**

   ```java
       /**
        * 要分配的最大数组大小
        */
       private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
   
       /**
        * ArrayList扩容的核心方法。
        */
       private void grow(int minCapacity) {
           // oldCapacity为旧容量，newCapacity为新容量
           int oldCapacity = elementData.length;
           //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
           //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
           int newCapacity = oldCapacity + (oldCapacity >> 1);
           //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
           if (newCapacity - minCapacity < 0)
               newCapacity = minCapacity;
          // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
          //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
           if (newCapacity - MAX_ARRAY_SIZE > 0)
               newCapacity = hugeCapacity(minCapacity);
           // minCapacity is usually close to size, so this is a win:
           elementData = Arrays.copyOf(elementData, newCapacity);
       }
   
   ```

   **int newCapacity = oldCapacity + (oldCapacity >> 1),所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）！** 奇偶不同，比如 ：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数.

   > ">>"（移位运算符）：>>1 右移一位相当于除 2，右移 n 位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了 1 位所以相当于 oldCapacity /2。对于大数据的 2 进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源

   **我们再来通过例子探究一下`grow()` 方法 ：**

   - 当 add 第 1 个元素时，oldCapacity 为 0，经比较后第一个 if 判断成立，newCapacity = minCapacity(为 10)。但是第二个 if 判断不会成立，即 newCapacity 不比 MAX_ARRAY_SIZE 大，则不会进入 `hugeCapacity` 方法。数组容量为 10，add 方法中 return true,size 增为 1。
   - 当 add 第 11 个元素进入 grow 方法时，newCapacity 为 15，比 minCapacity（为 11）大，第一个 if 判断不成立。新容量没有大于数组最大 size，不会进入 hugeCapacity 方法。数组容量扩为 15，add 方法中 return true,size 增为 11。
   - 以此类推······

6. **hugeCapacity()方法**

   从上面 `grow()` 方法源码我们知道： 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果 minCapacity 大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

   ```java
       private static int hugeCapacity(int minCapacity) {
           if (minCapacity < 0) // overflow
               throw new OutOfMemoryError();
           //对minCapacity和MAX_ARRAY_SIZE进行比较
           //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
           //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
           //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
           return (minCapacity > MAX_ARRAY_SIZE) ?
               Integer.MAX_VALUE :
               MAX_ARRAY_SIZE;
       }
   ```

   



#### 3.3: Collection接口 之 Set

##### 3.3.1 comparable 和 Comparator 的区别

- `comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序

  - 继承comparable<Object>接口,重写compare方法

- `comparator`接口实际上是出自 java.util 包它有一个`compare(Object obj1, Object obj2)`方法用来排序

  ```java
          // 定制排序的用法
          Collections.sort(arrayList, new Comparator<Integer>() {
  
              @Override
              public int compare(Integer o1, Integer o2) {
                  return o2.compareTo(o1);
              }
          });
  ```

##### 3.3.2 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

- `HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 null 值；
- `LinkedHashSet` 是 `HashSet` 的子类，能够按照添加的顺序遍历；
- `TreeSet` 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。



#### 3.4: Map接口

##### 3.4.1: HashMap和Hashtable的区别

|                        | HashMap                                                      | Hashtable                                                    |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **是否线程安全**       | 否                                                           | 是                                                           |
| **效率**               | 高                                                           | 低                                                           |
| **null key/value**     | 可以存储null键和null值,                                      | 不允许                                                       |
| **初始容量和扩充容量** | 默认初始化大小为16,之后每次扩充容量变为原来的两倍            | 创建时,如果不指定容量初始值,Hashtable默认的初始大小为11,之后每次扩充,容量变为原来的2n + 1 |
| **底层数据结构**       | 当链表长度大于阈值(默认为8)时,将链表转化为红黑树,以减少搜索时间,(将链表转化成红黑树之前,如果当前数组长度小于64,会先将数组扩容,而不是直接转换成红黑树) | 和JDK1.7及以前的HashMap底层数据结构类似采用数组 + 链表       |



**`HashMap` 中带有初始容量的构造函数**

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity); 🍉
    }
     public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

//下面这个方法保证了 HashMap 总是使用 2 的幂作为哈希表的大小。

    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

```



##### 3.4.2: HashMap和HashSet的区别

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

| `HashMap`                              | `HashSet`                                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| 实现了 `Map` 接口                      | 实现 `Set` 接口                                              |
| 存储键值对                             | 仅存储对象                                                   |
| 调用 `put()`向 map 中添加元素          | 调用 `add()`方法向 `Set` 中添加元素                          |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |



##### 3.4.3: HashMap 和 TreeMap的区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

<img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2020%201635391580%201635391580107%20AxxeEE%20image-20210805091937200.png" alt="image-20210805091937200" style="zoom:50%;float:left" />

实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。

实现`SortMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。



##### 3.4.4: HashMap底层实现

###### 3.4.4.1: JDK1.7及以前

HashMap底层是**数组和链表**结合在一起使用也就是**散列链表**,HashMap通过key的hashcode经过扰动函数处理后得到hash值,然后通过      (n - 1) & hash 判断当前元素存放的位置(这里n为数组长度),如果当前位置存在元素,就判断该元素与要存入元素的hash以及key是否相同,相同就直接覆盖,不相同就采用拉链法解决冲突;

**扰动函数:** 指HashMap的hash方法,使用扰动函数是为了防止一些实现比较差的hashcode()方法,换句话说就是使用扰动函数可以减少碰撞.

1.8 hash方法 (相较于1.7更加简化,原理不变)

```java
    static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

1.7 hash方法 (扰动4次)

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```



###### 3.4.4.2: JDK1.8开始

`1.8开始,解决hash冲突有了较大的变化.`

**当链表长度大于阈值(默认为8)时,将链表转化为红黑树,以减少搜索时间,(将链表转化成红黑树之前,如果当前数组长度小于64,会先将数组扩容,而不是直接转换成红黑树)**

<img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2020%201635391580%201635391580924%209TNtcW%20image-20210805100240587.png" alt="image-20210805100240587" style="zoom:30%;float:left" />

> TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。



##### 3.4.5: HashMap的长度为什么是2的幂次方

在得到hash值之后(hash值区间-2147483648 到 2147483647),需要对hash值根据数组长度取模运算得到存放的下标位置,计算方法为       (n - 1) & hash . 采取与算法的是比%更高效的,但是只有长度为2的幂次方时, (n - 1) & hash == hash % n



##### 3.4.6: HashMap在多线程下操作为什么不安全? (导致死循环

- **HashMap的 rehash**

  向hashmap中 put()新元素时,如果元素先前不存在,会检查容量是否超过阈值threshold.如果超过了,就会进行的resize()操作,因此会建一个两倍table.length的hash表,将老的数据迁移过去.

  **在多线程的情况下,迁移操作可能会导致元素之间形成一个循环链表.**

  虽然在jdk1.8以后解决了该问题,但还是不建议多线程下使用hashmap



##### [3.4.7: HashMap有哪几种常见的遍历方式?](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw) 



##### 3.4.8: ConcurrentHashMap和Hashtable的区别

|                        | ConcurrentHashMap                                            | Hashtable                                            |
| ---------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| **底层数据结构**       | **JDK1.7:** 底层采用**分段的数组 + 链表**<br />**JDK1.8:** 采和HashMap1.8一样采用 **数组 +  链表/红黑二叉树** | 和JDK1.7及以前的HashMap一致采用**数组+链表**         |
| **实现线程安全的方式** | **JDK1.7:** 采用**分段锁**对整个桶数组进行**分割分段(Segment)**,每一把锁中锁容器中一部分数据,多线程访问容器不同数据段,就不会存在锁竞争,提高了并发访问率;<br />**JDK1.8:** 摒弃了Segment概念,直接使用**Node数组 + 链表/红黑树**,并发控制使用**synchronized 和CAS**来操作.虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本； | (**同一把锁**),使用synchronized保证线程安全,效率低下 |

- **对比图**

  - **Hashtable**

    <img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2022%201635391582%201635391582014%20l5rTgA%20image-20210805145657746.png" alt="image-20210805145657746" style="zoom:50%;float:left" />

  - **JDK1.7 ConcurrentHashMap**

    <img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2024%201635391584%201635391584358%200TYzZA%20image-20210805145822194.png" alt="image-20210805145822194" style="zoom:50%;float:left" />

  - **JDK1.8 ConcurrentHashMap**

    <img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2025%201635391585%201635391585569%20OvEVKH%20image-20210805150140857.png" alt="image-20210805150140857" style="zoom:50%;float:left" />

    JDK1.8 的 `ConcurrentHashMap` 不在是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。



##### 3.4.9: ConcurrentHashMap线程安全的具体实现方式/底层实现方式

###### JDK: 1.7

首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。

Segment 实现了 `ReentrantLock`,所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

```JAVA
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```



###### JDK: 1.8

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）

`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。







### P4: Lambda表达式

#### 4.1: Lambda表达式的简介

##### 4.1.1: Lambda表达式的概念

- lambda表达式， 是Java8的一个新特性， 也是Java8中最值得学习的新特性之一。
- lambda表达式， 从本质来讲， 是一个匿名函数。 可以使用使用这个匿名函数， 实现接口中的方法。
  对接口进行非常简洁的实现， 从而简化代码。

##### 4.1.2: Lambda表达式的使用场景 

通常来讲， 使用lambda表达式， 是为了简化接口实现的。
关于接口实现， 可以有很多种方式来实现。 例如: 设计接口的实现类、 使用匿名内部类。 但是 lambda表达式， 比这两种方式都简单。

```java
public class Program {
  public static void main(String[] args) {
// 无参、无返回值的函数式接口
      interfaceImpl();
  }
private static void interfaceImpl() {
// 1. 使用显式的实现类对象
SingleReturnSingleParameter parameter1 = new Impl(); 
// 2. 使用匿名内部类实现
SingleReturnSingleParameter parameter2 = new SingleReturnSingleParameter() {
          @Overridepublic int test(int a) {
                return a * a;
} };
// 3. 使用lambda表达式实现
SingleReturnSingleParameter parameter3 = a -> a * a;
        System.out.println(parameter1.test(10));
        System.out.println(parameter2.test(10));
        System.out.println(parameter3.test(10));
    }
private static class Impl implements SingleReturnSingleParameter {
				@Override
        public int test(int a) {
            return a * a;
} }
}
```



```java
void process(int a[], int n)
{   
  int low = 0, high = n-1;
 	while(low < high){
		while(low < high && a[low] < 0)
			++low;
		while(low < high && a[high] > 0)
			--high;
		if(low < high){
      int tmp = a[low];
      a[low] = a[high];
      a[high] = tmp;
      low++;
      high--;
    }
}
```



##### 4.1.3: Lambda表达式对接口的要求

- 虽然说， lambda表达式可以在一定程度上简化接口的实现。 但是， 并不是所有的接口都可以使用
  lambda表达式来简洁实现的。

- lambda表达式毕竟只是一个匿名方法。 当实现的接口中的方法过多或者多少的时候， lambda表达式
  都是不适用的。

- lambda表达式，只能实现函数式接口。

  

##### 4.1.4: 函数式接口 

###### 4.1.4.1: 基础概念

如果说,一个接口中,要求实现类必须实现的抽象方法,有且只有一个! 这样的接口,就是函数式接口。

```java
// 这个接口中， 有且只有一个方法， 是实现类必须实现的， 因此是一个函数式接口 
interface Test1 {
    void test();
}
// 这个接口中， 实现类必须要实现的方法， 有两个! 因此不是一个函数式接口 
interface Test2 {
    void test1();
    void test2();
}
// 这个接口中， 实现类必须要实现的方法， 有零个! 因此不是一个函数式接口 
interface Test3 {
}
// 这个接口中， 虽然没有定义任何的方法， 但是可以从父接口中继承到一个抽象方法的。 是一个函数式 接口
interface Test4 extends Test1 {
}
// 这个接口， 虽然里面定义了两个方法， 但是defualt方法子类不是必须实现的。
// 因此， 实现类实现这个接口的时候， 必须实现的方法只有一个! 是一个函数式接口。 
interface Test5 {
    void test5();
    default void test() {}
}
// 这个接口中的 toString 方法， 是Object类中定义的方法。
// 此时， 实现类在实现接口的时候， toString可以不重写的! 因为可以从父类Object中继承到! 
// 此时， 实现类在实现接口的时候， 有且只有一个方法是必须要重写的。 是一个函数式接口! 
interface Test6 {
    void test6();
    String toString();
}
```

 思考题: 下面的两个接口是不是函数式接口?

```java
interface Test7 {
	String toString();
} 
//不是


interface Test8 {
	void test();
	default void test1() {}
	static void test2() {}
	String toString();
}
//是
```

###### 4.1.4.2:  @FunctionalInterface

是一个注解， **用在接口之前， 判断这个接口是否是一个函数式接口。** 如果不是就会报错,如果是就不会有什么问题, 功能类似于 @Override。

```JAVA
@FunctionalInterface
interface FunctionalInterfaceTest {
    void test();
}
```

###### 4.1.4.3: 系统内置的若干函数式接口

<img src="https://gitee.com/breeze1002/upic/raw/master/JAVASE/JAVASE/2021%2010%2028%2011%2026%2026%201635391586%201635391586316%20DiizVF%20image-20210420150656784.png" alt="image-20210420150656784" style="zoom:50%;float:left" />


#### 4.2: Lambda表达式的语法 

##### 4.2.1: Lambda表达式的基础语法

- lambda表达式， 其实本质来讲， 就是一个匿名函数。 因此在写lambda表达式的时候， 不需要关心方法名是什么。
- 实际上， 我们在写lambda表达式的时候， 也不需要关心返回值类型。
- 写lambda表达式的时,只需要关注两部分内容即可: 参数列表 和 方法体

**lambda表达式的基础语法:**

```java
(参数) -> { 
  	方法体
};
```

- **参数部分** : 方法的参数列表， 要求和实现的接口中的方法参数部分一致， 包括参数的数量和类型。
- **方法体部分** : 方法的实现部分， 如果接口中定义的方法有返回值， 则在实现的时候， 注意返回值的返回。
- **->** : 分隔参数部分和方法体部分。


```java
public class Syntax {
    public static void main(String[] args) {
			// 1. 无参、无返回值的方法实现 
    	NoneReturnNoneParameter lambda1 = () -> {
					System.out.println("无参、无返回值方法的实现");
      };
      lambda1.test();
      
			// 2. 有参、无返回值的方法实现 
      NoneReturnSingleParameter lambda2 = (int a) -> {
					System.out.println("一个参数、无返回值方法的实现: 参数是 " + a); 
      };
      lambda2.test(10);

      // 3. 多个参数、无返回值方法的实现
			NoneReturnMutipleParameter lambda3 = (int a, int b) -> {
					System.out.println("多个参数、无返回值方法的实现: 参数a是 " + a + ", 参数b是 " + b); 
      };
			lambda3.test(10, 20);
      
       // 4. 无参、有返回值的方法的实现 
      SingleReturnNoneParameter lambda4 = () -> {
					System.out.println("无参、有返回值方法的实现");
    			return 666;
			};
			System.out.println(lambda4.test());

      // 5. 一个参数、有返回值的方法实现 
      SingleReturnSingleParameter lambda5 = (int a) -> {
  				System.out.println("一个参数、有返回值的方法实现: 参数是 " + a);
    			return a * a;
			};
			System.out.println(lambda5.test(9));

      // 6. 多个参数、有返回值的方法实现
			SingleReturnMutipleParameter lambda6 = (int a, int b) -> {
					System.out.println("多个参数、有返回值的方法实现: 参数a是 " + a + ", 参数b return a * b;
			};
        System.out.println(lambda6.test(10, 20));
    }
}
```



##### 4.2.2: Lambda表达式的语法进阶

​	在上述代码中， 的确可以使用lambda表达式实现接口， 但是依然不够简洁， 有简化的空间。

###### 4.2.2.1: 参数部分的精简 

- **参数的类型**

  - 由于在接口的方法中，已经定义了每一个参数的类型是什么。 而且在使用lambda表达式实 现接口的时候， 必须要保证参数的数量和类型需要和接口中的方法保持一致。 因此， 此时 lambda表达式中的参数的类型可以省略不写。

  - 注意事项:

    - 如果需要省略参数的类型,要保证: 要省略,每一个参数的类型都必须省略不写.绝对不能出现,有的参数类型省略了,有的参数类型没有省略。

    - ```JAVA
      // 多个参数、无返回值的方法实现 
      NoneReturnMutipleParameter lambda1 = (a, b) -> {
      	System.out.println("多个参数、无返回值方法的实现: 参数a是 " + a + ", 参数 b是 "+b);
      };
      ```

- **参数的小括号**

  - 如果方法的参数列表中的参数数量 有且只有一个 ，此时，参数列表的小括号是可以省略不写 的。

  - 注意事项:

    - 只有当参数的数量是一个的时候， 多了、少了都不能省略。 

    - 省略掉小括号的同时， 必须要省略参数的类型。

    - ```java
      // 有参、无返回值的方法实现 
      oneReturnSingleParameter lambda2 = a -> {
      		System.out.println("一个参数、无返回值方法的实现: 参数是 " + a); 
      };
      ```

    

###### 4.2.2.2:方法体部分的精简 

- **方法体大括号的精简**

  - 当一个方法体中的逻辑， 有且只有一句的情况下， 大括号可以省略。

  - ```java
    // 有参、无返回值的方法实现
    NoneReturnSingleParameter lambda2 = a -> System.out.println("一个参数、无 返回值方法的实现: 参数是 " + a);
    ```

- **return的精简**

  - 如果一个方法中唯一的一条语句是一个返回语句,此时在省略掉大括号的同时,也必须省略掉return.

  - ```java
    SingleReturnMutipleParameter lambda3 = (a, b) -> a + b;
    ```

    

#### 4.3: 函数引用

- lambda表达式是为了简化接口的实现的。 在lambda表达式中， 不应该出现比较复杂的逻辑。 如果在 lambda表达式中出现了过于复杂的逻辑， 会对程序的可读性造成非常大的影响。 如果在lambda表达 式中需要处理的逻辑比较复杂， **一般情况会单独的写一个方法。 在lambda表达式中直接引用这个方法即可。**

> 或者， 在有些情况下， 我们需要在lambda表达式中实现的逻辑， 在另外一个地方已经写好了。 此时我们就不需要再单独写一遍， 只需要直接引用这个已经存在的方法即可。

- **函数引用**: 引用一个已经存在的方法， 使其替代lambda表达式完成接口的实现。



#####  4.3.1: 静态方法的引用

- **语法:**

  - 类::静态方法 

- 注意事项:

  - 在引用的方法后面， 不要添加小括号。

  - 引用的这个方法， 参数(数量、类型) 和 返回值， 必须要跟接口中定义的一致。

  - 示例:

  - ```java
    public class Syntax1 { 
      // 静态方法的引用
    	public static void main(String[] args) {
    			// 实现一个多个参数的、一个返回值的接口
    			// 对一个静态方法的引用
    			// 类::静态方法
    			SingleReturnMutipleParameter lambda1 = Calculator::calculate;
        	System.out.println(lambda1.test(10, 20));
    }
      private static class Calculator {
            public static int calculate(int a, int b) {
    					// 稍微复杂的逻辑:计算a和b的差值的绝对值 
              if (a > b) {
                return a - b; 
              }
    					return b - a; 
            }
      }
    }
    ```



##### 4.3.2: 非静态方法的引用

- **语法:**

  - 对象::非静态方法 

  - 注意事项:

    - 在引用的方法后面， 不要添加小括号。
    - 引用的这个方法， 参数(数量、类型) 和 返回值， 必须要跟接口中定义的一致。

  - 示例:

    ```java
    public class Syntax2 {
        public static void main(String[] args) {
    				// 对非静态方法的引用，需要使用对象来完成
            SingleReturnMutipleParameter lambda = new Calculator()::calculate;
            System.out.println(lambda.test(10, 30));
        }
        private static class Calculator {
            public int calculate(int a, int b) {
        			return a > b ? a - b : b - a;
            }
        }  
    }
    ```

    

#####  4.4.3: 构造方法的引用 

- **使用场景**

  - 如果某一个函数式接口中定义的方法， 仅仅是为了得到一个类的对象。 此时我们就可以使用
    构造方法的引用， 简化这个方法的实现。

- **语法**

  - 类名::new 

- **注意事项**

  - 可以通过接口中的方法的参数， 区分引用不同的构造方法。

- **示例**

  ```java
  public class Syntax3 {
      private static class Person {
          String name;
          int age;
          public Person() {
  					System.out.println("一个Person对象被实例化了");
          }
  				public Person(String name, int age) { 
            System.out.println("一个Person对象被有参的实例化了"); 
            this.name = name;
  					this.age = age;
  				} 
      }
      
    	@FunctionalInterface
  		private interface GetPerson {
  			// 仅仅是希望获取到一个Person对象作为返回值 
        Person test();
  		}
    
      private interface GetPersonWithParameter {
          Person test(String name, int age);
  		}
    
  		public static void main(String[] args) { 
        // lambda表达式实现接口
  			GetPerson lambda = Person::new;
  			// 引用到Person类中的无参构造方法，获取到一个Person对象
  			Person person = lambda.test();
        // 引用到Person类 中的有参构造方法，获取到一个Person对象
  			GetPersonWithParameter lambda2 = Person::new; 
        lambda2.test("xiaoming", 1);
      }
  }
  ```

  

##### 4.3.4: 对象方法的特殊引用

如果在使用lambda表达式，实现某些接口的时候。 lambda表达式中包含了某一个对象，此时方法体中， 直接使用这个对象调用它的某一个方法就可以完成整体的逻辑。其他的参数，可以作为调用方法 的参数。此时,可以对这种实现进行简化。

```java
public class Syntax {
    public static void main(String[] args) {
			// 如果对于这个方法的实现逻辑，是为了获取到对象的名字 
      GetField field = person -> person.getName(); 
      // 对于对象方法的特殊引用
			GetField field = Person::getName;
      
			// 如果对于这个方法的实现逻辑，是为了给对象的某些属性进行赋值 
      SetField lambda = (person, name) -> person.setName(name); 
      SetField lambda = Person::setName;

      // 如果对于这个方法的实现逻辑，正好是参数对象的某一个方法 
      ShowTest lambda2 = person -> person.show(); 
      ShowTest lambda2 = Person::show;

      
interface ShowTest {
    void test(Person person);
}
interface SetField {
    void set(Person person, String name);
}
interface GetField {
    String get(Person person);
}
class Person {
    private String name;
    public void setName(String name) {
        this.name = name;
}
    public String getName() {
        return name;
}
    public void show() {
} }
```





#### 4.4: Lambda表达式需要注意的问题 

这里类似于局部内部类、匿名内部类，依然存在闭包的问题。

如果在lambda表达式中,使用到了局部变量,那么这个局部变量会被隐式的声明为 final.是一个常量,不能修改值。





#### 4.5: Lambda表达式的实例 

##### 4.5.1: 线程的实例化

```java
Thread thread = new Thread(() -> { 
  	// 线程中的处理
});
```

##### 4.5.2: 集合的常见方法

```java
ArrayList<String> list = new ArrayList<>(); 
Collections.addAll(list, "千锋", "大数据", "好程序员", "严选", "高薪");

// 按照条件进行删除
list.removeIf(ele -> ele.endsWith(".m")); 
// 批量替换
list.replaceAll(ele -> ele.concat("!")); 
// 自定义排序
list.sort((e1, e2) -> e2.compareTo(e1)); // 遍历
list.forEach(System.out::println);
```

##### 4.5.3:  集合的流式编程

```java
ArrayList<String> list = new ArrayList<>(); 
Collections.addAll(list, "千锋", "大数据", "好程序员", "严选", "高薪");

list.parallelStream().filter(ele -> ele.length() >
2).forEach(System.out::println);
```



### P5: Streams

`java.util.Strem` 表示能应用在一组元素上的一次执行的操作序列；

Stream操作可以分位 **中间操作** 和 **最终操作** 两种；

- **最终操作** 返回一个特定类型的计算结果
- **中间操作** 返回Stream本身，因此可以将多个操作依次串起来

Stream的创建需要指定一个数据源，例如：java.util.Collection的子类，List或者Set , 不支持Map;

Stream的操作可以串行或者并行执行；

```java
package com.breeze2495.learn.streams;

import com.sun.org.apache.xerces.internal.xs.StringList;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * @author breeze
 * @date 2021/8/26 3:50 下午
 */
public class StreamsTest {
    public static void main(String[] args) {

        List<String> stringList = new ArrayList<>();
        stringList.add("ddd2");
        stringList.add("aaa2");
        stringList.add("bbb1");
        stringList.add("aaa1");
        stringList.add("bbb3");
        stringList.add("ccc");
        stringList.add("bbb2");
        stringList.add("ddd1");


        //Java 8扩展了集合类，可以通过 Collection.stream() 或者 Collection.parallelStream() 来创建一个Stream。

        /**
         * filter  过滤
         *
         * 过滤通过一个predicate接口规则来过滤并只保留符合条件的元素，该操作属于中间操作，
         * 所以我们可以在过滤后的结果来应用其他Stream操作（比如forEach）。
         * forEach需要一个函数来对过滤后的元素依次执行。forEach是一个最终操作，所以我们不能在forEach之后来执行其他Stream操作。
         *
         * forEach 是为 Lambda 而设计的，保持了最紧凑的风格。而且 Lambda 表达式本身是可以重用的，非常方便。
         */
        stringList
                .stream()
                .filter( s -> s.endsWith("2"))
                .forEach(System.out::println);


        System.out.println("--------------------------------------------");

        /**
         * sorted 排序
         *
         * 排序是一个 中间操作，返回的是排序好后的 Stream。如果你不指定一个自定义的 Comparator 则会使用默认排序。
         *
         * 需要注意的是，排序只创建了一个排列好后的Stream，而不会影响原有的数据源，排序之后原数据stringCollection是不会被修改的：
         */
        stringList
                .stream()
                .sorted((a,b) -> (b.compareTo(a)))
                .forEach(System.out::println);

        System.out.println("--------------------------------------------");

        /**
         * map 映射
         *
         * 中间操作 map 会将元素根据指定的 Function 接口来依次将元素转成另外的对象。
         *
         * 下面的示例展示了将字符串转换为大写字符串。你也可以通过map来将对象转换成其他类型，
         * map返回的Stream类型是根据你map传递进去的函数的返回值决定的。
         */
        stringList
                .stream()
                .map(String::toUpperCase)
                .sorted()
                .forEach(System.out::println);

        System.out.println("--------------------------------------------");

        /**
         * match 匹配
         *
         * Stream提供了多种匹配操作，允许检测指定的predicate接口规则是否匹配整个stream
         * 所有的匹配操作都是最终操作，最终会返回一个boolean值
         *
         */
        boolean anyStartWithA =
            stringList
                    .stream()
                    .anyMatch(a -> a.startsWith("a"));
        System.out.println(anyStartWithA);

        boolean allStartWithA =
            stringList
                    .stream()
                    .allMatch(a -> a.startsWith("a"));
        System.out.println(allStartWithA);

        boolean noneStartWithA =
            stringList
                    .stream()
                    .noneMatch(a -> a.startsWith("a"));
        System.out.println(noneStartWithA);


        System.out.println("--------------------------------------------");


        /**
         * count 计数
         *
         * 计数是一个 最终操作，返回Stream中元素的个数，返回值类型是 long。
         */
        long startsWithB =
                stringList
                        .stream()
                        .filter((s) -> s.startsWith("b"))
                        .count();
        System.out.println(startsWithB);

        System.out.println("--------------------------------------------");


        /**
         * reduce 规约
         *
         * 这是一个 最终操作 ，允许通过指定的函数来将stream中的多个元素规约为一个元素，规约后的结果是通过Optional 接口表示的
         *
         */

        Optional<String> reduce = stringList
                .stream()
                .sorted()
                .reduce((s1, s2) -> s1 + " - " + s2);
        reduce.ifPresent(System.out::println);

        /**
         * 这个方法的主要作用是把 Stream 元素组合起来。
         * 它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。
         * 从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。
         * 例如 Stream 的 sum 就相当于Integer sum = integers.reduce(0, (a, b) -> a+b);
         * 也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。
         */
        
    }
}
```



**Parallel Streams** 并行流

前面提到过Stream有串行和并行两种，串行Stream上的操作是在一个线程中依次完成，而并行Stream则是在多个线程上同时执行。

```JAVA
/** 
下面的例子展示了是如何通过并行Stream来提升性能：
首先我们创建一个没有重复元素的大表：
*/

int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}

//我们分别用串行和并行两种方式对其进行排序，最后看看所用时间的对比。

//Sequential Sort 串行

long t0 = System.nanoTime();

long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));

//打印 
1000000
sequential sort took: 709 ms//串行排序所用的时间

//Parallel Sort 并行

long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

//打印
1000000
parallel sort took: 475 ms//串行排序所用的时间

/**
上面两个代码几乎是一样的，但是并行版的快了 50% 左右，唯一需要做的改动就是将 stream() 改为parallelStream()。
*/

```



















































