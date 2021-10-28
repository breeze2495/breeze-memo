

# BreezeMVC 文档

## ⭐ 前言

### 1. 简介

​	SpringMVC可以说是目前最优秀的轻量MVC框架，其采用了松散耦合可插拔组件结构，比其他MVC框架更具扩展性和灵活性；因此理解SpringMVC的原理，在面试或工作中都十分的重要。

​	笔者发现SpringMVC的原理在网络上随处可见，但是写的都比较概括、零散；对于阅读源码经验较少(包括笔者自己)的小伙伴来说， 看源码又会被很多底层细节所干扰，不能够很好的抽离出springMVC原理的主线；

​	本项目从手写简易版的SpringMVC框架出发， 抽取出SpringMVC的主线，将其核心组件各个击破，达到深入理解SpringMVC的原理的目的。



### 2. 项目结构

```
BreezeMVC
├── README.md -- 项目开发/总结文档
├── breeze-mvc-core -- 实现mvc功能的核心代码
├── breezemvc-springboot-autoconfigure -- BreezeMVC的自动化配置
├── breezemvc-springboot-demo -- BreezeMVC的demo项目
├── breezemvc-springboot-starter -- BreezeMVC的starter
└── spring-mvc-demo -- SpringMVC 参考demo
```



### 3. 约定

- 为了更好的理解和使用SpringMVC，所以在BreezeMVC中所有组件的名称都和SpringMVC保持一致
- 参考 springMVC版本：version 5.2.9 
- 单元测试 Junit：version 4.13
- JSON转换：version 1.2.60
- 工具：代码Idea + UML类图ProcessOn + 文档Typora 



## P1: 总体架构规划

在正式开始各个组件的研发之前，我们需要先设计好BreezeMVC的整体框架结构，也就是先整体后局部。

当然，由于是参考springMVC并对其进行核心组件抽取，因此整体架构和springMVC一致

![image-20211018222116591](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2041%201635402401%201635402401226%200fYNJ2%20image-20211018222116591.png)



## P2: RequestMappingHandlerMapping

### D1: 概述

首先可能我们会存在这样的疑问：SpringMVC是如何通过request找到我们所写的controller中的方法，这其中需要经历怎样的环节？

本节就介绍了 **实现HandlerMapping的初始化过程**，把Controller中的方法转换成我们定义的HandlerMethod对象(架构图中的Handler)



### D2: 组件

![image-20210907153003355](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2042%201635402402%201635402402560%207EetLx%20image-20210907153003355.png)

#### C1: HandlerMapping接口

```java
/**
* 该接口中只有一个方法，通过request找到需要执行的handler，包装成HandlerExecutionChain返回。
* 目前暂不实现，具体实现在后续内容中。
*/
public interface HandlerMapping {
    HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

#### C2: RequestMapping 注解

```java
/**
* RequestMapping提供两个属性
* 1.path：url中的路径
* 2.method：http请求的方式(post,get) 默认为GET
*/
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestMapping {

    String path();

    RequestMethod method() default RequestMethod.GET;

}
```

#### C3: RequestMappingInfo

```java
/**
* 对应配置在Controller上的@RequestMapping注解，将其转换成RequestMappingInfo对象
*/
public class RequestMappingInfo {
    private String path;
    private RequestMethod httpMethod;

    public RequestMappingInfo(String prefix, RequestMapping requestMapping) {
        this.path = prefix + requestMapping.path();
        this.httpMethod = requestMapping.method();
    }

    public String getPath() {
        return path;
    }

    public RequestMethod getHttpMethod() {
        return httpMethod;
    }

}
```

#### C4: HandlerMethod

```java
/**
*	HandlerMethod是一个比较重要的组件，其主要是对应Controller中的每个方法，也就是实际处理业务的handler
* 目前我们暂时定义四个属性：
* 1. bean: 表示该方法的实例对象，也就是Controller的实例对象
* 2. beanType: 表示的是我们写的Controller的类型；比如：UserController，CompanyController等等
* 3. method: 表示Controller中的方法
* 4. parameters: 表示方法中的所有参数的定义，这里引用了Spring中提供的MethodParameter工具类，里面封装了一些实用的方法，比如说后面会用到获取方法上的注解等等
*
*/
public class HandlerMethod {

    private Object bean;
    private Class<?> beanType;
    private Method method;

    private List<MethodParameter> parameters;

    public HandlerMethod(Object bean, Method method) {
        this.bean = bean;
        this.beanType = bean.getClass();
        this.method = method;

        this.parameters = new ArrayList<>();
        int parameterCount = method.getParameterCount();
        for (int index = 0; index < parameterCount; index++) {
            parameters.add(new MethodParameter(method, index));
        }
    }
}
```

#### C5: MappingRegistry

```java
/**
 * MappingRegistry 是 RequestMappingInfo 和 HandlerMethod 的注册中心
 * 当解析完一个Controller的method后就会向MappingRegistry中注册一个
 * 当接收到用户请求，会根据url在MappingRegistry找到对应的HandlerMethod
 *
 */
public class MappingRegistry {
    private Map<String, RequestMappingInfo> pathMappingInfo = new ConcurrentHashMap<>();
    private Map<String, HandlerMethod> pathHandlerMethod = new ConcurrentHashMap<>();

    /**
     * 1.把解析完成的RequestMappingInfo注册到map中
     * 2.通过handler和method构建HandlerMethod对象，也注册到map中
     *
     */
    public void register(RequestMappingInfo mapping, Object handler, Method method) {
        pathMappingInfo.put(mapping.getPath(), mapping);

        HandlerMethod handlerMethod = new HandlerMethod(handler, method);
        pathHandlerMethod.put(mapping.getPath(), handlerMethod);
    }

    public RequestMappingInfo getMappingByPath(String path) {
        return this.pathMappingInfo.get(path);
    }

    public HandlerMethod getHandlerMethodByPath(String path) {
        return this.pathHandlerMethod.get(path);
    }

}
```

#### C6: RequestMappingHandlerMapping

```java
/**
* 1.在初始化过程中，我们需要获取容器中所有Bean对象，所以RequestMappingHandlerMapping需要继承：
*   ApplicationObjectSupport：其为我们提供了方便访问容器的方法
* 2.需要实现：
*   InitializationBean：提供了afterPropertiesSet方法，在对象创建城后，spring容器会调用这个方法。
*   初始化代码的入口就在afterPropertiesSet中。
*
*/
public class RequestMappingHandlerMapping extends ApplicationObjectSupport implements HandlerMapping, InitializingBean {

    private MappingRegistry mappingRegistry = new MappingRegistry();


    public MappingRegistry getMappingRegistry() {
        return mappingRegistry;
    }

    public void afterPropertiesSet() throws Exception {
        initialHandlerMethods();
    }
	
  	/**
		* 初始化
		*/
    private void initialHandlerMethods() {
				//从容器中拿出所有的Bean，该方法会返回beanName和bean实例对应的map
      	Map<String, Object> beansOfMap = BeanFactoryUtils.beansOfTypeIncludingAncestors(obtainApplicationContext(), Object.class);
      
      	//stream流过滤出标记了@Controller的类
      	//解析出handler中被@RequestMapping注解的方法
        beansOfMap.entrySet().stream()
                .filter(entry -> this.isHandler(entry.getValue()))
                .forEach(entry -> this.detectHandlerMethods(entry.getKey(), entry.getValue()));
    }

