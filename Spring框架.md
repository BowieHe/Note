# Spring框架

1. Spring的出现时为了简化企业级开发，使用Spring代码可以将Bean对象，Dao组件对象，Service组件对象等交给Spring容器管理。使得很多复杂的代码变得简洁，有效降低代码的耦合度，极大的方便项目的后期维护，升级和拓展
2. Spring是一个IOC和AOP容器框架
3. Spinrg优良特性

- 非侵入式：基于Spring开发的应用中的对象不依赖于Spring的API
- 控制反转：IOC-Inversion of Control。值的将对象的创建权交个Spring去创建。使用Spring前，对象的创建是通过new，使用Spring之后对象的创建都是交给Spring
- 依赖注入：DI-Dependency Injection，依赖的对象不需要手动调用set方法去设置，通过配置赋值
- 面向切面编程-Aspect Oriented Programming
- 容器：Spring是一个容器，因为包含并且还礼应用对象的生命周期
- 组件化：Spring实现了使用简单的组件配置组合成一个复杂的应用。在Spring中可以使用XML和Java注解组合对象
- 一站式：在IOC和AOP的基础上整合各种企业级开源框架和优秀的第三方类库



#### Spring框架分为四大模块

- Core核型模块，负责管理组件的Bean对象
- 面向切面编程
- 数据库操作
- Web模块

# Hello World

1. 搭载Spring开发环境，通过Maven创建，并且在`pom.xml`配置文件中注入依赖，同时在src路径下创建main和resource文件夹

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.0.4.RELEASE</version>
    </dependency>
```

2. 编写一个Java实体Bean对象，Person类,同时创建对应setter&getter，constructor，toString

```java
public Person(int id, String name, int age, String phone) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.phone = phone;
    }
```

3. 新建`applicationContext.xml`的Spring配置文件

```xml
<!-- 
		bean标签配置一个组件类======会由Spring容器来管理
			id 属性给bean添加唯一标识
			class 属性设置 配置的类的全类名
	 -->
<bean id="person" class="com.atguigu.pojo.Person">
  <!-- property 标签 设置 属性信息 -->
  <property name="id" value="1" />
  <property name="name" value="张三" />
  <property name="age" value="18" />
  <property name="phone" value="18688888888" />
</bean>
```

### 编写测试代码，通过id获取对象

```java
public void test1() {
  // applicationContext 就是 IOC容器
  // ClassPathXmlApplicationContext是容器的实现类
  ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
  // 从容器中获取 id 为 person 定义的对象
  Person person = (Person) applicationContext.getBean("person");
}
```

### 通过类型获取对象

将上面一行代码，改成
`Person person = (Person) applicationContext.getBean(Person.class);`

可以获取到IOC容器中获取Person.class示例对象。但是只适用于只有一个对象，如果有多个同Person.class类实现的时候，就会报错

### 通过构造方法参数名注入值

通过构造器为bean的属性赋值，修改`applicationContextxml`配置文件

```xml
<bean id="person03" class="com.atguigu.pojo.Person">
  <!-- constructor-arg 表示有参构造方法中的一个参数 -->
  <constructor-arg name="id" value="3"></constructor-arg>
  <constructor-arg name="name" value="王五"></constructor-arg>
  <constructor-arg name="age" value="18"></constructor-arg>
  <constructor-arg name="phone" value="18610101010"></constructor-arg>
</bean>	
```

### index属性指定参数的位置

在`applicationContext.xml`文件中修改配置文件，name改为index

```xml
<bean id="person04" class="com.atguigu.pojo.Person">
  <!-- index 表示参数的索引位置。索引值从零0开始算 -->
  <constructor-arg index="0" value="4"></constructor-arg>
  <constructor-arg index="1" value="王五"></constructor-arg>
  <constructor-arg index="3" value="18610101010"></constructor-arg>
  <constructor-arg index="2" value="18"></constructor-arg>
</bean>
```

### 根据参数类型注入

```xml
<!-- int id, String name, int age, String phone -->
<bean id="person05" class="com.atguigu.pojo.Person">
  <!-- index 表示参数的索引位置。索引值从零0开始算 -->
  <constructor-arg index="3" value="18610101010" type="java.lang.String"></constructor-arg>
  <constructor-arg index="1" value="王五" type="java.lang.String"></constructor-arg>
  <!-- 
           使用类型区分重载的构造函数
   这个地方有一点需要特别注意：
    如果代码中的类型是Integer , 那么type类型是 java.lang.Integer
    如果代码中的类型是int , 那么type类型是int
   -->
  <constructor-arg index="2" value="18" type="int"></constructor-arg>
  <constructor-arg index="0" value="4" type="int"></constructor-arg>
</bean>
```

### 使用P名称空间

在`applicationContext.xml`文件前面添加指令添加P名称空间
`xmlns:p="http://www.springframework.org/schema/p"`

在Bean中做以下配置
`<bean id="person6" class="com.spring.demo.Person" p:id="6" p:name="zhaoliu" p:age="18" p:phone="1234"/>`

### 添加null值

在`applicationContext.xml`中修改如下配置内容

```xml
<bean id="person07" class="com.atguigu.pojo.Person" p:id="7" p:age="18" p:name="第七个人">
  <property name="phone">
    <null />
  </property>
</bean>
```

### IOC子对象的赋值测试

引用其他的Bean。创建新的类Car，并添加属性name和carNo。同时在Person类中添加属性Car car。

在`applicationContext.xml`文件中添加bean对象

```xml
<!-- 定义一个Car类 -->
<bean id="car" class="com.atguigu.pojo.Car" p:name="宝马" p:carNo="京A12312" />
<!--  定义一个Person类-->
<bean id="person10" class="com.atguigu.pojo.Person">
  <property name="id" value="10" />
  <property name="name" value="perosn10" />
  <property name="age" value="18" />
  <property name="phone" value="0101001" />
  <!-- ref 表示引用一个对象 -->
  <property name="car" ref="car" />
</bean>
```

### IOC内部Bean的使用

引用内部Bean，将代码`<property name="car" ref="car" />` 修改为

```xml
<property name="car">
  <bean id="car02" class="com.atguigu.pojo.Car" p:name="保时捷" p:carNo="京B12341"></bean>
</property>
```

使用测试代码`System.out.println(person11);`可以获得该bean内容，但是如果print`applicationContext.getBean("car02")` 则会报错，因为内部的bean不能被外部使用

### List属性的赋值

在Person类中添加一个List集合属性->`private List<String> phones`

在`applicationContext.xml`配置文件中添加List内容

```xml
<!-- 给list集合赋值 -->
<property name="phones">
  <list>
    <value>18611110000</value>
    <value>18611110001</value>
    <value>18611110002</value>
  </list>
</property>
```

### IOC的Map属性赋值

Person添加Map属性`private Map<String, Object> map`

在`applicationContext.xml`中添加配置内容

```xml
</property>
<!-- map对象 -->
<property name="map">
  <map>
    <!-- entry 表示map中有每一项 -->
    <entry key="aaa" value="aaaValue" />
    <entry key="bbb" value="bbbValue" />
    <entry key="ccc" value="cccValue" />
  </map>
