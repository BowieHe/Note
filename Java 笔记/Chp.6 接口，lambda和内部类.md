# 接口

在 Java 程序设计语言中， **接口不是类， 而是对类的一组需求描述**， 这些类要遵从接口描 述的统一格式进行定义。

**接口没有实例，因此不能含有实例域**，提供实例域和方法实现的任务应该由实现接口的那个类来完成。 因此，可以将接口看成是没有实例域的抽象类

为了让类实现一个接口，通常有两个步骤

- 将类声明为实现给定的接口

- 对接口中的所有方法进行定义

  `class Employee implements Comparable`

比如Array中sort可以对对象中的数组进行排序，但前提是对象所属的类满足Comparable接口,任何实现Comparable接口的类都需要有compareTo方法

```java
public interface Comparable{
  int compareTo(Object other);
}

//实现
class Employee implements Comparable<Employee>{
  public int compareTo(Employee other){
    return DOuble.compare(salary, other.salary)
  }
}
```

## 接口特性

接口不是类，不能用new运算符实例化一个接口 `x = new Comparable()//ERROR`
但是可以声明接口变量 `Comparable x；//OK`
接口中不能包含实例域和静态方法，但是可以包含常量`int age = 18;//OK`
使用`if(abyObject instanceof Comparable)`检查对象是否实现了某接口

接口和抽象类区别：每个抽象类只能拓展一个类，但是每个类可以实现多个接口

```java
class Employee extends Person, Comparable //ERROR
class Employee extends Person implements Comparable //OK
```

## 解决冲突

如果一个接口中将一个方法定义为默认方法，然后在另一个父类或者接口中定义了相同的方法，这时应对规则如下：

- 父类优先，如果父类提供了具体方法，同名且有相同参数的默认方法会被忽略
- 接口冲突。如果一个超接口提供了一个默认方法， 另一个接口提供了一个同名而且 参数类型(不论是否是默认参数)相同的方法， 必须覆盖这个方法来解决冲突。
- 一个类拓展了父类，同时实现了接口，此时如果有相同的方法，Java只会考虑父类方法，接口所有默认方法都会被忽略

# lambda

lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）
常见表达形式` (参数) -> {statements}`
`Arrays.sort(age, (first,second)->first.length() - second.length());`
lambda方法即使没有参数也要提供空括号，而且如果参数类型可以推导出来，可以忽略类型。
如果只有一个参数，且类型可以推导出，那甚至可以省略小括号

对于一个只有抽象方法的接口，需要接口对象时，可以提供一个lambda表达式（函数式接口）

### 方法引用

`Timer t = new Timer(1000, event -> System.out.println(event)):`
可以使用方法引用，等价于lambda表达式 ` x一> System.out.println(x)`
`Timer t = new Timer(1000, Systei.out::println);`

有如下三种情况

- object::instanceMethod
- Class::staticMethod
- Class::instanceMethod

前两种等同于lambda表达式，`Math::pow 等价于(x，y) ->Math.pow(x,y)`
第三种第一个参数会成为方法目标，例如String::compareToIgnoreCase  等同于`(x, y) -> x.compareToIgnoreCase(y)`

由于lambda可以捕获外围作用域中变量的值，但是在lambda表达式中，只能引用值不会改变的变量

使用 lambda 表达式的重点是延迟执行 deferred execution ) 毕竟， 如果想耍立即执行代 码， 完全可以直接执行， 而无需把它包装在一个丨ambda 表达式中。

要接受lambda表达式，需要选择一个函数式接口，比如Runnable，Consumer，Function，BinaryOperator等等



# 内部类

inner class是定义在另一个类中的类

- 内部类方法可以访问该类定义所在的作用域中的数据， 包括私有的数据。
- 内部类可以对同一个包中的其他类隐藏起来。
- 当想要定义一个回调函数且不想编写大量代码时，使用匿名(anonymous) 内部类比较便捷。

内部类所有的静态域都必须是final，而且内部类不能用static方法

局部内部类不能用private或者public访问说明符进行声明，作用域被限定在声明这个局部类的块中

匿名内部类主要用来实现事件监听和其他回调，通常的语法是

```Java
new SuperType(construction parameter){
  inner class methods and data};
```

由于匿名构造类没有类名，因此不能有构造器。取而代之，是将构造器参数传递给superClass构造器

静态内部类：为了把一个类隐藏在另一个类内部，不需要内部类引用外围对象，可将内部类声明为static，取消产生的引用
只有内部类可以声明为static。静态内部类的对象除了没有对生成它的外围类对象 的引用特权外， 与其他所冇内部类完全一样。



# 代理（Proxy）

利用代理在运行时创建一个实现了一组给定接口的新类。只在编译时无法确定要实现哪个接口时使用

Java里存在**静态代理和动态代理**

## 静态代理

静态代理在使用的时候，需要定义接口或者父类，被代理的对象与代理对象一起实现相同的接口或者继承相同父类

比如模拟保存动作，定义一个保存动作的接口:IUserDao.java，然后目标对象实现这个接口方法UserDao.java，此时如果使用静态代理，需要在代理对象(UserDaoProxy.java)中也实现IUserDao接口。调用的时候通过调用代理对象的方法来调用目标对象。

