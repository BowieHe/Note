# 简介

Servlet是专门接受客户端发来请求的小web程序，并且可以把数据回传给客户端。

要编写一个servlet程序，这个程序必须实现servlet接口。通常通过HTTP接受和响应来自web客户端的请求。要实现这个接口，可以编写一个拓展 `javax.servlet.GenericServlet `的一般servlet，或者编写一个拓展`javax.servlet.http.HttpServlet`的HTTPservlet

这两个接口定义了初始化servlet的方法，为请求提供服务的方法和从服务器移除servlet的方法。称为声明周期方法，并按照以下顺序调用

1. 构造Servlet，使用init方法初始化
2. 处理来自客户端的对servide方法的调用
3. 从服务器取出servlet，俄案后使用destroy方法销毁，最后进行垃圾回收并终止

Servlet作用：

1. 接受浏览器发送过来的信息
2. 给浏览器返回消息，浏览器认识html，可以动态去输出html

# servlet入门

sun公司定义了servlet规范

1. 实现servlet接口(javax.servlet.Servlet)
2. 重写service方法（每次请求都会被调用一次）
3. 在`WebContent/WEB_INF/web/xml`中配置servlet的访问路径。浏览器访问servlet的路径，在根标签`web-app`下直接书写以下内容

```javascript
<servlet>
  <servlet-name>hello</servlet-name> 给servlet配置一个名字
	<servlet-class>cn.itcast.servlet.hello</servlet-class> 
	hello的Java文件所在地
</servlet>
配置servlet访问地址
<servlet-mapping>
    <servlet-name>hello</servlet-name> 给哪一个servlet配置地址
		<url-pattern>/hello</url-pattern> 配置访问地址
				完整地址http://ip:端口号/工程路径/servlet访问地址
</servlet-mapping>
```

4. 把项目部署到Tomcat，启动Tomcat，在地址栏中输入信息，访问servlet

## 解析url到servlet访问细节

访问地址：`http"//127.0.0.1:8080/day/hello`
其中http是请求的协议，127.0.0.1是访问的服务器IP地址
8080是服务器电脑上，Tomcat监听的端口号
day是Tomcat的工程名， hello是工程中servlet的访问地址

1. 通过IP地址找到对应的服务器（服务器有自己的IP）
2. 通过8080找到监听的程序->Tomcat。
3. day对应Tomcat中具体工程
4. 资源地址对应一个servlet的访问地址，Tomcat会先去判断当前访问的servlet是否已经创建。如果没有就创建一个servlet。如果已经有了就调用servlet的service方法。
5. 步骤4 是根据web.xml 文件中 描述 servlet内容来找到servlet的具体实现类

![WechatIMG1](/Users/bowei/Documents/Note/Pic/WechatIMG1.jpeg)



### 模拟GET请求和POST请求的分发

```javascript
// 将servletRequest对象强转成为 HttpServletRequest对象
HttpServletRequest httpRequest =  (HttpServletRequest) req;
// 获取请求的方式
String method = httpRequest.getMethod();
// get请求获取到"GET" ,post请求获取到"POST"字符串
// 根据请求的方式 执行不同的方法去做不同的处理
if ("GET".equals(method)) {
  doGet();
} else if ("POST".equals(method)) {
  doPost();
}
}
public void doPost() {System.out.println("这是POST请求的处理方法");}
public void doGet() {System.out.println("这是GET请求的处理方法");}

//HTML表单内容
<body>
	<form action="http://127.0.0.1:8080/web06/helloServlet2" method="get">
		用户名:<input name="username" type="text" /><br/>
		密码:<input name="password" type="password" /> <br/>
		<input type="submit" />
	</form>
</body>
```

## servlet生命周期

1. 先调用servlet的构造方法
2. 调用init方法初始化servlet
3. 调用servlet中的service方法，处理请求操作
4. 调用destroy方法，执行servlet销毁操作

init方法：当服务器创建一个servlet的时候，会调用init方法。第一次访问一个servlet的时候，会创建这个servlet对象，并且只会创建一次。

