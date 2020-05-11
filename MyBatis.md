# 入门

## 安装

如果使用jar包，直接将`mybatis-X.X.X.jar`置于classpath中即可
使用maven构建项目的，将下面的denpendency代码加入`pom.xml`中

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

## 从XML中构建SqlSessionFactory

每个Mybatis都是以一个SqlSessionFactory的实例为中心。SqlSessionFactory的实例可以通过SqlSessionFactoryBuilder来获得，SqlSessionFactoryBuilder可以从XML配置文件或一个预先定制的Configuration的实例构建出SqlSessionFactory实例. **使用Mybaits开发，一个数据库只能有一个SqlSessionFactory对象**

XML配置文件中则包含了对Mybatis系统核心设置，包含DataSource(数据库连接实例的数据源)和TransactionManager(决定事物范围和控制方式的事物管理器)

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

## 从SqlSessionFactory中获取SqlSession

有了SqlSessionFactory之后就可以获得SqlSession实例了。SqlSession完全包含了面向数据库执行SQL命令所需要的所有方法，可以通过SqlSession实例来执行已映射SQL语句。

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  User user = session.selectOne ("com.mybatis.pojo.User.selectUserById", 1);//查找语句
} finally {
  session.close();
}
```

### SQL语句映射

MyBatis提供的全部特性可以利用基于XML的映射语言来实现。在项目中，这个XML文件应建立在相同的package下，命名规则为`calssName + Mapper.xml`。里面的内容如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace 1对应的javabean的全类名 2，对应的Mapper接口全类名-->
<mapper namespace="com.mybatis.pojo.User">
    <!--select 标签用来定义一个select查询语句
        id属性 给select语句定义唯一标识 parameterType定义参数类型 resultType定义返回值数据类型-->
    <select id="selectUserById" parameterType="int" resultType="com.mybatis.pojo.User">
        select id, last_name, sex from t_user where id = #{id}
    </select>
    <delete id="deleteUserById">
        delete from t_user where id = #{id}
    </delete>
    <update id="updateUser" parameterType="com.mybatis.pojo.User">
        update t_user set last_name=#{lastName}, sex=#{sex} where id=#{id}
    </update>
    <insert id="saveUser" parameterType="com.mybatis.pojo.User" >
        insert into t_user (`last_name`,`sex`) values(#{lastName}, #{sex})
    </insert>
</mapper>
```

## Tips

在测试的时候，如果显示
`A query was run and no Result Maps were found for the Mapped Statement`
可能是`Mapper.xml`文件里对应语句的`resultType`没有设置。

在`Mapper.xml`文件中，

- parameterType：传入的参数类型。如果是一个参数，设置String，int即可。如果是好几个变量，可以设置为类(packageName.ClassName)
  有时候可以省略，但是为了阅读方便，建议添加
- resultType：在增删改的时候不需要有。有查询语句的时候需要定义一个返回类型，填写内容同上

如果显示`Mapped Statements collection does not contain value for...`，可能是匹配问题，仔细检查下面几项

- 检查映射器配置文件中的命名空间是不是对应接口的全限定名；
- 检查映射器接口中的方法、参数、返回类型和对应mapper.xml文件中sql语句的id一致（重新复制了一下）；
- 检查是否将mapper.xml文件添加到我在SqlSessionFactory（spring中配置）中配置的configLocation值对应的MyBatis配置文件中；
- 检查我传入SqlSessionTemplate中的sql路径是否是命名空间＋sqlId；
- 检查在SqlSessionFactory（spring中配置）中配置的configLocation值是不是我配置的mybatis配置文件；

## 插入记录并返回主键

在数据库插入数据之后有两种办法可以返回主键信息：

1. 使用insert标签中useGeneratedKeys属性(返回生成的主键)和keyProperty	属性(把返回的主键注入到返回值的哪个属性中，如果是id，则主键注入到返回对象的id属性中)组合获取主键信息
2. 使用子元素 selectKey标签执行sql语句获取

```xml
<insert id="saveUser" parameterType="com.atguigu.pojo.User" useGeneratedKeys="true" keyProperty="id">
		insert into t_user(last_name,sex) values(#{lastName},#{sex})
	</insert>
```

在insert标签中添加一个selectKey来返回插入记录后，自动生成的主键信息

