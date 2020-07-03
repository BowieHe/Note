使用泛型是代码具有更好可读性，而且调用get时进行类型转换就知道返回值类型为String为不实Object，如`ArrayList<String> files = new ArrayListoO;`

泛型类（generic class）是具有一个或多个类型变量的类
`public class Pair<T>`
在泛型类里面只关注泛型，不会为数据存储的细节烦恼

泛型变量：，任意字符都可以表示任意类型的变量，但是一般来说**E**表示集合的元素类型； **K，V**表示表的关键字与值的类型； **T**表示任意类型

泛型方法可以在普通类中定义，也可以在泛型类中定义
`String middle = ArrayAlg.<String>getMiddle("bowie","he")`
但在以上情况下可以省略<String>，因为可以推断出来。

泛型中类型变量可以进行限定，比如我们要对T所属类用compareTo方法，那我们就可以将T限制为实现了Comparable接口的类
`public static <T extends Comparable> T min(T[] a)`
一个变量也可以有多个限定，比如` extends Comparable & Serializable`

## 泛型代码和虚拟机

当定义泛型类型时，都会自动提供一个原始类型（删去类型参数后的泛型类型名）
擦出类型变量，替换为限定类型（无限定的变量用Object）

如Pair<T>, T是一个无限定的变量，直接用Object替换
Pair<String>, Pair<LocalDate>擦出类型后就变成原始Pair类型

当调用泛型方法时，如果擦出返回类型，编译器会插入强制类型转换

```Java
Pair<Employee> buddies = ,,,;
Employee buddy = buddies.getFirst();//返回Object类型，但会自动转换成Employee类型
```



---

## 约束和局限性

**不能基本类型实例化类型参数**，既没有 `Pair<double>` ，只有`Pair<Double>`。

**运行时类查询只适用于原始类型**：`if(a instanceof Pair<String>)//ERROR`

**不能创建参数化类型数组**：
`Pair<String>[] table = new Pair<String>[10]; // Error`
只是不允许创建数组，声明Pair<String>[] table是合法的，不过不能用`new Pair<String>[10]`来初始化变量

**不能实例化类型变量**：不能使用`new T(...)`这样表达

**禁止使用带有类型变量的静态域和方法**：

**注意擦除后的冲突**：

```Java
boolean equals(String) //defined in Pair<T>
boolean equals(Object) //inherited from Object
擦出boolean equals(T) -> boolean equals(Obecjt)冲突
```

---

## 泛型类型的继承规则

例如 Manager 是 Employee的子类，但是 Pair<Manager> 不是Pair<Employee>的子类

但是可以将参数化类型转换为一个原始类型，如Pair<Manager> 是Pair的一个子类

泛型类型可以拓展或实现其他泛型，如 ArrayList<Manager> 可以转换成List<Manager>

---

## 通配符类型

`Pair<? extends Employee>`，表示任何泛型Pair类型，类型参数是Employee的子类，如`Pair<Manager>`，但不是`Pair<String>`

父类限定 `Pair<? super Manager>`限制为Manager的所有父类型，可以为方法提供参数，但不能使用返回值

无限定通配符。如`Pair<?>`。和Pair不同点在于，可以用任意Object 对象调用原始 Pair 类的 setObject方法。

---

## 反射和泛型

