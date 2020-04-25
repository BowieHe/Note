Array（[]）：最高效；但是其容量固定且无法动态改变；
`Object [] array = new Object[10]`创建长度为10的数组
`Object [] array = {1,'b','c'}`创建数组，长度为3
`Object [] array =new Object[] {1,'b',"c"};`此类初始化不能在[]中制定长度

---

ArrayList：容量可动态增长；但牺牲效率；
`ArrayList<T> Obj = new ArrayList<>();    obj.add();`
`ArrayList<Type> obj = new ArrayList<Type>(Arrays.asList(Object o1, Object o2, Object o3)`
**ArrayList是Array的复杂版本，因为ArrayList内封装了一个Object类型数组，从一般意义上来说和数组没有区别**

ArrayList实现了List接口，有add，remove，clear等方法

*一般能用数组就用数组，无法确定大小的情况下才使用ArrayList*

> 因为每当执行Add，AddRange，Insert，InsertRange等添加元素时，都会检查内部数组的容量是否足够。若不够则会以原数组的1.5倍容量重新构建一个数组，丢弃久数组。

---

使用ArrayList存入对象时，会抛弃类型信息，所有对象屏蔽为Object。
ArrayList中可以存入各种Object，如String，Employee等，但是不支持基本数据类型（char，int，boolean等），除非用wrapper
Array元素类型既可以是对象，也可以是基本类型

---

### 获取长度

数组：length属性（由于数组长度不可变，所以长度是一个public final成员变量）
ArrayList：size()  （长度可变，类内部可读可写，外部仅可读，因此封装为private）

---

### 增加元素

数组：直接通过序号赋值 `array[0] = 3;`
ArrayList: 通过add方法 `list.add(0, 3);`

---

### 相互转换

- ArrayList转换为Array

  ```java
  ArrayList<String> list = new ArrayList<String>();
  list.add("bowie");
  String[] str = new String[list.size()]; 
  str = list.toArray(str);
  ```

- Array转换为ArrayList

  ```java
  Element[] array = {new Element(1),new Element(2)};
  ArrayList<Element> arrayList = new ArrayList<Element>(Arrays.asList(array));
  ```

  

# Queue

和其他的list不同，Queue一般作为LinkedList出现，而且里面的元素遵循先进先出的原则（First in First out），在尾部添加，头部删除

### 什么时候使用队列

一般情况下，如果是对一些及时消息的处理，并且处理的事件很短的情况下是不需要队列的，直接用阻塞式的方法调用就可以了。但是如果消息处理的时候特别费时间，这个时候如果有新消息来，那就只能处于阻塞状态，造成用户等待。这个时候就要引入队列。当消息接收到之后，先把消息堆入队列中，然后再用可用线程进行处理，这样就不会有消息阻塞了。

### 队列种类

1. 单队列（常见队列，每次都是队尾添加，队首删除）
2. 循环队列

## Queue语法

- add:添加一个元素，如果队列满了抛出IIIegaISlabEepeplian异常
- remove：移除并返回头部元素，空则抛出NoSuchElementException异常
- element：返回头部元素，空则抛出NoSuchElementException异常
- offer：添加元素并返回true，满则返回false
- poll：移除并返回头部元素，空则返回null
- peek：返回头部元素，空则返回null
- put：添加一个元素，如果满了则阻塞
- take：移除并返回头部元素，如果列队为空，则阻塞

|        | Throws exception | return special value |
| ------ | ---------------- | -------------------- |
| insert | Add(e)           | Offer(e)             |
| remove | Remove()         | Poll()               |
| exmain | Element()        | Peek()               |

## 其他

1. 虽然LinkedList没有禁止添加null，但是一般情况下都Queue的实现类都不允许添加null。因为poll和peek方法在异常时都会返回null，无法分别返回的是否正确
2. Queue一般都是FIFO，但是也有例外。比如优先队列(priority queue，顺序根据自然排序或者自定义comparator)；或者LIFO队列(和栈类似，先进后出)
3. 无论进出顺序如何，remove和poll方法操作的都是头部元素，而插入的位置就不一定在队尾了，不同的queue有不同的插入逻辑

