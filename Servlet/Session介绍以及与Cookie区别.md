# Session

> Session 是另一种记录浏览器状态的机制。不同的是Cookie保存在浏览器中，Session保存在服务器中。用户使用浏览器访问服务器的时候，服务器把用户的信息以某种的形式记录在服务器，这就是Session

如果说Cookie是检查用户身上的”通行证“来确认用户的身份，那么Session就是通过检查服务器上的”客户明细表“来确认用户的身份的。Session相当于在服务器中建立了一份“客户明细表”。

相比Cookie，Session的使用更加方便，而且可以解决Cookie不能解决的问题（Session可以存储对象，而Cookie只能存储字符串）

# Session API

- long getCreationTime();【获取Session被创建时间】
- **String getId();【获取Session的id】**
- long getLastAccessedTime();【返回Session最后活跃的时间】
- ServletContext getServletContext();【获取ServletContext对象】
- **void setMaxInactiveInterval(int var1);【设置Session超时时间】**
- **int getMaxInactiveInterval();【获取Session超时时间】**
- **Object getAttribute(String var1);【获取Session属性**】
- Enumeration<String> getAttributeNames();【获取Session所有的属性名】
- **void setAttribute(String var1, Object var2);【设置Session属性】**
- **void removeAttribute(String var1);【移除Session属性】**
- **void invalidate();【销毁该Session】**
- boolean isNew();【该Session是否为新的】

# Session作为域对象

从上面的API看出，Session有着request和ServletContext类似的方法。其实**Session也是一个域对象**。Session作为一种记录浏览器状态的机制，**只要Session对象没有被销毁，Servlet之间就可以通过Session对象实现通讯**。一般来说我们要存的是**用户级别的数据**就用Sessio，即只要浏览器不关闭，希望数据还在，就用Session来保存。

比如测试在demo1中往session添加变量，在demo4中读取

```java
//Demo1
//得到Session对象
HttpSession httpSession = request.getSession();
//设置Session属性
httpSession.setAttribute("name", "my name is ");

//Demo4
//获取到从Demo1的Session存进去的值
HttpSession httpSession = request.getSession();
String value = (String) httpSession.getAttribute("name");
System.out.println(value);
```

# Session的生命周期和有效期

- Session在用户**第一次访问服务器Servlet，jsp等动态资源就会被自动创建，Session对象保存在内存里**，这也就为什么上面的例子可以**直接使用request对象获取得到Session对象**。
- 如果访问HTML,IMAGE等静态资源Session不会被创建。
- Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，无论**是否对Session进行读写，服务器都会认为Session活跃了一次**。
- 由于会有越来越多的用户访问服务器，因此Session也会越来越多。**为了防止内存溢出，服务器会把长时间没有活跃的Session从内存中删除，这个时间也就是Session的超时时间**。
- Session的超时时间默认是30分钟，有三种方式可以对Session的超时时间进行修改

**第一种方式：**在tomcat/conf/web.xml文件中设置，时间值为20分钟，**所有的WEB应用都有效**

```xml
<session-config>
  <session-timeout>20</session-timeout>
</session-config>  
```

**第二种方式：**在单个的web.xml文件中设置，对单个web应用有效，**如果有冲突，以自己的web应用为准**。代码与上面相同

**第三种方式：**通过setMaxInactiveInterval()方法设置，单位为秒
`httpSession.setMaxInactiveInterval(60);`

### Session的有效期和Cookie是不同的

1. Session周期值得是不活动的时间，如果设置Session是10s，那么在10s内，没有访问Session，那么Session中的属性就会失效；但是如果在第9s的时候访问了，那么就重新计时。
2. 如果重新启动的Tomcat或者reload web应用，或者关机，Session也会失效。同时也可以通过函数让Session失效。invalidate()方法是让Session中的所有属性失效，常用语安全退出
3. 如果希望某个Session属性失效，通过方法removeAttribute()可以实现

