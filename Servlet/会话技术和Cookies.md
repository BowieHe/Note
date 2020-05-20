# 会话技术

> 基本概念：指用户开一个浏览器，访问一个网站，只要不关闭该浏览器，不管用户点击多少个超链接，访问多少资源，知道用户关闭浏览器，整个过程称为一次会话

会话跟踪技术可以解决很多问题，比如论坛的自动登录，根据浏览过的商品猜喜欢什么商品。

# Cookie

会话跟踪技术由Cookie和Session，Cookie是先出现的，先了解Cookie

>  Cookie是由W3C组织提出，最早由netscape社区发展的一种机制

- 网页之间的**交互是通过HTTP协议传输数据的，**而Http协议是**无状态的协议**。无状态的协议是什么意思呢？**一旦数据提交完后，浏览器和服务器的连接就会关闭，再次交互的时候需要重新建立新的连接**。
- 服务器无法确认用户的信息，于是乎，W3C就提出了：**给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息**。通行证就是Cookie

**Cookie的流程**

浏览器访问服务器，**如果服务器需要记录该用户的状态，就使用response向浏览器发送一个Cookie，浏览器会把Cookie保存起来。当浏览器再次访问服务器的时候，浏览器会把请求的网址连同Cookie一同交给服务器**。

## Cookie API

- Cookie类用于创建一个Cookie对象
- response接口中定义了一个addCookie方法，它用于在其响应头中增加一个相应的Set-Cookie头字段
- request接口中定义了一个getCookies方法，它用于获取客户端提交的Cookie

常见的Cookie方法

- public Cookie(String name,String value)
- setValue与getValue方法
- setMaxAge与getMaxAge方法
- setPath与getPath方法
- setDomain与getDomain方法
- getName方法

## 简单使用Cookie

创建Cookie对象并将Cookie发送给浏览器

```java
//设置response的编码
response.setContentType("text/html;charset=UTF-8");

//创建Cookie对象，指定名称和值
Cookie cookie = new Cookie("username", "dylandy");
//设置Cookie的时间
cookie.setMaxAge(1000);

//向浏览器给一个Cookie
response.addCookie(cookie);

response.getWriter().write("我已经向浏览器发送了一个Cookie");
```

## Cookie细节

### Cookie不可跨域名

Cookie具有不可跨域名性。浏览器判断**一个网站是否能操作另一个网站的Cookie的依据是域名**。所以一般来说，**当我访问baidu的时候，浏览器只会把baidu颁发的Cookie带过去，而不会带上google的Cookie。**

所以在访问Servlet的时候浏览器不会把所有的Cookie都带过去给服务器**，**更不会修改了别的网站的Cookie**

### Cookie保存中文

由于中文属于Unicode字符，英文数据根据ASCII字符，中文占4个或者3个字符，英文占2个字符。
在使用Cookie时对Unicode字符进行编码即可解决

`Cookie cookie = new Cookie("name",URLEncoder.encode(name,"UTF-8"))`

同时Cookie保存在硬盘的中文数据是经过编码的，那么在取出Cookie的时候要对中文数据进行解码

```java
Cookie[] cookie = request.getCookies();
for(int i = 0;cookies != null && i < cookies.length; i++){
  //经过URLEncoding就要经过URLDecoding
  String value = URLDecoder.decode(cookies[i].getValue(),"UTF-8");
  printWriter.write(name + "---" + value);
}
```

## Cookie的有效期

**Cookie的有效期是通过setMaxAge()来设置的**。

- 如果MaxAge为**正数**，**浏览器会把Cookie写到硬盘中，只要还在MaxAge秒之前，登陆网站时该Cookie就有效**【不论关闭了浏览器还是电脑】
- 如果MaxAge为**负数**，**Cookie是临时性的，仅在本浏览器内有效**，关闭浏览器Cookie就失效了，Cookie不会写到硬盘中。Cookie默认值就是-1。这也就为什么在我第一个例子中，如果我没设置Cookie的有效期，在硬盘中就找不到对应的文件。
- 如果MaxAge为**0**，则表示**删除该Cookie**。Cookie机制没有提供删除Cookie对应的方法，把MaxAge设置为0等同于删除Cookie

## Cookie的修改和删除

Cookie机制并没有提供修改和删除的机制，但是由于Cookie存储的方式类似Map集合，所以Cookie名称相同时，通过response添加浏览器中，会覆盖掉原来的Cookie

比如之前以name为名保存的，现在再以name为名，修改一个值，原来的文件不变，但是里面的值被修改了

如果要删除Cookie，将MaxAge设置为0，添加到浏览器中即可。
**注意**：删除或者修改Cookie时，**新建的Cookie出了value，maxAge之外所有的属性都要与原Cookie相同**，否则浏览器将视为不同的Cookie，不会覆盖，导致删除修改失败