</property>
```

### IOC Properties属性的赋值

使用prop子元素为Properties类型的属性赋值,给Person对象添加Properties属性->`private Properties prop`，并修改`applicationContext.xml`配置文件

```xml
<property name="props">
  <props>
    <prop key="username">root</prop>
    <prop key="password">root</prop>
    <prop key="drivers">com.mysql.jdbc.Driver</prop>
    <prop key="url">jdbc:mysql://localhost:3306/spring</prop>
  </props>
</property>
```

### IOC的Util名称空间

在`applicationContext.xml`文件前面添加名称空间`xmlns:util="http://www.springframework.org/schema/util"`,来添加Util名称空间，同时需要导入语义约束。之后在下面配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util-4.0.xsd">

<!-- 定义一个list集合 -->
<util:list id="list1">
  <value>string1</value>
  <value>string2</value>
  <value>string3</value>
</util:list>

<bean id="person15" class="com.atguigu.pojo.Person">
  <property name="id" value="10" />
  <property name="name" value="perosn10" />
  <property name="age" value="18" />
  <property name="phone" value="0101001" />
  <!-- list对象 ref表示引用 -->
  <property name="phones" ref="list1" />
</bean>
```

### IOC的级联属性赋值

在Bean中添加以下代码即可

`<property name="car" ref="car">`
`<property name="car.name" value="audi">` 

使用这种方法赋值，使用car.name之前car对象必须有值，不然会报空指针错误。同时使用car.name赋值之后，会覆盖掉原来的值

### IOC 静态工厂方法创建Bean

创建一个工厂类

```java
public class PersonFactory {
  public static Person createPerson() {
    Person person = new Person();
    person.setId(17);
    person.setName("这是静态工厂方法");
    return person;
  }
}
```

在`applicationContext.xml`中通过下列语句创建bean
`<bean id="person11" class="com.spring.demo.PersonFactory" factory-method="createPerson"/>`

### IOC工厂实例方法创建Bean

在PersonFactory中添加获取Bean的方法：

```java
public Person getPerson(){
  Person person = new Person();
  person.setId(17);
  person.setName("zheshi shenme a ");
  return person;
}
```

然后在配置文件中添加如下内容

```xml
<!-- 工厂实例方法分两步：
  第一步：先使用bean标签配置一个bean对象
  第二步:使用bean标签配置factory-bean和factory-method方法
  -->
<bean id="personFactory" class="com.atguigu.pojo.factory.PersonFactory" />
<!-- 
  factory-bean表示使用哪一个工厂对象的实现
  factory-method表示调用对象的哪一个方法去获取对象实例
  -->
<bean id="person18" factory-bean="personFactory" factory-method="getPerson">
```

### IOC的FactoryBean接口方式创建对象

创建一个FactoryBean的实现对象，并重写其中的方法

```java
public class PersonFactoryBean implements FactoryBean<Person> {
	public Person getObject() throws Exception {
		Person person = new Person();
		person.setId(20);
		person.setName("这是PersonFactoryBean创建出来的");
		return person;
	}
	public Class<?> getObjectType() {
		return Person.class;
	}
	/**
	 * 判断是否是单例<br/>
	 * 		返回true是单例 <br/>
	 *  	返回false是多例
	 */
	public boolean isSingleton() {
		return true;
	}
}
```

在配置文件中通过下面的方式创建bean对象

`<bean id="person13" class="com.spring.demo.PersonFactoryBean" />`

### IOC继承Bean配置

在配置文件中配置如下两个代码

```xml
<!-- 先定义一个base的 person对象 -->
<bean id="base" class="com.atguigu.pojo.Person" p:name="basePerson" p:age="18" p:id="12312"/>
<!-- 然后在另一个bean中。使用parent继承所有的 -->
<bean id="person20" class="com.atguigu.pojo.Person" parent="base" p:id="20" />
```

### IOC组件创建顺序

bean之间的依赖，通过depends-on属性来配置，首先创建三个bean对象

```java
public class A {
	public A() {
		System.out.println("A 被创建了");
	}
}
public class B {
	public B() {
		System.out.println("B 被创建了");
	}
}
public class C {
	public C() {
		System.out.println("C 被创建了");
	}
}
```

在配置文件中添加下列配置.在该配置下，创建一个applicationContext对象，会同发现b,c先被执行，然后a再执行

```xml
<!-- 默认情况下。bean对象创建的顺序，是从上到下
   depends-on 可以设定依赖
  -->
<bean id="a" class="com.atguigu.pojo.A" depends-on="b,c"/>
<bean id="b" class="com.atguigu.pojo.B" />
<bean id="c" class="com.atguigu.pojo.C" />
```

### 基于XML配置文件的自动注入

在配置文件中设置如下内容

```xml
<bean id="car1" class="com.atguigu.pojo.Car">
  <property name="carNo" value="京B23412" />
  <property name="name" value="劳死来死"/>
</bean>
<bean id="car2" class="com.atguigu.pojo.Car">
  <property name="carNo" value="京B23412" />
  <property name="name" value="劳死来死2"/>
</bean>
<!-- 
  autowire 属性设置是否自动查找bean对象并给子对象赋值

  default 和 no 表示不自动查找并注入（你不赋值，它就null）
  byName 	是指通过属性名做为id来查找bean对象，并注入
     1、找到就注入
     2、找不到就为null
  byType  是指按属性的类型进行查找并注入
     1、找到一个就注入
     2、找到多个就报错
     3、没有找到就为null
  constructor 是指按构造器参数进行查找并注入。
     1、先按照构造器参数类型进行查找并注入
     2、如果按类型查找到多个，接着按参数名做为id继续查找并注入。
     3、按id查找不到，就不赋值。
  -->
<bean id="p19" class="com.atguigu.pojo.Person" autowire="constructor">
  <property name="name" value="p19" />
</bean>
```

### IOC Bean的单例和多例

关于bean标签的scope属性设置

```xml
<!-- 
  scope 属性设置对象的域
   singleton:表示单例（默认）
        1、Spring容器在创建的时候，就会创建Bean对象
        2、每次调用getBean都返回spring容器中的唯一一个对象

   prototype:表示多例
        1、多例在Spring容器被创建的时候，不会跟着一起被创建。
        2、每次调用getBean都会创建一个新对象返回

   request:在一次请求中，多次调用getBean方法都是返回同一个实例。
        getBean("p20"); 底层大概的实现原理
        Object bean = request.getAttribute("p20");
        if (bean == null) {
         bean = new 对象();
         request.setAttribute("p20",bean);
        }
        return bean;

   session:在一个会话中，多次调用getBean方法都是返回同一个实例。
        getBean("p20"); 底层大概的实现原理
        Object bean = session.getAttribute("p20");
        if (bean == null) {
         bean = new 对象();
         session.setAttribute("p20",bean);
        }
        return bean;
  -->
<bean id="p20" class="com.atguigu.pojo.Person" scope="singleton">
  <property name="name" value="p20" />
</bean>
```