```xml
insert id="saveUser" parameterType="com.atguigu.pojo.User">
		<!-- 
			order 表示执行的顺序，AFTER表示在插入之后执行。BEFORE在插入之前执行。
			keyProperty属性设置对象的哪个属性接收
			resultType属性设置返回值类型。
		 -->
		<selectKey order="AFTER" keyProperty="id" resultType="int">
			SELECT LAST_INSERT_ID()
		</selectKey>
		insert into t_user(last_name,sex) values(#{lastName},#{sex})
	</insert>
```

# Mapper接口实现增删改查

如何确定执行什么语句，比如selectOne 还是selectList：根据接口对应方法的返回值类型来选择不同的语句

Mapper接口方式变成的接口命名一般是`模块名+Mapper`。类似对应模块的配置文件`模块名+Mapper.xml`

#### 编写规范

1.  对应Mapper配置文件的namespace属性值必须是Mapper接口的全类名
2. Mapper接口中的方法名必须与对应Mapper配置文件中对应id相同
3. Mapper接口方法参数类型必须与对应Mapper配置文件的parameterType类型匹配
4. Mapper接口方法的返回值类型必须与对应Mapper配置文件resultType类型匹配

```JAVA
编写测试
static SqlSessionFactory sqlSessionFactory;
String url = "mybatis-config.xml";// 读取配置文件
InputStream inputStream = Resources.getResourceAsStream(url);
// 创建SqlSessionFactory对象
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

// 打印出数据库中所有数据的范例
SqlSession session = sqlSessionFactory.openSession();
try {
  UserMapper userMapper = session.getMapper(UserMapper.class);
  System.out.println(userMapper.findUsers());
} finally {
  session.close();
}
```



## MyBatis核心配置-properties

一般数据库的连接信息会存放在一个`jdbc.porperties`的属性配置文件中

```properties
username=root
password=root
driverClass=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis
```

在`mybatis-config.xml`中修改引入properties资源

```xml
<properties resource="jdbc.properties">
  <!-- 也可以在properties配置中定义一些属性。当然并不推荐 -->
  <property name="username" value="root"/>
  <property name="password" value="root"/>
</properties>

<!-- 在environment中修改信息 -->
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC" />
    <dataSource type="POOLED">
      <!-- 配置数据库连接信息 -->
      <property name="driver" value="${driverClass}" />
      <property name="url" value="${url}" />
      <property name="username" value="${username}" />
      <property name="password" value="${password}" />
    </dataSource>
  </environment>
</environments>
```

## typeAliases

为Java类型设置一个较短的名字，之和XML配置有关。存在的意义只是减少类完全限定名的冗余，在`<configuration>`中设置

```XML
<typeAliases>
	<typeAlias type="com.mybatis.pojo.User" alias="user"/>
  <package name="com.mybatis.pojo"/>
  //通过包来扫描所有的类，自动配上别名
</typeAliases>

也可以在类前定义
@Alias("user2")
public class User
```

## environments

`<environments>`标签可以包含多个环境，其中default表示默认使用的环境。

定义多个环境可以使项目用于不同的场合，比如开发，测试和run

## transactionManager标签说明

`<transactionManager type=*"JDBC"* />`这个配置是直接使用了JDBC的提交和回滚设置，依赖于从数据源得到的连接管理事务范围

```XML
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

MANAGED则几乎不做什么，不提交也不会滚

## dataSource标签说明

`<dataSource type="POOLED">`这个标签配置是否启动数据库连接池配置，有三种属性：

POOLED：池概念，避免了创建新的实例必须要的初始化和认证时间，使兵法Web应用快速响应的流行处理方式

UNPOOLED：数据源的实现知识每次被请求时打开和关闭连接。虽然慢，但是对于没有性能要求的应用程序是一个很好的选择

JNDI：为了能在如EJB或应用服务器这类容器中使用

## databaseIdProvider 

MyBatis可以根据不同的数据库厂商执行不同的语句

```XML
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver" />
  <property name="MySQL" value="mysql" />
  <property name="DB2" value="db2" />
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

## Mapper

把mapper配置文件注入`mybatis-config`配置文件有三种方式

1. 在classpath路径下引入
   `<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>`
2. 使用mapper接口的形式导入配置
   `<mapper class="org.mybatis.builder.AuthorMapper" />`
3. 使用包扫描的方式引入配置文件
   `<package name="org.mybatis.builder" />`



