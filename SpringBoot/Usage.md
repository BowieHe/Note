# 依赖 dependency

## 依赖管理：

SpringBoot的每个版本都都提供了支持的依赖项的策划列表。实际上在构建配置中不需要提供这些依赖的版本，因为SpringBoot会管理，而且升级引导Spring时，这些依赖也会以一致的方式升级；如果有需要，仍然可以指定版本来覆盖SpringBoot的建议

jar列表以`spring-boot-dependencies`的形式提供，SpringBoot每个版本都与Spring框架基本版本相关联，因此强烈建议不要指定版本

## **Maven**

Maven可以继承`spring-boot-starter-parent`项目来获得合理的默认值，该类提供以下功能：

- Java 1.8作为默认编译器级别
- UTF-8编码
- 依赖管理部分，允许在pom中使用时省略依赖项的<version>标记
- 使用repackage执行ID执行repackage目标
- 资源过滤
- 插件配置
- `application.properties`,`application.yml`的资源过滤，包括特定于配置文件的文件

也可以通过覆盖自己项目中的属性来覆盖单个依赖项。比如想要升级到另一个Spring数据发布列表，将下列内容添加到pom.xml

```xml
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

### 没有父POM情况下使用Spring Boot

如果不选择继承`spring-boot-starter-parent`POM，仍可以使用`scope=import`依赖来保持依赖管理(不是插件管理)

同时由于不允许使用属性覆盖单个依赖项，因此需要`spring-boot-dependencies`之前的`dependencyManagement`中添加条目；下面是升级到另一个Spring数据发布列

```xml
<dependencyManagement>
	<dependencies>
		<!-- Override Spring Data release train provided by Spring Boot -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-releasetrain</artifactId>
			<version>Fowler-SR2</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.1.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

## Starters

Starters是一组依赖描述，。可以在应用程序中包含这些描述符。可以获得所需的Spring和相关技术的所有服务，不需要搜索示例代码和复制粘贴依赖描述。比如要开始使用Spring和JPA进行数据库访问，在项目中包含`spring-boot-starter-data-jpa`依赖项即可

# 构建代码

Spring Boot不需要任何特定的代码布局

当一个类不包含`package`声明的时候，是被认为在默认包中；通常不鼓励使用默认包，尽量避免使用。对于使用`@ComponentScan` , `@EntityScan` 或`@SpringBootApplication`注解的Spring Boot应用程序，会导致特定的问题，因为每个jar中的每个类都会被读取。

## 找到主应用程序类

通常会将主应用程序类放在其他类上的根包中，`@SpringBootApplication`注解往往放在主类中。它可以搜索特定的类；比如在写一个JPA应用，`@SpringBootApplication`注解包会搜索`@Entity`项，使用根包还允许组件扫描仅应用于此项目

如果不想使用`@SpringBootApplication`， 该注解导入的`@EnableAutoConfiguration` , `@ComponentScan`注解会定义该行为

下面是典型的布局

```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`Application.java` 文件会声明mian 方法和基本`@SpringBootApplication`

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

# 配置类

虽然SpringApplication可以和XML源一起使用。但是通常建议主要来源是单个`@Configuration`类，通常定义main方法的类是主要的`@Configuration`

不需要将所有`@Configuration`放入一个类。`@Import`注释可用于导入其他配置类。或者，您可以使用`@ComponentScan`自动选取所有Spring组件，包括`@Configuration`类。

如果使用基于XML的配置，仍建议使用`@Configuration`类，然后使用`@ImportResource`注解来加载XML文件

## 自动配置

Spring Boot会自动配置根据添加的jar依赖项目自动配置Spring应用程序。通过向`@Configuration`类之一添加`@EnableAutoConfiguration` , `@SpringApplication`注释来选择加入自动配置。应该只添加其中一个，

使用自动配置之后，如果发现正在应用不需要的特定自动配置类，可以使用`@EnableAutoConfiguration`的exclude属性禁用
`@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})`

## Spring Beans 和依赖注入

通常使用`@ComponentScan`来寻找beans和使用`@Autowired`作构造函数注入

如果在根包中定位应用程序类，则可以添加`@ConponentScan`不需要带任何的参数，所有的应用程序组件（`@Component` , `@Service` , `@Repository` , `@Controller`等）都会被自动注册为Spring Beans

下面为@Service bean 使用构造函数注入来获取所需的RiskAssessor bean
如果bean有一个构造函数，还可以省略`@Autowired`

```java
package com.example.service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {
  private final RiskAssessor riskAssessor;
  @Autowired
  public DatabaseAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
  }
  // ...
}
```

## 使用@SpringBootApplication Annotation

使用`@SpringBootApplication`注解可以启用三个功能的默认值：

- `@EnableAutoConfiguration`：启用Spring Boot自动配置机制
- `@ComponentScan`：对应用程序所在软件包启用`@Component`扫描
- `@Configuration`：允许在上下文中注册额外beans或导入其他类

