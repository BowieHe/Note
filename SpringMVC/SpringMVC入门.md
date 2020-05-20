# SpringMVC入门

SpringMVC是现在开发中流行的组件进行组合而成的一个框架，用在基于MVC的表现层开发，类似于struts2

# 回顾struts2开发

在Struts2中，开发的特点是

- Action类继承着ActionSupport类【如果要使用Struts2提供的额外功能，就要继承它】
- Action业务方法总是返回一个字符串，再由Struts2内部通过我们手写的Struts.xml配置文件去跳转到对应的view
- Action类是多例的，接收Web传递过来的参数需要使用实例变量来记住，通常我们都会写上set和get方法

## Struts2工作流程

1. Struts2接收到Request请求
2. 将请求转向我们的过滤分批器进行过滤
3. 读取Struts2对应的配置文件
4. 经过默认的拦截器之后创建对应的Action（多例）
5. 执行完业务方法就返回给Response对象

# SpringMVC快速入门

导入开发包：前六个是Spring核心功能包，第七格式Web的包，第八个是SpringMVC包

- org.springframework.context-3.0.5.RELEASE.jar
- org.springframework.expression-3.0.5.RELEASE.jar
- org.springframework.core-3.0.5.RELEASE.jar
- org.springframework.beans-3.0.5.RELEASE.jar
- org.springframework.asm-3.0.5.RELEASE.jar
- commons-logging.jar
- org.springframework.web-3.0.5.RELEASE.jar
- org.springframework.web.servlet-3.0.5.RELEASE.jar

## 编写Action

```java
public class HelloAction implements Controller {
  @Override
  public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
    ModelAndView modelAndView = new ModelAndView();

    //跳转到hello.jsp页面。
    modelAndView.setViewName("/hello.jsp");
    return modelAndView;
  }
}
```

这里只需要实现handleRequest方法就好了。该方法已经说明了request和response对象传递给我们使用了。是我们熟悉的request和response对象。但是该方法的返回时ModelAndView对象。

ModelAndView其实就是将我们的视图路径和数据封装起来而已（我们想要跳到那里，把什么数据存入request域中，设置ModelAndView的属性就好了）

## 注册核心控制器

使用SpringMVC需要在`web.xml`中配置核心控制器

```xml
<!-- 注册springmvc框架核心控制器 -->
<servlet>
  <servlet-name>DispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

  <!--到类目录下寻找我们的配置文件-->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:application-context-mvc.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>DispatcherServlet</servlet-name>
  <!--映射的路径为.action-->
  <url-pattern>*.action</url-pattern>
</servlet-mapping>
```

## 创建SpringMVC控制器

在`application-context-mvc.xml`配置文件中创建MVC控制器

`<bean class="org.demo.MvcComtroller" name="/hello.action"></bean>`

## 访问

之后配置Tomcat，运行之后。访问`http://localhost:8080/springmvcdemo_war_exploded/hello.action`的时候，Spring会读取到我们的访问路径，然后比对配置文件中是否有配置/hello.action，如果有，就交给对应的Action类来处理。Action类的业务方法将请求输出到hello.jsp页面上

# SpringMVC工作流程

- **用户发送请求**
- **请求交由核心控制器处理**
- **核心控制器找到映射器，映射器看看请求路径是什么**
- **核心控制器再找到适配器，看看有哪些类实现了Controller接口或者对应的bean对象**
- **将带过来的数据进行转换，格式化等等操作**
- **找到我们的控制器Action，处理完业务之后返回一个ModelAndView对象**
- **最后通过视图解析器来对ModelAndView进行解析**
- **跳转到对应的JSP/html页面**

## 映射器

我们在`web.xml`配置文件中规定只要是后缀名为.action的请求都会经过SpringMVC的和姓Servlet。

当我们接收到请求的时候，发现时hello.action，会经过我们的核心Servlet，因此核心Servlet回去找有没有专门的Action类来处理hello.action请求。

->>映射器就是用于处理什么样的请求交给Action处理的

在上面的例子中，`application-context-mvc.xml`的配置文件中，bean对象name属性的/hello.action就是到HelloAction控制中处理

在创建控制器的时候，也可以不使用name属性来指定路径，可以使用映射器来配置：

如果有多个请求都要交给helloAction控制器处理的话，只需要添加prop标签就可以了

```xml
<bean id="mvcComtroller" class="org.demo.MvcComtroller" name="/hello.action" />
<!-- 注册映射器(handler包)(框架) -->
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping"><!--映射器默认值，可以省略-->
  <property name="mappings">
    <props>
      <prop key="/hello.action">helloAction</prop>
      <prop key="/bye.action">helloAction</prop>
    </props>
  </property>
</bean>
```

**这里有个疑惑，如果配置mvcController没有name参数的话，会找不到该网页**