1. Cookie的生命周期就是按照累计时间计算，无论用户有没有访问过Session或者Cookie

# 使用Session实现简单的购物功能

在主页显示所有的书和对应的购买链接

```java
resp.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = resp.getWriter();

printWriter.write("网页上所有的书籍：" + "<br/>");

//拿到数据库所有的书
LinkedHashMap<String, Book> linkedHashMap = Search.getAll();
Set<Map.Entry<String, Book>> entry = linkedHashMap.entrySet();

//显示所有的书到网页上
for (Map.Entry<String, Book> stringBookEntry : entry) {

  Book book = stringBookEntry.getValue();

  String url = "/servletDemo_war_exploded/demo4?id=" + book.getId();
  printWriter.write(book.getName());
  printWriter.write("<a href='" + url + "'>购买</a>");
  printWriter.write("<br/>");
```

在demo4上获得用户想要购买的书籍（由于用户想购买的不一定是一本书，因此使用list容器来存储书籍）。先遍历Cookie，判断是不是第一次访问Servlet的逻辑思路，先获取到Session的属性，如果Session的属性是null，则判定为没有属性。最后将页面跳转到demo2显示购物车

这里存放List的时候不能`new Arraylist`，因为这样每次Servlet被访问的时候都会创建一个新的ArrayList集合，书籍会被分配到不同的list中

```java
//得到用户想买书籍的id
String id = req.getParameter("id");

//根据书籍的id找到用户想买的书
Book book = (Book) Search.getAll().get(id);

//获取到Session对象
HttpSession httpSession = req.getSession();

//由于用户可能想买多本书的，所以我们用一个容器装着书籍
List list = (List) httpSession.getAttribute("list");
if (list == null) {
  list = new ArrayList();
  //设置Session属性
  httpSession.setAttribute("list",list);
}
//把书籍加入到list集合中
list.add(book);

String url = "/servletDemo_war_exploded/demo2";
resp.sendRedirect(url);
```

在购物车页面提供显示用户购买过的书籍，并且显示出来

```java
//要得到用户购买过哪些书籍，得到Session的属性遍历即可
resp.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = resp.getWriter();
HttpSession httpSession = req.getSession();
List<Book> list = (List) httpSession.getAttribute("list");

if (list == null || list.size() == 0) {
  printWriter.write("对不起，你还没有买过任何商品");

} else {
  printWriter.write("您购买过以下商品：");
  printWriter.write("<br/>");
  for (Book book : list) {
    printWriter.write(book.getName());
    printWriter.write("<br/>");
  }
}
```

# Session实现原理

如果在demo1中设置Seesion属性，在demo4中可以取出。但是如果新建一个会话，再次访问demo4会报空指针异常，那么接下来讨论**服务器是如何实现一个session为一个用户浏览器服务的？换个说法：为什么服务器能够为不同的用户浏览器提供不同session？**

- HTTP协议是无状态的，**Session不能依据HTTP连接来判断是否为同一个用户**。于是乎：**服务器向用户浏览器发送了一个名为JESSIONID的Cookie，它的值是Session的id值**。其实Session依据Cookie来识别是否是同一个用户。
- 简单来说：Session **之所以可以识别不同的用户，依靠的就是Cookie**
- 该Cookie是**服务器自动颁发给浏览器的**，不用我们手工创建的。**该Cookie的maxAge值默认是-1，也就是说仅当前浏览器使用，不将该Cookie存在硬盘中**
- 我们来捋一捋思路流程：当我们访问demo1的时候，**服务器就会创建一个Session对象，执行我们的程序代码，并自动颁发个Cookie给用户浏览器**
- 当使用同一个浏览器访问demo4的时候，浏览器会把Cookie的值通过http协议带过去给服务器，这样服务器就知道使用哪一个Session。
- 而当使用新会话的浏览器访问demo4的时候，该新浏览器没有Cookie，因此服务器无法辨别使用哪一个Session，因此无法获取到值

