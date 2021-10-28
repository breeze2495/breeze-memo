

## D1: 概述

### P1: 概述

1. Spring是一个开源框架

2. Spring为简化企业级开发而生，使用Spring，JavaBean就可以实现很多以前要靠EJB才能实现的功能。同样的功能，在EJB中要通过繁琐的配置和复杂的代码才能够实现，而在Spring中却非常的优雅和简洁.

3. Spring是一个**IOC**(DI)和**AOP**容器框架。

4. Spring的优良特性

   [1]**非侵入式**：基于Spring开发的应用中的对象可以不依赖于Spring的API

   [2]**依赖注入**：DI——Dependency Injection，反转控制(IOC)最经典的实现。

   [3]**面向切面编程**：Aspect Oriented Programming——AOP

   [4]**容器**：Spring是一个容器，因为它包含并且管理应用对象的生命周期

   [5]**组件化**：Spring实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。

   [6]**一站式**：在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上Spring 自身也提供了表述层的SpringMVC和持久层的Spring JDBC）。

### P2: Spring模块划分

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2028%201635402088%201635402088673%20tAaXr1%20image-20210714234336602.png" alt="image-20210714234336602" style="zoom:50%;float:left" />

- **核心容器：**

  - beans
  - context
  - core
  - spel(expression)：spring expression language：spring表达式语言

  ```java
  spring-beans-4.0.0.RELEASE.jar
  spring-context-4.0.0.RELEASE.jar
  spring-core-4.0.0.RELEASE.jar
  spring-expression-4.0.0.RELEASE.jar
  ```

  

## D2: IOC容器和bean的配置

### P1: IOC 和 DI：

#### d1:IOC

`(Inversion of Control):控制翻转`

* 何为控制：资源获取方式；
    * 主动式：需要什么资源自己创建什么资源；

        ```java
         BookServlet {
         
              BookService bs = new BookService();//简单对象创建
        			AirPlane ap = new AirPlane();//对于复杂对象的创建是比较庞大的工程
        
        }
        ```

    * 被动式:

        ```java
        @AutoWired
        BookService bs ;
        
        public void test1(){
        
        	bs.method();
        
        }
        ```

- 容器
  - 容器其实就是一个map,其中保存了所有创建好的bean,并提供外界获取功能
  - 管理所有的组件（有功能的类）；
  - BookServlet受容器管理，BookService也受容器管理。容器可以自动探查出那些组件（类）需要用到的另一些组件；容器帮我们创建BookService对象，并把BookService对象赋值过去；

* **控制反转**:主动的从容器中获取资源变成由容器主动的将资源推送给需要的组件;



#### d2: DI：

`（dependency injection）依赖注入`

- **概念:** 把有依赖关系的类放入 (ioc)容器 中,然后解析出这个类的实例;



### P2: Bean的作用域（scope）

* **singleton（默认）**: 单实例
    * 在容器启动完成之前就已经创建好对象，保存在容器中了；
    * 任何获取都是获取之前创建好的那个对象；
* **prototype**: 多实例
    * 容器启动默认不会去创建多实例bean；
    * 获取的时候创建这个bean；
    * 每次获取都会创建一个新的对象；
* **request: **在web环境下：同一次请求创建一个Bean实例（没用）；
* **session**: 在web环境下：同一次会话创建一个Bean实例（没用）；



### P3: Bean的生命周期

* **单例：**
    1. 通过构造器或工厂方法创建bean实例
    2. 为bean的属性设置值和对其他bean的引用
    3. 调用bean的初始化方法
    4. bean就可以使用了
    5. 容器关闭,调用bean的销毁方法
* **多实例:**
    * 获取Bean（构造器----->初始化方法）----->容器关闭不会调用bean的销毁方法

### P4: Bean的后置处理器

`在初始化方法前后对bean进行额外处理`

* 接口（BeanPostProcessor）
    * 实现类
        * postProcessBeforeInitialization()
        * postProcessAfterIntialization()