# 使用注解方式

主要还是在XML文件当中写语句。如果使用注解方式，则不需要配置Mapper.xml文件，直接在接口文件上编程

```java
public interface UserMapperAnnotation {
	@Select("select id,last_name userName ,sex from t_user where id = #{id}")
	public User selectUser(int id);

	@Select("select * from t_user")
	public List<User> selectUserList();

	@Update("update t_user set last_name = #{lastName}, sex = #{sex} where id = #{id}")
	public int updateUser(User user);

	@Delete("delete from t_user where id = #{id}")
	public int deleteUserById(int id);

	@Insert("insert into t_user(`last_name`,`sex`) values(#{lastName},#{sex})")
	@SelectKey(before = false, keyProperty = "id", resultType = Integer.class, statement = { "select last_insert_id()" })
	public int insertUser(User user);
}
```

此外`mybatis-config.xml`文件中`<mapper>`标签需要修改为
`<mapper class="com.atguigu.dao.UserMapperAnnotation" />`

#### 数据增删减查方法的返回值问题

对于insert，delete和update三个方法，返回的数值是int，为返回影响行数。比如insert：插入n条记录；对于select语句，由于只是执行查询功能，并不影响数据库，因此方法的返回值类型为void

# mybatis参数传递(直接使用Mapper接口)

### 一个普通数据类型

一个方法只有一个普通数据类型时，在mapper配置文件中使用`#{}`占位符进行占位输出

```xml
public int deleteUserById(int id)

<delete id="deleteUserById" parameterType="int">
  delete from t_user where id = #{id}
</delete>
```

### 多个普通数据类型

有多个普通的参数，需要用占位符输出时，可以用`param1,param2...paramN`输出，即`#{param1}..#{paramN}`;
或者使用@Param命名参数(使用了这个之后，param1，0，1就不能用了)

```xml
public List<User> findUserByNameAndSex(String username, int sex);
//使用param1占位输出参数

<select id="findUserByNameAndSex" resultType="com.atguigu.bean.User" >
  select id,last_name lastName,sex from t_user where last_name = #{param1} and sex = #{param2}
</select>
```

```xml
public List<User> findUserByNameAndSex(@Param("username") String username, @Param("sex") int sex);
  //使用@Param注解命名参数

  <select id="findUserByNameAndSex" resultType="com.atguigu.bean.User" >
    select id,last_name lastName,sex from t_user where last_name = #{lastName} and sex = #{sex}
  </select>
```

## 传递Map对象为参数

可以用map对象的key来作为占位符，输出数据

```java
public List<User> findUserByMap(Map<String, Object> map);

//Test code
public void findUserByMap() {
  SqlSession session = sqlSessionFactory.openSession();
  try {
    UserMapper userMapper = session.getMapper(UserMapper.class);
    Map<String, Object>map = new HashMap<String, Object>();
    map.put("lastName", "admin");
    map.put("sex", 1);
    System.out.println( userMapper.findUserByMap(map) );
  } finally {
    session.close();
  }
}

//Configuration file
<select id="findUserByMap" resultType="com.atguigu.bean.User">
  select id,last_name lastName,sex from t_user where last_name = #{lastName} and sex = #{sex}
</select>
```

## 一个pojo数据类型

可以使用对象的属性名当作占位符名称，
`public int insertUser(User user);`

```xml
<insert id="insertUser" parameterType="com.atguigu.bean.User" useGeneratedKeys="true" keyProperty="id">
  insert into t_user(`last_name`,`sex`) values(#{lastName},#{sex})
</insert>
```

## 多个pojo数据类型

当有多个pojo对象作为参数传递给方法使用的时候，要取出数据作为sql的参数，使用如下的方式

1. #{param1.属性名} ... #{paramN.属性名}

`public List<User> findUserByTwoUser(User user1, User user2);`

```xml
<select id="findUserByTwoUser" resultType="com.atguigu.bean.User" >
  select id,last_name lastName,sex from t_user where last_name = #{param1.lastName} and sex = #{param2.sex}
</select>
```

2. @Param注解命名参数的形式

`public List<User> findUserByTwoUser(@Param("user1") User user1, @Param("user2") User user2);`

```xml
<select id="findUserByTwoUser" resultType="com.atguigu.bean.User" >
  select id,last_name lastName,sex from t_user where last_name = #{user1.lastName} and sex = #{user2.sex}
</select>
```

