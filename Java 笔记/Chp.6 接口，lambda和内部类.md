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

