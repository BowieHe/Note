## 多线程和多进程区别：

每个进程有自己一整套变量，而线程则共享数据。同时共享变量使线程之间的通信比进程之间更加有效和容易。

## 线程的6种状态

- New
- Runnable
- Blocked
- Waiting
- Timed waiting(计时等待)
- Terminated

## 线程属性（优先级，守护线程，线程组，处理未捕获异常的处理器）

1. 优先级：默认情况下，线程继承父线程的优先级。或者用`setPriority`提高或降低优先级（MIN_PRIORITY：1，NORM_PRIORITY：5，MAX_PRIORITY：10）
2. 守护线程：唯一用途是为其他线程提供服务，如计时线程。当只剩下守护线程时，虚拟机就退出了
   通过调用`t.setDaemon(true)`将线程转换为守护线程
   守护线程应该永远不去访问固有资源，如文件，数据库
3. 未捕获异常处理器：线程的run方法不能抛出任何异常，但是非受查异常会导致线程终止，因此需要未捕获异常处理器处理异常
   该处理器必须属于一个实现`THread.UncaughtExceptionHandle`接口类，这个接口只有一个方法
   `void uncaughtException(Thread t, Throwable e)`

## 同步

当两个或两个以上线程需共享同一对数据的存取。

锁对象 -> 用ReentrantLock保护代码块的基本结构如下

```java
private Lock myLock = new ReentrantLock();
private Condition sufficientFunds = myLock.newCondition();
myLock.lock(); // a ReentrantLock object 
try{
	while(!(ok to process))sufficientFunds.await()//当发现余额不足时被调用
	critical section;
	sufficientFunds.signalAll();}//释放await
finally{
myLock.unlock()}//make sure the lock is unlocked even if an exception is thrown
```

在锁的时候要注意条件，比如银行取款，不能将锁设置在检查完余额是否足够，这样可能导致余额足够时线程被锁，等待执行时余额又不足的情况。

一个锁对象可以有一个或多个条件对象。通过`newCondition`方法获得一个条件对象，如上示例

等待获得锁的线程和调用await方法存在本质上的不同。等待获得锁在上一个线程释放锁之后就可以进行。而调用 await 方法，会让该线程进入该条件的等待集，当锁可用时，该线程也不能马上接触阻塞。
处于阻塞状态时，直到另一个线程调用同一条件上`signalAll`方法为止
`sufficientFunds.signalAll();`
调用后会激活所有因这条件等待的线程

对await的调用通常在`while(!(ok to process))`中 

---

前面的Lock和Condition提供了高度锁控制，大多数情况下用synchronized关键字声明一个方法，则对象锁会保护整个方法
内部对象锁只有一个相关条件，wait方法添加一个线程到等待集中，notifyAll / notify 用来解除等待线程的阻塞状态，等同于Condition.await()/signalAll()

```java
public synchronized void transfer(...)throws InteruptedException{
  while(accounts[from] < amount) wait();
  process.....
  notifyAll();}
```

tryLock：使用tryLock试图申请一个锁，成功获得后返回true，否则返回false，而且线程可以立即离开去做别的事情

```java
if(myLock.tryLock()){
  try{....//now the thread own the lock}
  finally{myLock.unlock();}
```

可以使用带超时参数的`if (myLock.tryLock(100,TineUnit.MILLISECONDS)) `避免死锁，等待条件时也可以使用超时
`myCondition.await(100, TineUniBILLISECONDS))`

## 执行器

如果程序中有大量的生命周期很短的线程，应使用线程池。一个线程池中包含许多准备运行的 空闲线程。 将 Runnable 对象交给线程池， 就会有一个线程调用 run 方法。 当 run 方法退出时，线程不会死亡，而是在池中准备为下一个请求提供服务

`newCachedThreadPool`构建了线程池，如有有空闲线程，则立即执行任务，没有则新建一个线程
`newFixedThreadPool`构建固定大小的线程池
`newSingleThreadExecutor`退化了的大小为1的线程池

`ScheduledExecutorService`接口具有为预定执行或重复执行任务而设计的方法