# 对象生命周期

### IOC Bean的生命周期

创建有生命周期方法的Bean

```xml
<!-- 
  init-method配置初始化方法(bean对象创建之后)
  destroy-method配置销毁方法（在spring容器关闭的时候,只对单例有效）
  -->
<bean id="p21" class="com.atguigu.pojo.Person" init-method="init" destroy-method="destroy" scope="singleton">
  <property name="name" value="p21"/>
</bean>
```

测试案例：

```java
ConfigurableApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
applicationContext.getBean("p21");
applicationContext.close();
//会先执行init方法，在执行到close时，运行destroy方法
```

### Bean的后置处理器BeanPostProcessor

后置处理器可以在bean对象的初始化方法前/后做一些工作，使用步骤如下：

1. 编写一个类去实现BeanPostProcessor接口

重写`postProcessAfterInitialization` （初始化方法之后调用）
和`postProcessBeforeInitialization`（初始化方法之前调用）这两个方法

2. 到Spring容器配置文件中配置

```xml
<bean id="p21" class="com.atguigu.pojo.Person" init-method="init" destroy-method="destroy" scope="singleton">
  <property name="name" value="p21"/>
</bean>
<!-- 配置自定义的后置处理器,实现接口的类 -->
<bean class="com.atguigu.postprocessor.MyBeanPostProcessor" />
```

在处理的时候会先进行初始化的操作，然后才会执行init的方法和destroy的方法



# Spring管理数据库连接池

引入对应的durid和mysql-connection-java 两个jar包，并且在配置文件中进行如下设置

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
  <property name="username" value="bowie" />
  <property name="password" value="123" />
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
  <property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
  <property name="initialSize" value="5" />
  <property name="maxActive" value="10" />
</bean>
```

## Spring引入单独的`jdbc.properties`配置文件

写配置文件`jdbc.properties`，包含username，password，initialSize，maxActive等放在配置文件路径下。在`applicationContext.xml`配置文件中添加如下内容

```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
  <!--location表示要加载的文件路径，classpath表示从类路径下搜索-->
  <property name="location" value="classpath:jdbc.properties"/>
</bean>
<!--之后的username等value可以改为 value="${user}"-->
```

## 使用context名称空间加载jdbc.properties配置文件

添加context名称空间，可以通过context标签代替PropertyPlaceholderConfigurer加载属性配置文件
`<context:property-placeholder location="classpath:jdbc.properties"/>`

**Tips**:由于在properties中如果用户名标签为username，可能会和本计算机的本地用户名冲突，因此不建议使用username作为变量名称

## 使用Spring EL表达式

```xml
<bean id="car" class="com.atguigu.pojo.Car">
  <property name="carNo" value="京B123412" />
  <property name="name" value="蓝脖鸡泥" />
</bean>
<bean id="person" class="com.atguigu.pojo.Person">
  <!-- 实验26：[SpEL测试I]在SpEL中使用字面量 -->
  <!-- 使用格式：#{数值} 		#{“字符串” || ‘字符串’} -->
  <property name="id" value="#{100}" />
  <!-- 		<property name="name" value="#{'我是SpringEL输出的字符串'}" /> -->
  <!-- 实验27：[SpEL测试II]在SpEL中引用其他bean -->
  <!-- 使用格式：#{bean的id} -->
  <property name="car" value="#{car}"></property>
  <!-- 实验28：[SpEL测试III]在SpEL中引用其他bean的某个属性值 -->
  <!-- 使用格式： #{bean.属性名} -->
  <!-- 		<property name="name" value="#{car.name}"></property> -->
  <!-- 实验29：[SpEL测试IV]在SpEL中调用非静态方法 -->
  <!-- 使用格式： #{bean.方法名(参数)} -->
  <!-- 		<property name="name" value="#{car.fun()}"></property> -->
  <!-- 实验30：[SpEL测试V]在SpEL中调用静态方法 -->
  <!-- 使用格式：#{T(全名类).方法名(参数)} -->
  <property name="name" value="#{T(com.atguigu.pojo.Car).staticFun()}"></property>

  <!-- 实验31：[SpEL测试VI]在SpEL中使用运算符 -->
  <!-- 使用格式：#{表达式} -->
  <property name="salary" value="#{10000 * 10}" />
</bean>
```

# 注解功能

注解配置Dao,Service,Controller组件

新建class -> BookService,BookController,BookDao.在对应的Java文件class上加上注解，来实现Bean对象的建立

```java
//@Component 注解表示<br/>
//<bean id="book" class="com.atguigu.pojo.Book" />
@Component
public class Book {
}
/**@Repository 注解表示<br/>
 * <bean id="bookDao" class="com.atguigu.dao.BookDao" /><br/>
 * @Repository("abc") 表示
 * <bean id="abc" class="com.atguigu.dao.BookDao" /><br/>
 Scope表示scope，默认为单例*/
@Repository
@Scope("prototype")
public class BookDao {
}
/** @Service 注解表示<br/>
 * <bean id="bookService" class="com.atguigu.service.BookService" />*/
@Service
public class BookService {
}
 /** @Controller 注解表示<br/>
 * <bean id="bookController" class="com.atguigu.controller.BookController" /> */
@Controller
public class BookController {
}
```

在配置文件中添加下列语句，来查找对应包下有无注解：
`<context:component-scan base-package="com.atguigu"></context:component-scan>`

## 指定扫描包时的过滤内容

1. 使用context:include-filter指定扫描包时要包含的类
   通常需要与`use-default-filter`属性设置为false配合才能达到仅包含某些组件
2. 使用context:exclude-filter指定扫描包时不包含的类

```xml
<!-- 配置包扫描
     base-package	设置需要扫描的包名（它的子包也会被扫描）
     use-default-filters="false" 去掉包扫描时默认包含规则
    -->
<context:component-scan base-package="com.atguigu" use-default-filters="false">
  <!-- 
       type="annotation" 按注解进行过滤
	expression 注解的表达式（什么注解或注解的全类名）
      -->
  <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
  <context:include-filter type="assignable" expression="com.atguigu.controller.BookController"/>
</context:component-scan>
```

- annotation：根据目标组件是否标注了指定类型的注解进行过滤
- assignable：根据目标组件是否指定类型的子类方式进行过滤
- aspectj：`com.demo.*Service+`所哟类名以Service结束的，或者这样的子类。规则根据AspectJ表达式过滤
- regex：`com.demo.anno.*`所有anno包下的类，根据政策表达式匹配到的类名比较
- custom：`com.demo.xxTypeFilter`通过编码方式自定义过滤规则，该类必须实现`org.springframework.core.type.filter.Typefilter`接口

## 使用注解@Autowired 自动装配

使用`@Autowired`注解来实现根据类型实现自动装配。根据注入对象的类型在Spring容器中查找相对应的类，如果有找到，就自动装配。使用`@Autowired`注解不需要通过get/set方法

1. 按照类型查找并注入
2. 如果找到多个，就接着按照属性名作为id继续查找并注入
3. 如果有多个，但是属性名作为id找不到，可以继续使用 `@Qualifier("book")`注解指定id查找并注入
4. 通过修改`@Autowired(required=false)`允许字段值为null

```java
@Autowired(reuqired=false)
@@Qualifier("bookDaoExt")
private BookDao bookdao1;
```

### 将Autowired标注在方法上

1. 先按类型查找参数并调用方法传递
2. 如果找到多个，就接着按参数名做为id继续查找并注入
3. 如果找到多个。但是参数名做为id找不到，可以使用@Qualifier("bookDao")注解指定id查找并调用
4. 可以通过修改@Autowired(required=false)允许不调用此方法也不报错

```java
@Autowired(required=false)
public void setBookDao(@Qualifier("bookDaoExt")BookDao abc){
	System.out.print(abc);}