当继承HttpServlet类实现Servlet程序，重写`init(Servletconfig config)`方法的时候，一定要写`super.init(config)`，不然会报空指针错误

### servlet的类继承体系

从上到下为父类到子类

1. **Interface Servlet**：servlet是JavaEE Web程序中一个组件，Servlet是一个接口，描述了Servlet实例需要实现的方法
2. **Class GenericServlet**：GenericServlet实现了Servlet接口，帮助实现很多空方法，这样在继承GenericServlet实现自己Servlet的时候，不用每个方法都实现
3. **Class HttpServlet**：HttpServlet继承了GenericServlet类。是具体Http请求Servlet分支。HttpServlet会在service中根据用户的请求方式(GET/ POST)，分发到不同的处理方式：doGet/doPost
4. **根据业务需求自己实现的HttpServlet子类**：HttpServlet实现只需要关心doGet和doPost的方法实现

# ServletConfig类

ServletConfig类是Servlet类的配置文件。封装了Servlet的配置文件信息

常用功能有三个：

1. 获取Servlet在`web.xml`文件中配置的Servlet名称(servlet-name)
2. 获取Servlet初始化信息(web.xml中<servlet>标签中<init-param>的初始化信息)
3. 获取ServletContext域对象

# ServletContext说明

定义：

1. ServletContext是一个接口
2. ServletContext是一个域对象
3. 每个Web工程，都对应一个ServletContext对象

作用：

1. ServletContext可以获取web.xml文件中的配置上下文参数
2. ServletContext可以获取web工程在服务器的工程名
3. ServletContext可以获取web工程中文件夹或文件在服务器硬盘上的绝对路径
4. ServletContext可以设置，获取web工程的全局属性

# HTTP请求协议

格式： 请求首行；请求头信息；空行；请求体

### GET请求协议格式

1. 请求行
   1. 请求方式GET
   2. 提交表单内容
2. 请求头
   1. host：请求服务器地址
   2. connection：表示请求的链接状态，需要保持一小段的链接，默认3000
   3. user-agen：客户端浏览器的信息
   4. accept：告诉服务器客户端可以接受的数据类型
   5. accept-encoding：告诉服务器客户端浏览器可以支持的数据压缩类型
   6. accept-language：告诉服务器客户端浏览器可以支持的语言类型

### POST请求协议格式

1. 请求行：`POST/hello/index.js HTTP/1.1`
   1. 请求的方式 POST
   2. 请求的资源路径
   3. 请求的协议和版本号
2. 请求头（格式： key，value ）
   1. 和GET类型一致
   2. content-length：表示请求体长度
   3. cache-control：缓冲的控制
   4. content-type：提交的内容

### GET和POST请求进行的操作

GET：

1. 在浏览器地址栏中输入地址按回车
2. 点击超链接<a>
3. GET请求表单提交，<form method="get">
4. `Script src=""`引入外部文件
5. `img src=""`引入图片
6. 引入外部CSS

POST：

只有在表单提交的时候`method="post"`，提交表单就是发post请求,`<form method=”POST” />`

## 响应的协议格式

响应行 - > 响应头 -> 空行 -> 响应体（HTML内容）

HTTP回传给客户端的数据内容：响应行，响应头，空行，响应体

**响应行**：`HTTP/1.1 200 OK` 
协议和版本 + 响应码（200） + 响应状态描述（OK）

**响应头**：

- server：表示服务器信息
- content-type：服务器传回给客户端的数据类型
- content-length：表示响应体的长度
- date：表示请求响应的时间

### 常见的响应码

- 200:请求成功，浏览器会把响应体内容显示在浏览器中
- 404:请求的资源没有找到，说明客户端错误的请求了不存在的资源
- 500:请求资源找到了，但是服务器内部出现了错误
- 302:重定向，表示服务器要求浏览器重新再发送一个请求，服务器会发送一个响应头location，指定了新请求的URL地址

## 概括HTTP请求和响应