* 单例：
    * （容器启动）构造器--后置before-->初始化方法--后置after-->(容器关闭)销毁方法
* 无论bean是否有初始化方法，后置处理器都会默认其有，还会继续工作。



### P5: 通过注解配置bean

* 通过给bean添加某些注解，可以快速的将bean加入到ioc容器中
* 某个类添加上任何一个注解都能快速的将这个组件加入到ioc容器的管理中；
    * Spring有四个注解：
        * @Controller：控制器
        * @Service：业务逻辑层
        * @Repository：持久化层/dao层
        * @Component：给不属于以上基层的组件添加这个注解：
    * 注解可以随便加，spring底层不会去验证是否你所注解的是否就是一个dao service；只是推荐加，方便程序员看

* 使用注解将组件快速的加入到容器中需要如下几步：
    * 标注注解；
    * 告诉spring，自动扫描添加了注解的组件：依赖context名称空间
    * 一定要导入aop包，支持加注解模式；
* 组件的id和作用域配置：
    * id：
        * 默认为自检的类名首字母小写; 自定义:@Repository{"bookdaoxixi"}
    * 组件作用域:
        * 默认为单例; 自定义:  @Scope(value="prototype")

* * * 

### P6: @Autowired

- **根据类型实现自动装配**
- 构造器,普通字段(即使是非public),一切具有参数的方法都可以应用该注解
- 默认情况下，所有使用@Autowired注解的属性都需要被设置。当Spring找不到匹配的bean装配属性时，会抛出异常。
- 若某一属性允许不被设置，可以设置@Autowired注解的required属性为 false
- 默认情况下，当IOC容器里存在多个类型兼容的bean时，Spring会尝试匹配bean的id值是否与变量名相同，如果相同则进行装配。如果bean的id值不相同，通过类型的自动装配将无法工作。此时可以在@Qualifier注解里提供bean的名称。Spring甚至允许在方法的形参上标注@Qualifiter注解以指定注入bean的名称。
- @Autowired注解也可以应用在数组类型的属性上，此时Spring将会把所有匹配的bean进行自动装配。
- @Autowired注解也可以应用在集合属性上，此时Spring读取该集合的类型信息，然后自动装配所有与之兼容的bean。
- @Autowired注解用在java.util.Map上时，若该Map的键值为String，那么 Spring将自动装配与值类型兼容的bean作为值，并以bean的id值作为键。

- **@Resource**
  - @Resource注解要求提供一个bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为bean的名称。

- **@Inject**
  - @Inject和@Autowired注解一样也是按类型注入匹配的bean，但没有reqired属性。

- @Autowired @Resource @Inject(EJB) 都是自动装配的意思
  - @Autowired:最强大; 是spring自己的注解
  - @Resource: j2EE;java的标准
  - 两者区别:
    - @Autowired脱离spring就不能使用了,所以@Resource扩展性更强




* **源码分析:**
    * IOC:
        * IOC是一个容器;
        * 容器启动的时候创建所有单实例对象;
        * 我们可以直接从容器中获取这个对象
    * springIOC:
        * ioc容器的启动过程?启动期间都做了什么?(什么时候创建所有单实例bean);
        * ioc是如何创建这些单实例bean,并如何管理的?到底保存在了那里?
* **思路:**

* **BeanFactory和ApplicationContext的区别:**
    * 口头回答:
        * BeanFactory是最底层的接口,ApplicationContext里给程序员使用的ioc容器接口;
        * ApplicationContext是BeanFactory的子接口;
    * ApplicationContext是BeanFactory的子接口;
        * BeanFactory:bean工厂接口;负责创建bean实例;容器里面保存的所有单例bean其实是一个map;spring最底层的接口;
        * ApplicationContext:是容器接口;更多的负责容器功能的实现;(可以基于beanfactory创建好的对象之上完成强大的容器),容器可以从map获取这个bean,在ApplicationContext接口下的这些类里面,实现AOP,DI;
    * Spring里面最大的模式就是工厂模式
        * Beanfactory:bean工厂;工厂模式;帮用户创建bean