## 模糊查询

如需要实现如下语句
`select * from t_user where user_name like '%张%'`

有如下两种方法实现

```xml
<select id="findUserLikeName" resultType="com.atguigu.bean.User" >
  select id,last_name lastName ,sex from t_user where last_name like #{name}
  select id,last_name lastName ,sex from t_user where last_name like '%${name}%'
</select>
```

使用第一种`#{name}`会被编译成`?`。而如果要实现模糊搜索，需要传入name值需要带%符号，`%name%`。但是在很多时候收到的是用户输入的，用户并不会输入%，因此这种不适用。需要使用`concat`函数实现：`concat('%',#{name},'%')`

使用第二种`${name}`，会将参数的值原样输出到mysql，然后再进行字符串拼接，最后再mysql语句中执行。这里会编译为`%name%`

# 自定义结果集<resultMap></resultMap>

resultMap标签为自定义结果集标签，可以把查询结果封装为复杂的javaBean对象(属性里有javaBean类型或集合类型的属性的对象为复杂的javaBean)
默认的resultType只能将结果转换为简单的javaBean对象(属性里没有javaBean或集合类型的对象)

其中<id/>中间包含的为主键列，<result/>包含的为非主键列
column为数据库中列名，property为class中变量名

```XML
<!-- 
  resultMap标签专门用来定义自定义的结果集数据。
   type属性设置返回的数据类型
   id属性定义一个唯一标识
  -->
<resultMap type="com.atguigu.bean.Key" id="queryKeyForSimple_resultMap">
  <!-- id负责将主键列转换到javaBean对象的属性，
		column设置哪个主键列转换到指定对象属性
 		property设置将值注入到哪个对象的属性-->
  <id column="id" property="id"/>
  <!-- result 定义一个列和属性的映射 -->
  <result column="name" property="name"/>
  <!-- lock.id 和  lock.name 叫级联属性映射 -->
  <result column="lock_id" property="lock.id"/>
  <result column="lock_name" property="lock.name"/>
</resultMap>
<!-- 
  select 标签用于定义一个select语句
   id属性设置一个statement标识
   parameterType设置参数的类型
   resultMap 设置返回的结果类型
  -->
<select id="queryKeyForSimple" parameterType="int" resultMap="queryKeyForSimple_resultMap">
  select t_key.*,t_lock.name lock_name 
  from 
  t_key left join t_lock
  on
  t_key.lock_id = t_lock.id
  where 
  t_key.id = #{id}
</select>
```

## <association />嵌套结果集映射配置

<association />标签可以返回结果中对象的属性是子对象的情况，进行映射。比如上面Key对象中有一个子对象Lock，则可以通过这种方式映射

```xml
<resultMap type="com.atguigu.bean.Key" id="queryKeyForSimple_resultMap_association">
  <id column="id" property="id"/>
  <result column="name" property="name"/>
  <!-- 
   association 标签可以给一个子对象定义列的映射。
    property 属性设置 子对象的属性名 lock
    javaType 属性设置子对象的全类名
   -->
  <association property="lock" javaType="com.atguigu.bean.Lock">
    <!-- id 属性定义主键 -->
    <id column="lock_id" property="id"/>
    <!-- result 标签定义列和对象属性的映射 -->
    <result column="lock_name" property="name"/>
  </association>
</resultMap>
```

相比resultMap，association标签嵌套的可以在查询的同时获得一个字对象，可以用于实现分布查询：比如一个表有很多的列，但是只有几个列是常用的。为了提高查询效率，我们搜索两个，第一次返回常用列，如果有事务提交，则在查询第二次

此时代码如下，有多步操作：

1. 添加一个LockMapper接口

```java
public interface LockMapper {
	public Lock queryLockById(int lockId);
}
```

2. 添加LockMapper接口对应的配置文件

```xml
<mapper namespace="com.atguigu.dao.LockMapper">
  <!-- 定义一个根据id查询锁的select -->
  <select id="queryLockById" parameterType="int" resultType="com.atguigu.bean.Lock">
    select id , name from t_lock where id = #{value}
  </select>
</mapper>
```

3. 在KeyMapper接口中添加另一个方法：分布查询
   `public Key queryKeyByTwoStep(int id);`

4. 修改KeyMapper配置

