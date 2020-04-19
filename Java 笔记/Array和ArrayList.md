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

  