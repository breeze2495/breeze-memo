## P1: Lambda表达式的简介

### p1: Lambda表达式的概念

- lambda表达式， 是Java8的一个新特性， 也是Java8中最值得学习的新特性之一。
- lambda表达式， 从本质来讲， 是一个匿名函数。 可以使用使用这个匿名函数， 实现接口中的方法。
  对接口进行非常简洁的实现， 从而简化代码。

### p2: Lambda表达式的使用场景 

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



### p3: Lambda表达式对接口的要求

- 虽然说， lambda表达式可以在一定程度上简化接口的实现。 但是， 并不是所有的接口都可以使用
  lambda表达式来简洁实现的。

- lambda表达式毕竟只是一个匿名方法。 当实现的接口中的方法过多或者多少的时候， lambda表达式
  都是不适用的。

-  lambda表达式，只能实现函数式接口。

  

### p4: 函数式接口 

#### D1: 基础概念

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

#### D2:  @FunctionalInterface

是一个注解， **用在接口之前， 判断这个接口是否是一个函数式接口。** 如果不是就会报错,如果是就不会有什么问题, 功能类似于 @Override。

```JAVA
@FunctionalInterface
interface FunctionalInterfaceTest {
    void test();
}
```

#### D3: 系统内置的若干函数式接口

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20210420150656784.png" alt="image-20210420150656784" style="zoom:50%;float:left" />


## P2: Lambda表达式的语法 

### p1: Lambda表达式的基础语法

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



### p2: Lambda表达式的语法进阶

​	在上述代码中， 的确可以使用lambda表达式实现接口， 但是依然不够简洁， 有简化的空间。

#### D1: 参数部分的精简 

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

    

#### D2:方法体部分的精简 

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

    

## P3: 函数引用

- lambda表达式是为了简化接口的实现的。 在lambda表达式中， 不应该出现比较复杂的逻辑。 如果在 lambda表达式中出现了过于复杂的逻辑， 会对程序的可读性造成非常大的影响。 如果在lambda表达 式中需要处理的逻辑比较复杂， **一般情况会单独的写一个方法。 在lambda表达式中直接引用这个方法即可。**

> 或者， 在有些情况下， 我们需要在lambda表达式中实现的逻辑， 在另外一个地方已经写好了。 此时我们就不需要再单独写一遍， 只需要直接引用这个已经存在的方法即可。

- **函数引用**: 引用一个已经存在的方法， 使其替代lambda表达式完成接口的实现。



###  p1: 静态方法的引用

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



### p2: 非静态方法的引用

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

    

###  p3: 构造方法的引用 

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

  

### p4: 对象方法的特殊引用

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





## P4: Lambda表达式需要注意的问题 

这里类似于局部内部类、匿名内部类，依然存在闭包的问题。

如果在lambda表达式中,使用到了局部变量,那么这个局部变量会被隐式的声明为 final.是一个常量,不能修改值。





## P5: Lambda表达式的实例 

### p1: 线程的实例化

```java
Thread thread = new Thread(() -> { 
  	// 线程中的处理
});
```

### p2: 集合的常见方法

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

### p3:  集合的流式编程

```java
ArrayList<String> list = new ArrayList<>(); 
Collections.addAll(list, "千锋", "大数据", "好程序员", "严选", "高薪");

list.parallelStream().filter(ele -> ele.length() >
2).forEach(System.out::println);
```