## D3: AOP 

### P1: 概述

* **Aspect Oriented Programming  面向切面编程:**
  * 概念:
    * 基于OOP基础之上的新的编程思想;
    * 指在程序运行期间,将某段代码动态的切入到指定方法的指定位置进行运行的编程方式
  * 场景:
    * 计算器运行计算方法的时候进行日志记录;
      * 原始方法:
        * 直接写在方法的内部(不推荐,修改维护麻烦,耦合度高);
      * 改进:
        * 业务逻辑:(核心功能);日志模块能够在核心功能运行期间,自己动态加上;
        * 可以使用动态代理类将日志代码动态的在目标方法执行前后执行;

```java
/**
 * 帮Calculator.java生成代理对象的类
 * Object newProxyInstance
 * (ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
 */
public class CalculatorProxy 
    /**
     * 为传入的参数对象创建一个动态代理对象
     * @param calculator
     * @return
     *
     * Calculator calculator:被代理对象；（宝宝）
     * 返回的：宋喆
     */
    public static Calculator getProxy(final Calculator calculator) {
        //方法执行器。帮我们目标对象执行目标方法
        InvocationHandler h = new InvocationHandler() {
            /**
             * Object proxy：代理对象；给jdk使用，任何时候都不要动这个对象
             * Method method：当前将要执行的目标对象的方法
             * Object[] args：这个方法调用时外界传入的参数值
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args)throws Throwable {
              
                //System.out.println("这是动态代理将要帮你执行方法...");
                Object result = null;
                try {
                    LogUtils.logStart(method, args);
                    // 利用反射执行目标方法
                    //目标方法执行后的返回值
                    result = method.invoke(calculator, args);
                    LogUtils.logReturn(method, result);
                } catch (Exception e) {
                    LogUtils.logException(method,e);
                }finally{
                    LogUtils.logEnd(method);
                }
                //返回值必须返回出去外界才能拿到真正执行后的返回值
                return result;
            }
        };
        Class<?>[] interfaces = calculator.getClass().getInterfaces();
        ClassLoader loader = calculator.getClass().getClassLoader();

        //Proxy为目标对象创建代理对象；
        Object proxy = Proxy.newProxyInstance(loader, interfaces, h);
        return (Calculator) proxy;
    }
}
---------------------------------
public class AOPTest {
	/**
	 * 有了动态代理，日志记录可以做的非常强大；而且与业务逻辑解耦
	 * 
	 * jdk默认的动态代理，如果目标对象没有实现任何接口，是无法为他创建代理对象的；
	 * 
	 */
	@Test
	public void test() {
		Calculator calculator = new MyMathCalculator();
		
		calculator.add(1, 2);
		
		calculator.div(2, 1);
				
		//如果是拿到了这个对象的代理对象；代理对象执行加减乘除;
		Calculator proxy = CalculatorProxy.getProxy(calculator);
		//proxy的类型:com.sun.proxy.$Proxy2也是实现了Calculator接口
		//代理对象和被代理对象唯一能产生的关联就是实现了同一个接口
		System.out.println(Arrays.asList(proxy.getClass().getInterfaces()));
		proxy.add(2, 1);
		proxy.div(2, 0);
	}

}
```

* **动态代理:**
  * 写起来麻烦(难);
  * JDK默认的动态代理,如果目标对象没有实现任何接口,是无法为他创建代理对象的;
  * **因此:**
    * spring实现了AOP功能,底层就是动态代理;
    * 可以利用spring一句代码都不写的去创建动态代理;
    * 实现简单,没有强制要求目标对象必须实现接口;
  * AOP:将某段代码（日志）动态的切入（不把日志代码写死在业务逻辑方法中）到指定方法（加减乘除）的指定位置（方法的开始、结束、异常。。。）进行运行的这种编程方式（Spring简化了面向切面编程）

