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


# Java IO

# Java反射、动态代理


# JVM


# JUC