    /**
     * 类上有标记Controller的注解就是我们需要找的handler
     *
     * @param handler
     * @return
     */
    private boolean isHandler(Object handler) {
        Class<?> beanType = handler.getClass();
      	//Spring中的工具类AnnotatedElementUtils.hasAnnotation
        return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class));
    }

    /**
     * 解析出handler中 所有被RequestMapping注解的方法
     *
     * @param beanName
     * @param handler
     */
    private void detectHandlerMethods(String beanName, Object handler) {
        Class<?> beanType = handler.getClass();
      	//工具类MethodIntrospector.selectMethods判断方法是否添加注解
        Map<Method, RequestMappingInfo> methodsOfMap = MethodIntrospector.selectMethods(beanType,
                (MethodIntrospector.MetadataLookup<RequestMappingInfo>) method -> getMappingForMethod(method, beanType));
				//筛选后的方法注册到 MappingRegistry
        methodsOfMap.forEach((method, requestMappingInfo) -> this.mappingRegistry.register(requestMappingInfo, handler, method));
    }

    /**
     * 查找method上面是否有RequestMapping，有则构建RequestMappingInfo
     *
     * @param method
     * @param beanType
     * @return
     */
    private RequestMappingInfo getMappingForMethod(Method method, Class<?> beanType) {
        RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(method, RequestMapping.class);
        if (Objects.isNull(requestMapping)) {
            return null;
        }
        String prefix = getPathPrefix(beanType);
        return new RequestMappingInfo(prefix, requestMapping);
    }

    private String getPathPrefix(Class<?> beanType) {
        RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(beanType, RequestMapping.class);
        if (Objects.isNull(requestMapping)) {
            return "";
        }
        return requestMapping.path();
    }
		
  	//本节暂不实现
    @Override
    public HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        return null;
    }

}
```

### 

### D3: 单元测试

到此，Controller中的方法解析过程已初步完成，接下来写一点简单的测试用例。

```java
@Configuration
@ComponentScan(basePackages = "com.breeze9527.breezemvc")
public class AppConfig {

    @Bean
    public RequestMappingHandlerMapping handlerMapping() {
        return new RequestMappingHandlerMapping();
    }

}
```

为了测试我们的Controller解析是否正确，我们需要的测试用例：

- 在IndexController中我们在类上面配置path的前缀`/index`；解析完成后的path要拼接上`/index`
- 在IndexController中添加三个方法，其中`test`、`test2`两个用`@RequestMapping`标注，`test3`不标注；解析完成后`test3`不再我们注册中心里面，`test`、`test2`两个在注册中里面，并且`@RequestMapping`中的属性正确解析成`RequestMappingInfo`对象
- 创建TestConroller类，添加一个方法`test4`，在类上面标注注解`@Service`，解析完成后`test4`不能在注册中心里面找到

```java
@Controller
@RequestMapping(path = "/index")
public class IndexController {

    @RequestMapping(path = "/test", method = RequestMethod.GET)
    public void test(String name) {

    }

    @RequestMapping(path = "/test2", method = RequestMethod.POST)
    public void test2(String name2) {

    }

    public void test3(String name3) {

    }

}
```

```java
@Service
public class TestController {

    @RequestMapping(path = "/test4", method = RequestMethod.POST)
    public void test4(String name2) {

    }

}
```

接下来建立我们的单元测试类`RequestMappingHandlerMappingTest`，继承于我们上一节写好的测试基类`BaseJunit4Test`

```java
public class RequestMappingHandlerMappingTest extends BaseJunit4Test {

    @Autowired
    private RequestMappingHandlerMapping requestMappingHandlerMapping;

    @Test
    public void test() {
        MappingRegistry mappingRegistry = requestMappingHandlerMapping.getMappingRegistry();

        String path = "/index/test";
        String path1 = "/index/test2";
        String path4 = "/test4";

        Assert.assertEquals(mappingRegistry.getPathHandlerMethod().size(), 2);

        HandlerMethod handlerMethod = mappingRegistry.getHandlerMethodByPath(path);
        HandlerMethod handlerMethod2 = mappingRegistry.getHandlerMethodByPath(path1);
        HandlerMethod handlerMethod4 = mappingRegistry.getHandlerMethodByPath(path4);

        Assert.assertNull(handlerMethod4);
        Assert.assertNotNull(handlerMethod);
        Assert.assertNotNull(handlerMethod2);


        RequestMappingInfo mapping = mappingRegistry.getMappingByPath(path);
        RequestMappingInfo mapping2 = mappingRegistry.getMappingByPath(path1);

        Assert.assertNotNull(mapping);
        Assert.assertNotNull(mapping2);
        Assert.assertEquals(mapping.getHttpMethod(), RequestMethod.GET);
        Assert.assertEquals(mapping2.getHttpMethod(), RequestMethod.POST);
    }

}
```



### D4: 小结与延展

- **本节主要实现了HandlerMapping初始化，将Controller中的方法转换成HandlerMethod对象**
- **和SpringMVC相比存在的不足**
  - SpringMVC中不仅仅有`RequestMappingHandlerMapping`，还有基于xml配置的`SimpleUrlHandlerMapping`以及`BeanNameUrlHandlerMapping`等等，
  - `RequestMappingHandlerMapping`在SpringMVC中多了几层用于支持默认处理程序、处理程序拦截器等功能的抽象类。
    - RequestMappingInfoHandlerMapping:
    - AbstractHandlerMethodMapping<T> 
    - AbstractHandlerMapping



## P3: HandlerInteceptor

![image-20210907120231549](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2043%201635402403%201635402403430%20ii350p%20image-20210907120231549.png)

### D1: 概述

本节主要实现拦截器部分

### D2: 组件

#### C1: HandlerInterceptor

```java
public interface HandlerInterceptor {
		
    /**
    * preHandle: 在执行Handler之前被调用，如果返回的是false，那么Handler就不会在执行
    */
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
        return true;
    }
		
  	/**
  	* postHandle: 在Handler执行完成之后被调用，可以获取Handler返回的结果ModelAndView
  	*/
  	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }
		
  	/**
  	*	afterCompletion: 该方法是无论什么情况下都会被调用，比如：preHandle返回false，Handler执行过程中抛出异常，Handler正常执行完成
  	*/
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable Exception ex) throws Exception {
    }
}
```

#### C2: MappedInterceptor

```JAVA
/**
* 通常拦截器需要设置对哪些url生效/以及排除哪些url，但是HandlerInterceptor接口中我们没有看到相关定义
*	为了实现以上功能并达到配置与业务分离，我们需要开发MappedInterceptor，所需实现的功能有：
*	1.作为HandlerInteceptor代理类，需要实现HandlerInterceptor，实现其三个接口，并且内部需要包含真正HandlerInteceptor实例
*	2.管理对哪些url有效，需要排除哪些url
*	3.matches()方法，判断传入的url是否复合条件，本项目只简单实现了path的完整匹配。
*/
public class MappedInterceptor implements HandlerInterceptor {
    private List<String> includePatterns = new ArrayList<>();
    private List<String> excludePatterns = new ArrayList<>();

    private HandlerInterceptor interceptor;

    public MappedInterceptor(HandlerInterceptor interceptor) {
        this.interceptor = interceptor;
    }

    /**
     * 添加支持的path
     *
     * @param patterns
     * @return
     */
    public MappedInterceptor addIncludePatterns(String... patterns) {
        this.includePatterns.addAll(Arrays.asList(patterns));
        return this;
    }

    /**
     * 添加排除的path
     *
     * @param patterns
     * @return
     */
    public MappedInterceptor addExcludePatterns(String... patterns) {
        this.excludePatterns.addAll(Arrays.asList(patterns));
        return this;
    }


    /**
     * 根据传入的path, 判断当前的interceptor是否支持
     *
     * @param lookupPath
     * @return
     */
    public boolean matches(String lookupPath) {
        if (!CollectionUtils.isEmpty(this.excludePatterns)) {
            if (excludePatterns.contains(lookupPath)) {
                return false;
            }
        }
        if (ObjectUtils.isEmpty(this.includePatterns)) {
            return true;
        }
        if (includePatterns.contains(lookupPath)) {
            return true;
        }
        return false;
    }


    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
        return this.interceptor.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable ModelAndView modelAndView) throws Exception {
        this.interceptor.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable Exception ex) throws Exception {
        this.interceptor.afterCompletion(request, response, handler, ex);
    }
}
```

#### C3: InterceptorRegistry

```java
/**
*	在开发完处理拦截业务逻辑的接口HandlerInterceptor和管理HandlerInterceptor与请求路径的映射关联类MappedInterceptor后，我们还需要一个拦截器的注册中心来管理所有的拦截器
*/
public class InterceptorRegistry {
    private List<MappedInterceptor> mappedInterceptors = new ArrayList<>();

    /**
     * 注册一个拦截器到Registry
     * @param interceptor
     * @return
     */
    public MappedInterceptor addInterceptor(HandlerInterceptor interceptor) {
        MappedInterceptor mappedInterceptor = new MappedInterceptor(interceptor);
        mappedInterceptors.add(mappedInterceptor);
        return mappedInterceptor;
    }