```xml
<resultMap type="com.atguigu.bean.Key" id="queryKeyByTwoStep_resultMap">
  <id column="id" property="id"/>
  <result column="name" property="name"/>
  <association property="lock" select="com.atguigu.dao.LockMapper.queryLockById" column="lock_id" />
</resultMap>
<!-- 
  定义分步查询的select
  -->
<select id="queryKeyByTwoStep" parameterType="int" resultMap="queryKeyByTwoStep_resultMap">
  select id,name,lock_id from t_key where id = #{value}
</select>
```

完成之后，测`queryKeyByTwoSteps`可以看到会查询两次，第一次是`queryKeyByTwoSteps` 在key中的查询，第二次执行的是lock对象中的信息，即`queryLockById`

同时为了完善功能，还可以在`mybatis-config.xml`文件中加入延迟加载开关

使用延迟加载可以减少没有必要的查询，提升和优化数据库服务器性能，设置时需要添加两个全局的settings配置

```xml
<!-- 配置全局mybatis的配置 -->
<settings>
  <!-- 启用驼峰标识 -->
  <setting name="mapUnderscoreToCamelCase" value="true" />
  <!-- 打开延迟加载的开关 -->
  <setting name="lazyLoadingEnabled" value="true" />
  <!-- 将积极加载改为消息加载即按需加载 -->
  <setting name="aggressiveLazyLoading" value="false" />
</settings>
```

代码的实现，先执行`Key key = mapper.queryKryByTwoStep(1)`获取key对象。当执行`key.getName`时执行第一次查询。然后可以新建线程等待`Thread.sleep(5000)`，再执行`key.getLock()`进行第二次查询



# 一对多和多对一的使用实例

```My
## 一对多数据表
## 创建班级表
create table t_clazz(
	`id` int primary key auto_increment,
	`name` varchar(50)
);

## 创建学生表
create table t_student(
	`id` int primary key auto_increment,
	`name` varchar(50),
	`clazz_id` int,
	foreign key(`clazz_id`) references t_clazz(`id`)
);
```

## <collection/>一对多，立即加载

1. 定义类：

```java
//学生类
public class Student {
	private Integer id;
	private String name;
//班级类
public class Clazz {
	private Integer id;
	private String name;
	private List<Student> stus;//需要collection标签来处理
```

2. 定义ClazzMapper接口

`public Clazz queryClazzByIdForSample(Integer id);`

3. ClazzMapper.xml配置文件

```xml
<mapper namespace="com.atguigu.mapper.ClazzMapper">
	<resultMap type="com.atguigu.pojo.Clazz" id="queryClazzByIdForSample_resultMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<!-- 
			collection 标签是专门用来配置集合属性的标签
				property属性设置你要配置哪个集合属性
				ofType 属性设置这个集合中每个元素的具体类型
		 -->
		<collection property="stus" ofType="com.atguigu.pojo.Student">
			<id column="stu_id" property="id"/>
			<result column="stu_name" property="name"/>
		</collection>
	</resultMap>

<!-- 		/** -->
<!-- 	 * 根据班级id的信息，直接查询出班级信息，以及这个班的全部学生信息。 -->
<!-- 	 */ -->
<!-- 	public Clazz queryClazzByIdForSample(Integer id); -->
	<select id="queryClazzByIdForSample" resultMap="queryClazzByIdForSample_resultMap">
		select
			t_clazz.*,t_student.id stu_id,t_student.name stu_name
		from 
			t_clazz left join t_student
		on 
			t_clazz.id = t_student.clazz_id
		where 
			t_clazz.id = #{id}	
	</select>
</mapper>
```

## 一对多，赖加载(<collection/>)

1. 新建StudentMapper接口

`public List<Student> queryStudentsByClazzId(Integer clazzId);`

2. 配置StudentMapper.xml文件

```xml
<select id="queryStudentsByClazzId" resultType="com.atguigu.pojo.Student">
  select id,name from t_student where clazz_id = #{clazzId}
</select>
```

3. 修改ClazzMapper.xml配置文件

