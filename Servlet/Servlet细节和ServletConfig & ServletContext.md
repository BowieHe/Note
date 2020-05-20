一个被注册的Servlet可以被映射到多个URL上面，修改`web.xml`配置文件

```xml
<servlet>
  <servlet-name>Demo1</servlet-name>
  <servlet-class>zhongfucheng.web.Demo1</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>Demo1</servlet-name>
  <url-pattern>/Demo1</url-pattern>
</servlet-mapping>
<servlet-mapping>
  <servlet-name>Demo1</servlet-name>
  <url-pattern>/ouzicheng</url-pattern>
</servlet-mapping>
<servlet-mapping>
  <servlet-name>Demo1</servlet-name>
  <url-pattern>/*</url-pattern>  
  <!--为通配符,（*.拓展名）或者（/开头 /*结束）-->
</servlet-mapping>
<servlet-mapping>
  <servlet-name>Demo1</servlet-name>
  <url-pattern>*.jsp</url-pattern>
</servlet-mapping>
```

# Servlet的单例

**浏览器多次对Servlet的请求**，一般情况下，**服务器只创建一个Servlet对象**，也就是说，Servlet对象**一旦创建了**，就会**驻留在内存中，为后续的请求做服务，直到服务器关闭**。

对于每次访问请求，Servlet引擎都会**创建一个新的HttpServletRequest请求对象和一个新的HttpServletResponse响应对象**，然后将这两**个对象作为参数传递给它调用的Servlet的service()方法**，**service方法再根据请求方式分别调用doGet或者doPost方法**。

### 线程安全问题

当有多个用户访问Servlet时，**服务器会为每个用户创建一个线程**。**当多个用户并发访问Servlet共享资源的时候就会出现线程安全问题**。

原则上，如果一个变量**需要多个用户共享**，那在访问该变量时，加同步机制Synchronized(对象){}
如果一个变量不需要共享，则直接在doGet()或者doPost()中定义。

# <load-on-startup>

如果在<servlet>元素中配置了一个<load-on-startup>元素，那么**WEB应用程序在启动时**，就会**装载并创建Servlet的实例对象**、以及**调用Servlet实例对象的init()方法**。其中数字表明优先级。如果为负数或不填则第一次请求时被创建。为0或正数则在当前web应用被Servlet容器加载时创建实例

```xml
<servlet>
  <servlet-name>ServletDemo</servlet-name>
  <servlet-class>Demo1</servlet-class>
  <init-param>
    <param-name>name</param-name>
    <param-value>myservlet</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
```

**作用**：

1. 为web应用写一个InitServlet，这个**servlet配置为启动时装载**，为整个web应用**创建必要的数据库表和数据**
2. 完成一些定时的任务【定时写日志，定时备份数据】

# Web资源访问

在web上访问任何的资源都是在访问Servlet

当在web.xml文件中找不到匹配的`<servlet-mapping>`元素的URL，那这些访问请求都将交给缺省的Servlet处理，即**缺省Servlet用于处理所有其他Servlet都不处理的访问请求**

**无论在web中访问什么资源【包括JSP】，都是在访问Servlet。**没有手工配置缺省Servlet的时候，**你访问静态图片，静态网页，缺省Servlet会在你web站点中寻找该图片或网页**，如果有就返回给浏览器，没有就报404错误

# ServletConfig

ServletConfig对象可以读取`web.xml`中配置的初始化参数

在`web.xml`中为Servlet添加一个配置参数

```xml
<servlet>
  <servlet-name>ServletDemo</servlet-name>
  <servlet-class>Demo1</servlet-class>
  <init-param>
    <param-name>name</param-name>
    <param-value>myservlet</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
```

在Servlet中获取ServletConfig对象，通过ServletConfig对象获取在`web.xml`文件配置的参数

```java
protected void doPost(javax.servlet.http.HttpServletRequest req, javax.servlet.http.HttpServletResponse resp) throws ServletException, IOException {

  ServletConfig servletConfig = this.getServletConfig();
  String value = servletConfig.getInitParameter("name");
  System.out.println(value);
}
```