```

## 范型注入

先创立一个BookDao<T>的抽象类，由UserDao<User>和BookDao<Book>实现。

在创建BaseService<T>，里面有变量`private BookDao<T> baseDao`，并且打印。该抽象类有BookService<Book>和UserService<User>实现。

1. 当执行代码`BookService bookService = (BookService) applicationContext.getBean("bookservice")`时可以得到BookService对象
2. 通过BookService对象可以确定范型为Book
3. 开始查找BaseDao的子类得到UserDao和BookDao两个
4. 比较BookService和两个Dao的范型，范型一致的为BookDao，将BookDao注入

# Spring拓展的Junit测试

```java
/**
 * Spring扩展了Junit测试。测试的上下文环境中自带Spring容器。<br/>
 * 我们要获取Spring容器中的bean对象。就跟写一个属性一样方便。
 */
// @ContextConfiguration配置Spring容器
@ContextConfiguration(locations="classpath:applicationContext.xml")
// @RunWith配置使用Spring扩展之后的Junit测试运行器
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringJunitTest {

  @Autowired
  UserService userService;

  @Autowired
  BookService bookService;

  @Test
  public void test1() throws Exception {
    bookService.save(new Book());
    userService.save(new User());
  }
}
```

# AOP切面编程

面向切面编程指的是，程序运行期间，动态的将某段代码插入到原来方法的某些位置中，这就叫面向切面编程

## 通过动态代理统一日志

```java
//创建一个计算器代理工具类
public class CalculateProxyFactory {

	public static Calculate getProxy(Calculate target) {
		// 定义一个计算器拦截处理类
		class CalculateInvocationHandler implements InvocationHandler {
			Calculate target;

			public CalculateInvocationHandler(Calculate calculate) {
				this.target = calculate;
			}
			/**proxy	代理对象 method	调用的方法 args		方法的参数*/
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				if (method.getDeclaringClass().equals(Object.class)) {
					return method.invoke(target, args);
				}
				LogUtil.log(method.getName(), (int) args[0], (int) args[1]);
				Object result = null;
				try {
					result = method.invoke(target, args);
					System.out.println("后置日记");
				} catch (Exception e) {
					System.out.println("异常日记");
				} finally {
					System.out.println("finally日记");
				}
				return result;
			}
		}

		return (Calculate) Proxy.newProxyInstance(
      target.getClass().getClassLoader(),
      target.getClass().getInterfaces(), 
      new CalculateInvocationHandler(target));
	}
}

//test代码
public static void main(String[] args) {
  Calculate calculate = CalculateProxyFactory.getProxy( new Calculator() );
  int result = calculate.add(12, 12);
  System.out.println("相加的结果：" + result);

  result = calculate.mul(12, 12);
  System.out.println("相乘的结果：" + result);
}
```

## 使用Cglib代理

```java
public class CGLibProxyFactory implements MethodInterceptor {

  public static Object getCGLibProxy(Object target, Callback callback) {
    // 创建一个CGLig生成器
    Enhancer enhancer = new Enhancer();
    // 设置父类。因为cglib是通过类，进行代码，不是通过接口
    enhancer.setSuperclass(target.getClass());
    // 设置拦截的代理方法
    enhancer.setCallback(callback);
    // create 方法创建一个代理对象并返回
    return enhancer.create();
  }

  @Override
  public Object intercept(Object proxy, Method method, Object[] params, MethodProxy methodProxy)
    throws Throwable {
    LogUtil.log(method.getName(), (int) params[0], (int) params[1]);
    // 调用实际的对象的方法
    // 一定要使用methodProxy对象
    // 第一个参数是proxy代码对象的父类方法
    Object result = methodProxy.invokeSuper(proxy, params);
    System.out.println("这是后置代码");

    return result;
  }

  public static void main(String[] args) {
    Calculator calculator = (Calculator) CGLibProxyFactory.getCGLibProxy(new Calculator(),
                                                                         new CGLibProxyFactory());
    calculator.add(12, 13);
  }
}
```

在没有接口的情况下，同样可以实现代理的效果。

但是依然需要自己编码实现代理全部过程。

# AOP编程术语

### 连接点(Joinpoint)

连接点就是程序执行的某个特定的位置，如：类开始初始化前、类初始化后、类的某个方法调用前、类的某个方法调用后、方法抛出异常后等。
Spring 只支持类的方法前、后、抛出异常后的连接点。

### 切点（Pointcut）

一个项目中有很多的类，一个类有很多个连接点，当我们需要在某个方法前插入一段增强（advice）代码时，我们就需要使用切点信息来确定，要在哪些连接点上添加增强。
如果把连接点当做数据库中的记录，那么切点就是查找该记录的查询条件。
所以，一般我们要实现一个切点时，那么我们需要判断哪些连接点是符合我们的条件的，如：方法名是否匹配、类是否是某个类、以及子类等。

### 增强（Advice）

增强就很好理解了，AOP（切面编程）是用来给某一类特殊的连接点，添加一些特殊的功能，那么我们添加的功能也就是增强。比如：添加日志、管理事务。
不过增强不仅仅包含需要增加的功能代码而已，它还包含了方位信息。
方位信息就是相对于方法的位置信息，如：方法前、方法后、方法环绕
为什么要方位信息呢？切点不是确定了需要增强的位置了吗？
切点定位的是“在什么类的什么方法上”，也就是说，切点只是定位到了方法本身（也叫执行点，特殊的连接点），但是我们增强的内容是放在该方法的前面呢、后面呢？还是前后都要呢？这些切点却没有告诉我们，那么我们该如何确定具体位置呢？
所以，我们才需要用到方位信息，进一步的定位到具体的增强代码放置的位置。

咦？增强即包含了【功能】又包含了【方位】，那我是不是不用切点就可以匹配哪些方法，并添加功能了呢？
恩，确实如此，因为通过方位信息，虽然只是简单的描述了【功能】需要放在方法前、后、还是前后都要等信息，但是我们还是可以通过方位定位到位置。只不过，是匹配到所有类的所有方法！因为方位只是说明在方法前还是方法后，并没有要求是哪些类？哪些方法？ — So，我们可以直接使用增强来生成一个切面，而不需要切点，但这并不怎么推荐，因为它是匹配所有方法的。所以，我们才需要用切点来进一步确认位置。

### 目标对象（Target）

目标对象就是我们需要对它进行增强的业务类~
如果没有AOP，那么该业务类就得自己实现需要的功能。

## 引介（Introduction）

引介是一种特殊的增强。
它为类添加一些属性和方法。这样，即使一个业务类原本没有实现某个接口，通过AOP的引介功能，我们可以动态的为该业务类添加接口的实现逻辑，让业务类成为这个接口的实现类。

### 织入（Weaving）

织入就是将增强添加到目标类具体连接点上的过程。

1. 编译期织入，这要求使用特殊java编译器
2. 类装载期织入，这要求使用特殊的类装载器
3. 动态代理织入，在运行期为目标类添加增强生成子类的方式
   Spring采用的是动态代理织入，而AspectJ采用编译期织入和类装载期织入。

### 代理（Proxy）

一个类被AOP织入后生成出了一个结果类，它是融合了原类和增强逻辑的代理类根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能就是原类的子类，所以我们可以采用调用原类相同的方式调用代理类。

### 切面（Aspect）

切面又切点和增强（引介）组成，或只是由增强（引介）实现

## 使用Spring实现简单的AOP编程

```java
//定义接口
public interface Calculate {
	public int add(int num1, int num2);
}
//实现接口的方法
@Component
public class Calculator implements Calculate {
	@Override
	public int add(int num1, int num2) {
		return num1 + num2;
	}
}
//日志文件
@Aspect//表示切面
@Component
public class LogUtil {
	@Before(value = "execution(public int com.atguigu.aop.Calculator.add(int, int))")
	public static void logBefore() {
		System.out.println("前置 日记 ");
	}