**请求**：（POST请求中，HTTP协议由三部分，GET请求有两部分）

1. 请求行（请求方式+资源路径+请求的协议和版本号）
2. 请求头
3. 空行
4. 请求体

**响应**：

1. 响应行（HTTP协议和版本号+响应码+响应描述）
2. 响应头
3. 空行
4. 响应体

#### MIME类型

MIME(multipurpose internet mail extensions)是HTTP协议中的数据类型。MIME类型的格式是 大类型/小类型，并与某一种文件的拓展名相对应



# HttpServletRequest类

HttpServletRequest类封装了从客户端传递过来的信息。每次请求，Tomcat都会把客户端请求的信息封装在一个HttpServletRequest对象实例，传递到service请求的方法中来使用。

## Request对象常用方法

- getRequestURI()            获取请求的资源路径
- getRequestURL()            获取请求的统一资源定位符
- getRemoteHost()            获取请求的客户端ip地址
- getHeader()              获取请求头信息
- getParameter()             获取请求参数值
- getParameterValues()       获取请求的多个值（常用于复选框和下拉列表多选）。
- getRequestDispatcher()      获取请求的转发对象。转发请求（转发的请求和原请求共享request对象和response对象）
-  getMethod()             获取请求的方式GET 或 POST
- setAttribute(key, value); 设置Request请求范围的属性值。
- getAttribute(key);        获取Request请求范围的属性值。

#### Request对象的常用方法测试代码

1. 获取请求的资源路径
2. 获取请求的统一资源定位符
3. 获取请求的客户端IP地址
4. 获取请求头信息
5. 获取请求的方式GET或POST

#### 在web.xml文件中配置信息

```html
<servlet>
  <servlet-name>Request1</servlet-name>
  <servlet-class>com.atguigu.servlet.Request1</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>Request1</servlet-name>
  <url-pattern>/request1</url-pattern>
</servlet-mapping>
```

#### 直接访问http://127.0.0.1:8080/day07/request1 测试



## 获取请求参数的值

请求页面`register.html`内容

```html
<form action="http://127.0.0.1:8080/day07/params" method="get">
  用户名：<input name="username" type="text" /><br/>
  密　码：<input name="password" type="password" /><br/>
  兴趣爱好：
  <input name="hobby" type="checkbox" value="c" />C
  <input name="hobby" type="checkbox" value="cpp" />C++
  <input name="hobby" type="checkbox" value="java" />java
  <input name="hobby" type="checkbox" value="php" />php
  <br/>
  <input type="submit" />
</form>
```

接受请求参数的代码

```javascript
public class Params extends HttpServlet {
	private static final long serialVersionUID = 1L;
	protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		//获取客户端传递过来的用户名参数值
		String username = request.getParameter("username");
		System.out.println("用户名:" + username);
	
		// 获取密码
		String password = request.getParameter("password");
		System.out.println("密码：" + password);
		
		// 获取兴趣爱好,这个方法只能获取到一个兴趣爱好。
//		String hobby = request.getParameter("hobby");
//		System.out.println("getParameter得到的兴趣爱好：" + hobby);
		
		// 如果获取的参数有多个值的情况下。我们都使用getParameterValues方法来获取
		// 因为它可以获取到多个值。而getParameter只能获取到一个值。
		String[] hobbys = request.getParameterValues("hobby");
		if (hobbys != null) {
			for (String hb : hobbys) {
				System.out.println("getParameterValues兴趣爱好："  + hb);
			}
		}
	}

	protected void doPost(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}
}
```

Web.xml文件中配置信息

```xml
<servlet>
  <display-name>Params</display-name>
  <servlet-name>Params</servlet-name>
  <servlet-class>com.atguigu.servlet.Params</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>Params</servlet-name>
  <url-pattern>/params</url-pattern>
</servlet-mapping>
```



## GET请求中文参数值乱码问题解决

解决核心思路是把得到的乱码按照原来的乱码步骤逆序操作

1. 先以 iOS-8859-1进行编码
2. 再以utf-8进行解码