# 浏览器禁用了Cookie，Session还能用吗？

上面说了Session是依靠Cookie来识别用户浏览器的。如果我的用户浏览器禁用了Cookie了呢？绝大多数的手机浏览器都不支持Cookie，那Session怎么办？

- 当用户浏览器访问demo1的时候，服务器向用户浏览器颁发了一个Cookie
- 但是访问demo4的时候，由于禁用了Cookie，所以用户浏览器没有吧Cookie带给服务器

但是Java提供了解决方案：通过URL地址重写来解决。

HttpServletResponse类提供了两个URL地址重写的方法，`encodeURL(String url)`和 `encodeRedirectURL(String url)`

这两个方法会自动判断该浏览器是否支持Cookie，如果支持Cookie，重写后的URL地址就不会带有jsessionid了【当然了，即使浏览器支持Cookie，第一次输出URL地址的时候还是会出现jsessionid（因为没有任何Cookie可带）】

URL地址重写的原理：**将Session的id信息重写到URL地址中**。**服务器解析重写后URL，获取Session的id**。这样一来，即使浏览器禁用掉了Cookie，但Session的id通过服务器端传递，还是可以使用Session来记录用户的状态。

# Session禁用Cookie

 

```xml
<?xml version='1.0' encoding='utf-8'?>

<Context path="/ouzicheng" cookies="false">
</Context>
```

# Session和Cookie的区别

- **从存储方式上比较**
  - Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
  - Session可以存储任何类型的数据，可以把Session看成是一个容器
- **从隐私安全上比较**
  - **Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密
  - **Session存储在服务器上，对客户端是透明的**。不存在敏感信息泄露问题。
- **从有效期上比较**
  - Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在的
  - **Session的保存在服务器中，设置maxInactiveInterval属性值来确定Session的有效期。并且Session依赖于名为JSESSIONID的Cookie，该Cookie默认的maxAge属性为-1。如果关闭了浏览器，该Session虽然没有从服务器中消亡，但也就失效了。**
- **从对服务器的负担比较**
  - Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
  - Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。
- **从浏览器的支持上比较**
  - 如果浏览器禁用了Cookie，那么Cookie是无用的了！
  - 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。
- **从跨域名上比较**
  - Cookie可以设置domain属性来实现跨域名
  - Session只在当前的域名内有效，不可夸域名

# Cookie和Session共同使用

- **如果仅仅使用Cookie或仅仅使用Session可能达不到理想的效果**。这时应该尝试一下同时使用Session和Cookie
- 那么，什么时候才需要同时使用Cookie和Session呢？
- 在上一篇博客中，我们使用了Session来进行简单的购物，功能也的确实现了。现在有一个问题：**我在购物的途中，不小心关闭了浏览器。当我再返回进去浏览器的时候，发现我购买过的商品记录都没了！！为什么会没了呢？原因也非常简单：服务器为Session自动维护的Cookie的maxAge属性默认是-1的，当浏览器关闭掉了，该Cookie就自动消亡了。当用户再次访问的时候，已经不是原来的Cookie了。**
- 我们现在想的是：**即使我不小心关闭了浏览器了，我重新进去网站，我还能找到我的购买记录**。
- 要实现该功能也十分简单，问题其实就在：服务器为Session自动维护的Cookie的maxAge属性是-1，Cookie没有保存在硬盘中。我现在要做的就是：**把Cookie保存在硬盘中，即使我关闭了浏览器，浏览器再次访问页面的时候，可以带上Cookie，从而服务器识别出Session。**
- **第一种方式：**只需要在处理购买页面上创建Cookie，Cookie的值是Session的id返回给浏览器即可

```java
Cookie cookie = new Cookie("JSESSIONID",session.getId());
cookie.setMaxAge(30*60);
cookie.setPath("/Downloads/");
response.addCookie(cookie);
```