	@After(value = "execution(public int com.atguigu.aop.Calculator.add(int, int))")
	public static void logAfter() {
		System.out.println("后置 日记 ");
	}

	@AfterReturning(value = "execution(public int com.atguigu.aop.Calculator.add(int, int))")
	public static void logAfterReturn() {
		System.out.println("返回之后： 日记");
	}

	@AfterThrowing(value = "execution(public int com.atguigu.aop.Calculator.add(int, int))")
	public static void logThrowException() {
		System.out.println("抛异常：日记 ");
	}
}
//添加applicationContext.xml配置文件
<context:component-scan base-package="com.atguigu" />
<aop:aspectj-autoproxy />

//测试
public class CalculateTest {
    @Test
    public void test1() throws Exception{
      ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
      Calculator calculator = (Calculator) applicationContext.getBean("calculate");
      calculator.add(100,100);
    }
}
```

## Spring的切入点表达式

@PointCut切入点表达式的语法格式：`execution(访问权限 返回值类型 方法全限定名(参数类型列表))`

限定符：

*：

1. 匹配全类名，任意或者多个方法
   `execution(public int com.atguigu.aop.Calculator.a*(int, int))`表示匹配任何以a打头的方法
2. 在Spring中只有public权限能拦截到，因此访问权限可以省略
   `execution(int com.atguigu.aop.Calculator.*(int, int))`
3. 匹配任意类型的返回值
   `execution(* com.atguigu.aop.Calculator.*(int, int))`
4. 匹配任意子包
   `execution(* com.*.aop.Calculator.*(int, int))`
5. 任意类型参数
   `execution(* com.atguigu.aop.Calculator.*(int,*))`

..:  可以匹配多层路径，或任意多个任意类型参数
`execution(* com..aop.Calculator.*(int,int))`表示com和aop之间可以有任意层级的包
`execution(* com.atguigu.aop.Calculator.*(int,..))`第一个参数是int，后面可以有任意个任意类型的参数

模糊匹配：

- `execution(* *(..))`表示任意返回值，任意方法全限定符，任意参数
- `execution(* *.*(..))`任意返回值，任意包名+任意方法名，任意参数

精确匹配：

`execution(public int com.atguigu.aop.Calculator.add(int, int))`
访问权限 返回值类型 全类名.方法名 （参数类型）

切入点表达式连接： &&, ||

```java
@Before("execution(public int com.atguigu.aop.Calculator.add(int, int))"
    + " && "
    + "execution(public *com.atguigu.aop.Calculator.add(..))")
//表示需要同时满足两个表达式，如果是||则满足一个就会被匹配到
```

## Spring切面中 的代理对象

在Spring中可以对有接口的对象和无接口的对象分别进行处理

1. 如果被代理的对象实现了接口，那么获取对象的时候必须以接口来接受返回的对象
2. 被切面拦截的代理对象如果没有实现接口。获取对象的时候使用对象类型本身

## Spring通知执行顺序

正常情况：前置通知 --后置通知 -- 返回值通知
异常情况：前置通知 --后置通知 -- 抛异常通知

## 获取连接点信息

JoinPoint是连接点的信息，在通知方法的参数重加入一个JoinPoint参数，就可以获取到拦截方法的信息（org.aspectj.lang.JoinPoint这个类）

```java
@Before(value = "execution(public int com.atguigu.aop.Calculator.add(int, int)) || execution(public * com.atguigu.aop.Calculator.d*(..))")
public static void logBefore(JoinPoint joinPoint) {
  System.out.println("前置 日记 ：【" +joinPoint.getSignature().getName() + "】 方法调用前 。参数是："
+ Arrays.asList(joinPoint.getArgs()));
}
```

通过JoinPoint可以获取到执行的方法名和参数

## 获取拦截方法的返回值和抛出的异常信息

获取方法返回值分两步：

1. 在返回值通知的方法中，追加一个参数 Object result
2. 在@AfterReturning注解中添加参数`returning="参数名"`

获取方法抛出的异常分两步

1. 在异常通知爹方法中，追加一个参数 Exception exception
2. 在@AfterThrowing注解中添加参数`throwing="参数名"`

```java
@AfterReturning(value = "execution(public int com.atguigu.aop.Calculator.add(int, int))"  + " || " + "execution(public * com.atguigu.aop.Calculator.d*(..))", returning = "result")
public static void logAfterReturn(JoinPoint joinPoint, Object result) {
  System.out.println("返回之后： 日记 ：【" + joinPoint.getSignature().getName() + "】 方法调用 。参数是："
+ Arrays.asList(joinPoint.getArgs()) + ",返回值：" + result);
}
```

## Spring的环绕通知

1. 环绕通知使用@Around注解
2. 环绕通知如果和其他通知同时执行，环绕通知会优先于其他通知之前执行
3. 环绕通知一定要有返回值(环绕通知如果没有返回值，后面其他的通知就无法接收到目标方法执行的结果)
4. 在环绕通知中，如果拦截异常，一定要往外抛出。否则无法捕获其他异常通知

```java
@Around(value = "execution(* *(..))")
public static Object logAround(ProceedingJoinPoint proceedingJoinPoint) {
  //获取请求参数
  Object[] args = proceedingJoinPoint.getArgs();
  Object resultObject = null;
  try {
    System.out.println("环绕前置");
    //调用目标方法
    resultObject = proceedingJoinPoint.proceed(args);
    System.out.println("环绕后置");
  } catch (Throwable e) {
    System.out.println("环绕异常：" + e);
    throw new RuntimeException(e);
  } finally {
    System.out.println("环绕返回后");
  }
  //返回返回值
  return resultObject;
}
//如有有其他通知，环绕前置 - 普通前置 -目标方法执行 - 环绕后置-环绕错误/返回 -普通后置 -普通后置/返回
```

## 切入点表达式的复用

切入点表达式的复用，分三个步骤

1. 在切面类中定义一个空方法
   `public static void pointcut(){};`
2. 在方法中使用@Pointcut定义一个切入点表达式

```java
@Pointcut(value="execution(public int com.atguigu.aop.Calculator.add(int, int))" + " || "
          + "execution(public * com.atguigu.aop.Calculator.*(..))")
