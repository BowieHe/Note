# HttpServletRequest

> HttpServletRequest对象代表客户端的请求，当客户端通过HTTP协议访问服务器时，HTTP请求头中的所有信息都封装在这个对象中，开发人员通过这个对象的方法，可以获得客户这些信息。

简单来说，**要得到浏览器信息，就找HttpServletRequest对象**

# HttpServletRequest常用方法

## 获得浏览器信息

- **getRequestURL方法返回客户端发出请求时的完整URL。**
- getRequestURI方法返回请求行中的资源名部分。
- **getQueryString 方法返回请求行中的参数部分。**
- getPathInfo方法返回请求URL中的额外路径信息。额外路径信息是请求URL中的位于Servlet的路径之后和查询参数之前的内容，它以“/”开头。
- **getRemoteAddr方法返回发出请求的客户机的IP地址**
- getRemoteHost方法返回发出请求的客户机的完整主机名
- getRemotePort方法返回客户机所使用的网络端口号
- **getLocalAddr方法返回WEB服务器的IP地址。**
- getLocalName方法返回WEB服务器的主机名

## 获得客户机请求头

- **getHeader方法**
- getHeaders方法
- getHeaderNames方法

## 获得客户机请求参数（客户机提交的数据）

- **getParameter方法**
- **getParameterValues（String name）方法**
- getParameterNames方法
- getParameterMap方法

# HttpServletRequest应用

## 防盗链

防盗链就是我有一个主页，上面有一个超链接指向资源页。为了防止别人直接访问资源页，添加防盗链，这样别人必须从主页开始访问

想要实现这样的功能需要获取Refresher这个消息头，来判定Refresher是不是从我的首页过来的，如果不是从我的首页过来的，则跳转回我的首页

```java
//获取到网页是从哪里来的
String referer = request.getHeader("Referer");

//如果不是从我的首页来或者从地址栏直接访问的，
if ( referer == null || !referer.contains("localhost:8080/zhongfucheng/index.jsp") ) {

  //回到首页去
  response.sendRedirect("/zhongfucheng/index.jsp");
  return;
}

//能执行下面的语句，说明是从我的首页点击进来的，那没问题，照常显示
response.setContentType("text/html;charset=UTF-8");
response.getWriter().write("路飞做了XXXXxxxxxxxxxxxxxxxx");
```

## 表单数据提交（POST）

```html
<form action="/zhongfucheng/Servlet111" method="post">
  <table>
    <tr>
      <td>用户名</td>
      <td><input type="text" name="username"></td>
    </tr>
    <tr>
      <td>密码</td>
      <td><input type="password" name="password"></td>
    </tr>
    <tr>
      <td>性别</td>
      <td>
        <input type="radio" name="gender" value="男">男
        <input type="radio" name="gender" value="女">女
      </td>
    </tr>
    <tr>
      <td>爱好</td>
      <td>
        <input type="checkbox" name="hobbies" value="游泳">游泳
        <input type="checkbox" name="hobbies" value="跑步">跑步
        <input type="checkbox" name="hobbies" value="飞翔">飞翔
      </td>
    </tr><!--隐藏域-->
    <input type="hidden" name="aaa" value="my name is zhongfucheng">
    <tr>
      <td>你的来自于哪里</td>
      <td>
        <select name="address">
          <option value="广州">广州</option>
          <option value="深圳">深圳</option>
          <option value="北京">北京</option>
        </select>
      </td>
    </tr>
    <tr>
      <td>详细说明:</td>
      <td>
        <textarea cols="30" rows="2" name="textarea"></textarea>
      </td>
    </tr>
    <tr>
      <td><input type="submit" value="提交"></td>
      <td><input type="reset" value="重置"></td>
    </tr>
  </table>
```

Java中获取到提交的数据

```java
//设置request字符编码的格式
request.setCharacterEncoding("UTF-8");

//通过html的name属性，获取到值
String username = request.getParameter("username");
String password = request.getParameter("password");
String gender = request.getParameter("gender");

//复选框和下拉框有多个值，获取到多个值
String[] hobbies = request.getParameterValues("hobbies");
String[] address = request.getParameterValues("address");

//获取到文本域的值
String description = request.getParameter("textarea");

//得到隐藏域的值
String hiddenValue = request.getParameter("aaa");
```

