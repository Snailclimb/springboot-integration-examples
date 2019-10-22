### 1. Filter 介绍

Filter 过滤器这个概念应该大家不会陌生，特别是对与从 Servlet 开始入门学 Java 后台的同学来说。那么这个东西我们能做什么呢？Filter 过滤器主要是用来过滤用户请求的，它允许我们对用户请求进行前置处理和后置处理，比如实现 URL 级别的权限控制、过滤非法请求等等。

如果我们需要自定义 Filter 的话非常简单，只需要实现 `javax.Servlet.Filter` 接口，然后重写里面的 3 个方法即可！

`Filter.java`

```java
public interface Filter {
  
   //初始化过滤器后执行的操作
    default void init(FilterConfig filterConfig) throws ServletException {
    }
   // 对请求进行过滤
    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;
   // 销毁过滤器后执行的操作，主要用户对某些资源的回收
    default void destroy() {
    }
}
```

### 2. Filter是如何实现拦截的？

`Filter`接口中有一个叫做 `doFilter` 的方法，这个方法实现了对用户请求的过滤。具体流程大体是这样的：

1. 用户发送请求到 web 服务器,请求会先到过滤器；
2. 过滤器会对请求进行一些处理比如过滤请求的参数、修改返回给客户端的 response  的内容、判断是否让用户访问该接口等等。
3. 用户请求响应完毕。
4. 进行一些自己想要的其他操作。

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/filter1.png)

### 3. 如何自定义Filter

 下面提供两种方法。

#### 3.1自己手动注册配置实现

自定义的 Filter 需要实现`javax.Servlet.Filter`接口，并重写接口中定义的3个方法。

`MyFilter.java`

```java
/**
 * @author shuang.kou
 */
@Component
public class MyFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public void init(FilterConfig filterConfig) {
        logger.info("初始化过滤器：", filterConfig.getFilterName());
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //对请求进行预处理
        logger.info("过滤器开始对请求进行预处理：");
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        String requestUri = request.getRequestURI();
        System.out.println("请求的接口为：" + requestUri);
        long startTime = System.currentTimeMillis();
        //通过 doFilter 方法实现过滤功能
        filterChain.doFilter(servletRequest, servletResponse);
        // 上面的 doFilter 方法执行结束后用户的请求已经返回
        long endTime = System.currentTimeMillis();
        System.out.println("该用户的请求已经处理完毕，请求花费的时间为：" + (endTime - startTime));
    }

    @Override
    public void destroy() {
        logger.info("销毁过滤器");
    }
}
```

`MyFilterConfig.java`

```java
@Configuration
public class MyFilterConfig {
    @Autowired
    MyFilter myFilter;
    @Bean
    public FilterRegistrationBean<MyFilter> thirdFilter() {
        FilterRegistrationBean<MyFilter> filterRegistrationBean = new FilterRegistrationBean<>();

        filterRegistrationBean.setFilter(myFilter);

        filterRegistrationBean.setUrlPatterns(new ArrayList<>(Arrays.asList("/api/*")));

        return filterRegistrationBean;
    }
}
```

#### 3.2 通过提供好的一些注解实现

在自己的过滤器的类上加上`@WebFilter` 然后在这个注解中通过它提供好的一些参数进行配置。

```java
@WebFilter(filterName = "MyFilterWithAnnotation", urlPatterns = "/api/*")
public class MyFilterWithAnnotation implements Filter {

   ......
}
```

另外，为了能让 Spring 找到它，你需要在启动类上加上 `@ServletComponentScan` 注解。

### 4.定义多个拦截器，并决定它们的执行顺序