---

### P2: AOP专业术语

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2035%201635402095%201635402095098%20xyEm0Y%20image-20210701141208251.png" alt="image-20210701141208251" style="zoom:50%;float:left" />



### P3: spring实现AOP步骤

* 相关jar包;
* 写配置:
  * 将目标类和切面类(封装了通知方法(在目标方法执行前后执行的方法))加入到ioc容器中
  * 告诉spring哪个是切面类@Aspect
  * 告诉spring,切面类里面的每一个方法

```java
**
 * 如何将这个类（切面类）中的这些方法（通知方法）动态的在目标方法运行的各个位置切入
 *
 */
@Aspect
@Component
public class LogUtils {
	
	/**
	 * 告诉Spring每个方法都什么时候运行；
	 * try{
	 * 		@Before
	 * 		method.invoke(obj,args);
	 * 		@AfterReturning
	 * }catch(e){
	 * 		@AfterThrowing
	 * }finally{
	 * 		@After
	 * }
	 * 	
	 * 正常执行：  @Before（前置通知）=====@After（后置通知)====@AfterReturning（正常返回）；
	 * 异常执行： @Before（前置通知）=====@After（后置通知）===@AfterThrowing（方法异常）
	 * 5个通知注解
	 * @Before：在目标方法之前运行；  					    	前置通知
   * @After：在目标方法结束之后						    		 后置通知
	 * @AfterReturning：在目标方法正常返回之后			  返回通知
	 * @AfterThrowing：在目标方法抛出异常之后运行		  异常通知
	 * @Around：环绕								                环绕通知
	 * 
	 */
	
	//想在执行目标方法之前运行；写切入点表达式
	//execution(访问权限符  返回值类型  方法签名)
	@Before("execution(public int com.atguigu.impl.MyMathCalculator.*(..))")
	public static void logStart(JoinPoint joinPoint){
		//获取到目标方法运行是使用的参数
		Object[] args = joinPoint.getArgs();
		//获取到方法签名
		Signature signature = joinPoint.getSignature();
		String name = signature.getName();
		System.out.println("【"+name+"】方法开始执行，用的参数列表【"+Arrays.asList(args)+"】");
	}
	
	/**
	 * 切入点表达式的写法；
	 * 固定格式： execution(访问权限符  返回值类型  方法全类名(参数表))
	 *   
	 * 通配符：
	 * 		*：
	 * 			1）匹配一个或者多个字符:execution(public int com.atguigu.impl.MyMath*r.*(int, int))
	 * 			2）匹配任意一个参数：第一个是int类型，第二个参数任意类型；（匹配两个参数）
	 * 				execution(public int com.atguigu.impl.MyMath*.*(int, *))
	 * 			3）只能匹配一层路径
	 * 			4）权限位置*不能；权限位置不写就行；public【可选的】
	 * 		..：
	 * 			1）匹配任意多个参数，任意类型参数
	 * 			2）匹配任意多层路径:
	 * 				execution(public int com.atguigu..MyMath*.*(..));
	 * 
	 * 记住两种；
	 * 最精确的：execution(public int com.atguigu.impl.MyMathCalculator.add(int,int))
	 * 最模糊的：execution(* *.*(..))：千万别写；
	 * 
	 * 告诉Spring这个result用来接收返回值：
	 * 	returning="result"；
	 */
	//想在目标方法正常执行完成之后执行
	@AfterReturning(value="execution(public int com.atguigu..MyMath*.*(..))",returning="result")
	public static void logReturn(JoinPoint joinPoint,Object result){
		Signature signature = joinPoint.getSignature();
		String name = signature.getName();
		System.out.println("【"+name+"】方法正常执行完成，计算结果是："+result);
	}
	
	/**
	 * 细节四：我们可以在通知方法运行的时候，拿到目标方法的详细信息；
	 * 1）只需要为通知方法的参数列表上写一个参数：
	 * 		JoinPoint joinPoint：封装了当前目标方法的详细信息
	 * 2）、告诉Spring哪个参数是用来接收异常
	 * 		throwing="exception"：告诉Spring哪个参数是用来接收异常
	 * 3）、Exception exception:指定通知方法可以接收哪些异常
	 * 
	 * ajax接受服务器数据
	 * 	$.post(url,function(abc){
	 * 		alert(abc)
	 * 	})
	 */
	//想在目标方法出现异常的时候执行
	@AfterThrowing(value="execution(public int com.atguigu.impl.MyMathCalculator.*(..))",throwing="exception")
	public static void logException(JoinPoint joinPoint,Exception exception) {
		System.out.println("【"+joinPoint.getSignature().getName()+"】方法执行出现异常了，异常信息是【"+exception+"】：；这个异常已经通知测试小组进行排查");
	}
	//想在目标方法结束的时候执行
	/**
	 * Spring对通知方法的要求不严格；
	 * 唯一要求的就是方法的参数列表一定不能乱写
	 * 	通知方法是Spring利用反射调用的，每次方法调用得确定这个方法的参数表的值；
	 * 	参数表上的每一个参数，Spring都得知道是什么？
	 * 	JoinPoint:认识
	 * 	不知道的参数一定告诉Spring这是什么？
	 * 
	 * @param joinPoint
	 */
	@After("execution(public int com.atguigu.impl.MyMathCalculator.*(..))")
	private int logEnd(JoinPoint joinPoint) {
		System.out.println("【"+joinPoint.getSignature().getName()+"】方法最终结束了");
		return 0;
	}
}
```