- **第二种方式：** **在server.xml文件中配置，将每个用户的Session在服务器关闭的时候序列化到硬盘或数据库上保存**。但此方法不常用，知道即可！

# Session应用

## 使用Session完成用户简单登陆

首先创建User类，

```java
private String username = null;
private String password = null;
```

再简单的集合模拟数据库，在新的类Search中

```java
public class Search {
  private static List<User> list = new ArrayList<>();
  static {
    list.add(new User("aaa","111"));
    list.add(new User("bbb","222"));
    list.add(new User("ccc","333"));
  }

  public static User find(String username, String passwords){
    for(User user: list){
      if(user.getUsername().equals(username) && user.getPassword().equals(passwords)){
        return user;
      }
    }
    return null;
  }
}
```

新建upload.jsp和index.jsp，分别用来提交表单和表单成功后的页面

```jsp
<%@ page contentType="text/html; charset=UTF-8"%>
<html>
  <head>
    <meta charset="UTF-8">
    <title>表单提交</title>
  </head>
  <body>
    <form action="/servletDemo_war_exploded/demo" method="post">
      用户名：<input type="text" name="username"><br/>
      密码：<input type="password" name="password"><br/>
      <input type="submit" value="提交">
    </form>
  </body>
</html>

<!--index.jsp-->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  成功登陆
  </body>
</html>
```

在demo1的Java文件中获取到表单提交的数据，查找数据库是否有对应的用户名和密码。如果没有就提示用户名密码错误，出错了；有就跳转到`index.jsp`页面

```java
String username = req.getParameter("username");
String password = req.getParameter("password");
User user = Search.find(username, password);
//如果找不到，就是用户名或密码出错了。
if (user == null) {
  resp.getWriter().write("you can't login");
  return;
}
//标记着该用户已经登陆了！
HttpSession httpSession = req.getSession();
httpSession.setAttribute("user", user);

//跳转到其他页面，告诉用户成功登陆了。
resp.sendRedirect(resp.encodeURL("index.jsp"));
```

## 利用Session防止表单重复提交

重复提交可能导致在投票网页上不断提交，实现刷票效果；或者在论坛注册多个用户不断发水帖

常见的重复提交：

- 在处理表单的Servlet中刷新
- 后退再提交
- 网络延迟，多次点击提交按钮（客户端问题，可以通过javascript监听用户提交事件，只能让用户提交一次表单）

第三种情况通过往JSP文件中添加下列代码可以实现

```JSP
<script type="text/javascript">
  //定义一个全局标识量：是否已经提交过表单数据
  var isCommitted = false;
  function doSubmit() {
    //false表示的是没有提交过，于是就可以让表单提交给Servlet
    if(isCommitted==false) {
      isCommitted = true;
      return true;
    }else {
      return false;
    }
  }
</script>
```

或者当第一次点击提交按钮之后，把提交按钮隐藏起来，这样就无法多次点击了

```jsp
<script type="text/javascript">
  function doSubmit() {
    var button = document.getElementById("button");
    button.disabled = disabled;
    return true;
  }
</script>
```

- **在处理表单的Servlet中刷新**和**后退再提交**这两种方式不能只靠客户端来限制了。也就是说javaScript代码无法阻止这两种情况的发生。
- 于是乎，我们就想得用其他办法来阻止表单数据重复提交了。我们现在学了Session，**Session可以用来标识一个用户是否登陆了**。Session的原理也说了：**不同的用户浏览器会拥有不同的Session**。而request和ServletContext为什么就不行呢？**request的域对象只能是一次http请求**，**提交表单数据的时候request域对象的数据取不出来**。ServletContext代表整个web应用，**如果有几个用户浏览器同时访问**，**ServletContext域对象的数据会被多次覆盖掉**，也就是说域对象的数据就毫无意义了。
- 可能到这里，我们会想到：在提交数据的时候，存进Session域对象的数据，在处理提交数据的Servlet中判断Session域对象数据????。究竟判断Session什么？判断Session域对象的数据不为null？**没用呀，既然已经提交过来了，那肯定不为null。**
- 此时，我们就想到了，在**表单中还有一个隐藏域，可以通过隐藏域把数据交给服务器**。
  - **判断Session域对象的数据和jsp隐藏域提交的数据是否对应**。
  - **判断隐藏域的数据是否为空**【如果为空，**就是直接访问表单处理页面的Servlet**】
  - **判断Session的数据是否为空**【servlet判断完是否重复提交，**最好能立马移除Session的数据**，不然还没有移除的时候，客户端那边儿的请求又来了，就又能匹配了，产生了重复提交。**如果Session域对象数据为空，证明已经提交过数据了！**】