public static void pointcut1() {}
//其他通知的注解可以转换成
@Before("pointcut1()")
```

3. 其他的通知注解中使用方法名()的形式引用方法上定义的切入点表达式
   `@Before("pointcut1()")`

## 多个通知的执行顺序

当有多个通知和多个切面的时候

1. 通知的执行顺序默认是由切面类的字母的先后顺序决定的
2. 在切面类上使用`@Order(number)`注解决定通知执行的顺序（值越小越先执行）

## 基于XML配置AOP程序

配置文件ApplicationContext.xml配置文件内容

```xml
<!-- 注册bean对象
  添加@Component
  -->
<bean id="calculator" class="com.atguigu.aop.Calculator" />
<bean id="logUtil" class="com.atguigu.aop.LogUtil" />
<bean id="validation" class="com.atguigu.aop.Validation" />
<!-- 配置AOP -->
<aop:config>
  <!-- 定义可以共用的切入点 -->
  <aop:pointcut expression="execution(* com.atguigu.aop.Calculator.*(..))" id="pointcut1"/>
  <!-- 定义切面类 -->
  <aop:aspect order="1" ref="logUtil">
    <!-- 这是前置通知 
     method属性配置通知的方法
     pointcut配置切入点表达式
    -->
    <aop:before method="logBefore" pointcut="execution(* com.atguigu.aop.Calculator.*(..))"/>
    <aop:after method="logAfter" pointcut="execution(* com.atguigu.aop.Calculator.*(..))"/>
    <aop:after-returning method="logAfterReturn" 
                         returning="result" pointcut="execution(* com.atguigu.aop.Calculator.*(..))"/>
    <aop:after-throwing method="logThrowException" throwing="exception"
                        pointcut="execution(* com.atguigu.aop.Calculator.*(..))"/>
  </aop:aspect>
  <!-- 定义切面类 -->
  <aop:aspect order="2" ref="validation">
    <aop:before method="before" pointcut-ref="pointcut1"/>
    <aop:after method="after" pointcut-ref="pointcut1"/>
    <aop:after-returning method="afterReturning" 
                         returning="result" pointcut-ref="pointcut1"/>
  </aop:aspect>
</aop:config>
```

# Spring数据访问

在src目录下创建`jdbc.properties`属性配置文件

```properties
jdbc.user=bowie
jdbc.password=123
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.driverClass=com.mysql.cj.jdbc.Driver
initialSize=5
macActive=10
```

配置`applicationContext.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"      xmlns:context="http://www.springframework.org/schema/context"      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

  <!--<context:component-scan base-package="com.demo" />-->
  <context:property-placeholder location="classpath:jdbc.properties"/>
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="${jdbc.user}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="driverClassName" value="${jdbc.driverClass}"/>
    <property name="initialSize" value="${initialSize}"/>
    <property name="maxActive" value="${macActive}"/>
  </bean>
</beans>
```

## jdbcTemplate 使用

在spring中提供了对jdbc的封装类jdbctemplate，可以方便的帮我们执行sql语句，操作数据库。先建立数据库表单

```Mysql
drop database if exists jdbcTemplate;
create database jdbcTemplate;
use jdbcTemplate;
create table `employee`(
`id` int(11) primary key auto_increment,
`name` varchar(100) default null,
`salary` decimal(11,2) default null
);

insert into `employee`(`id`,`name`,`salary`)
values (1,'hbw',9000.5),(2,'bowei',100000),(3,'hebo',40000),(5,'hewei',234652),(4,'bowe',123123);

select * from employee;
```

在`applicationContext.xml`中配置jdbcTemplate

```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <property name="dataSource"  ref="c3p0DataSource"/>
</bean>
```

配置完成后可以在测试文件中修改字段，如下

```java
@Autowired
Jdbctemplate jdbctemplate;
@Test
public void test2() throws Exception {
  // 实验2：将emp_id=5的记录的salary字段更新为1300.00
  String sql = "update employee set salary = ? where id = ?";
  jdbcTemplate.update(sql, 1300, 5);
}
```

#### 批量插入

```java
@Test
public void test3() throws Exception {
  String sql = "insert into employee(`name`,`salary`) values(?,?)";
  ArrayList<Object[]> params = new ArrayList<>();
  params.add(new Object[] {"aaa",100});
  params.add(new Object[] {"bbb",100});
  params.add(new Object[] {"ccc",100});
  jdbcTemplate.batchUpdate(sql, params);
}
```

#### 查询数据库记录，封装为一个java对象或List返回

```java
//创建一个Employee对象
public class Employee {
	private Integer id;
	private String name;
	private BigDecimal salary;

@Test
public void test4() throws Exception {
  // 实验4：查询id=5的数据库记录，封装为一个Java对象返回
  String sql = "select id ,name ,salary from employee where id = ?";
  /**
		 * 在queryRunner中使用的是ResultSetHandler
		 * 	在Spring的jdbcTemplate中，使用RowMapper。
		 * 		BeanPropertyRowMapper 可以把一个结果集转成一个bean对象
		 */
  Employee employee = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper(Employee.class), 5);
  System.out.println(employee);
  
  //查询salary>4000的数据库记录，封装为list返回
  String sql = "select id,name,salary from employee where salary > ?";
  List<Employee> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper(Employee.class), 4000);
  System.out.println(list);
  //查询最大salary
  String sql = "select max(salary) from employee";
  BigDecimal salary = jdbcTemplate.queryForObject(sql, BigDecimal.class);
  System.out.println(salary);
}
```

#### 使用带有具名参数的SQL语句插入一条记录，并以Map或SqlParameterSource形式传入参数值

配置文件中添加

```xml
<!-- namedParameterJdbcTemplate -->
<bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate" >
  <constructor-arg name="dataSource" ref="c3p0DataSource" />
</bean>
```

```java
@Autowired
NamedParameterJdbcTemplate namedParameterJdbcTemplate;