---


* @**Around通知**
* **多切面运行**

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2036%201635402096%201635402096255%20TsuZL2%20image-20210701141228719.png" alt="image-20210701141228719" style="zoom:50%;float:left" />



### P4: AOP使用场景

* AOP加日志记录;
* AOP做权限验证;
* AOP做安全检查;
* AOP做事务控制;



### P5: AOP配置和注解

* 注解:快速方便;
* 配置:
  * 功能完善:外部导入jar包的需要配置;
  * 重要的用配置,不重要的用注解;





## D4: 事务

* **事务（ACID）:**
  * **原子性（Atomicity）:**
    * 在事务结束时，其中包含的所有更新处理要么全都执行，要么全都不执行；
      * 不可能出现在更新操作中，在更改a和b的价格之后，a的价格上涨了，b的价格却没有变化的情况；
  * **一致性（Consistency）：**
    * 事务中包含的处理要满足数据库提前设置的约束，如主键约束或者not null等；
  * **隔离性（Isolation）：**
    * 是指不同事务之间互不干扰的特性；
      * 例如即使某个事务向表中添加了记录，在没有提交之前，其他事务也是看不到新添加的记录的；
  * **持久性（Durability）：**
    * 在事务（不管是提交还是回滚）结束后，数据库管理系统能够保证该时间点的数据状态会被保存的特性；
      * 最常见的是通过记录日志的方式.




* **spring事务管理**
  * 编程式事务:
    * 使用原生JDBC API进行事务管理
      * 获取数据库连接Connection对象;
      * 取消事务的自动提交;
      * 执行操作;
      * 正常完成操作时手动提交事务;
      * 执行失败时回滚事务;
      * 关闭相关资源;
    * 评价:
      *     声明式事务:
    * 将事务管理代码的固定模式作为一种横切关注点,通过AOP方法模块化,进而借助spring aop框架实现
    * 显然声明式事务更好:它将事务管理代码从业务方法中分离出来,以声明的方式来实现事务管理.

---


* **数据库事务并发问题:**
  * 假如Transaction01和Transaction02并发执行;
    * **脏读:**
      * 01将某条记录的age值从20**修改**到了30;
      * 02读取了01**更新后的值**:30;
      * 01**回滚**,age值恢复到了20;
      * 02读取的30是一个无效的值
    * **不可重复读:**
      * 01读取了age值为20;
      * 02将age修改为30;
      * 01再次读取age值为30,和第一次读取不一致;
    * **幻读:**
      * 01读取了Student表中的一部分数据;
      * 02向Student表中插入了新的行;
      * 01读取了Student,多出了一些行;