- 我们向Session域对象的存入数据究竟是什么呢？简单的一个数字？好像也行啊。因为**只要Session域对象的数据和jsp隐藏域带过去的数据对得上号就行了呀，反正在Servlet上判断完是否重复提交，会立马把Session的数据移除掉的**。更专业的做法是：**向Session域对象存入的数据是一个随机数【Token--令牌**】。**生成一个独一无二的随机数**

通过下面代码可以生成一个随机的Token

```java
public class TokenProcessor {
  private TokenProcessor() {
  }
  private final static TokenProcessor TOKEN_PROCESSOR = new TokenProcessor();
  public static TokenProcessor getInstance() {
    return TOKEN_PROCESSOR;
  }
  public static String makeToken() {
    //这个随机生成出来的Token的长度是不确定的
    String token = String.valueOf(System.currentTimeMillis() + new Random().nextInt(99999999));
    try {
      //我们想要随机数的长度一致，就要获取到数据指纹
      MessageDigest messageDigest = MessageDigest.getInstance("md5");
      byte[] md5 = messageDigest.digest(token.getBytes());
      //如果我们直接 return  new String(md5)出去，得到的随机数会乱码。
      //因为随机数是任意的01010101010，在转换成字符串的时候，会查gb2312的码表，gb2312码表不一定支持该二进制数据，得到的就是乱码
      //于是乎经过base64编码成了明文的数据
      BASE64Encoder base64Encoder = new BASE64Encoder();
      return base64Encoder.encode(md5);
    } catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
    }
    return null;
  }
}
```

创建Token随机数，并且跳转到jsp页面

```java
//生出随机数
TokenProcessor tokenProcessor = TokenProcessor.getInstance();
String token = tokenProcessor.makeToken();
//将随机数存进Session中
request.getSession().setAttribute("token", token);
//跳转到显示页面
request.getRequestDispatcher("/login.jsp").forward(request, response);
```

在jsp的隐藏域中获取Token值

```jsp
<form action="/ouzicheng/Servlet7" >
  用户名：<input type="text" name="username">
  <input type="submit" value="提交" id="button">
  <%--使用EL表达式取出session中的Token--%>
  <input type="hidden" name="token" value="${token}" >
</form>
```

在demo1，表单处理页面中判断jsp的隐藏域有没有带值过来，如果Session中的值是不是空的，Session中的值和隐藏域带来的值是不是一样

```java
String serverValue = (String) request.getSession().getAttribute("token");
String clientValue = request.getParameter("token");
if (serverValue != null && clientValue != null && serverValue.equals(clientValue)) {
  System.out.println("处理请求");
  //清除Session域对象数据
  request.getSession().removeAttribute("token");
}else {
  System.out.println("请不要重复提交数据！");
}
```

实现原理

- 在session域中存储一个token
- 然后前台页面的隐藏域获取得到这个token
- 在第一次访问的时候，我们就判断seesion有没有值，如果有就比对。对比正确后我们就处理请求，接着就把session存储的数据给删除了
- 等到再次访问的时候，我们session就没有值了，就不受理前台的请求了！

