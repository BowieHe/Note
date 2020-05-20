Tomcat就是一个运行Java的网络服务器。底层是一个Socket程序，也是JSP和Servlet的一个容器

### Tomcat启动

进入Tomcat目录下的bin文件，通过指令`sudo ./startup.sh` 来启动Tomcat。
在浏览器输入`localhost:8080`，如果出现Tomcat页面，则配置成功

### 无法正常启动：

1. JAVA_HOME配置问题
2. 端口被占用
   - 在Terminal中输入`netstat -anb`查看哪个进程用到了8080端口，关闭这个进程
   - 在Tomcat的`conf/server.xml`文件中修改8080端口到其他端口

## 将一个HTML文件作为站点的首页

在WEB-INF目录下创建一个`web.xml`文件，在文件中添加下面代码

```xml
<welcome-file-list>
  <welcome-file>index.html</welcome-file>
</welcome-file-list>
```

# 配置虚拟目录

## 原因

- 如果把所有的Web站点的目录都放在webapps下，可能会导致磁盘空间不足，同时也不利于web站点目录管理
- 把web站点的目录分散到其他磁盘管理需要配置虚拟目录（默认情况下只有webapps下的目录才能被Tomcat自动管理成一个web站点）
- 把web应用所在的目录交给web服务器管理，这个过程称为虚拟目录映射

## 配置虚拟目录

1. 在其他路径下创建web站点目录，并创建`WEB-INF`目录和一个HTML文件。
2. 在`Tomcat/conf/server.xml`文件中的`<host>`节点下添加代码，path表示访问时输入的web项目名，docBase表示站点目录的绝对路径
   `<Context path="/web" docBase="./Downloads/web1">`

### 方法二：

1. 在Tocmcat路径下，进入`confCatalinalocalhost`文件下创建XML文件，该文件名就是站点名
2. 在XML文件中添加代码，其中docBase是web站点的绝对路径

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<Context 
    docBase="D:\web1" 
    reloadable="true"> 
</Context> 
```

# 配置临时域名

访问Tomcat服务器的方式

1. 使用localhost域名访问
2. 使用IP地址 127.0.0.1访问
3. 使用机器名称访问（只限用于本季上或者局域网）
4. 使用本级IP地址访问
5. 为机器配置临时域名

# 设置虚拟主机

虚拟主机：多个不同的域名网站共存在一个Tomcat中

比如现在开发了四个网站，共有四个域名。一个Tomcat服务器只能运行一个网站。如果不配置虚拟主机，则需要四台电脑才可以运行四个网站

## 配置虚拟主机

1. 在Tomcat的`server.xml`文件中添加主机名

```xml
<Host name="zhongfucheng" appBase="D:\web1">
  <Context path="/web1" docBase="D:\web1"/>
</Host>`
```

# Tomcat体系结构

![Tomcat体系结构](/Users/bowei/Documents/Note/Pic/Tomcat体系结构.png)



# 浏览器访问Web资源

![浏览器访问Web资源过程解析](/Users/bowei/Documents/Note/Pic/浏览器访问Web资源过程解析.png)