---


* **事务的隔离级别:一个事物与其他事务隔离的程度**
  * **读未提交**:READ UNCOMMITTED:
    * 允许Transaction01 读取 Transaction02 未提交的修改;
  * **读已提交:**READ COMMITTED:
    * 要求Transaction01 只能读取 Transaction02 已提交的修改;
  * **可重复读:**REPEATABLE READ:
    * 在同一个事务期间.第一次读是什么,以后就是什么.(快照读)
  * **串行化:**SERIALIZABLE:
    * 确保Transaction01可以多次从一个表中读取到相同的行,在01执行期间,禁止其他事物对这个表进行添加,更新,删除操作,可以避免发生任何并发问题,但性能低下;

---


* **各隔离级别解决并发问题的能力:**

|                  | 脏读 | 不可重复读 | 幻读 |
| :--------------- | :--- | :--------- | :--- |
| READ UNCOMMITTED | 有   | 有         | 有   |
| READ COMMITTED   | 无   | 有         | 有   |
| REPEATABLE READ  | 无   | 无         | 有   |
| SERIALIZABLE     | 无   | 无         | 无   |

* **各数据库对事务隔离级别的支持:**

|                  | Oracle | MySQL   |
| :--------------- | :----- | :------ |
| READ UNCOMMITTED | ×      | √       |
| READ COMMITTED   | √      | √       |
| REPEATABLE READ  | ×      | √(默认) |
| SERIALIZABLE     | √      | √       |


---


* **触发事务回滚的异常:**
  * 默认情况:
    * 捕获到运行时异常和error时回滚;
    * 捕获到编译时异常不回滚;
  * 设置:

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2037%201635402097%201635402097500%2012BBb8%20image-20210701141311696.png" alt="image-20210701141311696" style="zoom:50%;float:left" />



---


* **事务的传播行为(propagation):**

  * 概念:

    * 当事务方法被另一个事务方法调用时,必须指定事务应该如何传播;
    * 例如:方法可能继续在现有事务中运行,也可能开启一个新事务,并在自己的事务中运行;

  * Spring定义了7种传播属性:

    * 最重要的两个传播属性:

      * REQUIRED:如果有事务在运行,当前的方法就在这个事务内运行;否则,就启动一个新的事务,并在自己的事务内运行;

      * REQUIRED_NEW:当前的方法必须启动新事务,并在自己的事务内运行,如果有事务正在运行,应该将它挂起;

        <img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2038%201635402098%201635402098941%203D8AZz%20image-20210701141332481.png" alt="image-20210701141332481" style="zoom:50%;float:left" />

  - 一些问题:

    <img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2039%201635402099%201635402099894%20xeZVDM%20image-20210701141423894.png" alt="image-20210701141423894" style="zoom:50%;float:left" />

  - <img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2041%201635402101%201635402101108%20Bjfsc7%20image-20210701141452812.png" alt="image-20210701141452812" style="zoom:50%;float:left" />

    任何处崩,已经执行的REQUIRES_NEW都会成功

  * 如果是REQUIRED:事务的属性都是继承大事务的;
    * 例如:在小事务中设置timeout就会没有效果,应该在大事务中设置

* 如果是REQUIRES_NEW则可以单独设置

* bookService类中的checkout()和updatePrice()都是REQUIRES_NEW属性:

  * 如果在bookService中写一个mulTx()方法直接调用上面两个方法,大事务中出了问题,这两个方法还是会回滚;

<img src="https://gitee.com/breeze1002/upic/raw/master/SSM/Spring/2021%2010%2028%2014%2021%2042%201635402102%201635402102079%202uXAaZ%20image-20210701141545293.png" alt="image-20210701141545293" style="zoom:50%;float:left" />





