@Test
public void test7() throws Exception {
  String sql = "insert into employee(`name`,`salary`) values(:name,:salary)";
  Map<String, Object> param = new HashMap<String,Object>();
  param.put("name", "小明");
  param.put("salary", new BigDecimal(55));
  namedParameterJdbcTemplate.update(sql, param);
  //SqlParameterSource形式
  SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(new Employee(0,                                                                                          "小新", new BigDecimal(11111)));
  namedParameterJdbcTemplate.update(sql, sqlParameterSource);
}
```

#### 创建Dao，自动装配JdbcTemplate对象

添加类Dao，并在applicationContext.xml文件中添加配置
`<context:component-scan base-package=*"com.atguigu"* />`

```java
@Repository
public class EmployeeDao {

  @Autowired
  JdbcTemplate jdbcTemplate;

  public int saveEmployee(Employee employee) {
    String sql = "insert into employee(`name`,`salary`) values(?,?)";
    return jdbcTemplate.update(sql, employee.getName(), employee.getSalary());
  }
}

//测试代码
@Autowired
EmployeeDao employeeDao;

@Test
public void test9() throws Exception {
  employeeDao.saveEmployee(new Employee(null, "ssssss", new BigDecimal(999)));
}
```

#### 通过继承JdbcDaoSupport创建JdbcTemplate的Dao

```java
@Repository
public class EmployeeDao extends JdbcDaoSupport {

  @Autowired
  protected void initJdbcTemplate(DataSource dataSource) {
    setDataSource(dataSource);
  }

  public int saveEmployee(Employee employee) {
    String sql = "insert into employee(`name`,`salary`) values(?,?)";
    return getJdbcTemplate().update(sql, employee.getName(), employee.getSalary());
  }
}
```

# 声明式事务

事务分为声明式和编程式两种

- 声明式事务：通过注解的形式对食物的各种特性进行控制和管理
- 编程式事务：通过编码的方式实现事务的声明

## 编码式实现业务

```java
//获取数据库连接
Connection connection = JdbcUtils.getConnection();
try{
  //设置手动管理事务
  connection.setAutoConnection(false);
  //执行若干事务
  //提交事务
  connection.commit();
}catch(Exception e){
  connection.rollback();//回滚事务
}finally{
  connection.close();//关闭数据库连接
}
```

## 声明式业务

准备数据库-database tx
table user(`id`,`username`,`monry`)
table book(`book`,`user`)

```JAVA
//创建测试类
public class Book {
	private Integer id;
	private String name;
	private int stock;

public class User {
	private Integer id;
	private String username;
	private int money;

//编写Dao类
@Repository
public class UserDao {
    @Autowired
    JdbcTemplate jdbcTemplate;
    public void updateuser(){
        jdbcTemplate.update("update user set username='user table changed'");
    }
}
  
@Repository
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void uodateBook(){
        jdbcTemplate.update("update book set name='book table changed'");
    }
}
  
//创建transactionService
@Service
public class TransactionService {
    @Autowired
    UserDao userDao;
    @Autowired
    BookDao bookDao;
    public void updateMulti(){
        userDao.updateuser();
        bookDao.uodateBook();
    }
}
  
//测试
@Test
public void test2() throws Exception{
  ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
  TransactionService transactionService = (TransactionService) applicationContext.getBean("transactionService");
  transactionService.updateMulti();
}
```

事务引入的分析-PlatformTransactionManager类介绍

PlatformTransactionManager是Spring提供用来统一管理的接口。
由于这里使用DataSource数据源访问数据库，Spring也提供了一个`DataSourceTransactionManager`类来对这个事务进行管理

```java
//获取数据库连接
Connection connection = JdbcUtils.getConnection();
try{
  //设置手动管理事务
  connection.setAutoConnection(false);//--前置通知 doBegin
  //执行若干事务
  //提交事务
  connection.commit();//----返回通知 doCommit
}catch(Exception e){
  //回滚事务
  connection.rollback();//---异常通知 doRollback
}finally{
  connection.close();//关闭数据库连接
}
//Spring底层事务管理原理：AOP（代理） + 切面（通知）
```

## 使用Spring注解实现声明式事务

在目标对象的方法上使用@Transactional注解来表示此方法交给Spring来管理事务

```java
@Transactional
public void updateUserAndBook() {
  // 更新图书
  bookDao.update(new Book(2, "xxx", 2222));
  // 更新用户
  userDao.update(new User(2, "ssssssss", 2222));
}
```

修改配置文件applicationContext.xml

```xml
<!-- 配置数据库连接池的事务管理器，一般ID只叫transactionManager-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!-- 这里的数据源一定是访问数据库时用的同一个数据源-->
  <property name="dataSource" ref="dataSource" />
</bean>
<!-- 表示支持 @Transactional注解声明式业务，而且会自动代理 
transaction-manager="transactionManager表示使用哪个事务管理器
如果id是transactionManager则此属性可以省略-->
<tx:annotation-driven />
```

## noRollbackFor和noRollbackForClassName测试不回滚的异常

默认情况下，@Transactional是回滚运行时异常，即RuntimeException和其子异常或Error错误

```java
@Transactional(noRollbackFor= {
			//表示算术异常和类型转换异常，不需要回滚事务
			ArithmeticException.class,ClassCastException.class
	})
@Transactional(noRollbackForClassName= {"java.lang.ArithmeticException","java.lang.ClassCastException"})
//表示效果和上面相同
```

## 自定义设置回滚异常

和上面类似。但是由于不是运行时异常在执行时不会回滚，因此可以选择添加

```java
@Transactional(RollbackFor= FileNotFoundException.class)
////表示文件未找到异常也回滚事务
@Transactional(noRollbackForClassName="java.io.FileNotFoundException")
//表示效果和上面相同
```

## 事务只读属性

readOnly属性设置事务是否允许写操作。readOnly默认值时false，表示可读可写。如果设置为true，则表示只支持读操作。
如果在写操作的方法上标为只读，会报异常

`@Transactional(readOnly=true)`

## 事务超时属性timeout

timeout时设置超时属性，单位为秒，表示多少秒之后不允许再执行sql语句

`@Transactional(timeout = 3)`-->表示3秒之后不允许执行sql语句

## 事务隔离级别

四种事务隔离级别：

1. 读未提交：read uncommitted
2. 读已提交：read committed
3. 可重复读：repeatable read
4. 串行事务：serializable

由事务隔离级别产生的几个常见问题

读未提交 ---> 脏读-在某一事务中读取到了其他事务中为提交的数据
读已提交 --->不可重复读-一个事务中多次读取，由于其他事务进行修改操作，导致两次得到的内容不同
重复度 ---> 幻读-每次读取的数据都和第一次的数据相同，不管数据本身是否已经被其他事务所修改

1. 查看当前会话隔离级别：`select @@tx_isolation;`
2. 查看系统当前隔离级别：`select @@global.tx_isolation;`
3. 设置当前会话隔离级别
   `set session transaction isolation level read uncommitted`
   `set session transaction isolation level read committed`
   `set session transaction isolation level repeatable read`
   `set session transaction isolation level serializable`
4. 开启事务： `start transaction`
5. 提交事务：`commit`
6. 回滚事务：`rollback`



## 事务的传播特性propagation

事务的传播行为：当事务方法被另一个事务方法调用时，必须制定事务应该如何传播。比如方法可能继续在现有的事务中运行，也可能开启一个新事物，并在自己的事务中运行。
事务的传播行为可以由传播属性制定。Spring定义了7种类传播行为

| 传播属性     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| REQUIRED     | 如果有事务在运行，当前的方法就在这个事务内运行，否则就启动一个新的事务，并在自己的事务内运行 |
| REQUIRED_NEW | 当前的方法必须启动新事务，并在自己的事务内运行。如果有事务正在运行，应该将它挂起 |
| SUPPORT      | 如果有事务正在运行，当前的方法就在这个事务内运行，否则可以不运行 在事务中 |
| NOT_SUPPORT  | 当前的方法不应该运行在事务中。如果有运行的事务，将它挂起     |
| MANDATORY    | 当前的方法必须运行在事务内部，如果没有正在运行的事务，就抛出异常 |
| NEVER        | 当前的方法不应该运行在事务中，如果有运行的事务，就抛出异常   |
| NESTED       | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行。否则就启动一个新的事务，并在它自己的事务内运行 |

## 注解演示事务的传播特性

创建测试类

```java
@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    @Transactional
    public void updateUser(){
        userDao.updateuser();
    }
}
@Service
public class BookService {
    @Autowired
    private BookDao bookDao;
    @Transactional
    public void updateBook(){
        bookDao.uodateBook();
    }
}
//修改TransactionService类
@Autowired
private BookService bookService;
@Autowired
private UserService userService;
@Transactional
public void multiTransaction(){
  userService.updateUser();
  bookService.updateBook();
}
//测试代码
@Test
public void testMultiUpdate() {
  transactionService.multiUpdate();
}
```

### 大小事务的传播特性都是`REQUIRED`

```java
@Transactional(propagation = Propagation.REQUIRED)
public void multiUpdate() 

