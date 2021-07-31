* **前端控制器的架构:****DispatcherServlet**

![图片](https://uploader.shimo.im/f/t1krSRW0om8mVvvD.png!thumbnail)


* **doDispatch()****细节:**
    * DispatcherServlet收到请求
        * 调用doDispatch()方法进行处理
            * **getHandler():**根据当前请求在HandlerMapping中找到这个请求的映射信息,获去目标处理器类
            * **getHandlerAdapter():**根据当前处理器类,找到当前类的HandlerAdapter(适配器)
            * 使用刚才获取到的适配器(AnnotationMethodHandlerAdapter)执行目标方法;
            * 目标方法执行后会返回一个ModelAndView对象
            * 根据ModelAndView的信息转发到具体的页面,并可以再请求域中取出ModelAndView中的模型数据;
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        try {
            ModelAndView mv = null;
            Exception dispatchException = null;
            try {
               //1、检查是否文件上传请求
                processedRequest = checkMultipart(request);
                multipartRequestParsed = processedRequest != request;
                // Determine handler for the current request.
               //2、根据当前的请求地址找到那个类能来处理；
                mappedHandler = getHandler(processedRequest);
               //3、如果没有找到哪个处理器（控制器）能处理这个请求就404，或者抛异常
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }
                // Determine handler adapter for the current request.
               //4、拿到能执行这个类的所有方法的适配器；（反射工具AnnotationMethodHandlerAdapter）
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (logger.isDebugEnabled()) {
                        String requestUri = urlPathHelper.getRequestUri(request);
                        logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
                    }
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }
                try {
                    // Actually invoke the handler.处理（控制）器的方法被调用
                    //控制器（Controller），处理器（Handler）
                    //5、适配器来执行目标方法；将目标方法执行完成后的返回值作为视图名，设置保存到ModelAndView中
                    //目标方法无论怎么写，最终适配器执行完成以后都会将执行后的信息封装成ModelAndView
                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                }
                finally {
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }
                }
                applyDefaultViewName(request, mv);//如果没有视图名设置一个默认的视图名；
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
            //转发到目标页面；
            //6、根据方法最终执行完成后封装的ModelAndView；转发到对应页面，而且ModelAndView中的数据可以从请求域中获取
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Error err) {
            triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                return;
            }
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
```


* **getHandler()****细节:怎么根据当前请求找到对应的类来处理**
    * 该方法返回的是目标处理器类的**执行链**:

![图片](https://uploader.shimo.im/f/Eq3NGx9j372byufO.png!thumbnail)

    * HandlerMapping:处理器映射
        * 里面保存了每一个处理器能处理哪些请求的映射信息

![图片](https://uploader.shimo.im/f/0eH390zZyYZS0Dey.png!thumbnail)

    * handlerMap:
        * IOC容器启动创建Controller的时候扫描每个处理器能处理什么请求,保存在HandlerMapping的handlerMap属性中;
        * 下一次请求过来时,就看哪一个HandlerMapping中有这个请求映射信息就行了;
            * BeanNameUrlHandlerMapping:基于配置;
            * DefaultAnnotationhandlerMapping:基于注解;

<img src="https://uploader.shimo.im/f/u2TniM7W0fsmORDR.png!thumbnail" alt="图片" style="zoom:67%;" />

相关代码:

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        for (HandlerMapping hm : this.handlerMappings) {
            if (logger.isTraceEnabled()) {
                logger.trace(
                        "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
            }
            HandlerExecutionChain handler = hm.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
        return null;
    }
```
* **getHandlerAdapter****():**

![图片](https://uploader.shimo.im/f/RIaWiavJHVe9Qwfz.png!thumbnail)

    * **AnnotationMethodHandlerAdapter**:能解析注解方法的适配器
        * 处理器类中只要标了注解的方法就能用;

相关代码:

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
        for (HandlerAdapter ha : this.handlerAdapters) {
            if (logger.isTraceEnabled()) {
                logger.trace("Testing handler adapter [" + ha + "]");
            }
            if (ha.supports(handler)) {
                return ha;
            }
        }
        throw new ServletException("No adapter for handler [" + handler +
                "]: The DispatcherServlet configuration needs to   a HandlerAdapter that supports this handler");
    }
