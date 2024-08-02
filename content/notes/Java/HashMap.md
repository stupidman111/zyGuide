
# HashMap 简介
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
	* 越接近 1，数组中存放的数据就越多，也就越密，查找效率越低；
	* 越接近 0，数组中存放的数据越少，越稀疏，数组利用率越低；
	* 默认的 HashMap 容量为 16，默认的 loadFactor 为 0.75，也就是当HashMap 中键值对数量超过了 阈值=`16*0.75` = 12 时，就需要将数组进行扩容，并且需要复制数据； 

* **hreshold = capacity * loadFactor**，**当 Size>threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准**。
## HashMap 构造函数
> 需要注意，当我们指定初始容量构造 HashMap 时，实际上 HashMap 不一定使用该值进行数组的初始化，需要对该 值扩大到最接近 2 的幂次方的大小。
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

## Node节点--链表节点


## TreeNode节点--红黑树节点


## put方法
> put 方法实际调用 putVal 方法。

```java

//调用 putVal 方法前，通过 扰动函数--hash() 计算 key 映射的位置；
public V put(K key, V value) {  
    return putVal(hash(key), key, value, false, true);  
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
               boolean evict) {  
    Node<K,V>[] tab; Node<K,V> p; int n, i;  
    if ((tab = table) == null || (n = tab.length) == 0)  
        n = (tab = resize()).length;  
    if ((p = tab[i = (n - 1) & hash]) == null)  
        tab[i] = newNode(hash, key, value, null);  
    else {  
        Node<K,V> e; K k;  
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            e = p;  
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        else {  
            for (int binCount = 0; ; ++binCount) {  
                if ((e = p.next) == null) {  
                    p.next = newNode(hash, key, value, null);  
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        treeifyBin(tab, hash);  
                    break;                }  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    break;  
                p = e;  
            }  
        }  
        if (e != null) { // existing mapping for key  
            V oldValue = e.value;  
            if (!onlyIfAbsent || oldValue == null)  
                e.value = value;  
            afterNodeAccess(e);  
            return oldValue;  
        }  
    }  
    ++modCount;  
    if (++size > threshold)  
        resize();  
    afterNodeInsertion(evict);  
    return null;}
```

* 调用 putVal 方法前，通过 扰动函数--hash() 计算 key 映射的位置；
* 如果定位到的位置没有元素，就直接插入；
* 如果定位到的数据位置有元素，将插入元素的 key 与该位置的 key 进行`.equals()`比较
	* 如果为 true，则覆盖该元素；
	* 如果为 false，则判断该位置是否为树节点(`instanceof TreeNode`)
		* 如果是，直接调用方法将元素插入到树中；
		* 如果不是，遍历链表，依次比较 key.equals()，如果不存在 true，则最终插入链表末尾；
> 注意：在 JDK1.7 中，使用头插法插入到 链表头部；

## get方法

```java

```

## resize方法 和 重新哈希
> 调用 resize 扩容时，需要复制元素，HashMap 对所有 key 进行一次重新 hash 分配；

* resize：判断是否超过HashMap 容量最大值：是：不再扩容；否：2 倍扩容（在构造时保证了初始容量是 2 的幂次方）
* 重新哈希：将每个 Key 进行重新哈希定位，具体操作：`key.hash & (newCap-1)`：
	即：key 经过扰动函数得到的 hash 值，与上 新容量-1：
	因为新容量是 2 的幂次方，-1，可以将低位全部置为1，高位全部置为 0，
	这样新位置要么是原来的位置，要么是`原来的位置+原数组的长度`