## 超链接方式提交数据

常见的Get方式提交数据有使用超链接，sendRedirect()，格式如下

`sendRedirect("servlet的地址?参数名="+参数值 &"参数名="+参数值);`

通过超链接将数据带给浏览器，通过如下代码

`<a href="/zhongfucheng/Servlet111?username=xxx">使用超链接将数据带给浏览器</a>`

在服务器端，通过如下代码可以接受数据

```java
//接收以username为参数名带过来的值
String username = request.getParameter("username");
System.out.println(username);
```

## Get和Post的中文乱码问题

在使用post方法获取表单数据的时候，通过添加代码
`request.setCharacterEncoding("UTF-8");`
可以解决乱码问题，但是这种方法不适用于get

Post方法在传递参数的时候。是在我们按下提交按钮之后，数据封装进了Form Data中，http请求中吧实体主体带过去了（传输的数据称之为实体主体），因为对request对象封装了http请求，所以request对象可以解析到发送过来的数据，只要吧编码格式设置成UTF-8就可以解决乱码问题了

而Get方法的数据是通过消息行带过去的，没有封装到request对象里，所以要使用request设置编码是无效的。

即然Tomcat的默认编码格式是IOS-8859-1，那么get方式由消息体带过去给浏览器的时候用的也是IOS-8859-1编码。因此可以通过反向查找的方式来获取字符串

```java
//此时得到的数据已经是被ISO 8859-1编码后的字符串了，这个是乱码
String name = request.getParameter("username");

//乱码通过反向查ISO 8859-1得到原始的数据
byte[] bytes = name.getBytes("ISO8859-1");

//通过原始的数据，设置正确的码表，构建字符串
String value = new String(bytes, "UTF-8");
```

此外get方法也可以使用修改Tomcat配置来实现，在8080端口的Connector上加入`URIEncoding="utf-8"`，这种改法固定使用UTF-8来编码，不推荐使用

```XML
<Connector port="8080" protocol="HTTP/1.1" 
           connectionTimeout="20000" 
           redirectPort="8443" URIEncoding="utf-8"/>
```

或者设置Tomcat的访问该端口时编码为页面编码，这种改法随着页面的编码而改变

```XML
<Connector port="8080" protocol="HTTP/1.1" 
           connectionTimeout="20000" 
           redirectPort="8443" URIEncoding="utf-8"/>
```

设置编码为UTF-8

```java
request.setCharacterEncoding("UTF-8");
String name = request.getParameter("name");
```

## 实现转发

之前的response的sendResirect()可以实现重定向，做到的是页面跳转。
使用request的getRequestDispatcher.forward(request,response)实现转发实现的功能也是页面跳转，代码如下

```java
//获取到requestDispatcher对象，跳转到index.jsp
RequestDispatcher requestDispatcher = request.getRequestDispatcher("/index.jsp");

//调用requestDispatcher对象的forward()实现转发,传入request和response方法
requestDispatcher.forward(request, resp);
```

通过sendRedirect()重定向可以在资源尾部添加参数提交的数据给服务器，那么穿法也可以把数据提交给服务器。

之前说过ServletContext可以用来实现实时通讯，ServletContext之所以称为与对象，二request可以称为域对象。只是ServletContext的域是整个web应用，而request只代表一个http请求，下面代码通过request实现了Servlet之间的通讯

```java
//Demo1
//以username为关键字存zhongfucheng值
request.setAttribute("username", "zhongfucheng");
//获取到requestDispatcher对象
RequestDispatcher requestDispatcher = request.getRequestDispatcher("/Servlet222");
//调用requestDispatcher对象的forward()实现转发,传入request和response方法
requestDispatcher.forward(request, response);

//Demo2
//获取到存进request对象的值
String userName = (String) request.getAttribute("username");
//在浏览器输出该值
response.getWriter().write("i am :"+userName);
```

通过上述代码，demo2拿到了request对象在demo1中存进去的数据。