## Cookie的域名

Cookie的**domain属性决定运行访问Cookie的域名。domain的值规定为“.域名”**

Cookie的隐私安全机制决定Cookie是不可跨域名的。也就是说www.baidu.com和www.google.com之间的Cookie是互不交接的。**即使是同一级域名，不同二级域名也不能交接**，也就是说：www.goole.com和www.image.goole.com的Cookie也不能访问

如果希望一级域名相同的网页Cookie之间可以互相访问，也就是说www.goole.com的Cookie可以在www.image.goole.com获取到，那么通过domain方法可以设置

```java
Cookie cookie = new Cookie("name", "google");
cookie.setMaxAge(1000);
cookie.setDomain(".google.com");
response.addCookie(cookie);

printWriter.write("使用www.google.com域名添加了一个Cookie,只要一级是google.com即可访问");
```

## Cookie路径

**Cookie的path属性决定允许访问Cookie的路径**

- 一般地，**Cookie发布出来，整个网页的资源都可以使用。现在我只想Servlet1可以获取到Cookie，其他的资源不能获取**。
- 使用Servlet2颁发一个Cookie给浏览器,设置路径为"/Servlet1"。

```java
Cookie cookie = new Cookie("username", "java");
cookie.setPath("/Servlet1");
cookie.setMaxAge(1000);
response.addCookie(cookie);

printWriter.write("该Cookie只有Servlet1获取得到");
```

## Cookie安全属性

- HTTP协议不仅仅是无状态的，而且是不安全的！如果不希望Cookie在非安全协议中传输，可以**设置Cookie的secure属性为true，浏览器只会在HTTPS和SSL等安全协议中传输该Cookie**。
- 当然了，设置secure属性不会将Cookie的内容加密。如果想要保证安全，最好使用md5算法加密。

## Cookie应用

### 显示上次访问时间

其实就是每次登录的时候，取到Cookie保存的值，然后再更新一下Cookie的值

访问Servlet有两种情况，一种是已经访问过了，另一种是第一次访问。通过下面代码实现

```JAVA
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
resp.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = resp.getWriter();

//获取网页上所有的Cookie
Cookie[] cookies = req.getCookies();

//判断Cookie的值是否为空
String cookieValue = null;
for (int i = 0; cookies != null && i < cookies.length; i++) {

  //获取到以time为名的Cookie
  if (cookies[i].getName().equals("time")) {
    printWriter.write("您上次登陆的时间是：");
    cookieValue = cookies[i].getValue();
    //由于时间格式中存在空格，在Cookie是非法字符，因此需要通过编码发送给浏览器，在读取的时候再解码来显示出来。通过查找ASCII可查找非法字符
    //解码读取
    printWriter.write(URLDecoder.decode(cookieValue,"UTF-8"));
    //编码来写入空格
    cookies[i].setValue(URLEncoder.encode(simpleDateFormat.
                  format(new Date()), "UTF-8"));
    resp.addCookie(cookies[i]);
    //既然已经找到了就可以break循环了
    break;
  }
}

//如果Cookie的值是空的，那么就是第一次访问
if (cookieValue == null) {
  //创建一个Cookie对象，日期为当前时间
  //解码显示时间
  Cookie cookie = new Cookie("time", URLEncoder.encode(
    simpleDateFormat.format(new Date()), "UTF-8"));

  //设置Cookie的生命期
  cookie.setMaxAge(20000);

  //response对象回送Cookie给浏览器
  resp.addCookie(cookie);

  printWriter.write("第一次登录");
}
```

按照正常的逻辑应该是先闯将Cookie对象，回送Cookie给浏览器，再遍历Cookie，更新Cookie值。

但是由于每次访问Servlet都会覆盖原来的Cookie，所以取到的Cookie永远是当前时间，而不是上次保存的时间。

所以逻辑变成先遍历所有的Cookie看有没有要的，如果没有那Cookie的值就是null，就是第一次登录

## 显示上次浏览的商品

首先创建Book对象，包含setter和getter方法还有构造器

```java
private String id ;
private String name ;
private String author;
```

添加一个简单的数据库存储数据，通过使用LinkedHashMap集合来实现，因为需要根据商品的id来寻找相关书籍，删改较多，因此使用linkedList

```java
private static LinkedHashMap<String, Book> linkedHashMap = new LinkedHashMap();

//简化开发复杂度，book的id和商品的id相同
static {
  linkedHashMap.put("1", new Book("1", "javaweb", "zhong"));
  linkedHashMap.put("2", new Book("2", "java", "fu"));
  linkedHashMap.put("3", new Book("3", "oracle", "cheng"));
  linkedHashMap.put("4", new Book("4", "mysql", "ou"));
  linkedHashMap.put("5", new Book("5", "ajax", "zi"));
}

//获取到所有书籍
public static LinkedHashMap getAll() {
  return linkedHashMap;
}
```