    public List<MappedInterceptor> getMappedInterceptors() {
        return mappedInterceptors;
    }
}
```



### D3: 单元测试

到此，拦截器部分开发完成，接下来写一些简单的测试用例：

1. 创建一个拦截器的实现，能够成功注册到`InterceptorRegistry`
2. 能够为注册的拦截器设置支持URL和排除的URL
3. 测试拦截器的match方法是否正确

`具体测试代码在项目中`



### D4: 小结与延展

- 本节完成了拦截器相关的功能组件。
- **和SpringMVC相比存在的不足**
  - matches功能，只简单实现了path的完整匹配。springMVC的PathMatcher接口提供了基于字符串的匹配策略，例如通过某个模式找出路径的哪一部分是动态匹配的，以及通配符"*"，"?"，"{"的处理



## P4: HandlerExecutionChain

![image-20210903223939817](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2044%201635402404%201635402404562%20RqI9Kn%20image-20210903223939817.png)

### D1: 概述

本节主要开发HandlerMapping接口中getHandler用到的HandlerExecutionChain对象，已经过程中使用到的NoHandlerFoundException异常

### D2: 组件

#### C1: HandlerExecutionChain

```java
/**
*	该类包含两个对象
*	1.HandlerMethod方法
*	2.根据url找到对本次请求有效的拦截器HandlerInterceptor
*/
public class HandlerExecutionChain {
    private HandlerMethod handler;
    private List<HandlerInterceptor> interceptors = new ArrayList<>();
    private int interceptorIndex = -1;

    public HandlerExecutionChain(HandlerMethod handler, List<HandlerInterceptor> interceptors) {
        this.handler = handler;
        if (!CollectionUtils.isEmpty(interceptors)) {
            this.interceptors = interceptors;
        }
    }
		
  	/**
  	*	applyPreHandle: 执行所有拦截器的preHandle方法，如果preHandle返回的是false，那么就执行triggerAfterCompletion
  	*/
    public boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        if (CollectionUtils.isEmpty(interceptors)) {
            return true;
        }
        for (int i = 0; i < interceptors.size(); i++) {
            HandlerInterceptor interceptor = interceptors.get(i);
            if (!interceptor.preHandle(request, response, this.handler)) {
                triggerAfterCompletion(request, response, null);
                return false;
            }
            this.interceptorIndex = i;
        }
        return true;
    }
		
  	/**
  	* applyPostHandle: 执行所有拦截器的postHandle方法
  	*/
    public void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
        if (CollectionUtils.isEmpty(interceptors)) {
            return;
        }
        for (int i = interceptors.size() - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors.get(i);
            interceptor.postHandle(request, response, this.handler, mv);
        }
    }
  
  	/**
  	* triggerAfterCompletion: 
  	*	HandlerExecutionChain中还定义了一个变量interceptorIndex，当每执行一个HandlerInterceptor的		preHandle方法后interceptorIndex的值就会被修改成当前执行拦截器的下标，
  	*triggerAfterCompletion中根据interceptorIndex记录的下标值反向执行拦截器的afterCompletion方法；
  	*
  	*	例如：
  	*	下标0，1的拦截器返回true 当前index==1。当执行下标为2的preHandle返回false，则会执行下标为0，1的拦截器的afterCompletion方法
  	*/
    public void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)
            throws Exception {
        if (CollectionUtils.isEmpty(interceptors)) {
            return;
        }
        for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors.get(i);
            interceptor.afterCompletion(request, response, this.handler, ex);
        }
    }

    public List<HandlerInterceptor> getInterceptors() {
        return interceptors;
    }

    public HandlerMethod getHandler() {
        return handler;
    }
}
```

#### C2: NoHandlerFoundException

```java
/**
*	在通过HandlerMapping.getHandler获取对应request处理器的时候，可能会遇到写错了请求的路径导致找不到匹配的Handler情况，这个时候需要抛出指定的异常，方便我们后续处理，比如说跳转到错误页面
*/
public class NoHandlerFoundException extends ServletException {
    private String httpMethod;
    private String requestURL;

    public NoHandlerFoundException(HttpServletRequest request) {
        this.httpMethod = request.getMethod();
        this.requestURL = request.getRequestURL().toString();
    }

    public String getHttpMethod() {
        return httpMethod;
    }

    public String getRequestURL() {
        return requestURL;
    }
}
```

#### C3:完善RequestMappingHandlerMapping

```java
//添加拦截器集合，包含了所有的拦截器
private List<MappedInterceptor> interceptors = new ArrayList<>();

public void setInterceptors(List<MappedInterceptor> interceptors) {
    this.interceptors = interceptors;
}

/**
*	获取到本次请求的path，然后根据path从MappingRegistry找到对应的HandlerMethod，如果找不到就抛出我们定义的异常NoHandlerFoundException；
*/
@Override
public HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    String lookupPath = request.getRequestURI();
    HandlerMethod handler = mappingRegistry.getHandlerMethodByPath(lookupPath);
    if (Objects.isNull(handler)) {
        throw new NoHandlerFoundException(request);
    }
    return createHandlerExecutionChain(lookupPath, handler);
}

/**
*	从所有拦截器中过滤出匹配本次请求path的拦截器，然后创建HandlerExecutionChain对象
*/
private HandlerExecutionChain createHandlerExecutionChain(String lookupPath, HandlerMethod handler) {
    List<HandlerInterceptor> interceptors = this.interceptors.stream()
            .filter(mappedInterceptor -> mappedInterceptor.matches(lookupPath))
            .collect(toList());
    return new HandlerExecutionChain(handler, interceptors);
}
```



### D3: 单元测试

本节的测试用例：

1. 测试getHandler返回的HandlerExecutionChain数据是否正确：
   a) HandlerMethod中的bean是否是正确的Controller实例；
   b) interceptors是否是匹配请求的path
2. 测试getHandler找不到Handler是否会抛出异常`NoHandlerFoundException`
3. 同一个拦截器添加多个includePatterns，能正确匹配

`具体测试代码在项目中`



### D4: 小结与延展

- 本节完成了getHandler的开发，其中主要的对象是`HandlerExecutionChain`，它包含了具体执行业务逻辑的`HandlerMethod`以及匹配的拦截器；
- **和SpringMVC相比存在的不足**
  - 相较于本项目的实现，springMVC提供的功能更加完善。
    - 例如：跨域配置CORS的相关功能



## P5: handlerMethod参数解析器

![image-20210831103436158](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2045%201635402405%201635402405395%20AHYj9k%20image-20210831103436158.png)

### D1: 概述

本节主要是在开发HandlerAdapter过程中使用到的组件HandlerMethodArgumentResolver。

在我们享受SpringMVC给我带来的便利的时候，Controller中方法的参数如何完成自动注入是一个需要思考的问题，
在添加上注解`@PathVariable`、`@RequestParam`、`@RequestBody`就能够把请求中的参数主动注入，
甚至在方法参数任意位置写`HttpServletRequest`、`HttpSession`等类型的参数，它自动就有值了便可直接使用；
现在我们就开始来逐步实现这个功能，本节主要实现参数的解析。

本节我们简单实现解析`HttpServletRequest`、`HttpServletResponse`、`Model`以及注解`@RequestParam`、`@RequestBody`的功能，SpringMVC提供了其他参数解析器实现。

### D2: 组件

#### C1: ModelAndViewContainer

```java
/**
*	该类主要用于保存Handler处理过程中的Model以及返回的View的对象
*	
*/
public class ModelAndViewContainer {

  	private Object view;
    private Model model;
    private HttpStatus status;
  	//标记本次请求是否已经处理完成，后期在处理注解@ResponseBody中会使用到
    private boolean requestHandled = false;

    public void setView(Object view) {
        this.view = view;
    }

    public String getViewName() {
        return (this.view instanceof String ? (String) this.view : null);
    }
    
    public void setViewName(String viewName) {
        this.view = viewName;
    }
    
    public Object getView() {
        return this.view;
    }

    public boolean isViewReference() {
        return (this.view instanceof String);
    }

    public Model getModel() {
        if (Objects.isNull(this.model)) {
            this.model = new ExtendedModelMap();
        }
        return this.model;
    }

    public void setStatus(HttpStatus status) {
        this.status = status;
    }

    public HttpStatus getStatus() {
        return this.status;
    }
    
