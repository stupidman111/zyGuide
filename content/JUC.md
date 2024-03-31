# Java中的线程

## Case1-Main线程与Main方法
> 线程是怎么执行的？main方法和当前线程是什么关系？
```java
public class Test01 {  
    public static void main(String[] args) {  
       System.out.println(Thread.currentThread().getName());  
    }  
}
```
> main方法和当前线程没有任何关系，当我们启动程序时，JVM创建主线程（main线程）来执行main方法

## Case2-run方法与target成员
> 为什么Thread类中有run方法？

> 因为Thread类继承了Runnable接口，实现了Runnable接口中的run方法。
```java
public class Thread implements Runnable {
}
```
> Thread类中实现的run方法。
```java
@Override  
public void run() {  
    if (target != null) {  
        target.run();  
    }  
}
```
> target是什么？
```java
/* What will be run. */  
private Runnable target;
```
> target是Thread方法中的一个成员变量，我们在创建Thread实例时，可以传入一个Runnable对象作为参数，会赋值给改实例的target成员，之后就会调用该target重写的run方法。

## Case3-分析线程信息的指令
> 可以使用 jps 列出当前正在运行的所有线程及其pid；
> 根据上一步得到的pid，使用 jstack pid 来查看改线程的详细信息；

## Case4-run方法与start方法
> 直接调用run方法能启动线程吗？
> 不能，直接调用run方法，执行改run方法的线程是调用run方法的线程

> start方法是什么？
> start方法内部会调用native方法start0，start0会启动一个线程并调用当前线程对象的run方法。
```java
public synchronized void start() {
	//...
	start0();  
	//...
}
```
> 多次调用start方法可以吗？ 
> 不可以，会抛出 java.lang.IllegalThreadStateException 异常

> 调用start方法代表了run的执行顺序吗？
> 不能代表，调用start方法会使JVM告知OS创建线程，然后调度线程，执行线程，具体什么时候调度、什么时候执行是由OS决定的，我们控制不了。

## Case5-Thread与Runnable
> 实现Runnable接口作为Thread实例的构造参数有什么好处？
> 因为Java是单继承的，如果我们采用自定义类继承Thread类的方式的话，这个自定义类就无法再继承其他类了；但是如果采用实现Runnable接口方式的话，该自定义类还有继承其他类的余地；
> 具体地，Thread类提供了多个带有Runnable类型参数的构造方法，我们将实现了Runnable接口的自定义类作为Thread实例的构造参数，就是将该自定义类转型为了Runnable，并赋值给了Thread的target成员。

> 继承自Thread类的自定义类，为什么还能够作为Thread实例的构造参数？
> 因为Thread类本身就实现了Runnable接口，所以继承自Thread类的自定义类也间接的实现了Runnable接口，所以可以作为Thread的构造参数。


# Java线程的优先级
> Java中为线程设置优先级只是我们提供给OS一个参考，具体是否优先级变得更高、更低完全由OS来掌握。

* 设置优先级：`setPriority()`
* 优先级范围：1~10（大于10或小于1会抛出`IllegalArgumentException`，Thread类为我们提供了`1, 5, 10`这三个取值的优先级，
```java
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```
* 线程的优先级具有继承性，例如，A线程启动B线程，则B线程的优先级与A线程是一样的。
* ...

# 守护线程
> Java中有两种线程：一种是用户线程，也称非守护线程；另一种是守护线程。

* 守护线程：
	* 当Java进程中不存在用户线程（非守护进程）了，则守护线程自动销毁；
	* 典型的守护线程是垃圾回收线程（GC线程）；
	* 守护线程（Daemon线程）的作用的是为其它线程提供服务，比如GC线程用于回收垃圾，以保证其他线程能够正常运行；
	* 使用`.setDaemon(true)`将一个线程对象设置为守护线程；
```java
//MyThread，run方法每1秒打印一次i的值
public class MyThread extends Thread {  
    private int i = 0;  
    @Override  
    public void run() {  
       try {  
          while (true) {  
             i++;  
             System.out.println("i=" + (i));  
             Thread.sleep(1000);  
          }  
       } catch (InterruptedException e) {  
          e.printStackTrace();  
       }  
    }  
}

//设置myThread线程并设置其为守护线程--setDaemon(true)
public class Run {  
    public static void main(String[] args) {   
       try {  
          MyThread myThread = new MyThread();  
          myThread.setDaemon(true);//设置myThread为守护线程  
          myThread.start();  
          Thread.sleep(5000);  
          //在Main线程睡眠5s后，Main线程执行完销毁，由于此时没有其他非守护线程了，那么守护线程myThread就会自动销毁  
       } catch (InterruptedException e) {  
          e.printStackTrace();  
       }  
    }  
}
```

# synchronized、volatile
> 非线程安全：多个线程在对同一个对象中的实例变量进行并发访问时产生，结果就是会产生脏读（读取到的数据是被更改过的）
> 线程安全：获取实例变量的值的动作是经过同步处理过的，不会出现

## synchronized
> synchronized可用来保证：原子性，可见性，有序性

* 方法内的变量是线程安全的；
* 实例变量是非线程安全的（多个线程可能操作同一个对象实例的实例变量）；
* `synchronized修饰方法`是通过在方法的`flags`中加上标记`ACC_SYNCHRONIZED`来实现的，当调用synchronized方法时，调用指令会检查方法的`ACC_SYNCHRONIZED`访问标志是否设置了，若设置了，执行线程需要先持有同步锁，才能执行方法，在方法完成时要释放同步锁
```java
public class Main {  
    synchronized public static void testMethod() {  
  
    }  
  
    public static void main(String[] args) {  
       testMethod();  
    }  
}

//使用javac Main.java编译成字节码Main.class
//使用javap -c -v Main.class将class文件转换为字节码指令
//其中，synchronized修饰的testMethod如下：
 public static synchronized void testMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=0, args_size=0
         0: return
      LineNumberTable:
        line 9: 0

//其中可以发现testMethod下的flags中带有一个 ACC_SYNCHRONIZED 标志
```
* `synchronized修饰代码块`，是在代码块开头加一个`monitorenter`、两个`monitorexit`实现的；
```java
public class Main {  
    public  void testMethod() {  
       synchronized (this) {  
          System.out.println("test");  
       }  
    }  
  
    public static void main(String[] args) {  
       Main main = new Main();  
       main.testMethod();  
    }  
}

//使用javac Main.java编译成字节码Main.class
//使用javap -c -v Main.class将class文件转换为字节码指令
//其中，synchronized修饰的代码块如下：
 public void testMethod();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter 
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3                  // String test
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return

```
* 多个线程执行同一个对象的synchronized实例方法时，先拿到对象锁的线程先执行，其他线程只能处于等待状态；
* 只有`共享资源`的读写访问才需要同步化，如果不是共享资源，那么就没有同步的必要；
* 在方法声明处添加synchronized并不是锁方法，而是锁当前类的对象。
* 在Java中只有“将对象作为锁”这种说法，并没有“锁方法”这种说法。
* 在Java语言中，“锁”就是“对象”，“对象”可以映射成“锁”，哪个线程拿到这把锁，哪个线程就可以执行这个对象中的synchronized同步方法。
* 如果在X对象中使用了synchronized关键字声明非静态方法，则X对象就被当成锁。
* 


## volatile
