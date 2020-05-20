# response、request对象

> Tomcat收到客户端的http请求，会针对每一次请求，分别创建一个**代表请求的request对象**、和**代表响应的response对象**

既然request对象代表http请求，那么我们**获取浏览器提交过来的数据，找request对象**即可。response对象代表http响应，那么我们**向浏览器输出数据，找response对象**即可。

### HttpServletResponse对象

http响应由状态行、实体内容、消息头、一个空行组成。**HttpServletResponse对象就封装了http响应的信息**

# HttpSevletResponse应用

## 调用getOutputStream()方法向浏览器输出数据

调用getOutputStream()方法向浏览器输出数据,**getOutputStream()方法可以使用print()也可以使用write()**。

使用**print()**方法如果输出英文没有问题，但是输出中文的时候由于编码不支持，因此会出现异常

```java
ServletOutputStream servletOutputStream = resp.getOutputStream();
servletOutputStream.print("aaaa");
```

使用**write()**方法不会出现乱码的问题，但是如果指定编码格式，而浏览器默认的编码格式不同，依旧会造成乱码的问题。通过设置Servlet消息头可以告诉浏览器回送的数据编码格式是什么

```java
//设置消息头
response.setHeader("Content-Type", "text/html;charset=UTF-8");
response.getOutputStream().write("你好呀我是中国".getBytes("UTF-8"));

//或者模拟HTML的<meta>标签模拟http消息头
//获取到servletOutputStream对象
ServletOutputStream servletOutputStream = response.getOutputStream();

//使用meta标签模拟http消息头，告诉浏览器回送数据的编码和格式
servletOutputStream.write("<meta http-equiv='content-type' content='text/html;charset=UTF-8'>".getBytes());
servletOutputStream.write("我是中国".getBytes("UTF-8"));
```

## 调用getWriter()方向向浏览器输出数据

**getWriter()**是Writer的子类，只能向浏览器输出字符数据，不能输出二进制数据。但是由于数据编码格式问题，当需要输出中文时，需要指定编码格式，同时设置浏览器的编码设置

```java
//设置浏览器用UTF-8编码显示数据，把中文转码的码表设置成UTF-8
response.setContentType("text/html;charset=UTF-8");

//获取到printWriter对象
PrintWriter printWriter = response.getWriter();
printWriter.write("read the message");
```

## 实现文件下载

在服务器内部添加资源可以在网上让其他用户实现下载。由于浏览器发送的所有请求都是去找Servlet，这样别的用户访问的时候就可以下载这个图片了。
Java文件上传下载都是通过io流来完成，即然要下载图片，首先是实现图片的读取。如果文件名是中文，则需要进行URL编码

```java
//获取到资源的路径
String path = this.getServletContext().
  getRealPath("MVC.png");

//读取资源
FileInputStream fileInputStream = new FileInputStream(path);

//获取到文件名,路径在电脑上保存是\\形式的。
String fileName = path.substring(path.lastIndexOf("\\") + 1);



//设置消息头，告诉浏览器要下载MVC,png这个文件,URL编码设置
resp.setHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode(fileName, "UTF-8"));

//把读取到的资源写给浏览器
int len = 0;
byte[] bytes = new byte[1024];
ServletOutputStream servletOutputStream = resp.getOutputStream();

while ((len = fileInputStream.read(bytes)) > 0) {
  servletOutputStream.write(bytes, 0, len);
}

//关闭资源
servletOutputStream.close();
fileInputStream.close();
```

## 实现自动刷新

在规定的时间内，让页面自动刷新，跟新资源，通过修改消息头来实现每3秒刷新一次页面

<center>`resp.setHeader("Refresh","3")`</center>

自动刷新页面往往和页面自动跳转是关联的，很多网页的3秒后自动跳转其实就是通过Refresh来实现的

```java
//登陆的页面设置
//3秒后会自动跳转到url所值的地址
resp.setContentType("text/html;charset=UTF-8");
resp.getWriter().write("3秒后跳转到别的页面");
resp.setHeader("Refresh","3;url='/servletDemo_war_exploded/demo");
```

## 设置缓存

浏览器本身就存在缓存机制。当第一次访问一个页面时，浏览器向服务器发送了两次请求，一个是网页的，一个是图片。

当第二次访问页面的时候，图片不再重新加载，而是从缓存里面直接读取。不过像有些网页，比如股票是不断更新数据的，因此可以设置禁止缓存功能，代码如下

```java
//浏览器有三消息头设置缓存，为了兼容性！将三个消息头都设置了
response.setDateHeader("Expires", -1);
response.setHeader("Cache-Control","no-cache");
response.setHeader("Pragma", "no-cache");

//这里为了看效果
PrintWriter printWriter = response.getWriter();
printWriter.print("你好啊" + new Date().toString());
```