    public boolean isRequestHandled() {
        return requestHandled;
    }

    public void setRequestHandled(boolean requestHandled) {
        this.requestHandled = requestHandled;
    }

}
```

#### C2: HandlerMethodArgumentResolver

```JAVA
/**
*	该方法是一个策略接口，作用时把请求中的数据 解析为Controller中方法的参数值
*	有该方法的加持，让springMVC在处理入参时显得很自动化
*/
public interface HandlerMethodArgumentResolver {
  	
  	//此方法判断当前的参数解析器是否支持传入的参数，返回true表示支持
    boolean supportsParameter(MethodParameter parameter);
  	
  	/**
  	* 从request对象中解析出parameter需要的值，除了MethodParameter和HttpServletRequest参数外，
还传入了ConversionService，用于在把request中取出的值需要转换成MethodParameter参数的类型。
这个方法的参数定义和SpringMVC中的方法稍有不同，主要是为了简化数据转换的过程
  	*/
    Object resolveArgument(MethodParameter parameter, HttpServletRequest request, 
                    HttpServletResponse response, ModelAndViewContainer container,
                           ConversionService conversionService) throws Exception;
}
```

#### C3: ServletRequestMethodArgumentResolver、ServletResponseMethodArgumentResolver

- **ServletRequestMethodArgumentResolver**

```java
public class ServletRequestMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        Class<?> parameterType = parameter.getParameterType();
      	//判断该类型是否是ServletRequest的子类
        return ServletRequest.class.isAssignableFrom(parameterType);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request,
                                  HttpServletResponse response, ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {
        return request;
    }

}
```

- **ServletResponseMethodArgumentResolver**

```java
public class ServletResponseMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        Class<?> parameterType = parameter.getParameterType();
        return ServletResponse.class.isAssignableFrom(parameterType);
    }
    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request, HttpServletResponse response,
                                  ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {
        return response;
    }
}
```

#### C4: ModelMethodArgumentResolver

```java
//用于解析出 Model 对象
public class ModelMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return Model.class.isAssignableFrom(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request,
                                  HttpServletResponse response, ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {

        Assert.state(container != null, "ModelAndViewContainer is required for model exposure");
        return container.getModel();
    }
}
```

#### C5: RequestParamMethodArgumentResolver

- **@RequestParam注解**

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam {
		
  	//从request中取参数的名字，该参数必填
    String name();
		//用于标记改参数是否必填，默认为true
    boolean required() default true;
		//默认值
    String defaultValue() default "";

}
```

- **RequestParamMethodArgumentResolver**

```java
public class RequestParamMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
      	//判断是否添加了 @RequestParam注解
        return parameter.hasParameterAnnotation(RequestParam.class);
    }
		
  	/**
  	*	从request中找指定name的参数，如果找不到用默认值赋值，如果默认值也没有，当required=true时抛出异常，
否则返回null; 
		*	如果从request中找到了参数值，那么调用conversionService.convert方法转换成正确的类型
  	*/
    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request,
                                  HttpServletResponse response, ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {

        RequestParam param = parameter.getParameterAnnotation(RequestParam.class);
        if (Objects.isNull(param)) {
            return null;
        }
        String value = request.getParameter(param.name());
        if (StringUtils.isEmpty(value)) {
            value = param.defaultValue();
        }
        if (!StringUtils.isEmpty(value)) {
            return conversionService.convert(value, parameter.getParameterType());
        }
        
        if (param.required()) {
            throw new MissingServletRequestParameterException(parameter.getParameterName(),
                    parameter.getParameterType().getName());
        }
        return null;
    }

}
```

#### C6: RequestBodyMethodArgumentResolver

- **@RequestBody注解**

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {
    boolean required() default true;
}
```

- **RequestBodyMethodArgumentResolver**

```java
public class RequestBodyMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(RequestBody.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request,
                                  HttpServletResponse response, ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {
      	//从request对象流中读取出数据转换成字符串
        String httpMessageBody = this.getHttpMessageBody(request);
        if (!StringUtils.isEmpty(httpMessageBody)) {
          	//将字符串转换成对应Object对象
            return JSON.parseObject(httpMessageBody, parameter.getParameterType());
        }

        RequestBody requestBody = parameter.getParameterAnnotation(RequestBody.class);
        if (Objects.isNull(requestBody)) {
            return null;
        }
        if (requestBody.required()) {
            throw new MissingServletRequestParameterException(parameter.getParameterName(),
                    parameter.getParameterType().getName());
        }
        return null;
    }

    private String getHttpMessageBody(HttpServletRequest request) throws IOException {
        StringBuilder sb = new StringBuilder();
        BufferedReader reader = request.getReader();
        char[] buff = new char[1024];
        int len;
        while ((len = reader.read(buff)) != -1) {
            sb.append(buff, 0, len);
        }
        return sb.toString();
    }
}
```

#### C7: HandlerMethodArgumentResolverComposite

`该组件的实现使用到了策略模式`

```java
/**
* HandlerMethodArgumentResolverComposite实现HandlerMethodArgumentResolver
*	内部定义list，找到支持的解析器就开始解析，找不到就抛出异常
*/
public class HandlerMethodArgumentResolverComposite implements HandlerMethodArgumentResolver {
    private List<HandlerMethodArgumentResolver> argumentResolvers = new ArrayList<>();
    
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return true;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, HttpServletRequest request,
                                  HttpServletResponse response, ModelAndViewContainer container,
                                  ConversionService conversionService) throws Exception {
        for (HandlerMethodArgumentResolver resolver : argumentResolvers) {
            if (resolver.supportsParameter(parameter)) {
                return resolver.resolveArgument(parameter, request, conversionService);
            }
        }
        throw new IllegalArgumentException("Unsupported parameter type [" +
                parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
    }

    public void addResolver(HandlerMethodArgumentResolver resolver) {
        this.argumentResolvers.add(resolver);
    }

    public void addResolver(HandlerMethodArgumentResolver... resolvers) {
        Collections.addAll(this.argumentResolvers, resolvers);
    }

    public void clear() {
        this.argumentResolvers.clear();
    }
}
```

### D3: 单元测试

**本节做了如下单元测试：**

- 验证`@RequestParam`: 创建TestController，方法test4的参数name, age, birthday, request，
- 验证解析器是否能够正常处理类型为String、Integer、Date、HttpServletRequest的解析
- 验证`@RequestBody`:创建TestController，方法user的参数UserVo, 验证解析器能够正确的把JSON字符串解析成UserVo对象

`具体测试代码在项目中`



### D4: 小结与延展

- 本节完成了handlerMethod参数解析器的开发，了解了springMVC中参数解析的过程；
- **和SpringMVC相比存在的不足**
  - 本节所开发的解析器只实现了对参数的自动封装，springMVC中的参数解析器还包含了参数的校验等等，
  - 此外。springMVC还为我们提供了丰富的解析器：
    - 如：`PathVariableMethodArgumentResolver`、`SessionAttributeMethodArgumentResolver`、`ServletCookieValueMethodArgumentResolver`等





## P6: handlerMethod返回值解析器

![image-20210831103359646](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2054%201635402414%201635402414252%20jP46Fo%20image-20210831103359646.png)

### D1: 概述

上一节我们完成了参数解析器组件，这一节我们将完成对返回值解析器的开发；

springMVC为我们提供了很多返回值解析器，本节我们选取其中几个常用的进行开发；

#### C1: HandlerMethodReturnValueHandler

```java
public interface HandlerMethodReturnValueHandler {
  
		//同参数解析器一样，判断处理器是否支持该返回值类型
    boolean supportsReturnType(MethodParameter returnType);
  
