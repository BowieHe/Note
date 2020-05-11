# Cookie

cookie是一种服务器告诉浏览器以键值形式存储小量信息的技术

### 创建

1. 先创建一个Servlet编写创建Cookie的代码
2. 然后打开浏览器访问Servlet程序。
3. 按下F12.查看Cookie内容。

- 浏览器第一次访问服务器，会创建cookies，`Cookie cookie = new Cookie("key","value")`
- 通过浏览器保存cookie，`response.addCookie(cookie)`
- 在响应的响应头重返回cookie的信息，`set-Cookie:key=value`的形式通知浏览器保存cookie信息
- 浏览器收到响应， 生成cookie信息

```java
//Cookie创建
Cookie cookie = new Cookie("cookie-name", "cookie-Value");
// 告诉浏览器保存
response.addCookie(cookie);
response.getWriter().write("已创建Cookie……");
```

```xml
//xml文件中的配置
<servlet>
  <servlet-name>CookieServlet</servlet-name>
  <servlet-class>com.atguigu.servlet.CookieServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>CookieServlet</servlet-name>
  <url-pattern>/cookieServlet</url-pattern>
</servlet-mapping>
```

修改html页面中链接的访问地址
`<li><a href=*"cookieServlet?action=createCookie"* target=*"target"*>Cookie的创建</a></li>`

在浏览器调试工具里 resource-cookies--localhost中查看 localhost域名下的cookie

### 获取Cookie

1. 编写获取cookie的代码
2. 修改html链接，点击访问
3. 打开浏览器工具查看HTTP协议内容
4. 查看服务器代码获取cookie后的输出

- 首先要先确认浏览器中有cookie，然后向服务器发送请求
- 浏览器向服务器发送请求会在请求头中携带cookie信息
  `cookie:cookie-name=cookie.Value;`
- 服务器收到请求后，通过request对象获取cookie信息
- 获取所有cookie对象，`cookie[] cookies=request.getCookies();`，咩有则返回null

```java
// 获取所有cookie对象
Cookie[] cookies = request.getCookies();
// 如果没有cookie，则返回null。
if (cookies != null) {
  // 有cookie则遍历
  for (Cookie cookie : cookies) {
    response.getWriter().write("Cookie名：" + cookie.getName() 
                               + "<br/>Cookie值：" + cookie.getValue() + "<br/><br/>");
  }
} else {
  response.getWriter().write("没有Cookie");
}
```

修改html页面中链接的访问地址
`<li><a href="cookieServlet?action=getCookie" target="target">Cookie的获取</a></li>`

### 修改cookie值

1. 在servlet中添加修改cookie值的代码
2. 修改html页面中cookie的连接并访问
3. 打开调试工具查看，请求头和响应头中cookie的信息

- 现在浏览器中生成cookie之后，发请求去服务器修改cookie
- 浏览器向服务器发送请求，会在请求头中携带cookie信息
  `cookie-name = cookie-Value`
- 服务器收到请求后，只需要生成一个和浏览器已存在名字cookie，然后修改cookie 的值，再通知浏览器
- 服务器在给浏览器的响应中，添加响应头信息
- 浏览器收到响应头信息后，会把对应的key的cookie值修改

```java
// 创建一个已存在key的Cookie对象
Cookie cookie = new Cookie("cookie-name", null);
// 修改Cookie的值
cookie.setValue("this is new value");
// 通知浏览器保存修改
response.addCookie(cookie);
response.getWriter().write("Cookie…已修改值");
```

修改html页面中update修改cookie的访问地址
`<li><a href="cookieServlet?action=updateCookie" target="target">Cookie值的修改</a></li>`



### Cookie生命控制

.setMaxAge() 方法控制cookie存活

1. cookie的默认存活时间setMaxAge()为负数，表示会话级。即浏览器一旦关闭，cookie就会被删除
2. setMaxAge() 为0表示马上删除，一收到响应就会删除
3. setMaxAge() 正数表示多少秒之后删除

修改cookie有效时间

1. 浏览器向服务器发起请求，通过请求头携带cookie信息到服务器
2. 服务器收到cookie，查出想要修改的cookie对象，`Cookie cookie = 要修改的cookie对象`，通过setMaxAge来修改，然后通知浏览器
3. 当cookie的属性发生变化之后，会通过响应头反馈给浏览器cookie的过期时间
4. 浏览器收到cookie的信息之后，马上对相应的cookie作出修改

```java
// 获取Cookie
Cookie[] cookies = request.getCookies();
Cookie cookie = null;
if (cookies != null) {
  // 查找出我们需要修改的Cookie对象
  for (Cookie c : cookies) {
    // 获取键为cookie-name的cookie对象
    if ("cookie-name".equals(c.getName())) {
      cookie = c;
      break;
    }
  }
}
if (cookie != null) {
  //          负数表示浏览器关闭后删除，正数表示多少秒后删除
  //			 设置为零，表示立即删除Cookie
  cookie.setMaxAge(0);
  response.addCookie(cookie);
  response.getWriter().write("删除Cookie……");
}
```

修改html页面中连接地址
`<li><a href=*"cookieServlet?action=deleteCookie"* target=*"target"*>Cookie立即删除</a></li>`

### cookie路径path设置

调用cookie的对象 setPath方法就可以设置路径。