```xml
<resultMap type="com.atguigu.pojo.Clazz" id="queryClazzByIdForTwoStep_resultMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
  <!-- 
			collection 标签是专门用来配置集合属性的（它可以通过调用一个select查询得到需要集合）。
				property属性设置你要配置哪个集合属性
				select 属性设置你要调用哪个查询
				column 将哪个列的值传递给查询做为参数
		 -->
		<collection property="stus" column="id"			select="com.atguigu.mapper.StudentMapper.queryStudentsByClazzId" />
	</resultMap>
	
	<select id="queryClazzByIdForTwoStep" resultMap="queryClazzByIdForTwoStep_resultMap">
		select id,name from t_clazz where id = #{id}
	</select>
```

## 双向关联

StudentMapper配置文件修改如下

```xml
resultMap type="com.atguigu.pojo.Student" 
		id="queryStudentsByClazzIdForTwoStep_resultMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<association property="clazz" column="clazz_id"
			select="com.atguigu.mapper.ClazzMapper.queryClazzByIdForTwoStep" />
	</resultMap>
<!-- 	/** -->
<!-- 	 * 把学生分两次查，一次还要查班级。 -->
<!-- 	 */ -->
<!-- 	public List<Student> queryStudentsByClazzIdForTwoStep(Integer clazzId); -->
	<select id="queryStudentsByClazzIdForTwoStep" 
		resultMap="queryStudentsByClazzIdForTwoStep_resultMap" >
		select id,name,clazz_id from t_student where clazz_id = #{clazzId}
	</select>
```

修改ClazzMapper配置文件

```xml
<resultMap type="com.atguigu.pojo.Clazz" id="queryClazzByIdForTwoStep_resultMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<collection property="stus" column="id"		select="com.atguigu.mapper.StudentMapper.queryStudentsByClazzIdForTwoStep" />
	</resultMap>
```

### 双向关联可能出现死循环的问题：

解决：

1. 不要调用toString方法
2. 在需要终止调用的最后一次查询使用resultType而不是resultMap

# 动态SQL语句

## if语句

可以动态根据提供的值来决定是否需要动态添加查询条件

```xml
<select id="queryUsersByNameAndSex" resultType="com.atguigu.pojo.User">
  select  id,last_name lastName,sex  from  t_user  where 
  <if test="lastName != null">
    last_name like concat('%',#{lastName},'%')
  </if>
  <if test="sex == 0 || sex == 1">
    and sex = #{sex}
  </if>
</select>
```

## where语句(建议where关键词都用标签替代)

可以在多个动态语句中，有效的去掉前面多余的`and` 和`or`之类的多余关键字。并且在有内容的前提下，会自动添加一个where关键字

```xml
<select id="queryUsersByNameAndSex" resultType="com.atguigu.pojo.User">
  select  id,last_name lastName,sex  from  t_user
  <where>
    <if test="lastName != null">
      last_name like concat('%',#{lastName},'%')
    </if>
    <if test="sex == 0 || sex == 1">
      and sex = #{sex}
    </if>
  </where> 
</select>
```

## trim语句

trim语句可以动态的在包含的语句前后添加内容或者删除前后给定的内容

- prefix 前面添加内容
- suffix 后面添加内容
- suffixOverrides 去掉后面的内容
- prefixOverrides 去掉前面的内容

```xml
<select id="queryUsersByNameAndSex" resultType="com.atguigu.pojo.User">
  select  id,last_name lastName,sex  from  t_user
  <trim suffixOverrides="and" prefix="where">
    <!-- if标签用来判断一个条件是否成立。如果成立 就执行里面的内容 -->
    <if test="lastName != null">
      last_name like concat('%',#{lastName},'%')  and 
    </if>
    <if test="sex == 0 || sex == 1">
      sex = #{sex}
    </if>
  </trim> 
</select>a
```

## choose语句

choose when otherwise可以执行多路选择判断，但是只有一个分支会被执行，类似switch case语句

```xml
<select id="queryUsersByNameAndSexChoose" resultType="com.atguigu.pojo.User">
  select  id,last_name lastName,sex  from  t_user 
  <where>
    <choose>
      <when test="lastName != null">
        last_name like concat('%',#{lastName},'%')
      </when>
      <when test="sex == 0 || sex == 1">
        sex = #{sex}
      </when>
      <otherwise>
        last_name = 'wzg168'
      </otherwise>
    </choose>
  </where> 
</select>
```

## set语句(建议set关键词都用标签替代)

删除条件后面的逗号