		//处理并返回值
  	//需要传入HttpServletResponse，可能会在直接处理完 整个请求，例如@ResponseBody
    void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                           ModelAndViewContainer mavContainer,
                           HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

#### C2: ModelMethodReturnValueHandler

```java
/**
*	上一节中提到的ModelAndViewContainer，它是一个ModelAndView的容器，每个请求都会新建一个对象，贯穿整个Handler执行前的参数解析、执行以及返回值处理；
*	这两个类的实现主要都是将Handler的返回值添加到Model中，用于后面构建ModeAndView对象以及实现渲染
*/
public class ModelMethodReturnValueHandler implements HandlerMethodReturnValueHandler {

    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return Model.class.isAssignableFrom(returnType.getParameterType());
    }

    @Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response) throws Exception {

        if (returnValue == null) {
            return;
        } else if (returnValue instanceof Model) {
            mavContainer.getModel().addAllAttributes(((Model) returnValue).asMap());
        } else {
            // should not happen
            throw new UnsupportedOperationException("Unexpected return type: " +
                    returnType.getParameterType().getName() + " in method: " + returnType.getMethod());
        }
    }
}
```

#### C3: MapMethodReturnValueHandler

```java
public class MapMethodReturnValueHandler implements HandlerMethodReturnValueHandler {
    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return Map.class.isAssignableFrom(returnType.getParameterType());
    }

    @Override
    @SuppressWarnings("unchecked")
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response) throws Exception {

        if (returnValue instanceof Map) {
            mavContainer.getModel().addAllAttributes((Map) returnValue);
        } else if (returnValue != null) {
            // should not happen
            throw new UnsupportedOperationException("Unexpected return type: " +
                    returnType.getParameterType().getName() + " in method: " + returnType.getMethod());
        }
    }
}
```

#### C4: ViewNameMethodReturnValueHandler

上方两个处理器主要负责ModelAndView中的Model部分

接下来我们要实现的处理器负责View部分

```java
/**
*	先构建View对象方便引用，具体暂时不实现
*/
public interface View {
}
```

```java
/**
*	如果returnValue是String，直接把这个返回值当做视图名，放到ModelAndViewContainer中
*/
public class ViewNameMethodReturnValueHandler implements HandlerMethodReturnValueHandler {

    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        Class<?> paramType = returnType.getParameterType();
        return CharSequence.class.isAssignableFrom(paramType);
    }

    @Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response) throws Exception {

        if (returnValue instanceof CharSequence) {
            String viewName = returnValue.toString();
            mavContainer.setViewName(viewName);
        } else if (returnValue != null) {
            // should not happen
            throw new UnsupportedOperationException("Unexpected return type: " +
                    returnType.getParameterType().getName() + " in method: " + returnType.getMethod());
        }
    }
}
```

#### C5: ViewMethodReturnValueHandler

```java
/**
*	如果returnValue是View对象，直接把这个视图放到ModelAndViewContainer中
*/
public class ViewMethodReturnValueHandler implements HandlerMethodReturnValueHandler {

    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return View.class.isAssignableFrom(returnType.getParameterType());
    }

    @Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response) throws Exception {

        if (returnValue instanceof View) {
            View view = (View) returnValue;
            mavContainer.setView(view);
        } else if (returnValue != null) {
            // should not happen
            throw new UnsupportedOperationException("Unexpected return type: " +
                    returnType.getParameterType().getName() + " in method: " + returnType.getMethod());
        }
    }

}
```

#### C6: ResponseBodyMethodReturnValueHandler

- **@ResponseBody**

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseBody {
}
```

```java
/**
*	1.标记出当前请求已经处理完成，后续的渲染无需在执行
*	2.使用fastJson把返回值转换成JSON字符串，在使用response输出给前端
*/
public class ResponseBodyMethodReturnValueHandler implements HandlerMethodReturnValueHandler {
    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return (AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ResponseBody.class) ||
                returnType.hasMethodAnnotation(ResponseBody.class));
    }

    @Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        //标记本次请求已经处理完成
        mavContainer.setRequestHandled(true);

        outPutMessage(response, JSON.toJSONString(returnValue));
    }

    private void outPutMessage(HttpServletResponse response, String message) throws IOException {
        try (PrintWriter out = response.getWriter()) {
            out.write(message);
        }
    }

}
```

#### C7: HandlerMethodReturnValueHandlerComposite

```java
public class HandlerMethodReturnValueHandlerComposite implements HandlerMethodReturnValueHandler {
    private List<HandlerMethodReturnValueHandler> returnValueHandlers = new ArrayList<>();

    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return true;
    }

    @Override
    public void handleReturnValue(Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer,
                                  HttpServletRequest request, HttpServletResponse response) throws Exception {
        for (HandlerMethodReturnValueHandler handler : returnValueHandlers) {
            if (handler.supportsReturnType(returnType)) {
                handler.handleReturnValue(returnValue, returnType, mavContainer, request, response);
                return;
            }
        }
        throw new IllegalArgumentException("Unsupported parameter type [" +
                returnType.getParameterType().getName() + "]. supportsParameter should be called first.");
    }

    public void clear() {
        this.returnValueHandlers.clear();
    }

    public void addReturnValueHandler(HandlerMethodReturnValueHandler... handlers) {
        Collections.addAll(this.returnValueHandlers, handlers);
    }
}
```



### D3: 单元测试

验证Model/Map/ViewName/View/ResponseBody是否分别能够正常工作

`具体测试代码在项目中`

### D4: 小结与延展

- 本节我们完成了5个常用返回值的解析器，支持Handler返回`Map`、` Modle`、 `View`、 `ViewName`以及被`@ResponseBody`标注；
- **和SpringMVC相比存在的不足**
  - springMVC中有数十种的返回值解析器，例如webAsyncTask，CallableMethod,StreamingResponseBody等等



## P7: InvocableHandlerMethod 

![image-20210831141033483](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2056%201635402416%201635402416301%20vdmvIr%20image-20210831141033483.png)

### D1: 概述

前两节我们完成了参数解析器和返回值解析器的开发，本节进行开发组件InvocableHandlerMethod

它是对HandlerMethod的扩展，基于一组HandlerMethodArgumentResolver，从请求上下文中解析出控制器方法的参数值，然后调用控制器方法。

### D2: 组件

#### C1: InvocableHandlerMethod

```java
/**
*	1.继承HandlerMethod，它是对HandlerMethod的扩展
*	2.初始化一个ParameterNameDiscoverer的默认实现，用于查找方法名
*	3.InvocableHandlerMethod本身是实现调用控制器方法的，所以包含了参数解析器和返回值处理器
*		过程中会使用到数据转换，定义了ConversionService
*	4.调用方法之前getMethodArgumentValues获取到方法的参数
* 5.解析完所有参数，通过反射调用控制器中的方法
* 6.执行完成后判断返回值是否为空，如果为空需要判断当前的response是否已经提交（有可能用户直接在控制的方		  法中使用response输出内容到前端），已提交标记本次请求已经处理完成mavContainer.setRequestHandled(true);		   
*/
public class InvocableHandlerMethod extends HandlerMethod {
    private ParameterNameDiscoverer parameterNameDiscoverer = new DefaultParameterNameDiscoverer();

    private HandlerMethodArgumentResolverComposite argumentResolver;
    private HandlerMethodReturnValueHandlerComposite returnValueHandler;
    private ConversionService conversionService;

    public InvocableHandlerMethod(HandlerMethod handlerMethod,
                                  HandlerMethodArgumentResolverComposite argumentResolver,
                                  HandlerMethodReturnValueHandlerComposite returnValueHandler,
                                  ConversionService conversionService) {
        super(handlerMethod);
        this.argumentResolver = argumentResolver;
        this.returnValueHandler = returnValueHandler;
        this.conversionService = conversionService;
    }

    /**
     * 调用handler
     *
     * @param request
     * @param mavContainer
     * @throws Exception
     */
    public void invokeAndHandle(HttpServletRequest request,
                                HttpServletResponse response,
                                ModelAndViewContainer mavContainer) throws Exception {

        List<Object> args = this.getMethodArgumentValues(request, response, mavContainer);
        Object resultValue = doInvoke(args);
        //返回为空
        if (Objects.isNull(resultValue)) {
            if (response.isCommitted()) {
                mavContainer.setRequestHandled(true);
                return;
            } else {
                throw new IllegalStateException("Controller handler return value is null");
            }
        }

        mavContainer.setRequestHandled(false);
        Assert.state(this.returnValueHandler != null, "No return value handler");

        MethodParameter returnType = new MethodParameter(this.getMethod(), -1);  //-1表示方法的返回值
        this.returnValueHandler.handleReturnValue(resultValue, returnType, mavContainer, request, response);

    }

    private Object doInvoke(List<Object> args) throws InvocationTargetException, IllegalAccessException {
        return this.getMethod().invoke(this.getBean(), args.toArray());
    }

