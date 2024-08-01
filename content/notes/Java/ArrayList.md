# ArrayList 简介

* 底层是 `Object[]`；
* 支持动态扩容；
* 可以在创建 ArrayList 时在构造中传入一个容量参数，或者在创建后使用`ensureCapacity`方法预分配一个容量，这样做的好处是可以减少添加元素过程中容量不够再分配的次数。

![](继承关系.png)

## ArrayList 继承关系

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	//... 
}
```

* ArrayList 继承自 AbstractList：

  * AbstractList 主要作用是为 `List` 接口的具体实现提供一个骨架实现，简化具体 `List` 实现类的开发；
  * AbstractList 提供了 `iterator()` 和 `listIterator()` 方法的基本实现，使得子类可以直接继承这些方法，而无需重新实现。这确保了所有继承自 `AbstractList` 的类都能通过迭代器进行遍历。

* ArrayList 实现了 List 接口、RandomAccess 接口、Cloneable接口、Serializable 接口：

  * 实现 List 接口：表明 ArrayList是一个列表，支持 添加、删除、查找 等操作，并且支持下标访问；
  * 实现 RandomAccess 接口：是一个标志接口，表明 ArrayList 支持快速随机访问；
  * 实现 Cloneable 接口：表明 ArrayList 具有拷贝能力，可以进行深拷贝、浅拷贝等动作；
  * 实现 Serializable 接口：表明 ArrayList 可以进行序列化操作；

  

# ArrayList 与 LinkedList 区别

* 线程安全问题：二者都不是线程安全的容器；
* 底层数据结构：
  * ArrayList 底层使用 Object 数组来实现；
  * LinkedList 底层使用 Node 节点组成的双向链表来实现；
* 插入和删除元素时间复杂度：
  * ArrayList 在末尾添加或者删除元素时，时间复杂度为 O(1)；在数组其余指定位置插入元素时，时间复杂度为 O(n)，因为需要移动后面的元素；
  * LinkedList 在头尾插入删除元素都是 O(1)；但在指定位置插入元素时，时间复杂度是 O(n)，因为需要先遍历找到该位置；
* 是否支持快速随机访问：
  * 快速随机访问指的是 可以通过元素的序号直接或许到元素；
  * ArrayList 支持
  * LinkedList 不支持
* 内存占用：
  * ArrayList 在底层数组末尾会预留一定空间，造成一定的空间浪费；
  * LinkedList 在 Node 节点中除了存储元素外，还需要存储两个指针；



# ArrayList 核心源码

## 构造方法

```java
    /**
     * 默认初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空实例，构造传入0时使用（类对象，所有 ArrayList 实例共享，避免空间浪费）
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 默认构造时使用（类对象，所有 ArrayList 实例共享，避免空间浪费）
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 底层数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * 实际存储数组元素个数
     */
    private int size;

    /**
     * 指定容量构造
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 默认构造
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

* 以无参构造或者初始化容量为 0 的构造 创建 ArrayList 时，底层实际是一个 空数组；当向数组中进行元素添加时，第一次直接扩容为 10；
* 指定初始化容量大于 0 的构造 创建 ArrayList时，才会初始化对应容量的数组；

## 扩容机制

### add方法

```java
    /** 
     * 在末尾添加指定元素
     */
		public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```



```java
	  /**
     * 确保底层数组容量达到指定的最小容量
     */
		private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```



```java
    /**
     * 根据指定的最小容量和当前数组容量 来计算所需容量
     */
		private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```





```java
   /**
    * 判断是否需要扩容
    */
	 private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```



* 以默认构造创建 ArrayList 为例，底层是一个空数组；
* 当 add 第 1 个元素时，执行ensureCapacityInternal，此时实际元素个数为 1，但 elementData.length 为 0，因此所需最小容量为 10，因此需要进行扩容grow到 10；
* 当 add 第 2...10 个元素时，所需最小容量-底层数组大小 小于等于 0，因此不需要扩容；
* 当 add 第 11 个元素时，所需最小容量-底层数组大小 大于 0，此时需要进行扩容grow；

### grow方法

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //新容量变为旧容量的 1.5 倍左右
        int newCapacity = oldCapacity + (oldCapacity >> 1);
      	//若新容量还不够，则新容量直接变为所需最小容量（实际元素个数）
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
      	// 若新容量大于数组最大容量MAX_ARRAY_SIZE，执行 hugeCapacity
      		//若所需容量大于MAX_ARRAY_SIZE，则新容量为 Integer.MAX_VALUE
      		//否则为 MAX_ARRAY_SIZE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

* 以默认构造创建 ArrayList 为例，底层是一个空数组；
* 当 add 第 1 个元素时，执行ensureCapacityInternal，此时实际元素个数为 1，但 elementData.length 为 0，因此所需最小容量为 10，因此需要进行扩容grow到 10；
* 当 add 第 2...10 个元素时，所需最小容量-底层数组大小 小于等于 0，因此不需要扩容；
* 当 add 第 11 个元素时，所需最小容量-底层数组大小 大于 0，此时需要进行扩容grow；
* grow 流程：
  * 新容量为旧容量的 1.5 倍左右；
  * 若新容量还不够，则新容量直接等于所需容量，即数组实际元素个数；
  * 若新容量大于 最大容量（因为可能新容量是从 1.5 扩容得到的，且大于所需容量），则判断：所需容量 是否大于 最大容量
    * 若大于，则新容量为 Integer.MAX_VALUE；
    * 否则新容量为最大容量；

### 元素拷贝

在 grow 的最后，使用 `Arrays.copyOf()`，将原数组中的数据拷贝到新数组中，最后再将 add 的元素添加到新数组中；

`Arrays.copyOf()`底层调用了`System.arraycopy()`；

`System.arrayCopy()`负责具体的元素拷贝操作；

