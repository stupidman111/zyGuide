# Java基础
## equals() 和 hashCode() 
* `==` 用于基本类型变量值的相等判断；
* `equals()` 用于对象相等的判断：
	* Object类的equals方法内部是判断两个对象引用是否相等（地址是否相等）；
	* 一般需要判断的是两个引用类型变量的内部属性是否相等，所以需要重写equals；
* 重写equals：
	* 判断引用是否相等；
	* 判断对象类型是否相等；
	* 判断对象内部属性是否相等；
* `hashCode()`用于获取对象的哈希码：
	* Object类的hashCode方法是native的，底层由C/Cpp实现；**原生的hashCode()方法为不同的对象返回不同的整型值** ，这个整形值可能与对象的内存地址有关，但具体实现要看JVM版本；
	* 对象用于`散列表` 时，需要重写hashCode方法；
* 重写hashCode：
	* 可以使用`Objects.hash()` 方法等等；
	* 对对象的属性值按某个哈希函数进行哈希运算；
* 为什么重写equals，一定要重写hashCode？
	* 原因：出于效率
	* 在HashSet、HashMap、HashTable等集合中元素是无序的且不重复的，若单靠equals方法比较，时间复杂度是O(n)，但有了hashCode方法后，我们可以获取该对象的hash值用O(1)的时间复杂度来定位；若该位置上已有对象，使用equals方法进行判断对象内部属性是否相等。
	* 重写equals是为了判断两个对象内部属性是否相等，重写hashCode是为了保证equals相等的两个对象，一定能在散列表中落到同一个`桶` 中。
* 总结：
	* 两个对象的hashCode相等，那么这两个对象不一定相等（哈希碰撞）；
	* 两个对象的hashCode相等，且两个对象的equals返回true，那么认为这两个对象相等；
	* 两个对象的hashCode不相等，那么可以直接认为这两个对象不相等；

## String、StringBuilder、StringBuffer


## 常量折叠


## 包装类型
>Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。

`Byte、Short、Integer、Long`四种包装类型默认创建了`[-128, 127]` 的相应类型的缓存数据；
`Character` 创建了数值在`[0, 127]` 范围的缓存数据；
`Boolean` 直接返回`true` Or`false`；
* 自动拆箱与自动装箱
	* 装箱：将基本数据类型变量用它们对应的引用类型包装起来;
	* 拆箱：将包装类型变量转换为对应的基本数据类型变量；
```java
class Main {
	public static void main(String[] args) {
		Integer i = 10;//自动装箱：Integer i = new Integer(10);
		int j = i;//自动拆箱：
	}
}
```

## BigDecimal


# Java集合
![](./img/Java容器继承图.png)

## Collection接口
> 三大子接口：Set、List、Queue


## Map接口

* `Collection<V> values();`：返回map的所有value的集合，可以作为ArrayList的构造参数。


## ArrayList
```java
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
{
	//...
}
```
* 是否线程安全：否，方法都没有使用`synchronized`关键字；
* 底层数据结构：底层是Object数组，支持动态扩容；
* 插入和删除是操作：
	* 直接插入：默认插入到数组尾部，时间复杂度为O(1)；
	* 指定位置插入：需要移动大量元素，时间复杂度为O(n)；
	* 指定元素移除/指定位置移除：时间复杂度O(n)；
* 快速访问：实现了`RandomAccess`接口，支持快速访问，通过索引直接访问底层数组；
* 内存空间占用：数组尾部会预留一定空间；
### 扩容机制
* 未指定容量初始化的空list，底层数组容量为0；
* 当加入第1个元素后，底层数组容量变为10；
* 当加入第11个元素后，底层数组触发`grow`扩容，扩容到原来的`1.5`倍；
### System.arraycopy() 和 Arrays.copyOf()
* ArrayList中的add方法用到了System.arraycopy()：当在指定位置添加元素时，需要将原有的一部分数据向后拷贝一位；
* ArrayList中的toArray()方法用到了Arrays.copyOf()：从原来的底层数组中，复制出一个新的数组并返回；

## LinkedList
```java
public class LinkedList<E> extends AbstractSequentialList<E>
		implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
	//...
}
```
* 是否线程安全：否
* 插入和删除操作：底层使用双向链表结构（非循环）
	* 头部插入、删除：O(1)；
	* 尾部插入、删除：O(1)；
	* 指定位置插入、删除：O(n)；
* 快速访问：LinkedList底层是链表，地址不连续，不支持快速访问，因此不能实现`RandomAccess`接口；
* 内存空间占用：链表的每个节点都拥有两个指针
## 删除节点
> 删除节点的核心方法是`unlink(Node<E> x)`方法
* 首先获取待删除节点x的前驱pre和后继节点next；
* 判断待删除节点x是否为头结点或尾节点：
	* 若x是头节点，则将虚拟头结点的后继指针指向x的后继节点；
	* 若x是尾节点，则将虚拟尾节点的前驱指针指向x的前驱节点；
	* 若x不是头节点，也不是尾节点，执行下一步操作；
		* 将待删除节点x的前驱节点的后继指针指向x的后继节点pre，将x的前驱指针置为null；
		* 将待删除节点x的后继节点的前驱指针指向x的前驱节点next，将x的后继指针置为null；
* 将待删除节点x置为nul，修改链表长度；
# Java IO

# Java反射、动态代理


# JVM


# JUC
