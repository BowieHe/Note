### 子类定义

当子类（subclass）继承了父类（parent class）之后，子类对象可以使用子类里面自定义的方法，同时也自动继承父类中所有的方法和域。
但是父类对象却不能使用子类中定义的方法

### 覆盖方法

由于父类中的方法访问到了私有域，所以子类虽然继承了父类，但是没有权利访问其中的私有域，只能通过公有接口来访问，比如父类中的公有方法。

但是如果我们希望调用父类中的公有方法而不是当前类的方法，用super解决
`double baseSalary = super.getSalry()`

### 子类构造器

由于子类的构造器不能访问父类的私有域，因此必须利用父类的构造器对这部分的数据进行初始化，即通过super实现对构造器调用。
`super(name, salry, age)`
使用super调用构造器的语句必须是子类构造器的第一条语句
_如果子类构造器中没有调用父类的构造器，则默认调用父类的无参构造器_

### 多态

在Java中对象的变量的多态的，一个Employee变量既可以引用一个Employee类对象，也可以引用任何一个Employee的子类对象
`Employee name = new Manager();`

```java
Manager boss = new Manager();
Employee[] stuff = new Employee[3];
stuff[0] = boss;
boss.setBonus(500); //OK
stuff[0].setBonus(500) //Wrong
```

在这个例子中，stuff[0] 和 boss 调用同一个对象，编译器将stuff[0] 看成Employee的对象，因此可以直接调用
不能第二种方法调用是因为setBonus 是Manager的方法，而不是Employee的方法

同时也不能将父类对象赋值给子类对象。~~Manager m = stuff[0];~~

### final类和方法：阻止继承

`public final class Exclusive extends Manager`使用final可以阻止定义Exclusive的子类，但同时Exclusive里的所有方法都自动编程final方法

### 强制转换类型

`Manager boss = (Manager) stuff[0]`
强制转换的唯一原因是需要将元素复原成子类，来访问新增加的变量
**将一个子类的引用赋给一个父类是允许的，但是一个父类的引用赋给子类，必须要类型转换**
在将父类转换成子类前先用`x instanceof c`来检查，不会返回错误，只返回false

### 抽象类

对于一些类只想作为基类，而不是使用的特定的实例类。使用 abstact 可以创建方法和类，但是不需要实现这个方法
抽象方法充当占位的角色，具体实现在子类中。
类即使没有抽象方法，也可以定义为抽象类。
**抽象类不能实例化**

```java
new Person("bowie")  //Wrong
Person p = new Student("bowie") //correct
```

### 受保护访问

通常父类中的域会被设为私有域。但是有时候会希望父类中的方法允许被子类访问，或者子类方法访问父类中的某个域，这时可以用 protected， 如Manage可以访问Employee的 age。
但是Manager的子类只能访问Manager的age，不能访问Employee中的age

- private 仅本类可见
- public 对所有类可见
- protected 对本包所有子类可见
- 默认。 对本包可见

---

# Object

Object是所有类的始祖，每个类都由Object拓展而来
可以使用Object类型的变量引用任何类型的对象
`Object obj = new Employee("bowie")`

Object里equals的方法可以用于检测一个对象是否等于另一个对象

也可以使用hashCode方法来判定是否相同，不同对象的hashCode基本不会相同

`toString()`用于返回表示对象值的字符串
重写toString可以`return getClass().getName() + "name" + name;`
如果父类写了`getClass().getName()` 那么子类只用`super.toString()`

# 泛型数组列表

ArrayList 是一个采用类型参数 (type parameter ) 的泛型类(generic class )。 为了指定数组列表保存的元素对象类型，需要如下定义 ArrayList < Employee>。
`ArrayList<Employee> staff = new ArrayList<Eniployee>(100);`

使用add将对象添加入Arraylist
`stuff.add(index, new Employee("bowie"))` /  在中间插入元素会使index后都后移一位（**remove同理**）
`stuff.add(new Employee("bowie"))` 

使用set方法设置ArrayList某元素
`stuff.set(0, "Harry")`

使用get方法获得ArrayList元素
`Employee e = stuff.get(0);`

---

### Wrapper

对象包装器用于将基本类型转换成对象，比如Integer，Character，Float，Long，Double，Short，Void，Boolean。
**对象包装器由final修饰，一旦构造，就不允许修改包装在其中的值**

为了便于将基本数据类型添加到ArrayList，使用`list.add(3)` 时会自动装箱成`list.add(Integer.valueOf(3))`
当一个Integer对象赋值给int时，则会自动拆箱`int n=list.get(i).intValue()`

> IntValue() : 以int形式返回Integer对象的值
> toString(int i, int radix) 以新String对象形式返回i以radix参数进制表示
> parseInt(String s, int radix)返回字符串s表示的int数值，以radix表示进制
> valueOf(String s, int radix) 返回s表示整型数值初始化后一个新的**Integer对象**，按照radix进制表示

---

