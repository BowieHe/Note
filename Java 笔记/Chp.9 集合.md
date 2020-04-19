# Java集合框架

## 集合的接口和实现分离

队列（queue）接口是指可以在队列尾部添加元素，然后在队列的头部删除元素，并可以查找队列中元素个数

```java
public interface Queue<E>{
  void add(E element);
  E remove();
  int size();
}
```

### collection接口

```java
public interface Collection<E>{
  boolean add(E element);
  Iterator<E> iterator();//返回一个实现了Iterator接口的对象
}
```

### 迭代器

```java
public interface Iterator<E>{
  E next();
  boolean hasNext();
  void remove();
  default void forEachRemaining(Consumer<?super E>action);
}
```

通过next逐个访问元素，再此之前用hasNext方法查看

## 集合框架中的接口

集合有两个基本接口，Collection和Map，可以用`boolean add(E element)`插入元素。但是由于映射包含键值，用 `V put(K key, V value)`插入，从映射中读取用`V get(K key)`

set接口等同于Collectoin接口，但是set添加元素时不允许添加重复的元素



# 具体的集合

实现Collection接口：ArrayList, LinkedList, ArrayQueue, HashSet, TreeSet, EnumeSet, LinkedHashSet, PriorityQueue

实现Map接口：HashMap, TreeMap, EnumMap, LinkedHashMap, WeakHashMap, IdentityHashMap

## 链表

通过`void add(E element);`添加元素，`E previous()`和`boolean hasPreviousO`反向遍历链表
`remove`指令，在调用next之后，remove会删除左侧的元素，但是如果调用的是previous，则会删除右侧的元素。同时==不能连续使用两次remove==

```java
				List<String> a = new LinkedList<>();
        a.add("new");
        a.add("next");
        List<String> b = new LinkedList<>();
        b.add("bowie");
        b.add("he");
        ListIterator<String> ait = a.listIterator();
        ListIterator<String> bit = b.listIterator();
        // add element in list b to a
        while(bit.hasNext()){
            if(ait.hasNext())ait.next();
            ait.add(bit.next());
        }
        System.out.println(a);
        //recover bit from b
        bit = b.listIterator();
        while(bit.hasNext()){
            bit.next();
            if(bit.hasNext()){
                bit.next();
                bit.remove();
  
 void addAll(int i, E element) //指定位置添加元素
 E set(int i, E element) //新元素取代指定位置元素
 int indexOf(Object element)/lastIndexOf //返回指定元素相等的元素在列表中第一次和最后一次出现的位置
 E get(int i) //获取指定位置元素
```

Queue

- boolean add()//添加元素到队列尾部并返回true，满了则抛出错误
- Boolean offer()//同上，满了返回false
- E remove()//如果列队不空，删除并返回头部元素，空则抛出异常
- E pool()//同上，空则返回null
- E element()//返回列队头部元素。如果列队为空抛出异常
- E peek()//同上，列队为空则返回null

Deque

- void addFirst/addLast(E element) //将对象添加到头尾，满了抛出异常
- Boolean offerFirst/offerLast(E element)//同上，满了返回false
- E removeFirst/removeLast//删除并返回列队头元素，为空则抛出异常
- E pollFirst/pollLast//同上，列队空则返回null
- E getFist/getLast//返回列队头元素，为空则抛出异常
- E peekFirst/peekLast//返回头尾元素，为空返回null

ArrayDeque

# 映射

通用实现为 HashMap和TreeMap

相比TreeMap，HashMap稍快一些，如果不需要按照排列顺序访问键，最好选择散列映射

```java
V get(Object key)//获取与键对应的值，没有则返回null
default V getOrDefault(Object key, V defaultValue)//获取与键关联的值，没有则返回defaultValue
V put(K key, V value)//将键值插入，如已经存在则替换，同时返回旧值
void putAll(Map<? extends K, ? extends V> entries)//将给定映射中所有条目添加到这个映射中
boolean containsKey(Object key)//如果有这个键，返回true
boolean containsValue(Object value)//如果有这个值，返回true
default void forEach(BiConsumer<? super K,? super V> action)//
  对所有键值做这个动作
```

更新映射对象值

```java
counts.put(word, counts.get(word)+ 1);
counts,put(word, counts.getOrDefault(word, 0) + 1);
counts.merge(word, 1, Integer::sum);
```

映射的视图-是实现了Collection接口或某个字接口的对象
有三种视图：

- 键集    ->  Set<K> keySet()
- 值集合  -> Collection<V> values()
- 键/值对集 ->  Set<Map, Entry<K, V>> entrySet<>

```java
Set<String> keys = map.keySetO; 
for (String key : keys) //枚举一个映射中所有的键
  
for (Map.Entry<String, Employee> entry : staff.entrySetO) {
String k = entry.getKey(); 
Employee v = entry.getValue(); //一个映射中国呢所有键/值
```

# 视图与包装器







# 算法

## 排序和混排

Collections类中sort方法实现了对List接口的集合的排序
`List<String> staff = new LinkedListoO;  Collections, sort (staff) ;`
如果想按照其他方式排序，可使用List接口的sort方法并传入Comparable对象
`staff _sort(Comparator.comparingDouble(Employee::getSalary));`

想要降序排列。可以使用`staff.sort (Comparator.reverseOrder())`
`staff.sort(Comparator.comparingDouble(Employee::getSalary).reversed())`

## 二分查找

要查找元素必须要提供集合和查找的元素，而且集合要实现List接口，没有则提供一个比较器
`Collections.binarySearch(C, element)`
`Collections.binarySearch(C, element, comparator)`

## 简单算法

1. 返回集合中最大最小元素
   static <T extends Comparab1e<? super T>>T min(Collection<T> elements) 
   static <T extends Comparable<? super T>> T max(Col1ection<T> elements) 
   static <T> min(Col1ection<T> elements, Comparator ? super T> c) 
   static <T> max(Col1ection<T> elements, Comparator ? super T> c)
2. 原列表中所有元素复制到目标列表
   static <T> void copy(List<? super T> to, List<T> from)
3. 列表中所有位置设为相同值
   static <T> void fi11(List<? super T> 1， T value)
4. 所有值添加到集合中
   static <T> boolean addAl1(Collection<? super T> c, T... values)
5. NewValue取代OldValue
   static <T> boolean replaceAl1(List<T> 1, T oldValue, T newValue)
6. 返回第一个和最后一个字列表索引
   static int indexOfSubList(List<?> 1, List<?> s) 
   static int 1astlndexOfSubList(List<?> 1, List<?> s) 
7. 逆转列表元素顺序
   static void reverse(List<?> 1)
8. 返回c中与o相同的元素个数
   static int frequency(Col1ection<?> c, Object o) 
9. 没有相同元素则返回true
   boolean disjoint(Collection<?> cl, Col1ection<?> c2)

## 批操作

1. coll1.removeAll(coll2) // 从coll1中删除所有coll2元素
2. set.retainAll(b) //set和b中的交集
3. removeAll

## 集合和数组转换

数组 -> 集合

```java
String[] values = . .
HashSet<String> staff = new HashSet<>(Arrays.asList(values));
```

集合 -> 数组

```java
Object[] values = stuff.toArray();
String[] values = stuff.toArray(new String[0];
stuff.toArray(new String[stuff.size()])
```

## 栈

1. E push(E item)
2. E pop()
3. E peek()