    private List<Object> getMethodArgumentValues(HttpServletRequest request,
                                                 HttpServletResponse response,
                                                 ModelAndViewContainer mavContainer) throws Exception {
        Assert.notNull(argumentResolver, "HandlerMethodArgumentResolver can not null");

        List<MethodParameter> parameters = this.getParameters();
        List<Object> args = new ArrayList<>(parameters.size());
      	//遍历方法所有的参数，处理每个参数前都需要先调用initParameterNameDiscovery，
      	//然后再通过参数解析器去找到想要的参数
        for (MethodParameter parameter : parameters) {
            parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
            args.add(argumentResolver.resolveArgument(parameter, request, response, mavContainer, conversionService));
        }
        return args;
    }

    public void setParameterNameDiscoverer(ParameterNameDiscoverer parameterNameDiscoverer) {
        this.parameterNameDiscoverer = parameterNameDiscoverer;
    }
}
```

### D3: 单元测试

### D4: 小结与延展

- 本节主要内容时把前两节的参数解析器和返回值解析器串联起来完成方法的调用

- **和SpringMVC相比存在的不足**

  - SpringMVC中的`ServletInvocableHandlerMethod`、`InvocableHandlerMethod`  提供了并发相关的支持，并能够基于注解@ResponseStatus设置响应状态

    



## P8: RequestMappingHandlerAdapter

![image-20210903222411447](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2057%201635402417%201635402417140%20SrpxTt%20image-20210903222411447.png)

### D1: 概述

本节完成HandlerAdapter最后一个组件，其实现类RequestMappingHandlerAdapter的开发

RequestMappingHandlerAdapter本身在springMVC中也比较重要，虽然它知识HandlerAdapter的一种实现，但是它是使用最多的一个实现类，主要用于将某个请求适配给`@RequestMapping`类型的Handler处理

### D2: 组件

#### C1: HandlerAdapter

```java
/**
*	该接口中我们只定义了一个handle方法，但是在springMVC中还有一个supports，概述中也提到springMVC中	* HandlerAdapter有多个实现，这也是策略模式的提现，因此需要supports方法；
* 本项目中我们只做了一个实现，因此只需要一个handler就够了
*
* handler方法返回值为ModelAndView，因此HandlerAdapter中也使用到了我们之前开发过的返回值处理
*/
public interface HandlerAdapter {
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response,
                        HandlerMethod handler) throws Exception;
}
```

#### C2: ModelAndView

```java
public class ModelAndView {
    private Object view;
    private Model model;
    private HttpStatus status;

    public void setViewName(String viewName) {
        this.view = viewName;
    }

    public String getViewName() {
        return (this.view instanceof String ? (String) this.view : null);
    }

    //省略getter setter
}
```

#### C3: RequestMappingHandlerAdapter

```java
/**
*	1.考虑到框架的扩展性，类中定义了customArgumentResolvers，customReturnValueHandlers
*	  如果breezeMVC提供参数解析器和返回值处理器不满足用户的需求，允许添加自定义参数解析器和返回值处理器
*	2.afterPropertiesSet方法中我们需要把系统默认支持的以及用户自定义的参数解析器和返回值处理器添加到系统中。
*	3.当DispatcherServlet处理用户请求的时候会调用HandlerAdapter的handle方法，这时候先通过传入HandlerMethod创建之前我们已经开发完成的组件InvocableHandlerMethod，然后调用invokeAndHandle执行控制器的方法
*/
public class RequestMappingHandlerAdapter implements HandlerAdapter, InitializingBean {

    private List<HandlerMethodArgumentResolver> customArgumentResolvers;
    private HandlerMethodArgumentResolverComposite argumentResolverComposite;

    private List<HandlerMethodReturnValueHandler> customReturnValueHandlers;
    private HandlerMethodReturnValueHandlerComposite returnValueHandlerComposite;

    private ConversionService conversionService;

    @Override
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response,
                               HandlerMethod handlerMethod) throws Exception {

        InvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
        ModelAndViewContainer mavContainer = new ModelAndViewContainer();

        invocableMethod.invokeAndHandle(request, response, mavContainer);

        return getModelAndView(mavContainer);
    }

    private ModelAndView getModelAndView(ModelAndViewContainer mavContainer) {
        if (mavContainer.isRequestHandled()) {
            //本次请求已经处理完成
            return null;
        }

        ModelAndView mav = new ModelAndView();
        mav.setStatus(mavContainer.getStatus());
        mav.setModel(mavContainer.getModel());
        mav.setView(mavContainer.getView());
        return mav;
    }

    private InvocableHandlerMethod createInvocableHandlerMethod(HandlerMethod handlerMethod) {
        return new InvocableHandlerMethod(handlerMethod,
                this.argumentResolverComposite,
                this.returnValueHandlerComposite,
                this.conversionService);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Assert.notNull(conversionService, "conversionService can not null");
        if (Objects.isNull(argumentResolverComposite)) {
            List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
            this.argumentResolverComposite = new HandlerMethodArgumentResolverComposite();
            this.argumentResolverComposite.addResolver(resolvers);
        }

        if (Objects.isNull(returnValueHandlerComposite)) {
            List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
            this.returnValueHandlerComposite = new HandlerMethodReturnValueHandlerComposite();
            this.returnValueHandlerComposite.addReturnValueHandler(handlers);
        }
    }

    /**
     * 初始化默认返回值处理器
     *
     * @return
     */
    private List<HandlerMethodReturnValueHandler> getDefaultReturnValueHandlers() {
        List<HandlerMethodReturnValueHandler> handlers = new ArrayList<>();

        handlers.add(new MapMethodReturnValueHandler());
        handlers.add(new ModelMethodReturnValueHandler());
        handlers.add(new ResponseBodyMethodReturnValueHandler());
        handlers.add(new ViewNameMethodReturnValueHandler());
        handlers.add(new ViewMethodReturnValueHandler());

        if (!CollectionUtils.isEmpty(getCustomReturnValueHandlers())) {
            handlers.addAll(getDefaultReturnValueHandlers());
        }

        return handlers;
    }

    /**
     * 初始化默认参数解析器
     *
     * @return
     */
    private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
        List<HandlerMethodArgumentResolver> resolvers = new ArrayList<>();

        resolvers.add(new ModelMethodArgumentResolver());
        resolvers.add(new RequestParamMethodArgumentResolver());
        resolvers.add(new RequestBodyMethodArgumentResolver());
        resolvers.add(new ServletResponseMethodArgumentResolver());
        resolvers.add(new ServletRequestMethodArgumentResolver());

        if (!CollectionUtils.isEmpty(getCustomArgumentResolvers())) {
            resolvers.addAll(getCustomArgumentResolvers());
        }

        return resolvers;
    }

    //省略getter setter
}
```



### D3: 单元测试

本节的单元测试目标时验证RequestMappingHandlerAdapter是否能成功调用到控制器中的方法并正确返回

### D4: 小结与延展

- 本节通过开发`RequestMappingHandlerAdapter`，把我们之前开发的多个组件都组合起来了，并且能够正确的工作。
- **和SpringMVC相比存在的不足**
  - SpringMVC中`HandlerAdapter`有多个实现类，都有不同的使用方式，而`RequestMappingHandlerAdapter`是使用最多的一个



## P9: RedirectView/InternalView

![image-20210903221551788](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2058%201635402418%201635402418060%20ezbW7D%20image-20210903221551788.png)

### D1: 概述

本节将开始开发视图渲染相关组件，包括jsp视图渲染以及重定向视图的渲染

### D2: 组件

#### C1: View

```java
public interface View {
  	//获取视图支持的contentType，默认返回空
    default String getContentType() {
        return null;
    }
  	
  	//通过response把model中的数据渲染成视图返回给用户
    void render(Map<String, Object> model, HttpServletRequest request, 	 HttpServletResponse response) throws Exception;
}
```

#### C2: AbstractView

```java
/**
*	因为视图可以有很多的实现类，比如：JSON、JSP、HTML、各类模板等等，所以我们定义一个抽象类AbstractView，通过模板方法定义出渲染的基本流程
*/
public abstract class AbstractView implements View {