```java
//接口：IUserDao.java
public interface IUserDao{
  void save();
}
//目标对象：UserDao.java
public class UserDao implements IUserDao{
  public void save(){
    System.out.println("data has been saved");
  }
}
//代理对象：UserDaoProxy.java
public class UserDaoProxy implements IUserDao{
  private IUserDao target;//保存目标对象
  public UserDaoProxy(IUserDao target){
    this.target=target;
  }
  public void save(){
    System.out.print("start...");
    target.save();//执行目标对象方法
    System.out.print("end....");
  }
}
//测试类：App.java
public class App{
  public static void main(String[] args){
    UserDao target = new UserDao();//目标对象
    //代理对象，把目标对象传给代理对象，建立代理关系
    UserDaoProxy proxy = new UserDaoProxy(target);
    proxy.save();//执行代理方法
  }
}
```

静态代理可以做到在不修改目标对象的功能前提下，对目标功能进行拓展

缺点：由于代理对象需要和目标对象实现一样的接口，所以会有很多代理类。类太多的同时，一旦接口增加方法，目标对象与代理对象都需要维护

## 动态代理

特点：

- 代理对象，不需要实现接口
- 代理对象的生成，是利用JDK的API，动态的在内存中构建代理对象（需要我们指定创建代理对象/目标对象实现的接口的类型）
- 动态代理也叫做：JDK代理，接口代理

JDK中生成代理对象的API：(java.lang.reflect.Proxy)
JDK实现代理只需要使用newProxyInstance方法，但是该方法需要接受三个参数
`static Object newProxyInsatance(ClassLoader loader, Class<?>[] interface, InvocationHandler h)`

- ClassLoader loader：指定当前目标对象使用类加载器，获取加载器的方法是固定的
- Class<?>[] interface：目标对象实现的接口的类型，使用范型方式确认类型 
- InvocationHandler h：事件处理，执行目标对象的方法时，会触发事件处理器的方法，会把当前执行目标对象的方法作为参数传入。

接口类IUserDao.java以及接口实现类，目标对象UserDao是一样的。在这个基础上，增加一个代理工厂类（ProxyFactory.java），将代理类写在这个地方，然后在测试类（需要用到代理的代码）中先建立目标对象和代理对象的联系，然后代用代理对象的同名方法

```java
//ProxyFactory.java
public class ProxyFactory{
  private Object target;
  public ProxyFactory(Object target){
    this.target.target;
  }
  public Object getProxyInstance(){
    return Proxy.newProxyInstance(
      target.getClass().getClassLoader(),
      target.getClass().getInterface(),
      new InvocationHandler(){
        @Override
        public Object invoke(Object proxy, Method method, Object[] args)throw Throwable{
          System.out.print("start1...");
          //执行目标对象方法
          Object.returnValue = method.invoke(target,args);
          System.out.print("submit...");
          return returnValue;
        }
      }
    );
  }
}

//App.java
public class App{
  public static void main(String[] args){
    //目标对象
    IUserDao target = new UserDao();
    //原始类型class cn.itcast.b_dynamic.UserDao
    System.out.print(target.getClass());
    //给目标对象，创建代理对象
    IUserDao proxy = (IUserDao)new ProxyFactory(target).getProxyInstance();
    //class $proxy 内存中动态生成的代理对象
    System.out.print(proxy.getClass());
    //执行方法（代理对象）
    proxy.save();
  }
}
```

其中代理对象不需要实现接口，但是目标对象一定要实现接口，否则不能用动态代理

## Cglib代理

静态代理和动态代理模式有个相同点就是都要求目标对象是实现一个接口的对象，但是并不时所有的对象都会实现一个接口，也存在没有任何接口的对象。这时就可以用继承目标类以目标对象子类的方式实现代理，这就是Cglib代理，也叫子类代理。是在内存中构建一个子类对象从而实现对目标对象功能的拓展。

- JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
- Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的interception(拦截)
- Cglib包的底层是通过使用一个小而块的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

Cglib代理的类不能为final，否则会报错。同时目标对象的方法如果是final/static，那么就不会执行目标对象额外的业务方法

```java
//目标对象
public class UserDao{
  public void save(){
    System.out.print("saved...");
  }
}
//Cglib子类代理工厂
public class ProxyFactory implements MethodInterceptor{
  //维护目标对象
  private Object target;
  public ProxyFactory(Object target){
    this.target = target;
  }
  //给目标对象创建一个代理对象
  public Object getProxyInsstance(){
    //1.工具类
    Enhancer en = new Enhencer();
    //2.设置父类
    en.setSuperclass(target.getClass());
    //3.设置回调函数
    en.setCallback(this);
    //4.创建子类（代理对象）
    return en.create();
  }
  @Override
  public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable{
    System.out.print("start..");
    //执行目标对象方法
    Object returnValue = method.invoke(target, args);
    System.out.print("submitted..");
    return returnVlaue;
  }
}
//测试
public class App{
  @Test
  public void test(){
    //目标对象
    UserDao target = new UserDao();
    //代理对象
    UserDao proxy = (UserDao)new ProxyFactory(target).
      getProxyInstance();
    //执行代理对象的方法
    proxy.save();
  }
}
```

