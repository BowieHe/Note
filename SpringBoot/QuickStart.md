Spring Boot以来使用org.springframework.boot，通常Maven的POM文件需要继承自spring-boot-starter-parent项目，并声明对一个或多个starter的依赖关系。

下面是典型的SpringBoot的`pom.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- Inherit defaults from Spring Boot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
	</parent>
	<!-- Add typical dependencies for a web application -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>
	<!-- Package as an executable jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

其中`spring-boot-starter-parent`是一个特殊的启动器，有maven 的默认值，还提供了`dependency-management`部分，可以省略version标签



在设置完SpringBoot之后，在DemoApplication.java 文件中输入下列代码可以快速运行程序

```java
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @GetMapping("/hello")
  public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
    return String.format("Hello %s!", name);
  }
}
```

其中添加的hello()方法是为了获取String参数name，与代码中的"Hello"组合成一个字符串。如果在url的最后添加`?name=any`，输出则会是`Hello any`

`@RestController`注解是`@Controller`的衍生。为Spring提供了该类扮演特定觉得的提示，在这种情况下，我们的类是web@Controller，所以Spring在处理传日的Web请求时会考虑。

`@EnableAutoConfiguration`注解告诉Spring Boot根据添加的jar依赖关系猜测如何配置Spring，由于`spring-boot-starter-web`添加了Tomcat和SpringMVC，因此会自动配置假定正在开发Web应用程序并相应设置Spring

`@RequestMapping("/" )`注解提供路由信息，告诉Spring任何带有"/"路径的HTTP请求都应该映射到home方法， `@RestController`注解告诉Spring将结果字符串直接呈现给调用者

`@GetMapping("/hello")`则告诉Spring使用`hello()`方法来回应请求，并转到地址`http://localhost:8080/hello`.`@RequestParam`是提取name值从request中，默认是World

main方法，在这里主要是遵行应用程序入口点的Java约定的标准方法。主要通过调用run来委托Spring Boot的SpringApplilcation类，SpringApplication引导我们的应用程序，从Spring开始，然后启动自动配置的Tomcat Web服务器。需要将`DemoApplication.class`作为参数传递给run方法，告诉SpringApplication那个是Spring的组建，还会传递args数组以公开任何命令行的参数

## 创建一个可运行的jar

在SpringBoot中，要创建可执行的jar，需要将`spring-boot-maven-plugin`添加到怕`pom.xml`。

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

保存并运行`mvn package`就可以将项目打包；通过指令`jar tvf jar_file_name`可以查看jar包内部;

同时还有一个`myproject-SNAPSHOT.jar.original`的文件，这是Maven在Spring Boot重新打包之前创建的原始jar文件，通过`java -jar file-name`命令可以运行。