    @Override
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.prepareResponse(request, response);
        this.renderMergedOutputModel(model, request, response);
    }
		
  	//在实施渲染前需要做的一些工作放入到这个方法中，例如：设置响应的头信息
    protected void prepareResponse(HttpServletRequest request, HttpServletResponse response) {
    }
		
  	//将执行渲染的逻辑放入到这个方法中
    protected abstract void renderMergedOutputModel(
            Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception;

}
```

#### C3: RedirectView

```java
/**
*	1.当在控制器中返回的视图名是以redirect:开头的都将视为重定向视图；
* 2.重定向视图需要继承于AbstractView
*/
public class RedirectView extends AbstractView {
  	
  	//表示重定向的地址，实际也就是控制器中返回的视图名截取redirect:之后的字符串
    private String url;

    public RedirectView(String url) {
        this.url = url;
    }

    @Override
    protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        String targetUrl = createTargetUrl(model, request);
        response.sendRedirect(targetUrl);
    }

    /**
     * 根据url拼接出重定向地址，如果有设置contextPath，则把contextPath拼接到链接前面
     * 如果model有属性值，则将model中的数据添加到URL后面作为参数
     *
     */
    private String createTargetUrl(Map<String, Object> model, HttpServletRequest request) {
        Assert.notNull(this.url, "url can not null");

        StringBuilder queryParams = new StringBuilder();
        model.forEach((key, value) -> {
            queryParams.append(key).append("=").append(value).append("&");
        });
        if (queryParams.length() > 0) {
            queryParams.deleteCharAt(queryParams.length() - 1);
        }
        StringBuilder targetUrl = new StringBuilder();
        if (this.url.startsWith("/")) {
            // Do not apply context path to relative URLs.
            targetUrl.append(getContextPath(request));
        }

        targetUrl.append(url);

        if (queryParams.length() > 0) {
            targetUrl.append("?").append(queryParams.toString());
        }
        return targetUrl.toString();
    }

    private String getContextPath(HttpServletRequest request) {
        String contextPath = request.getContextPath();
        while (contextPath.startsWith("//")) {
            contextPath = contextPath.substring(1);
        }
        return contextPath;
    }

    public String getUrl() {
        return url;
    }
}
```

#### C4: InternalResourceView

```java
/**
*	InternalResourceView用于支持JSP，HTML的渲染
*/
public class InternalResourceView extends AbstractView {
  
  	//表示JSP文件的路径
    private String url;

    public InternalResourceView(String url) {
        this.url = url;
    }

    @Override
    public String getContentType() {
        return "text/html";
    }

    @Override
    protected void renderMergedOutputModel(
            Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        exposeModelAsRequestAttributes(model, request);

        RequestDispatcher rd = request.getRequestDispatcher(this.url);
        rd.forward(request, response);
    }

    /**
     * 把model中的数据放入到request，方便在JSP中通过el表达式取值
     *
     * @param model
     * @param request
     */
    private void exposeModelAsRequestAttributes(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((name, value) -> {
            if (Objects.nonNull(value)) {
                request.setAttribute(name, value);
            } else {
                request.removeAttribute(name);
            }
        });
    }

    public String getUrl() {
        return url;
    }
}
```

### D3: 单元测试

本次单元测试我们先只测试`RedirectView`，`InternalResourceView`放在后面整体测试；

```java
@Test
public void test() throws Exception {
    MockHttpServletRequest request = new MockHttpServletRequest();
    request.setContextPath("/path");

    MockHttpServletResponse response = new MockHttpServletResponse();

    Map<String, Object> model = new HashMap<>();
    model.put("name", "breeze2495");
    model.put("url", "http://github.breeze2495.com");

    RedirectView redirectView = new RedirectView("/redirect/login");
    redirectView.render(model, request, response);

    response.getHeaderNames().forEach(headerName ->
            System.out.println(headerName + ":" + response.getHeader(headerName)));
}
```

1. 检查重定向地址是否有拼接上ContextPath
2. 检查重定向地址是否有拼接上model中的数据

### D4: 小结与延展

- 本节我们完成了`RedirectView`、`InternalResourceView`视图，后期通过自定义视图的方式实现excel视图；
- **和SpringMVC相比存在的不足**
  - springMVC中还有视图`MappingJackson2JsonView`(model转成json)、`FreeMarkerView`（freeMark模板引擎)等，实现逻辑类似



## P10: ViewResolver

![image-20210907155533176](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2058%201635402418%201635402418859%20Jf5Yxd%20image-20210907155533176.png)

### D1: 概述

ViewResolver组件负责将ViewName解析成View对象，View对象再调用render完成渲染。

### D2: 组件

#### C1: ViewResolver

```java
public interface ViewResolver {
    View resolveViewName(String viewName) throws Exception;
}
```

#### C2: AbstractCachingViewResolver

```java
/**
*	在启动过程中，很多用户可能会请求同一个视图名称，为了避免每次都需要把viewName解析成View，我们需要做一层缓存
* 在第一次成功解析了ViewName之后把返回的View对象缓存起来，下次直接从缓存中取
*/
public abstract class AbstractCachingViewResolver implements ViewResolver {
    private final Object lock = new Object();
  	//定义一个默认空视图UNRESOLVED_VIEW，通过viewName解析不到视图返回null时，将默认空视图放到缓存中
    private static final View UNRESOLVED_VIEW = (model, request, response) -> {
    };
    private Map<String, View> cachedViews = new HashMap<>();

    @Override
    public View resolveViewName(String viewName) throws Exception {
        View view = cachedViews.get(viewName);
        if (Objects.nonNull(view)) {
            return (view != UNRESOLVED_VIEW ? view : null);
        }
				//存在同一时刻多个用户请求同一视图的可能，synchronized加锁
        synchronized (lock) {
            view = cachedViews.get(viewName);
            if (Objects.nonNull(view)) {
                return (view != UNRESOLVED_VIEW ? view : null);
            }

            view = createView(viewName);
            if (Objects.isNull(view)) {
                view = UNRESOLVED_VIEW;
            }
            cachedViews.put(viewName, view);
        }
      	//如果获取到的视图是UNRESOLVED_VIEW，直接返回null
        return (view != UNRESOLVED_VIEW ? view : null);
    }

    protected abstract View createView(String viewName);

}
```

#### C3: UrlBasedViewResolver

```java
/**
*	1. 当viewName以redirect:开头，那么返回RedirectView视图
*	2. 当viewName以forward:开头，那么返回InternalResourceView视图
* 3. 如果都不是，直接模板方法buildView
*/
public abstract class UrlBasedViewResolver extends AbstractCachingViewResolver {
    public static final String REDIRECT_URL_PREFIX = "redirect:";
    public static final String FORWARD_URL_PREFIX = "forward:";

    private String prefix = "";
    private String suffix = "";


    @Override
    protected View createView(String viewName) {
        if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
            String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
            return new RedirectView(redirectUrl);
        }

        if (viewName.startsWith(FORWARD_URL_PREFIX)) {
            String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
            return new InternalResourceView(forwardUrl);
        }

        return buildView(viewName);
    }

    protected abstract View buildView(String viewName);

    //getter setter省略
}
```

#### C4: InternalResourceViewResolver

```java
/**
*	实现了UrlBasedViewResolver中的模板方法buildView，拼接了url的前缀和后缀，返回视图InternalResourceView
*/
public class InternalResourceViewResolver extends UrlBasedViewResolver {
    @Override
    protected View buildView(String viewName) {
        String url = getPrefix() + viewName + getSuffix();
        return new InternalResourceView(url);
    }
}
```

#### C5: ContentNegotiatingViewResolver

```java
/**
*	视图协同器ContentNegotiatingViewResolver定义了所有ViewResolver以及默认支持的View，
  当接收到用户的请求后根据头信息中的Accept匹配出最优的视图
*/
public class ContentNegotiatingViewResolver implements ViewResolver, InitializingBean {
    private List<ViewResolver> viewResolvers;
    private List<View> defaultViews;

    @Override
    public View resolveViewName(String viewName) throws Exception {
      	//1.通过视图名使用ViewResolver解析出所有不为null的视图，如果默认视图不为空，则把全部默认视图添加为候选视图
        List<View> candidateViews = getCandidateViews(viewName);
       	//2.从request中拿出头信息accept，根据视图的contentType从候选视图中匹配出最优的视图返回
      	View bestView = getBestView(candidateViews);
        if(Objects.nonNull(bestView)){
            return bestView;
        }
        return null;
    }
  