```xml
<update id="updateUser" parameterType="com.atguigu.pojo.User">
  update  t_user 
  <set>
    <if test="lastName != null">
      last_name = #{lastName},
    </if>
    <if test="sex == 0 || sex == 1">
      sex = #{sex}
    </if>
  </set>
  where   id = #{id}
</update>
```

## foreach语句

对于UserMapper接口内的方法：`public List<User> queryUsersByIds(List<Integer> ids);`由于传入的是一个List对象，因此需要遍历输出

- collection 表示要遍历的数据源
- items 表示当前遍历的数据
- open 表示遍历之前添加的内容
- close 表示遍历之后添加的内容
- separator 给遍历的每个元素中间添加的内容

```xml
<select id="queryUsersByIds" resultType="com.atguigu.pojo.User">
  select  id,last_name lastName,sex  from  t_user 
  <where>
    id in 
    <foreach collection="list" item="i" open="(" close=")" separator=",">
      #{i}
    </foreach>
  </where>
</select>
```

# MyBatis缓存

缓存是指

1. 把经常需要读取的数据保存到一个告诉的缓冲区中，这个行为叫缓存
2. 也可以指被保存到告诉缓冲区中的数据

一级缓存：把数据保存到SqlSession中
二级缓存：把数据保存到SqlSessionFactory中

### 一级缓存失效的四种情况

1. 不在同一个SqlSession对象中
2. 执行语句的参数不同，缓存中也不存在数据。比如给定`id=1`，传入id和1会存为两条语句
3. 执行增删改语句，会清空缓存
4. 手动清空缓存数据``session.clearCache()`

### 二级缓存

MyBatis的二级缓存默认不开启，需要在MyBatis核心配置文件中配置setting选项
`setting name="cacheEnable" value="true"`

并在Mapper配置文件中加入cache标签(`<cache></cache>`)

同时需要被二级缓存的对象必须要实现java的序列化接口
`public class User implements Serializable`

#### useCache="false"

在select标签中，useCache属性表示是否使用二级缓存，默认是true。即每次查询完之后，会把这个查询数据放到二级缓存中。

#### flushCache="false"

在insert,update,delete标签中，都有flushCache属性，这个属性决定执行完增删改语句之后要不要清空二级缓存，默认值为true

#### <cache></cache>标签的介绍和说明

默认的<cache/>作用：

1. 映射语句文件中的所有select语句会被缓存
2. 映射语句中所有的insert，update,dalete语句会刷新缓存
3. 缓存会使用Latest Recently Used（LRU）算法回收
4. 根据时间表(比如no Flush Interval)，缓存不会以任何的时间顺序来刷新
5. 缓存会存储列表集合或对象的1024个引用
6. 缓存会被视为是read/write的缓存，意味对象检索不是共享的，且可以安全的被调用者修改，而不干扰其他调用者或线程所做的潜在修改

`<cache  eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>`

eviction属性表示缓存策略

- LRU：移除最长时间不被使用的对象
- FIFO：先进先出，按对象进入缓存的顺序来移除
- SOFT：软引用，移除基于垃圾回收器状态和软引用规则的对象
- WEAK：弱引用，更积极地移除基于垃圾回收器状态和弱引用规则的对象

flushInterval：表示间隔多长时间刷新缓冲区，清理溢出单位。以毫秒为单位

size：表示缓存中可以保存多少个对象，默认1024

readOnly：是否只读。设置为true表示缓存中只有一个对象。false则每次取出来都会反序列化拷贝一份

type：表示自定义二级缓存对象

### 缓存使用顺序

1. 当执行一个查询语句的时候，MyBatis会先去二级缓存中查询数据，如果二级缓存中没有，则会去一级缓存中查找
2. 如果二级缓存和一级缓存都没有，会发送sql语句到数据库中查找
3. 查询出来之后马上把数据保存到一级缓存中
4. 但SqlSession关闭的时候，会把一级缓存中的数据保存到二级缓存中

# MyBatis逆向工程

简称MBG，为MyBatis框架使用者定制的代码生成器。可以快速的根据表生成对应的映射文件，接口和Bean类对象。
插件`mybatis-generator-core-1.3.2`可以自动对表单生成增删改查代。在对比数据库表之后，生成大量的基础代码，包含

- 数据库表对用的javaBean对象
- 这些JavaBean对象对应的Mapper接口
- 这些Mapper接口对应的配置文件

生成的MyBatis代码，在每次生成新的代码的时候都需要将原来的旧代码删除