```java
// 创建一个Cookie对象
Cookie cookie = new Cookie("cookie-path", "test");
// 设置Cookie的有效访问路径为/day14/abc/下所有资源
cookie.setPath(request.getContextPath() + "/abc");
// 通知浏览器保存修改
response.addCookie(cookie);
response.getWriter().write("设置Cookie…的path路径");
```

一个一周免登录cookie设置

```java
//服务器servlet代码
protected void login(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {
  // 获取请求参数
  String username = request.getParameter("username");
  String password = request.getParameter("password");

  if ("admin".equals(username) && "admin".equals(password)) {
    // 创建Cookie
    Cookie cookie = new Cookie("username", username);
    // 设置过期时间为一个星期
    cookie.setMaxAge(60 * 60 * 24 * 7);
    // 通知浏览器保存
    response.addCookie(cookie);
    response.getWriter().write("登录成功！");
  } else {
    response.sendRedirect(request.getContextPath() + "/login.jsp");
  }
}

//WebContent/login.jsp页面
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="pragma" content="no-cache" />
<meta http-equiv="cache-control" content="no-cache" />
<meta http-equiv="Expires" content="0" />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<form action="cookieServlet?action=login" method="post">
		用户名：<input name="username" type="text" value="${ cookie.username.value }" /><br /> 
		密码：<input name="password" type="text" value="" /><br /> 
			<input type="submit" value="提 交" />
	</form>
</body>
</html>
```



# session会话

session是jsp中内置对象之一，是一个域对象
是在服务器端用来保护用户数据的一种技术，session会话技术是基于Cookie实现的

## Session的使用

### 创建和获取（ID号）

`request.getSession()`创建或获取Session对象，第一次是创建，之后都是获取
而且在浏览器关闭之后，Session失联

`session.getId()` 返回Session 的唯一编号
`session.isNew()`返回当前Session是否是刚创建的

```xml
//在web.xml文件中的配置
<servlet>
  <servlet-name>SessionServlet</servlet-name>
  <servlet-class>com.atguigu.servlet.SessionServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>SessionServlet</servlet-name>
  <url-pattern>/sessionServlet</url-pattern>
</servlet-mapping>
```

### Session数据的存取

Session域对象数据的存取和其他三个域对象`PageContext` ,`Request`, `ServletContext`一样，只需要调用下面两个方法
`setAttribute` 设置属性
`getAttribute` 获取属性

```java
//setAttribute
HttpSession session = request.getSession();
//设置数据
session.setAttribute("abc","abc value");
response.getWriter().write("设置属性成功")；

  //getAttribute
HttpSession session = request.getSession();
//获取数据
String value = (String) session.getAttribute("abc");
response.getWriter().writer("get the attribute value of abc" + value);
```

修改HTML中访问的连接地址
`<li><a href=*"sessionServlet?action=setAttribute"* target=*"target"*>Session域数据的存储</a></li>`

` <li><a href=*"sessionServlet?action=getAttribute"* target=*"target"*>Session域数据的获取</a></li>`

### Session生命周期控制

`int getMaxInactiveInterval()`获取超时时间，以秒为单位
`setMaxInactiveInterval(time)`设置用户多长时间没有操作就会session过期，以秒为单位。正数表示时间，负数则为永不过期

**Session默认存活时间**为30分钟，默认在tomcat的conf目录下web.xml配置文件中

```xml
<session-config>
  <session-timeout>30</session-timeout>
</session-config>

设置3秒后超时
//获取新session，如果已经创建就获取原来的会话
HttpSession session = request.getSession();
//设置时间。用不超时则为-1
session.setMaxInactivaInterval(3);
session.invalidate()//立即失效
```

### 浏览器和Session关联的技术内幕

1. 浏览器第一次发起请求，创建session对象，`request.getSession()`
2. 在响应中会通过cookie技术把创建的session对象的id属性返回给浏览器
   `set-cookie:JSESSIONID=......`
3. 浏览器收到响应后，会生成一个cookie
4. 之后每次请求，浏览器都会在请求头中携带这个cookie告诉服务器当前对象的会话编号
   `cookie:JSESSIONID=...`
5. `request.getSession()`调用的时候会到服务器中心去查找对应seesion id的session，如果找不到则创建新session
6. 此时浏览器如果关闭，cookie被销毁
7. 此时再次发出请求，由于cookie已经被销毁，所以没有携带cookie。此时服务器因为没有拿到session id的信息，找不到对应session，因此创建一个新session对象返回

### 关闭服务器后使用session

如果想要在浏览器关闭之后同样可以关联上之前创建的session对象，只要让JSESSIONID的cookie在浏览器关闭之后不被销毁即可。即修改key是JSESSIONID的cookie存活时间。

```java
// 第一个调用就是获取一个新的Session。如果Session已经创建过。就获取原来的会话。
HttpSession session = request.getSession();

// 手动创建一个key是JSESSIONID的Cookie
Cookie cookie = new Cookie("JSESSIONID", session.getId());
// 设置一个存活时间，确保浏览器关闭后Cookie没有销毁
cookie.setMaxAge(60 * 60 * 24);
response.addCookie(cookie);
// 输出会话id号，和是否是新创建
// session.getId()返回Session的唯一编号
// session.isNew()返回当前Session是否是刚创建的
response.getWriter().write(
  "session ID:" + session.getId() + "<br/>是否是新的：" + session.isNew());
```

