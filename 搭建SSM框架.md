首先要创建一个动态web目录，建立好相关目录结构：

```
src
--main
----java:存放Java代码
----resource：存放资源文件，spring，mybatis，log配置文件
------mapper：dao中每个方法对应sql，无需daoimpl
------spring：spring相关配置，有dao，service，web三层
------webapp：存放前端静态资源
--------resource：项目静态资源，css，jsp，js等
--------WEB-INF：web.xml等内部目录
--test：测试分支
```

```
-java
----dao:数据访问层（接口）：数据库操作，文件读写，redis缓存
----entity：实体类，也叫pojo，封装dap层出来的数据为一个对象
----dto：数据传输层：用于service和web层之间的传输
----service（接口）：业务逻辑：写业务逻辑
----serviceimpl：业务逻辑（实现）：实现业务接口
----web：控制器，springmvc的controller在这里
```

## 配置文件

在Spring文件夹中创建三个配置文件，分为三层，dao，service，web

### spring dao

1. 读入数据库连接相关参数
2. 配置数据连接
3. 配置连接属性，可以不读配置文件直接这里写
4. 配置c3p0
5. 配置sqlSessionFactory对象(mybatis)
6. 扫描dao层接口，动态实现dao接口，即不需要daoimpl，sql和参数都写在xml文件里

```xml
<!--配置数据库相关参数-->
<context:property-placeholder location="classpath:jdbc.properties" />

<!--数据库连接池-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
  <property name="driverClass" value="${jdbc.driver}" />
  <property name="jdbcUrl" value="${jdbc.url}" />
  <property name="user" value="${jdbc.username}" />
  <property name="password" value="${jdbc.password}" />
  <property name="maxPoolSize" value="30" />
  <property name="minPoolSize" value="10" />
  <property name="checkoutTimeout" value="1000" />
  <property name="acquireRetryAttempts" value="2" />
  <property name="autoCommitOnClose" value="false" />
</bean>

<!--配置SqlSessionFactory 对象-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="typeAliasesPackage" value="entity" />
  <property name="configLocation " value="classpath:mybatis-config.xml" />
  <property name="mapperLocations" value="classpath:mapper/*.xml" />
</bean>
<!--配置扫描dao接口包，注入spring容器-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" >
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
  <property name="basePackage" value="dao" />
</bean>
```



### jdbc.properties

配置jdbc的username时，如果使用jdbc,username，可能会与系统环境中的username变量冲突，所以真正连接数据的时候，用户名会被替换成系统的用户名（可能是administrator）

### mybatis-config.xml

1. 使用自增主键 useGeneratedKeys
2. 使用列别名   useColumnLabel
3. 开启驼峰命名转换  mapUnderscoreToCamelCase

```xml
<settings>
  <setting name="useGeneratedKeys" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### spring-service

1. 扫描service包所有的注解@Service

   `<context:component-scan base-package="com.core.service" />`

2. 配置事务管理器，把事务管理器交由spring完成

   transactionManager

   `<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">`

3. 配置基于注解的声明式事务，可直接在方法上加上@Transaction

   `<tx:annotation-driven transaction-manager="transactionManager" />`

```xml
<import resource="spring-dao.xml" />
<context:component-scan base-package="service" />

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" >
  <property name="dataSource" ref="dataSource" />
</bean>

<tx:annotation-driven transaction-manager="transactionManager" />
```

### spring-web

1. 开启springMVC注解模式，可以使用@RequestMapping，@PathVariable，@ResponseBody等

   `<mvc:annotation-driven />`

2. 对静态资源处理，例如js，jsp，css

   `<mvc:default-servlet-handler />`

3. 配置jsp显示ViewResolver，例如在controller中某个方法返回一个string类型的"login"，实际上会返回"WEB-INF/login.jsp"

   `<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">`

4. 扫描web层@Controller

   `<context:component-scan base-package="com.core.web" />`

```xml
<mvc:annotation-driven />

<mvc:default-servlet-handler />

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="WEB-INF/jsp/" />
  <property name="suffix" value=".jsp" />
  <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
</bean>

<context:component-scan base-package="web" />
```

### web.xml

web.xml 文件在WEB-INF文件夹中，设置servlet参数

```xml
<servlet>
  <servlet-name>secdmeo-dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/spring-*.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>secdmeo-dispatcher</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

## entity

设置对应的实体类，比如Book, Appointment。并添上对应构造方法，getter和setter

## dao

在dao包中创建对应的接口，对应上面的BookDao.java 和AppointmentDao.java。并添加相应的业务方法

这里不需要实现dao接口，mybatis会动态实现，但是需要编写相应的mapper

使用@Param注解来实现方法在mybatis中实现（出现两个及以上参数的时候）

## mapper

上面所说的，因此mapper包中需要添加BookDao.xml和 AppointmentDap.xml，分别对应上面两个dao接口

mapper总结：namespace是该xml对应的接口全名，select和update中的id对应方法名，resultType是返回值类型，parameterType是参数类型（这个其实可选），最后#{...}中填写的是方法的参数.还有一个小技巧要交给大家，就是在返回Appointment对象包含了一个属性名为book的Book对象，那么可以使用"book.属性名"的方式来取值，看上面queryByKeyWithBook方法的sql。

## service

在service中创建BookService 和AppointmentService两个业务接口

可以在service.impl的包下面创建对应的BookServiceImpl.java 和AppointmentServiceImpl.java实现service接口里面的方法

通过使用@Service注解，@Autowire注入service依赖，

## dto

在dto包里面封装一个json返回结果的类, Result,java

## web

最后写web层，即controller，在web中创建BookController.java, AppointmentController.java两个控制器

 