在主页上添加下列代码来显示书籍，并添加相关超链接到跳转页面

```java
resp.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = resp.getWriter();
printWriter.write("网页上所有的书籍" + "<br/>");

//拿到数据库中的书
LinkedHashMap<String, Book> linkedHashMap = Search.getAll();
Set<Map.Entry<String, Book>> entry = linkedHashMap.entrySet();

//显示所有的书
for(Map.Entry<String,Book> stringBookEntry : entry){
  Book book = stringBookEntry.getValue();
  printWriter.write("<a href='/servletDemo_war_exploded/demo4?id=" + book.getId() + "'target='_blank' >" + book.getName() + "</a>");
  printWriter.write("<br/>");
}
```

在跳转页面接收id，找到用户想看的书籍，显示出来

```java
String id = req.getParameter("id");

//由于book的id和商品id一致，因此可以获得用户的书
Book book = (Book) Search.getAll().get(id);
resp.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = resp.getWriter();
//输出书籍的详细信息
printWriter.write("书的编号是" + book.getId() + "<br/>");
printWriter.write("the name of the book" + book.getName() + "<br/>");
printWriter.write("the author" + book.getAuthor() + "<br/>");
```

此时已经完成了主页显示书籍和相关页面跳转的功能，接下来实现Cookies发送

在用户点击书籍的同时，服务器应该发送Cookie给浏览器，记住用户点击了该书籍。由于一会浏览器还要讲浏览过的书籍显示出来，所以使用书籍的id是最佳选择。

由于用户可能点击很多的书籍，所以这里值选择显示最近的三本书。

由于书籍内都是数字，为了读取方便，将存储到Cookie的书籍id用"_"分隔符分割。

因此接下来在跳转页新建makeHistpry方法。要做的是先遍历Cookie，查看有没有想要的Cookie，如果找到想要的Cookie，就取出Cookie值

```java
String bookHistory = null;
Cookie[] cookies = request.getCookies();
for (int i = 0; cookies != null && i < cookies.length; i++) {
  if (cookies[i].getName().equals("bookHistory")) {
    bookHistory = cookies[i].getValue();
  }
}
```

取出来的Cookie也分几种情况

1. Cookie的值为null【直接把传入进来的id当做是Cookie的值】
2. Cookie的值长度有3个了【把排在最后的id去掉，把传进来的id排在最前边】
3. Cookie的值已经包含有传递进来的id了【把已经包含的id先去掉，再把id排在最前面】
4. Cookie的值就只有1个或2个，直接把id排在最前边

并且在方法的最后把集合中的值取出，并且拼接成字符串

```java
if (bookHistory == null) {
    return id;
}

//如果Cookie的值不是null的，那么就分解Cookie的得到之前的id。
String[] strings = bookHistory.split("\\_");

//为了增删容易并且还要判断id是否存在于该字符串内-----我们使用LinkedList集合装载分解出来的id
List list = Arrays.asList(strings);
LinkedList<String> linkedList = new LinkedList<>();
linkedList.addAll(list);

if (linkedList.contains(id)) {
    linkedList.remove(id);
    linkedList.addFirst(id);
}else if (linkedList.size() >= 3) {
        linkedList.removeLast();
        linkedList.addFirst(id);
} else {
        linkedList.addFirst(id);
}
StringBuffer stringBuffer = new StringBuffer();

//遍历LinkedList集合，添加个下划线“_”
for (String s : linkedList) {
    stringBuffer.append(s + "_");
}

//最后一个元素后面就不需要下划线了
return stringBuffer.deleteCharAt(stringBuffer.length() - 1).toString();
```

然后在上面设置Cookie的生命周期，将Cookie回传给浏览器即可

```java
String bookHistory = makeHistory(req, id);
Cookie cookie = new Cookie("bookHistory",bookHistory);
cookie.setMaxAge(30000);
resp.addCookie(cookie);
```

接下来在首页获取Cookie的值，显示用户浏览过什么商品就好了

```java
printWriter.write("the items you looked");
printWriter.write("<br/>");
//显示浏览过的商品
Cookie[] cookies = req.getCookies();
for(int i = 0;cookies != null && i < cookies.length; i++){
    if(cookies[i].getName().equals("bookHistory")){
      //获得的数据类似"1_2_3"，需要拆解
        String bookHistory = cookies[i].getValue();
        String[] ids = bookHistory.split("\\_");
      //通过id找到每本书
        for(String id:ids){
            Book book = linkedHashMap.get(id);
            printWriter.write(book.getName());
            printWriter.write("<br/>");
        }
      break；
    }
}
```