相比ServletContext，能用request就用request，因为ServletContext代表着整个web应用，使用时会消耗大量的资源。而request对象会随着请求的结束而结束，资源会被回收。使用request在Servlet之间通讯是很常见的。

## 转发时序

1. 发送http请求
2. 解析主机
3. 解析出web域名
4. 解析出资源名
5. 查询demo1的路径
6. init
7. 创建request，response对象，并传递给doPost
8. doPost()（转发在服务器上发生）
9. 告诉web服务器要找demo2
10. 查询demo2
11. 调用demo2
12. response对象返回
13. 把response的信息拆解
14. 把结果返回

## 请求转发的细节

如果在**调用forward方法之前**，在**Servlet程序中写入的部分内容已经被真正地传送到了客户端**，forward方法将抛出IllegalStateException异常。 也就是说：**不要在转发之前写数据给浏览器**。

如果在调用forward方法之前向Servlet引擎的缓冲区中写入了内容，**只要写入到缓冲区中的内容还没有被真正输出到客户端**，forward方法就可以被正常执行，**原来写入到输出缓冲区中的内容将被清空**，但是，**已写入到HttpServletResponse对象中的响应头字段信息保持有效**。

## 转发和重定向的区别

### 实际发生位置不同，地址栏不同

- 转发是发生在服务器的
  - **转发是由服务器进行跳转的**，细心的朋友会发现，在转发的时候，**浏览器的地址栏是没有发生变化的**，在我访问Servlet111的时候，即使跳转到了Servlet222的页面，浏览器的地址还是Servlet111的。也就是说**浏览器是不知道该跳转的动作，转发是对浏览器透明的**。通过上面的转发时序图我们也可以发现，**实现转发只是一次的http请求**，**一次转发中request和response对象都是同一个**。这也解释了，为什么可以使用**request作为域对象进行Servlet之间的通讯。**
- 重定向是发生在浏览器的
  - **重定向是由浏览器进行跳转的**，进行重定向跳转的时候，**浏览器的地址会发生变化的**。曾经介绍过：实现重定向的原理是由response的状态码和Location头组合而实现的。**这是由浏览器进行的页面跳转**实现重定向**会发出两个http请求**，**request域对象是无效的，因为它不是同一个request对象**

### 用法不同

很多人都搞不清楚转发和重定向的时候，**资源地址究竟怎么写**。有的时候要把应用名写上，有的时候不用把应用名写上。很容易把人搞晕。记住一个原则：**给服务器用的直接从资源名开始写，给浏览器用的要把应用名写上**

- request.getRequestDispatcher("/资源名 URI").forward(request,response)
  - **转发时"/"代表的是本应用程序的根目录【zhongfucheng】**
- response.send("/web应用/资源名 URI");
  - **重定向时"/"代表的是webapps目录**

### 能够去往的URL的范围不一样

- **转发是服务器跳转只能去往当前web应用的资源**
- **重定向是浏览器跳转，可以去往任何的资源**

### 传递数据的类型不同

- **转发的request对象可以传递各种类型的数据，包括对象**
- **重定向只能传递字符串**

### 跳转的时间不同

- **转发时：执行到跳转语句时就会立刻跳转**
- **重定向：整个页面执行完之后才执行跳转**

### 使用场景

根据上面说明了转发和重定向的区别也可以很容易概括出来**。转发是带着转发前的请求的参数的。重定向是新的请求**。

典型的应用场景：

1. 转发: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
2. 重定向: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了

## RequestDispatcher说明

RequestDispatcher对象调用forward()可以实现转发上面已经说过了。RequestDispatcher还有另外一个方法include()，该方法可以实现包含，有什么用呢？

我们在写网页的时候，一般网**页的头部和尾部是不需要改变的**。如果我们**多个地方使用Servlet输出网头和网尾的话，需要把代码重新写一遍**。而使用RequestDispatcher的**include()方法就可以实现包含网头和网尾的效果了**。

```java
request.getRequestDispatcher("/Head").include(request, response);
//指向Head的class文件

response.getWriter().write("-------------------------------");


request.getRequestDispatcher("/Foot").include(request, response);
```