      /**
     * 先找出所有候选视图
     *
     * @param viewName
     * @return
     * @throws Exception
     */
    private List<View> getCandidateViews(String viewName) throws Exception {
        List<View> candidateViews = new ArrayList<>();
        for (ViewResolver viewResolver : viewResolvers) {
            View view = viewResolver.resolveViewName(viewName);
            if (Objects.nonNull(view)) {
                candidateViews.add(view);
            }
        }
        if (!CollectionUtils.isEmpty(defaultViews)) {
            candidateViews.addAll(defaultViews);
        }
        return candidateViews;
    }
  

    /**
     * 根据请求找出最优视图
     *
     * @param candidateViews
     * @return
     */
    private View getBestView(List<View> candidateViews) {
        Optional<View> viewOptional = candidateViews.stream()
                .filter(view -> view instanceof RedirectView)
                .findAny();
        if (viewOptional.isPresent()) {
            return viewOptional.get();
        }

        HttpServletRequest request = RequestContextHolder.getRequest();
        Enumeration<String> acceptHeaders = request.getHeaders("Accept");
        while (acceptHeaders.hasMoreElements()) {
            for (View view : candidateViews) {
                if (acceptHeaders.nextElement().equals(view.getContentType())) {
                    return view;
                }
            }
        }
        return null;
    }



    @Override
    public void afterPropertiesSet() throws Exception {
        Assert.notNull(viewResolvers, "viewResolvers can not null");
    }

    //getter setter 省略
}
```

#### C6: RequestContextHolder

```java
/**
*	在当前线程中存放了当前请求的HttpServletRequest
*/
public abstract class RequestContextHolder {
    private static final ThreadLocal<HttpServletRequest> inheritableRequestHolder =
            new NamedInheritableThreadLocal<>("Request context");

    /**
     * Reset the HttpServletRequest for the current thread.
     */
    public static void resetRequest() {
        inheritableRequestHolder.remove();
    }

    public static void setRequest(HttpServletRequest request) {
        inheritableRequestHolder.set(request);
    }

    public static HttpServletRequest getRequest() {
        return inheritableRequestHolder.get();
    }
}
```

### D3: 单元测试

本节单元测试主要测试ContentNegotiatingViewResolver，检查能否正确返回视图对象。

```java
@Test
public void resolveViewName() throws Exception {
    ContentNegotiatingViewResolver negotiatingViewResolver = new ContentNegotiatingViewResolver();
    negotiatingViewResolver.setViewResolvers(Collections.singletonList(new InternalResourceViewResolver()));

    MockHttpServletRequest request = new MockHttpServletRequest();
    request.addHeader("Accept", "text/html");
    RequestContextHolder.setRequest(request);

    View redirectView = negotiatingViewResolver.resolveViewName("redirect:/breeze2495.cn");
    Assert.assertTrue(redirectView instanceof RedirectView); //判断是否返回重定向视图

    View forwardView = negotiatingViewResolver.resolveViewName("forward:/breeze2495.cn");
    Assert.assertTrue(forwardView instanceof InternalResourceView); //

    View view = negotiatingViewResolver.resolveViewName("/breeze2495.cn");
    Assert.assertTrue(view instanceof InternalResourceView); //通过头信息`Accept`，判断是否返回的`InternalResourceView`

}
```



### D4: 小结与延展

- 本节完成了ViewResolver组件的开发，对springMVC的视图解析过程有了一定的了解。

- meanWhile，也进行了synchronized锁，ThreadLocal等并发编程知识的应用，加深了先前对并发编程相关知识的理解

- **相比springMVC的不足**

  - 相较于springMVC中的ContentNegotiatingViewResolver，仅仅实现的是其简版，springmvc中通过mediaType来封装的请求信息，更加复杂。

  - springMVC中还有一些其他的Resolver，BeanNameViewResolver，XmlViewResolver等。

    

## P11: DispatcherServlet

![image-20210909100922245](https://gitee.com/breeze1002/upic/raw/master/breezeMVC/breezeMVC/2021%2010%2028%2014%2026%2059%201635402419%201635402419718%205WZQVO%20image-20210909100922245.png)

### D1: 概述

本节进行DispatcherServlet的开发

### D2: 组件

#### C1: DispatcherServlet

DispatcherServlet继承自 HttpServlet，通过使用Servlet API对HTTP请求进行响应。

其工作大致可分为两部分：

##### E1: 初始化部分

1. 当Servlet在第一次初始化的时候会调用init方法，在该方法里对诸如handlerMapping，ViewResolver等进行初始化

```java
@Override
public void init() {
    this.handlerMapping = this.applicationContext.getBean(HandlerMapping.class);
    this.handlerAdapter = this.applicationContext.getBean(HandlerAdapter.class);
    this.viewResolver = this.applicationContext.getBean(ViewResolver.class);
}
```

2. 对HTTP请求进行响应，作为一个Servlet，当请求到达时Web容器会调用其service方法; 通过`RequestContextHolder`在线程变量中设置request，然后调用`doDispatch`完成请求

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    logger.info("DispatcherServlet.service => uri:{}", request.getRequestURI());
    RequestContextHolder.setRequest(request);

    try {
        doDispatch(request, response);
    } catch (Exception e) {
        logger.error("Handler the request fail", e);
    } finally {
        RequestContextHolder.resetRequest();
    }

}
```

##### E2: doDispatch 方法执行逻辑

1. 首先通过handlerMapping获取到处理本次请求的HandlerExecutionChain
2. 执行拦截器的前置方法
3. 通过handlerAdapter执行handler返回ModelAndView
4. 执行拦截器的后置方法
5. 处理返回的结果processDispatchResult
6. 在处理完成请求后调用`executionChain.triggerAfterCompletion()`，完成拦截器的`afterCompletion`方法调用

```java
private void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    Exception dispatchException = null;
    HandlerExecutionChain executionChain = null;
    try {
        ModelAndView mv = null;
        try {
    				//1.首先通过handlerMapping获取到处理本次请求的HandlerExecutionChain
            executionChain = this.handlerMapping.getHandler(request);
						//2.执行拦截器的前置方法
            if (!executionChain.applyPreHandle(request, response)) {
                return;
            }
            //3.通过handlerAdapter执行handler返回ModelAndView
            mv = handlerAdapter.handle(request, response, executionChain.getHandler());
						//4.执行拦截器的后置方法
            executionChain.applyPostHandle(request, response, mv);
        } catch (Exception e) {
            dispatchException = e;
        }
      	// 5.处理返回的结果
        processDispatchResult(request, response, mv, dispatchException);
    } catch (Exception ex) {
        dispatchException = ex;
        throw ex;
    } finally {
        if (Objects.nonNull(executionChain)) {
          	//6.在处理完成请求后调用executionChain.triggerAfterCompletion()，完成拦截器的		afterCompletion方法调用
            executionChain.triggerAfterCompletion(request, response, dispatchException);
        }
    }

}
```

###### M1: `processDispatchResult`方法中又分为两个逻辑

- 如果时正常返回ModelAndView，那么就执行render方法；
- 如果执行过程中抛出了任何异常，就会执行processHandlerException，方便做全局异常处理

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   ModelAndView mv, Exception ex) throws Exception {
    if (Objects.nonNull(ex)) {
        //error ModelAndView
        mv = processHandlerException(request, response, ex);
    }

    if (Objects.nonNull(mv)) {
        render(mv, request, response);
        return;
    }

    logger.info("No view rendering, null ModelAndView returned.");
}
```

###### M2: processHandlerException

返回的是一个异常处理后返回的ModelAndView，处理方法本节暂不实现；下一节在实现全局异常后再进行实现

```java
//出现异常后的ModelAndView
private ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,Exception ex) {
    return null;
}
```

###### M3: render

首先通过ViewResolver解析出视图，然后在调用View的render方法实施渲染逻辑

```java
private void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response)
        throws Exception {

    View view;
    String viewName = mv.getViewName();
    if (!StringUtils.isEmpty(viewName)) {
        view = this.viewResolver.resolveViewName(viewName);
    } else {
        view = (View) mv.getView();
    }

    if (mv.getStatus() != null) {
        response.setStatus(mv.getStatus().getValue());
    }
    view.render(mv.getModel().asMap(), request, response);
}
```











































