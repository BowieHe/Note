# SpringMVC过滤编码器

和Servlet一样，在SpringMVC的控制器中，如果没有对编码器进行任何的操作，获得到的中文数据都是乱码的。

即使在handle() 方法中，使用request对象设置编码也不行。因为SpringMVC接受参数是通过控制器中的无餐构造方法，再经过handle()方法的object对象得到的具体的参数类型。

在PSringMVC中只需要要web.xml配置文件中设置过滤编码器就可以解决问题了。

```xml
<!-- 编码过滤器 -->
<filter>
  <filter-name>CharacterEncodingFilter</filter-name>
  <filter-class>
    org.springframework.web.filter.CharacterEncodingFilter
  </filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CharacterEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 注解开发SpringMVC

上面是通过XML来配置开发SpringMVC，和Spring一样，SpringMVC也支持注解。

在使用Action接口的时候，要么继承AbstractCommandController类，或者使用注解Controller接口。当使用了注解之后，就不用显示继承任何类了。

使用@Controller这个注解，就表明这是一个SpringMVC的控制器.

```java
@Controller
public class Action{
  /**
     * @RequestMapping 表示只要是/hello.action的请求，就交由该方法处理。当然了.action可以去掉
     * @param model 它和ModelAndView类似，它这个Model就是把数据封装到request对象中，我们就可以获取出来
     * @return 返回跳转的页面【真实路径，就不用配置视图解析器了】
     */
    @RequestMapping(value="/hello.action")
    public String hello(Model model) throws Exception{
        System.out.println("HelloAction::hello()");
        model.addAttribute("message","你好");
        return "/index.jsp";
    }
}
```

同时需要在配置文件中配置扫描注解。在配置扫描路径的时候，不要加上.*

` <context:component-scan base-package="com.pojo"/>`

之后会跳转到/index.jsp文件，首页就能得到对应的值

-----

基于注解和基于XML来开发SpringMVC都是通过映射器，适配器和视图解析器的。只是映射器，适配器略有不同，但都是可以省略的

```xml
<!-- 基于注解的映射器(可选) -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

<!-- 基于注解的适配器(可选) -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

<!-- 视图解析器(可选) -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
```

# RequestMapping

@RequestMapping可以控制请求路径和请求方式

## 一个控制器写多个业务方法

通过@RequestMapping可以配置哪个请求对应哪个业务方法

当请求hello.action的时候，业务方法是hello，请求 bye.action的时候业务方法是bye

```java
@Controller
public class HelloAction {
  /**
     * @RequestMapping 表示只要是/hello.action的请求，就交由该方法处理。当然了.action可以去掉
     * @param model 它和ModelAndView类似，它这个Model就是把数据封装到request对象中，我们就可以获取出来
     * @return 返回跳转的页面【真实路径，就不用配置视图解析器了】
     */
  @RequestMapping(value="/hello.action")
  public String hello(Model model) throws Exception{
    System.out.println("HelloAction::hello()");
    model.addAttribute("message","你好");
    return "/index.jsp";
  }
  @RequestMapping(value = "/bye.action")
  public String bye(Model model) throws Exception {
    model.addAttribute("message","再见");
    return "/index.jsp";
  }
}
```

## 分模块开发

使用@RequestMapping注解同样可以实现分模块开发，只要把@RequestMapping这个注解写到类上面，就代表了分模块

```java
@COntroller
@RequestMapping("/mydemo")
public class Action{}
```

那么如果想要Action控制器处理我们的请求，访问的地址是http://localhost:8080/zhongfucheng/XXX

## 限定某个业务控制方法，只允许GET或者POST请求方式

通过设置@RequestMapping的method属性可以控制业务方法

```java
@RequestMapping(value = "/bye.action",method = RequestMethod.POST)
public String bye(Model model) throws Exception {
  model.addAttribute("message", "再见");
  return "/index.jsp";
}
```

此时如果使用GET方式来访问该业务则失败

# 业务方法写入传统web参数

业务方法除了可以写Model参数之外，如果有需要还可以写Request，Response等传统Servlet参数，但是并不建议使用传统的web参数，会耦合

```java
@RequestMapping(method=RequestMethod.POST,value="/register")
public String registerMethod(HttpServletRequest request,HttpServletResponse response) throws Exception{

  //获取用户名和薪水
  String username = request.getParameter("username");
  String salary = request.getParameter("salary");
  System.out.println("用户注册-->" + username + ":" + salary);

  //绑定到session域对象中
  request.getSession().setAttribute("username",username);
  request.getSession().setAttribute("salary",salary);

  //重定向/jsp/success.jsp页面
  //response.sendRedirect(request.getContextPath()+"/jsp/success.jsp");

  //转发/jsp/ok.jsp页面
 request.getRequestDispatcher("/jsp/ok.jsp").forward(request,response);

  //转发(提倡)
  return "/jsp/success.jsp";
}
```

如果我们返回值是一个真实路径，而在程序中又使用了转发或者重定向，那么具体跳转的位置就是按照程序中跳转的路径为准

# 业务方法收集参数

SpringMVC收集参数

- 业务方法上写上参数
- 参数的名称要和web端带来的数据名称一致

## 接受普通参数

普通参数可以直接在方法上写上与web端带来的名称相同的参数就可以了

```jsp
<form action="${pageContext.request.contextPath}/hello.action" method="post">
  <table align="center">
    <tr>
      <td>用户名：</td>
      <td><input type="text" name="username"></td>
    </tr>
    <tr>
      <td>编号</td>
      <td><input type="text" name="id"></td>
    </tr>
    <tr>
      <td colspan="2">
        <input type="submit" value="提交">
      </td>
    </tr>
  </table>
