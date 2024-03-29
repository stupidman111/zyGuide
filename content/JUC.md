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