```

* **SpringMVC的九大组件:**
    * 即DispatcherServlet()中几个引用类型的属性:
    * springMVC在在工作的时候,关键位置都是由这些组件完成的;
    * 共同点:九大组件全部都是接口
        * 接口就是规范;
        * 提供了强大的可扩展性;

九大组件:

```java
     /** 文件上传解析器*/
    private MultipartResolver multipartResolver;
    /** 区域信息解析器；和国际化有关 */
    private LocaleResolver localeResolver;
    /** 主题解析器；强大的主题效果更换 */
    private ThemeResolver themeResolver;
    /** Handler映射信息；HandlerMapping */
    private List<HandlerMapping> handlerMappings;
    /** Handler的适配器 */
    private List<HandlerAdapter> handlerAdapters;
    /** SpringMVC强大的异常解析功能；异常解析器 */
    private List<HandlerExceptionResolver> handlerExceptionResolvers;
    /**  */
    private RequestToViewNameTranslator viewNameTranslator;
    /** FlashMap+Manager：SpringMVC中运行重定向携带数据的功能 */
    private FlashMapManager flashMapManager;
    /** 视图解析器； */
    private List<ViewResolver> viewResolvers;
```
九大组件初始化的地方:

```java
protected void initStrategies(ApplicationContext context) {
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        initHandlerMappings(context); //*
        initHandlerAdapters(context); //*
        initHandlerExceptionResolvers(context);
        initRequestToViewNameTranslator(context);
        initViewResolvers(context);
        initFlashMapManager(context);
    }
```
初始化HandlerMapping代码:

    * 有些组件在容器中是使用**类型**找的,有些组件是使用**id**找的;
    * 取容器中找这个组件,如果没有找到就使用默认的配置;
```java
private void initHandlerMappings(ApplicationContext context) {
        this.handlerMappings = null;
        if (this.detectAllHandlerMappings) {
            // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
            Map<String, HandlerMapping> matchingBeans =
                    BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
                // We keep HandlerMappings in sorted order.
                OrderComparator.sort(this.handlerMappings);
            }
        }
        else {
            try {
                HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
                this.handlerMappings = Collections.singletonList(hm);
            }
            catch (NoSuchBeanDefinitionException ex) {
                // Ignore, we'll add a default HandlerMapping later.
            }
        }
        // Ensure we have at least one HandlerMapping, by registering
        // a default HandlerMapping if no other mappings are found.
        if (this.handlerMappings == null) {
            this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
            if (logger.isDebugEnabled()) {
                logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
            }
        }
    }
```
可在web.xml中修改DispatcherServlet某些属性的默认配置:

```xml
<init-param>
       <param-name>detectAllHandlerMappings</param-name>
       <param-value>false</param-value>
</init-param>
```
* 确定参数的值(源码分析较难)

SpringMVC确定POJO值的三步；

1、如果隐含模型中有这个key（标了ModelAttribute注解就是注解指定的value，没标就是参数类型的首字母小写）指定的值；

如果有将这个值赋值给bindObject；

2、如果是SessionAttributes标注的属性，就从session中拿；

3、如果都不是就利用反射创建对象；


---


两件事：

1）、运行流程简单版；

2）、确定方法每个参数的值；

1）、标注解：保存注解的信息；最终得到这个注解应该对应解析的值；

2）、没标注解：

1）、看是否是原生API；

2）、看是否是Model或者是Map，xxxx

3）、都不是，看是否是简单类型；paramName；

4）、给attrName赋值；attrName（参数标了@ModelAttribute("")就是指定的，没标就是""）

确定自定义类型参数：

1）、attrName使用参数的类型首字母小写；或者使用之前@ModelAttribute("")的值

2）、先看隐含模型中有每个这个attrName作为key对应的值；如果有就从隐含模型中获取并赋值

3）、看是否是@SessionAttributes(value="haha")；标注的属性，如果是从session中拿；

如果拿不到就会抛异常；

4）、不是@SessionAttributes标注的，利用反射创建一个对象；

5）、拿到之前创建好的对象，使用数据绑定器(WebDataBinder)将请求中的每个数据绑定到这个对象中；