使用String类的方法进行解码

```java
username = new String(username.getBytes("ISO-8859-1"), "UTF-8");
```

## POST请求中文参数值乱码问题解决

POST请求方法乱码的原因是Tomcat对服务器参数的默认编码是iso-8859-1.
只需要在请求参数之前调用`request.setCharacterEncoding("UTF-8"); `设置字符集即可

```java
request.setCharacterEncoding("UTF-8");
String username = request.getParameter("username");
```

## 请求转发

客户端浏览器发起请求 -> 
servlet1收到请求之后，处理完自己的工作，发现接下来的自己并不擅长，于是移交给servlet2完成 ->
servlet2收到任务后，完成自己的任务，并把数据返还给servlet1 ->
servlet1再整合数据，最后把数据回显到页面上

- 请求转发的时候，浏览器的地址还是原来servlet1的请求路径；
- 两个servlet共享同一个request对象和response对象
- 这是一次客户端到服务器的请求

请求转发最常见的地方就是servlet程序中处理业务数据。然后把数据传给jsp页面做展示

```java
//在request中设置属性值
request.setAttribute("abc","abcvalue");
//获取request域中的属性值
String abcAttr = (String)request.getAttribute("abc");
//请求转发的对象，转发的请求路径是/servlet2
RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
//转发请求，一般forward之后就不进行任何操作了
requestDispatcher.forward(request, response);
```



# HttpServletResponse类介绍

response对象封装了服务器到客户端返回的信息

response.getOutputStream()：二进制流，主要用于返回二进制数据，如下载等
response.getWriter()：拿到字符流，大部分的返回都用writer

这两个输出流不可以同时获取两个，只能使用一个作为返回

### 往客户端回传数据

1. 往客户端输出
   1. 先获取输出流（二进制用字节流，字符用字符流）
      `Writer writer = response.getWriter();`
   2. 调用输出流对象，写出数据到客户端
      `writer.write("this is response");`

2. 访问servlet。通过response对象输出数据到客户端

### 输出中文到客户端乱码问题

那writer字符流直接输出中文内容返回到客户端就会得到乱码。
主要是因为服务器输出的字符串编码和客户端显示字符串编码不一样。

**解决**：

```java
//设置服务器输出的编码为UTF-8
response.setCharacterEncoding("UTF-8");

//告诉浏览器输出的内容是html,并且以utf-8的编码来查看这个内容。
response.setContentType("text/html;charset=utf-8");
//这两行语句要在获取输出流之前执行才会生效
```

## 设置响应状态码，和设置响应头

```java
response.setStatus(302);
// 设置响应头 Location中的信息
// 浏览器会把Location中的值重新发起一次请求。
//	response.setHeader("Location", "http://127.0.0.1:8080/day07/response2");
// 浏览器会重新 发起请求http://www.baidu.com
response.setHeader("Location", "http://www.baidu.com");
```

请求重新定向：是两次请求，浏览器地址栏回发生改变。重定向的第二次请求不仅可以是站内资源，也可以是站外资源

第一次请求返回的响应码为302。并且有响应头location的内容
第二次请求的地址就是location对应的值

## 请求重定向

除了通过设置响应状态码和响应头可以实现重定向。还可以通过HttpServletResponse对象的sendRedirect方法实现重定向

`response.sendRedirect("http://www.baidu.com")`

浏览器访问后，会有两次请求：
第一次访问`127.0.0.1:8080/day07/response3`即java代码
第二次是访问location的地址

### 请求转发和重定向的对比

|                             | 转发                      | 重定向            |
| --------------------------- | ------------------------- | ----------------- |
| 浏览器地址栏                | 不会发生变化              | 会变化            |
| 几次请求                    | 同一个请求                | 两次请求          |
| API                         | Request对象               | Response对象      |
| WEB-INF                     | 可以访问                  | 不能访问          |
| 共享request<br />请求域数据 | 可以共享                  | 不可以共享        |
| 目标资源                    | 必须是当前web应用中的资源 | 不限于当前web应用 |

