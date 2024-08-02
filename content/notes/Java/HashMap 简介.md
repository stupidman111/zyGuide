* HashMap 主要用来存放键值对，它基于哈希表的 Map 接口实现，是常用的 Java 集合之一，是非线程安全的；
* 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；
* 底层数据结构：
	* JDK1.8 之前，HashMap 由 `数组+链表` 组成，数组是 HashMap 的主体，链表是为了解决 `哈希冲突` 而设计的（即拉链法）；
	* JDK1.8 开始，HashMap 由`数组+ 链表 + 红黑树` 组成，当链表长度大于默认阈值 8，且数组长度大于等于 64 时，链表会转向红黑树；

## HashMap 继承关系
```java
public class HashMap<K,V> extends AbstractMap<K,V>  
    implements Map<K,V>, Cloneable, Serializable {
	//
}
```

* 继承自 AbstractMap
* 实现了 Map 接口、Cloneable 接口、Serializable 接口

# HashMap 核心源码

## 扰动函数 -- HashMap 的 hash()方法
> 扰动函数的作用：用于计算键Key的哈希值，以帮助在哈希表中定位键值对的位置。它主要用于分布键的哈希值，以减少哈希冲突，从而提高哈希表的性能。


JDK1.7 中 HashMap 的 扰动函数--hash()：
```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

```

JDK1.8 中的 HashMap 的 扰动函数--hash()：
```java
/**
 * 先通过  key.hashCode()获取对象的原始哈希值
 * 将原始哈希值`右移` 16 位，得到高位的哈希值部分
 * 将原始哈希值 与 高位部分的哈希值(右移16位得到的) 进行按位 异或^操作
 * 
 * 目的：混合高位和低位的哈希值，以减少由于哈希玛的高位或低位分布不均匀而导致的哈希冲突。
 */
static final int hash(Object key) {  
    int h;  
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```

## 解决哈希冲突的策略
> 哈希冲突：指不同的 Key， 通过哈希函数，被散列（映射）到同一个哈希桶（数组的相同位置）的现象；

* JDK1.7 时 HashMap 解决 哈希冲突的策略：
	使用`拉链法`，即当不同的 Key 被哈希函数映射到同一个哈希桶中是，在该位置构建一个链表，存放这些 键值对；

* JDK1.8 时 HashMap 解决 哈希冲突的策略：
	当发生哈希冲突映射到同一个哈希桶的 不同 Key 不超过 8 个时，依然使用 `拉链法`解决哈希冲突；
	当大于 8 个时，且哈希桶个数（数组长度）大于或等于 64 时，才会执行转换红黑树操作，以减少搜索时间；否则（哈希桶长度不到 64），只对哈希桶进行扩容；

## HashMap 部分重要属性
```java
private static final long serialVersionUID = 362498820763181265L;

//默认的初始容量为 16，当不传入初始容量构造 HashMap时，就使用该值
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//HashMap 的最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认的负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//当一个哈希桶上的 链表长度大于等于该值时，链表可以`考虑`转换为红黑树形式
static final int TREEIFY_THRESHOLD = 8;

//当一个哈希桶上的 红黑树中节点个数小于等于该值时，红黑树可以`考虑`转换为链表
static final int UNTREEIFY_THRESHOLD = 6;

//链表有需求转换为红黑树时，要求数组容量的最小值
static final int MIN_TREEIFY_CAPACITY = 64;

//纯属元素的数组，总是为 2 的幂次倍
transient Node<K,V>[] table;

//包含了所有键值对的集合视图
transient Set<Map.Entry<K,V>> entrySet;

//存放键值对的个数
transient int size;

//每次扩容和更改 map 结构的计数器
transient int modCount;

//阈值，等于 容量 * 负载因子，当实际大小超过该值是，会进行扩容
int threshold;

//负载因子
final float loadFactor;

```

* `loadFactor`：负载因子，控制数组存放数据的疏密程度；
	* 越接近 1，数组中存放的数据就越多，也就越密；
	* 越接近 0，数组中存放的数据越少，
## HashMap 构造函数
```java
//传入初始容量、负载因子
public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}  

//传入初始容量
public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}  

//默认构造：负载因子为属性中默认的负载因子
public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted  
}  

//通过集合构造 HashMap
public HashMap(Map<? extends K, ? extends V> m) {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
    putMapEntries(m, false);  
}
```