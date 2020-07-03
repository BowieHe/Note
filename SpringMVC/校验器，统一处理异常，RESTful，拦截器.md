# Validation

SpringMVC使用Hibernate Validation

配置校验器

```xml
<bean id="validator"
class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
  <!-- 校验器 -->
  <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
  <!-- 指定校验使用的资源文件，如果不指定则默认使用classpath下的ValidationMessages.properties -->
  <property name="validationMessageSource" ref="messageSource" />
</bean>
```

错误信息的校验文件配置

```xml
<bean id="messageSource"      class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
  <!-- 资源文件名 -->
  <property name="basenames">
    <list>
      <value>classpath:CustomValidationMessages</value>
    </list>
  </property>
  <!-- 资源文件编码格式 -->
  <property name="fileEncodings" value="utf-8" />
  <!-- 对资源文件内容缓存时间，单位秒 -->
  <property name="cacheSeconds" value="120" />
</bean>
```

添加到自定义参数绑定的WebBindingInitialzer中

```xml
<bean id="customBinder"      class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
  <!-- 配置validator -->
  <property name="validator" ref="validator" />
</bean>
```

最终添加到适配器中

```xml
<bean      class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
  <!-- 在webBindingInitializer中注入自定义属性编辑器、自定义转换器 -->
  <property name="webBindingInitializer" ref="customBinder"></property>
</bean>
```

新建Item类定义规则，同时在Controller需要在校验的参数上添加@Validation注解，拿到BindingResult对象

```java
@RequestMapping("/validation")
public void validation(@Validated Items items, BindingResult bindingResult) {
  List<ObjectError> allErrors = bindingResult.getAllErrors();
  for (ObjectError allError : allErrors) {
    System.out.println(allError.getDefaultMessage());
  }
}
```

# 分组校验

分组校验是为了使校验更加灵活。有时候并不需要把当前配置爹属性都进行校验，需要的是当前方法仅仅校验某些属性。这个时候就可以用到分组校验了。

1. 定义分组的接口（主要是标识）
2. 定义校验规则属于哪个组
3. 在Controller方法中定义使用校验分组

```java
//1.定义接口方法，只标识哪些规则属于该ValidationGroup1
public interface ValidGroup1{}
//2.通过groups指定校验属于哪个分组，指定多个分组
@NotNull(message="{item.creaate.is.notnull}",group={ValidGroup1.class})
//3.在@Validate中定义使用ValidGroup1组下的校验
@Validated(value={ValidGroup1.class})@ModelAttribute(value="itemCustom")
```

# 统一异常处理

在Java中异常分为两类，编译时异常和运行时异常

在运行时期的异常只能通过代码质量，系统测试时 详细的排除运行时异常。
对于编译时的异常，可以在代码手动处理异常可以用try/catch捕获，向上抛出

同时可以转换思路，自定义一个模块化的异常信息，比如商品类别的异常

在Spring源码中，前端控制器DispatcherServlet在进行HandlerMapping，调用HandlerAdapter执行Handler过程中，如果遇到异常，在系统中自定义统一的异常处理器，写系统自己的异常代码处理

## 定义统一异常处理器类

```java
public class CustomExceptionResolver implements HandlerExceptionResolver  {
  //前端控制器DispatcherServlet在进行HandlerMapping、调用HandlerAdapter执行Handler过程中，如果遇到异常就会执行此方法
  //handler最终要执行的Handler，它的真实身份是HandlerMethod
  //Exception ex就是接收到异常信息
  @Override
  public ModelAndView resolveException(HttpServletRequest request,HttpServletResponse response, Object handler, Exception ex) {
    //输出异常
    ex.printStackTrace();

    //统一异常处理代码
    //针对系统自定义的CustomException异常，就可以直接从异常类中获取异常信息，将异常处理在错误页面展示
    //异常信息
    String message = null;
    CustomException customException = null;
    //如果ex是系统 自定义的异常，直接取出异常信息
    if(ex instanceof CustomException){
      customException = (CustomException)ex;
    }else{
      //针对非CustomException异常，对这类重新构造成一个CustomException，异常信息为“未知错误”
      customException = new CustomException("未知错误");
    }
    //错误 信息
    message = customException.getMessage();
    request.setAttribute("message", message);
    try {
      //转向到错误 页面
      request.getRequestDispatcher("/WEB-INF/jsp/error.jsp").forward(request, response);
    } catch (ServletException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (IOException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    return new ModelAndView();
  }
}
```

## 配置统一异常处理器

```xml
 <bean class="cn.itcast.ssm.exception.CustomExceptionResolver"></bean>
```

异常处理过程时 http请求 -> DispatcherServlet -> Handler(Controller) -> service  ->  mapper。要求dao，service,controller 如果有遇到异常就向上抛出。？
异常处理器要实现HandlerExceptionResolver接口

# RESTful接口

**RESTful(Representational State Transfer)软件开发理念，RESTful对http进行非常好的诠释**。

RESTful架构