</form>
```

```java
@RequestMapping(value = "/hello.action")
public String hello(Model model, String username, int id) throws Exception {
  System.out.println("用户名是：" + username);
  System.out.println("编号是：" + id);
  model.addAttribute("message", "你好");
  return "/index.jsp";
}
```

## 接受JavaBean

处理表单的参数，如果表单带来的数据较多，可以通过JavaBean对其进行封装。在SpringMVC也可以这么做。

- 创建JavaBean
- JavaBean属性与表单带过来的名称相同
- 在业务方法上写上JavaBean的名称

```java
//新建一个User类，里面有String id和String username
//在业务方法上写上JavaBean
@RequestMapping(value = "/hello.action")
public String hello(Model model,User user) throws Exception {

  System.out.println(user);
  model.addAttribute("message", "你好");
  return "/index.jsp";
}
```

## 收集数组

收集数组和收集普通的参数是类似的

```jsp
<form action="${pageContext.request.contextPath}/hello.action" method="post">
  <table align="center">
    <tr>
      <td>用户名：</td>
      <td><input type="text" name="username"></td>
    </tr>
    <tr>
      <td>爱好</td>
      <td><input type="checkbox" name="hobby" value="1">篮球</td>
      <td><input type="checkbox" name="hobby" value="2">足球</td>
      <td><input type="checkbox" name="hobby" value="3">排球</td>
      <td><input type="checkbox" name="hobby" value="4">羽毛球</td>
    </tr>
    <tr>
      <td colspan="2">
        <input type="submit" value="提交">
      </td>
    </tr>
  </table>
</form>
```

```java
@RequestMapping(value = "/hello.action")
public String hello(Model model,int[] hobby) throws Exception{
  for (int i : hobby) {
    System.out.println("喜欢运动的编号是：" + i);
  }
  model.addAttribute("message", "你好");
  return "/index.jsp";
}
```

## 收集List<JavaBean>集合

在Spring的业务方法里面是不能用List<JavaBean>这样的参数接收的，因此SpringMVC里使用另外一种方案

使用一个JavaBean吧集合封装起来，给出对应的set和get方法，这样在接受参数的时候，接收到的就是JavaBean

```java
/**
 * 封装多个Emp的对象 
 * @author AdminTC
 */
public class Bean {
  private List<Emp> empList = new ArrayList<Emp>();
  public Bean(){}
  public List<Emp> getEmpList() {
    return empList;
  }
  public void setEmpList(List<Emp> empList) {
    this.empList = empList;
  }
}

//业务方法接受JavaBean
/**
     * 批量添加员工
     */
@RequestMapping(value="/addAll",method=RequestMethod.POST)
public String addAll(Model model,Bean bean) throws Exception{
  for(Emp emp:bean.getEmpList()){
    System.out.println(emp.getUsername()+":"+emp.getSalary());
  }
  model.addAttribute("message","批量增加员工成功");
  return "/jsp/ok.jsp";
}
```

## 收集多个模型

在JSP页面可能有多个模型，比如User和Emp模型数据要收集，并且User模型的属性和Emp模型的属性一样。此时可以在User模型和Emp模型上向上抽象出一个Bean，改Bean有Emp和User对象

```java
/**
 * 封装User和Admin的对象
 * @author AdminTC
 */