## 适配器

当映射器找到对应的Action来处理请求的时候，核心控制器就会让适配器去找该类是否实现了Controller接口（默认可以省略）

-->>适配器的功能时寻找实现了Controller接口的类

`<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>`

## 视图解析器

当把结果封装在ModelAndView之后，SpringMVC会使用视图解析器来对MpdelAndView来进行解析（默认可以省略）

有一种情况时不能省略的。在上面的例子中，我们将结果封装在ModelAndView中，使用的是绝对真实路径。但是如果使用逻辑路径，那就必须对其配置，否则SpringMVC会找不到对应的路径。

在Action中返回hello，hello是一个逻辑路径，需要使用视图解析器把逻辑路径补全

```java
public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
  ModelAndView modelAndView = new ModelAndView();
  //跳转到hello.jsp页面。
  modelAndView.setViewName("hello");
  return modelAndView;
}
```

配置视图解析器：

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <!-- 路径前缀 -->
  <property name="prefix" value="/"/>
  <!-- 路径后缀 -->
  <property name="suffix" value=".jsp"/>
  <!-- 前缀+视图逻辑名+后缀=真实路径 -->
</bean>
```

# 控制器

## ParameterizableViewController

在使用Struts2的时候，如果仅仅只是要跳转到某个WEB-INF/JSP页面，也要写一个业务方法，这个业务方法也只是返回一个简单的字符串。在SpringMVC中，如果只是跳转到某个视图，可以省略Action和业务方法，配置的Action只要继承ParameterizableViewCOntroller这个类就可以了

```xml
<!-- 专用于jsp到jsp/html的转发控制器 -->
<bean name="/ok.action" class="org.springframework.web.servlet.mvc.ParameterizableViewController">
  <!-- 转发到真实视图名 -->
  <property name="viewName" value="/WEB-INF/ok.jsp"/>
</bean>
```

## AbstractCommandController

现在讲SpringMVC是怎么接受web端传递过来的参数的

我们在Struts2中，只要在Action类上写对应的成员变量，给出对应的set和get方法。那么Struts2就会帮我们把参数封装到对应的成员变量中，是非常方便的。

在SpringMVC中获取参数是通过讲Action继承AbstractCommadController类来实现

```java
public class HelloAction extends AbstractCommandController {
  @Override
  protected ModelAndView handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, BindException e) throws Exception {
    return null;
  } 
}
```

首先SpringMVC的控制器是单例的，而Struts2的控制器是多例的。

-->Struts2收集变量是定义成员变量来进行接受，二SpringMVC作为单例，是不可能使用成员变量来接受（因为会有多个用户访问，会出现数据的不合理性）

因此SpringMVC作为单例的，只能通过方法的参数来进行接受对应的参数。只有方法能保证不同的用户对应不同的数据。

### 实体

实体的属性和web页面上name提交过来的名称是一致的

比如配置一个User类，里面有String id 和String name两个参数

### 提交参数的JSP

# 小结

当web服务器启动时，SpringMVC核心控制器，在默认的情况下会加载`/WEB-INF/<servlet-name>-servlet.xml`配置文件。映射器，适配器，视图解析器会注册bean class。程序员的Action是必须的。

### 映射器：

- web.servlet.handler.BeanNameUrlHandlerMapping(将bean标签的name属性值设为请求的url)
- web.servlet.handler.SimpleIrlHandlerMapping(多个请求，对应一个Action)
- 属于BeanNameUrlHanderMapping（可省略）

### 适配器：实现接口

- web.servlet.SimpleControllerHandlerAdapter(只会找实现了Controller接口后的后端控制器)
- simpleControllerHandlerAdapter(可省略)

### 控制器：继承类

- web.servlet.ParameterizableViewController(根据请求，直接跳转到jsp页面，不经过action)
- web.servlet.mvc.AbstractCommandController(根据请求，调用处理action，以模型方式传入参数值)
- AbstractCommandController(可省略)

### 视图器

- org.springframework.web.servlet.view.InternalResourceViewResolver(根据逻辑名，找到真实名)
- InternalResourceViewResolver(如果视图是真实的，可以省略)

### Action

- 此外写的Action必须注册到SpringIOC容器中，不可以省略

# SpringMVC工作流程

SpringMVC工作流程

- 用户发送HTTP请求，SpringMVC核心控制器接受到请求
- 找到映射器看该请求是否交由对应的Action类处理
- 找到适配器看有无该Action类
- Action类处理完结果封装到ModelAndView中
- 通过视图解析器吧数据解析，跳转到对应的JSP页面

控制器介绍：

- ParameterizableViewController：可以是心啊跳转到WEB-INF下资源，并不用写处理方法
- AbstractCommandController（已不用）实现对参数数据的封装