@Transactional(propagation = Propagation.REQUIRED)
public void updateBook() 

@Transactional(propagation=Propagation.REQUIRED)
public void updateUser(){}

//当执行multiUpdate方法时，会创建一个事务(multiUpdate)，并获取一个链接
//执行updateUser方法时，加入到事务(multiUpdate)，执行updateUser的sql语句
//执行updateBook方法时，加入到事务(multiUpdate)，执行updateBook的sql语句
//当multiUpdate方法结束时，提交事务，关闭连接
//如果在最后结束之前抛出异常，则事务都不会被提交
```

### 大小传播事务传播特性都是`REQUIRES_NEW`

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void multiUpdate()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateBook()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateUser(){}

//首先开启一个新的事务，运行multiUpdate方法，获取连接
//第二步，执行updateBook方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateBook事务
//		执行updateBook中的操作
//		提交updateBook事务，让操作生效
//		恢复multiUpdate事务
//第三步，执行updateUser方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateUser事务
//		执行updateUser中的操作
//		提交updateUser事务，让操作生效
//		恢复multiUpdate事务
//程序全部执行完之后，提交multiUpdate事务释放资源，关闭连接
//每个事务都是独立的，所以事务提交不会印象
```

### 大事务时`REQUIRED`，小事务时`REQUIRES_NEW`

```java
@Transactional(propagation = Propagation.REQUIRED)
public void multiUpdate()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateBook()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateUser(){}

//当执行multiUpdate方法时，会创建一个事务(multiUpdate)，并获取一个链接
//第二步，执行updateBook方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateBook事务
//		执行updateBook中的操作
//		提交updateBook事务，让操作生效
//		恢复multiUpdate事务
//第三步，执行updateUser方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateUser事务
//		执行updateUser中的操作
//		提交updateUser事务，让操作生效
//		恢复multiUpdate事务
//程序全部执行完之后，提交multiUpdate事务释放资源，关闭连接
//如果代码中有一个位置抛出异常，就会从异常所在的事务开始，到之后所有代码所在的事务 都会失效
//这个测试中每个方法都会开启新的事务
```

### 大事务是`EQUIRED`，小1`REQUIRED`，小2`REQUIRES_NEW`

```java
@Transactional(propagation = Propagation.REQUIRED)
public void multiUpdate()
  @Transactional(propagation = Propagation.REQUIRED)
  public void updateBook()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateUser(){}

//当执行multiUpdate方法时，会创建一个事务(multiUpdate)，并获取一个链接
//执行updateBook方法时，加入到事务(multiUpdate)，执行updateBook的sql语句
//第三步，执行updateUser方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateUser事务
//		执行updateUser中的操作
//		提交updateUser事务，让操作生效
//		恢复multiUpdate事务
//提交multiUpdate方法的事务
//这个测试中，调用multiUpdate和updateUser方法的时候会开启事务。
//如果程序出现异常，异常出现的事务开始，后面的代码操作都不会成功
```

### 大事务是`REQUIRED`，小1`REQUIRES_NEW`，小2`REQUIRED`

```java
@Transactional(propagation = Propagation.REQUIRED)
public void multiUpdate()
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void updateBook()
  @Transactional(propagation = Propagation.REQUIRED)
  public void updateUser(){}

//当执行multiUpdate方法时，会创建一个事务(multiUpdate)，并获取一个链接
//第二步，执行updateBook方法的时候，程序会挂起multiupdate的事务
//		开启一个新的updateBook事务
//		执行updateBook中的操作
//		提交updateBook事务，让操作生效
//		恢复multiUpdate事务
//执行updateUser方法时，加入到事务(multiUpdate)，执行updateBook的sql语句
//提交multiUpdate方法的事务
//这个测试中，调用multiUpdate和updateUser方法的时候会开启事务。
//如果程序出现异常，异常出现的事务开始，后面的代码操作都不会成功
```

# XML配置式事务声明

事务又 AOP + 切面 + 事务属性

```XML
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!-- 事务管理器，需要数据库连接池对象 -->
  <property name="dataSource" ref="dataSource" />
</bean>
<!-- 需要配置方法的事务特性 -->
<tx:advice id="advice1" transaction-manager="transactionManager">
  <tx:attributes>
    <!-- 表示update打头的方法，都使用新的事务 -->
    <tx:method name="update*" propagation="REQUIRES_NEW" />
    <!-- 表示multiUpdate方法都使用新的事务 -->
    <tx:method name="multiUpdate" propagation="REQUIRES_NEW" />
  </tx:attributes>
</tx:advice>		
<!-- 配置切面的配置信息-->
<aop:config>
  <!-- 配置切入点。切入点可以知道有哪些类，哪些方法进入切入管理。 -->
  <aop:pointcut expression="execution(* com.atguigu.service.*Service.*(..))" id="pointcut1"/>
  <!-- 配置切入点中都使用哪些事务特性 -->
  <aop:advisor pointcut-ref="pointcut1" advice-ref="advice1"/>
</aop:config>
```

# Spring整合Web

由于官方不推荐使用new的方法创建Spring IOC容器
`ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");`

因此需要用其他方式创建
`ApplicationContext applicationContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext())`