## 实现数据压缩

网页上面的信息量是很大的，如果数据不压缩直接回传给浏览器，这样子十分耗费流量

**getOutputStream()和getWriter()都是直接把数据输出给浏览器的**。现在我要做的就是**让数据不直接输出给浏览器，先让我压缩了，再输出给浏览器**。**java提供了GZIP压缩类给我们**，那就使用GZIP来对数据进行压缩

```java
//GZIP的构造方法需要一个OutputStream子类对象，究竟哪个对象适合，我们看下write()方法
GZIPOutputStream gzipOutputStream = new GZIPOutputStream();

//查看了下API，write()接收的z是byte[]类型的。
gzipOutputStream.write();

//既然是byte[]类型，那么我就给他一个ByteArrayOutputStream
//创建GZIPOutputStream对象，给予它ByteArrayOutputStream
ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
GZIPOutputStream gzipOutputStream = new GZIPOutputStream(byteArrayOutputStream);

//GZIP对数据压缩，GZIP写入的数据是保存在byteArrayOutputStream上的
gzipOutputStream.write(ss.getBytes());

//gzipOutputStream有缓冲，把缓冲清了，并顺便关闭流
gzipOutputStream.close();

//将压缩的数据取出来
byte[] bytes = byteArrayOutputStream.toByteArray();

//告诉浏览器这是gzip压缩的数据
response.setHeader("Content-Encoding","gzip");

//将压缩的数据写给浏览器
response.getOutputStream().write(bytes);
```

## 生成随机图片

生成随机图片经常用于登陆时候的验证码填写，而那些验证码是通过HttpServletRespoinse写给浏览器的

Java提供了BufferedImage类来让我们生成图片

```java
//在内存中生成一张图片,宽为80,高为20，类型是RGB
BufferedImage bufferedImage = new BufferedImage(80, 20, BufferedImage.TYPE_INT_RGB);

//获取到这张图片
Graphics graphics = bufferedImage.getGraphics();

//往图片设置颜色和字体
graphics.setColor(Color.white);
graphics.fillRect(0, 0, 80, 20);
graphics.setFont(new Font(null, Font.BOLD, 20));

//往图片上写数据，先写个12345，横坐标是0，纵坐标是20【高度】
graphics.drawString("12345", 0, 20);
```

现在可以生成一张写上12345的图片。为了将图片传给浏览器，Java提供了图片流ImageIO来使用

```java
//要往浏览器写一张图片，那要告诉浏览器回送的类型是一张图片
response.setHeader("ContentType", "jpeg");

//java提供了图片流给我们使用，这是一个工具类
//把图片传进去，类型是jpg，写给浏览器
ImageIO.write(bufferedImage, "jpg", response.getOutputStream());
```

如果要生成随机数字，将12345替换成随机数字组成的String，如果想随机生成中文找到对应的中文映射表即可

## 重定向跳转

重定向跳转就是点击一个超链接，通知浏览器跳转到另外一个页面。页面之间的跳转有两种方式：重定向和转发。什么时候用哪一种在后面描述

使用HttpServletResponse对象的重定向

<center>`resp.sendRedirect("/demo2")`</center>

在重定向的过程中，可以看到两个状态码，一个是302，一个是200.
302在http协议中代表临时重定向。即这里的重定向是通过302状态码和跳转地址实现的。于是设置消息头也可以实现重定向。sendRedirect其实就是对于setStatus和setHeader的封装。

```java
//设置状态码是302
response.setStatus(302);

//HttpServletResponse把常用的状态码封装成静态常量了，所以我们可以使用SC_MOVED_TEMPORARILY代表着302
response.setStatus(HttpServletResponse.SC_MOVED_TEMPORARILY);

//跳转的地址是index.jsp页面
response.setHeader("Location", "/demo2");
```

## getWriter和getOutputStream区别

1. **getWriter()和getOutputStream()两个方法不能同时调用**。如果同时调用就会出现异常
2. Servlet程序向ServletOutputStream或PrintWriter对象中**写入的数据将被Servlet引擎从response里面获取，Servlet引擎将这些数据当作响应消息的正文，然后再与响应状态行和各响应头组合后输出到客户端**。
3. Servlet的**serice()方法结束后【也就是doPost()或者doGet()结束后】**，Servlet引擎将检查getWriter或getOutputStream方法返回的输出流对象是否已经调用过close方法，**如果没有，Servlet引擎将调用close方法关闭该输出流对象.**