# ServletContext对象

在Tomcat启动的时候，就会创建一个ServletContext对象，代表当前Web站点。

**作用**：

1. ServletContext既然代表着当前web站点，那么**所有Servlet都共享着一个ServletContext对象**，所以**Servlet之间可以通过ServletContext实现通讯**。
2. ServletConfig获取的是配置的是单个Servlet的参数信息，**ServletContext可以获取的是配置整个web站点的参数信息**
3. **利用ServletContext读取web站点的资源文件**
4. 实现Servlet的转发【用ServletContext转发不多，主要用request转发】

### Servlet之间实现通讯

ServletContext对象也可以被称为域对象，域对象可以简单的理解成一个容器（类似Map集合）

实现Servlet之间的通讯要用到ServletContext的serAttribute(String name, Object obj)方法，第一个参数是关键字，第二个参数是要储存的对象

```java
//save the value in Demo 2
//获取到ServletContext对象
ServletContext servletContext = this.getServletContext();
String value = "value";
//MyName作为关键字，value作为值存进   域对象【类型于Map集合】
servletContext.setAttribute("MyName", value);

//read the value in Demo3
//获取ServletContext对象
ServletContext servletContext = this.getServletContext();
//通过关键字获取存储在域对象的值
String value = (String) servletContext.getAttribute("MyName");
System.out.println(value);
```

在配置文件`web.xml`中添加如下配置来设置两个Servlet

```xml
<servlet>
  <servlet-name>ServletDemo</servlet-name>
  <servlet-class>Demo1</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>ServletDemo</servlet-name>
  <url-pattern>/demo</url-pattern>
</servlet-mapping>
<servlet>
  <servlet-name>demo3</servlet-name>
  <servlet-class>org.example.Demo3</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>demo3</servlet-name>
  <url-pattern>/demo3</url-pattern>
</servlet-mapping>
```

## 获得Web站点配置信息

web.xml文件支持对整个站点进行配置参数信息（所有的Servlet都可以取到该参数信息）

```XML
<context-param>
  <param-name>name</param-name>
  <param-value>zhongfucheng</param-value>
</context-param>
```

在Java类中添加下列代码来实现配置信息获取

```JAVA
//获取到ServletContext对象
ServletContext servletContext = this.getServletContext();
//通过名称获取值
String value = servletContext.getInitParameter("name");
System.out.println(value);
```

## 读取资源文件

之前在Java中如果程序和文件在同一包名下，可以直接通过文件名获取到资源信息。但这是在JVM中运行得到的，现在通过Tomcat运行，需要用另外的方式

1. 根据web目录规范，Servlet编译后的class文件会存放在WEB-INF中的classes文件夹。因此通过以下代码可以获得

   正常情况下需要写绝对路径，但是如果将读取文件的模块转移到其他web站点，代码需要重新修改。

   那可以通过ServletContext读取文件，因为ServletContext对象是根据当前web站点生成的

```java
FileInputStream fileInputStream = new FileInputStream("Servletdemo/web/WEB-INF/classes/web/1.png");
System.out.println(fileInputStream);
```

2. 如果文件放在web目录下，直接通过文件名就可以获取文件

```java
//获取到ServletContext对象
ServletContext servletContext = this.getServletContext();

//调用ServletContext方法获取到读取文件的流
InputStream inputStream = servletContext.getResourceAsStream("filename");
```

3. 通过类装载器读取资源文件

   将文件放在类目录下，或者包下，都可以通过如下代码获得。但是如果文件过大则不能这样读取，会导致内存溢出

```java
//获取到类装载器
ClassLoader classLoader = Servlet111.class.getClassLoader();

//通过类装载器获取到读取文件流
InputStream inputStream = classLoader.getResourceAsStream("3.png");
InputStream inputStream = classLoader.getResourceAsStream("/web/1.png");
```