- 每一个URI代表一种资源
- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过四个HTTP动做，对服务器端资源进行操作，实现“表现层状态转换”

简单的说，如果对象在请求的过程中发生变化（比如属性被修改），那这个非幂等的。多次重复请求，结果还是不变的话，那就是幂等的（状态统一）

PUT用于幂等请求，因此在更新的时候吧所有的属性都写完整，多次请求之后，其他属性时不会变的

一般的架构并不能完全支持RESTful，因此只要系统支持RESTful的某些功能，那一般就是称为支持RESTful结构

## URL的RESTful实现

非RESTful的http的url：[http://localhost](http://localhost/):8080/items/editItems.action?id=1&....

RESTful的url是简洁的：http:// localhost:8080/items/editItems/1

## 更改DispatcherServlet配置

在上面的url中，并没有action后缀，因此需要修改颗心分配器的配置

```xml
<servlet>
  <servlet-name>springmvc_rest</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 加载springmvc配置 -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <!-- 配置文件的地址 如果不配置contextConfigLocation， 默认查找的配置文件名称classpath下的：servlet名称+"-serlvet.xml"即：springmvc-serlvet.xml -->
    <param-value>classpath:spring/springmvc.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>springmvc_rest</servlet-name>
  <!-- rest方式配置为/ -->
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

在Controller上使用PathVariable注解来绑定对应的参数

```java
@RequestMapping("/viewItems/{id}")
public @ResponseBody ItemsCustom viewItems(@PathVariable("id") Integer id) throws Exception{
  //调用 service查询商品信息
  ItemsCustom itemsCustom = itemsService.findItemsById(id);
  return itemsCustom;
}
```

当DispatcherServlet拦截，开头的所有请求，对静态资源访问就报错，需要对静态资源解析

```xml
<mvc:resources location="/js/" mapping="/js/**" />
<mvc:resources location="/img/" mapping="/img/**" />
<!-- /** 表示不管有多少层都解析， /* 表示当前层的资源-->
```

# SpringMVC拦截器

用户请求到DispatcherServlet中，DispatcherServlet调用HandlerMapping查找Handler，HandlerMapping返回返回一个拦截的链，springmvc拦截器时通过HandlerMapping发起的

实现拦截器接口

```java
public class HandlerInterceptor1 implements HandlerInterceptor {
    //在执行handler之前来执行的
    //用于用户认证校验、用户权限校验
    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {        
        System.out.println("HandlerInterceptor1...preHandle");        
        //如果返回false表示拦截不继续执行handler，如果返回true表示放行
        return false;
    }
    //在执行handler返回modelAndView之前来执行
    //如果需要向页面提供一些公用 的数据或配置一些视图信息，使用此方法实现 从modelAndView入手
    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {        System.out.println("HandlerInterceptor1...postHandle");        
    }
    //执行handler之后执行此方法
    //作系统 统一异常处理，进行方法执行性能监控，在preHandle中设置一个时间点，在afterCompletion设置一个时间，两个时间点的差就是执行时长
    //实现 系统 统一日志记录
    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex)throws Exception {        System.out.println("HandlerInterceptor1...afterCompletion");
    }
}
```

## 配置拦截器

```xml
<!--拦截器 -->
<mvc:interceptors>
  <!--多个拦截器,顺序执行 -->
  <!-- <mvc:interceptor>
            <mvc:mapping path="/**" />
            <bean class="cn.itcast.ssm.controller.interceptor.HandlerInterceptor1"></bean>
        </mvc:interceptor>
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <bean class="cn.itcast.ssm.controller.interceptor.HandlerInterceptor2"></bean>
        </mvc:interceptor> -->
  <mvc:interceptor>
    <!-- /**可以拦截路径不管多少层 -->
    <mvc:mapping path="/**" />
    <bean class="cn.itcast.ssm.controller.interceptor.LoginInterceptor"></bean>
  </mvc:interceptor>
</mvc:interceptors>
```

# 总结

- 使用Spring的校验方式就将要校验属性前面加上注解声明
- 在Controller中的方法参数上加上加上@Validation注解，那么SpringMVC内部会帮我们进行处理（创建对应的bean，加载配置文件）
- BindingResult可以那我们校验错误的信息
- 分组校验就是将让我们校验更加灵活，某方法需要校验这个属性，某方法不用校验该属性。我们就可以使用分组校验。
- 对于异常处理，SpringMVC同一个统一异常处理器类的，实现了HandlerExceptionResolver接口
- 对模块细分多个异常类，都交由我们统一异常处理器类进行处理
- 对于RESTful规范，我们可以使用SpringMVC简单的支持的。将SpringMVC的拦截.action改成是任意的。同时，如果是静态的资源文件，我们应该设置不拦截
- 对于url上的参数，我们可以使用@PathVariable将url中的{}抱起来的参数进行绑定
- SpringMVC的拦截器和Strut2拦截器差不多。但是拦截器链的调用顺序Filter是没有区别的。