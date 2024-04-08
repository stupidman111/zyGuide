
# 第一章 并发编程的挑战
* 上下文切换
	* 任务从保存到再加载的过程就是一次上下文切换；
	* 减少上下文切换的方法：
		* 无锁并发编程：可以将数据的ID按照哈希算法进行取模分段，不同的线程处理不同段的数据；
		* CAS算法：Java的Atomic包使用CAS算法来更新数据，不需要加锁；
		* 使用最少线程：避免创建不必要的线程；
		* 使用协程：在单线程中实现多任务的调度，并维持多任务的切换。
* 死锁
	* 多个线程互相等待对方释放锁；
	* 避免死锁：
		* 避免一个线程同时获取多个锁；
		* 避免一个线程在锁内同时占有多个资源；
		* 使用定时锁，如`Lock.tryLock(timeOut)`方法；
		* 对于数据库锁，加锁、解锁必须在一个数据库连接内，否则会出现解锁失败的情况；
* 资源限制
	* 并发编程时，程序的执行受硬件资源或软件资源的限制；
	* 解决办法：
		* 对于硬件资源：使用集群；
		* 对于软件资源：将资源`池化`，使其能复用；
# 第二章 Java并发机制的底层实现原理
> volatile与synchronized

## volatile
* `volatile`：
	* 轻量级的`synchronized`；
	* 保证`共享变量的可见性`，可见性：当一个线程修改一个共享变量时，其他线程能够读到这个修改的值；
	* 原理：volatile修饰的共享变量进行写操作时，汇编代码会多出一条含有`lock前缀`指令的代码，`lock前缀`指向在多核处理器下会发生：
		* 将当前处理器核心的Cache行数据写回到内存中；
		* 通知其他核心，若Cache中含有对应内存地址的数据则失效；
	* ...

## synchronized
* `synchronized`：
	* synchronized实现同步的基础是Java中每一个对象都可以作为锁，具体表现为：
		* synchronized作用于普通同步方法，锁的是当前实例对象；
		* synchronized作用于静态同步方法，锁的是当前类的Class对象；
		* synchronized作用于同步代码块，锁的是括号内配置的对象；
	* synchronized的实现原理是JVM基于进入、退出`Monitor`对象来实现方法同步和代码块同步：
		* 代码块同步使用`monitorenter`、`monitorexit`指令来实现，执行`moniterenter`即尝试获取对象对应的`Monitor`的所有权，即尝试获取对象的锁；
		* 方法同步使用
	* Java对象头：
		* MarkWord字段默认存储：对象的hash码（25bit），分代年龄（4bit），锁标志位（1bit偏向锁标识，2bit锁标志位）；
		* 锁的四种状态：无锁、偏向锁、轻量级锁、重量级锁；
		* 锁只能升级不能降级；
		* 锁升级流程：
			* 没有线程进入同步块，锁对象的MarkWord字段内容为对象的HashCode、分代GC年龄、是否偏向锁标志、锁标志（25+4+1+2），锁标志为01，处于`无锁状态`；
			* 当有第一个线程进程同步块时，JVM发现锁对象的MarkWord中锁标志为01，且无偏向，则立马将锁对象的偏向线程ID设置为当前线程ID，并将是否偏向锁标志设置为1，MarkWord此时内容为偏向线程ID+epoch+分代GC年龄+是否偏向锁标志+锁标志（23+2+4+1+2），此时处于`偏向锁状态`；当该线程再次进入同步块时，判断偏向线程ID是自己，就会直接执行代码，这样就实现了就如同没加synchronized关键字一样，这种方式也实现了锁重入；
			* 当另一个线程进入同步块时，会CAS自旋等待很短的时间，如果此时偏向线程正好退出同步方法了，那么CAS就立即结束，同时锁升级为`轻量级锁`（轻量级锁是线程交替执行同步块）；如果CAS失败，那么锁就会晋升为`重量级锁`（重量级锁是多个线程同时争夺执行同步块），重量级锁由操作系统Mutex Lock来实现，需要进行用户态、内核态的切换，成本很高；

| 锁    | 优点                                   | 缺点                       | 适用场景                 |
| ---- | ------------------------------------ | ------------------------ | -------------------- |
| 偏向锁  | 加锁、解锁不需要额外的消耗；<br>相比执行非同步方法只存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗； | 适用于只有一个线程访问同步块的场景；   |
| 轻量级锁 | 线程的竞争不会阻塞，提高了程序的响应速度                 | 线程需要自旋等待锁，而自旋会消耗CPU      | 追求响应时间，同步块执行时间非常快的场景 |
| 重量级锁 | 线程竞争不使用自旋，不消耗CPU                     | 获取不到锁的线程将会阻塞，响应速度慢；      | 追求吞吐量，同步块执行时间较长的场景   |
## 原子操作实现原理
> 原子操作：不可被中断的一个或一系列操作。

* `CAS`：CAS操作需要输入两个数值，一个旧值（期望操作前的值）和一个新值，在操作期间先比较旧值有没有发送变化，如果没有发生变化，才替换为新值，发送了变化则不替换；
* Java实现原子操作：
	* 使用循环CAS实现原子操作：
		* Java中CAS操作使用处理器提供的`CMPXCHG`指令来实现；
		* 自旋CAS操作即循环进行CAS操作直到成功为止；
		* java.util.concurrent包中提供了一系列类来支持原子操作，如`AtomicInteger`、`AtomicLong`、`AtomicBoolean`、`AtomicReference`等，这些类都包含`compareAndSet`方法；
		* `CAS`实现原子操作的三大问题：
			* `ABA`问题：
				* 即CAS在操作值时，需要检查值有没有发生变化，若值从原来的A变成了B，最后在检查前又变成了A，这样CAS会判断该值没有发生变化，但实际上这个值发生了变化；
				* 解决：追加一个版本号，每修改一次值都将版本号+1，这样在检查时除了要判断值是否变化外还需要判断版本号是否变化；
				* Java中提供了`AtomicStampedReference`类以解决ABA问题：通过引用来比较值，并设置时间戳作为版本号的实现；
			* 循环时间长开销大：
			* 只能保证一个共享变量的原子操作：解决办法是可以将多个变量组合成一个对象，使用`AtomicReference`来保证引用对象的原子性；
	* 使用锁机制实现原子操作：
		* 锁机制保证只有获得了锁的线程才能操作指定的内存区域。
		* JVM中除了偏向锁，其他锁的实现方式都使用了循环CAS（循环CAS获取锁，循环CAS释放锁）
```
java
`AtomicStampedReference类的CAS方法`
/**
Params:
expectedReference – the expected value of the reference 
newReference – the new value for the reference 
expectedStamp – the expected value of the stamp 
newStamp – the new value for the stamp
*/
public boolean compareAndSet(V   expectedReference,  
                             V   newReference,  
                             int expectedStamp,  
                             int newStamp) {  
    Pair<V> current = pair;  
    return  
        expectedReference == current.reference &&  
        expectedStamp == current.stamp &&  
        ((newReference == current.reference &&  
          newStamp == current.stamp) ||  
         casPair(current, Pair.of(newReference, newStamp)));  
}
```