public class Bean {
  private User user;
  private Admin admin;
  public Bean(){}
  public User getUser() {
    return user;
  }
  public void setUser(User user) {
    this.user = user;
  }
  public Admin getAdmin() {
    return admin;
  }
  public void setAdmin(Admin admin) {
    this.admin = admin;
  }
}
```

```jsp
<form action="${pageContext.request.contextPath}/person/register.action" method="POST">
  <table border="2" align="center">
    <tr>
      <th>姓名</th>
      <td><input type="text" name="user.username" value="${user.username}"/></td>
    </tr>
    <tr>
      <th>月薪</th>
      <td><input type="text" name="user.salary" value="${user.salary}"></td>
    </tr>
    <tr>
      <th>入职时间</th>
      <td><input 
                 type="text" 
                 name="user.hiredate" 
                 value='<fmt:formatDate value="${user.hiredate}" type="date" dateStyle="default"/>'/></td>
    </tr>
    <tr>
      <td colspan="2" align="center">
        <input type="submit" value="普通用户注册" style="width:111px"/>
      </td>
    </tr>
  </table>    
</form>    
```

# 字符串转日期类型

在SpingMVC中，对于yyyy-mm-dd hh:MM:ss这种格式的日期类型SpringMVC也是不能帮忙自动解析的。

如果使用的是继承AbstractCommandController的类来开发，则可以通过重写initBinder方法来实现，可以通过注解@InitBinder来重写该方法

```java
//构建User对象来接受时间变量 getDate&& setDate
//业务方法获得Date值
@RequestMapping(value = "/hello.action")
public String hello(Model model, User user) throws Exception {
  System.out.println(user.getUsername() + "的出生日期是：" + user.getDate());
  model.addAttribute("message", "你好");
  return "/index.jsp";
}

//重写方法转换日期格式
@InitBinder
protected void initBinder(HttpServletRequest request, ServletRequestDataBinder binder) throws Exception {
  binder.registerCustomEditor(
    Date.class,
    new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
}
```

# 结果重定向和转发

在SpringMVC中，如果要实现重定向，直接在跳转前加上关键字就好了

同理，如果想再次请求的话，只要写上对应的请求路径即可

```java
@RequestMapping(value = "/hello.action")
public String hello(Model model, User user) throws Exception {
  System.out.println(user.getUsername() + "的出生日期是：" + user.getDate());
  model.addAttribute("message", user.getDate());
  return "redirect:/index.jsp";
}

@RequestMapping("/bye.action")
public String bye() throws Exception {
  System.out.println("我进来了bye方法");
  return "/index.jsp";
}
```

# 返回JSON文本

首先导入JSON开发包

- **jackson-core-asl-1.9.11.jar**
- **jackson-mapper-asl-1.9.11.jar**

在要返回JSON的业务方法上给注解，并配置JSON适配器

```java
@RequestMapping(value = "/hello.action")
public
  @ResponseBody
  User hello() throws Exception {
  User user = new User("1", "zhongfucheng");
  return user;
}
```

```xml
<!--  
1）导入jackson-core-asl-1.9.11.jar和jackson-mapper-asl-1.9.11.jar
2）在业务方法的返回值和权限之间使用@ResponseBody注解表示返回值对象需要转成JSON文本
3）在spring.xml配置文件中编写如下代码：
-->
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
  <property name="messageConverters">
    <list>
      <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"/>
    </list>
  </property>
</bean>
```

# 总结

- 使用注解的开发避免了继承多余的类，并且非常简洁高效。
- **想要中文不乱码，仅仅设置request的编码格式是不行的。因为SpringMVC是通过无参的构造器将数据进行封装的**。我们可以使用SpringMVC提供的过滤器来解决中文乱码问题。
- **RequestMapping可以设置我们具体的访问路径，还可以分模块开发**。基于这么两个原因，我们就可以在一个Action中写多个业务方法了。
- RequestMapping还能够限制该请求方法是GET还是POST。
- **在我们的业务方法中，还可以使用传统的request和response等对象**，只不过如果不是非要使用的话，最好就别使用了。
- **对于SpringMVC自己帮我们封装参数，也是需要使用与request带过来的名称是相同的。如果不相同的话，我们需要使用注解来帮我们解决的。**
- 如果是需要封装成集合，或者封装多个Bean的话，那么我们**后台的JavaBean就需要再向上一层封装，在业务方法上写上Bean进行了。当然了，在web页面上要指定对应Bean属性的属性**。
- 字符串转日期对象用到 **@InitBinder注解来重写方法。**
- **返回JSON对象，我们就需要用到@ResponseBody注解，如果接收JSON数据封装成JavaBean的话，我们就需要用到@RequestBody注解。随后在配置文件上创建对应的bean即可。**

