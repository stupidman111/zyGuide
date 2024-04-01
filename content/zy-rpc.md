
## 线程池
> 在基于Socket方式的RPC中，底层使用了自己配置的线程池；

## 负载均衡实现（一致性哈希）
> 参考Dubbo的负载均衡实现，使用一致性哈希 + 虚拟节点映射的方式来实现负载均衡，具体地：
> [负载均衡实现](https://cn.dubbo.apache.org/zh-cn/blog/2019/05/01/dubbo-%e4%b8%80%e8%87%b4%e6%80%a7hash%e8%b4%9f%e8%bd%bd%e5%9d%87%e8%a1%a1%e5%ae%9e%e7%8e%b0%e5%89%96%e6%9e%90/)

* 我们使用环形队列的方式，将节点映射到哈希环上，当要取值或调用服务时，将key也映射到哈希环上，顺时针方向离该key映射位置最近的节点就是我们要的目标节点；
* 但采用这种方式的情况下，所有节点不能保证均匀低分散在环上，很有可能出现节点集中在一块的情况，因此我们采用 为每个真实节点构造多个虚拟节点的方式，将这些虚拟节点尽量均匀分散映射到环上，当我们取值或调用服务时，首先将key映射到环上的某个位置，然后取该位置顺时针方向走得到的第一个虚拟节点，然后根据映射关系，找到这个虚拟节点所对应的真实节点，在真实节点中取到数据；
* 这种 添加虚拟节点实现一致性哈希的好处有：
	* 真实节点可以不需要直接映射到哈希环上，而是为每个真实节点构造多个虚拟节点再映射到环上，可以使映射更加均匀；
	* 且添加或者删除一个节点，需要迁移的数据量更小；

> 负载均衡的步骤：在所有的Provider中选出一个，作为当前Consumer的远程调用对象。

> 一致性Hash算法的两个目的：
>  1. 映射Provider至Hash值区间中（即哈希环上，项目中实际映射的是Invoker）
>  2. 映射请求，找到大于请求的哈希值的第一个Provider；


* 选择器---`selector`：一致性Hash实现中，承载着整个映射关系的数据结构，其中定义了：
	* `private final TreeMap<Long, Invoke<T>> virtualInvokers`：一个TreeMap，key是哈希值，value是节点，用于保存哈希值到节点的映射；
	* `private final int replicaNumber`：节点数目，默认为160；
	* `private final int identityHashCode`：用来识别Invoker列表是否发生变更的Hash码；
* 我们用一个key为String，value为`Selector`的`ConcurrentHashMap`来保存服务名-->一致性哈希结构的映射（每个服务的哈希环是独立的，一个服务的所有节点只会在同一个哈希环上）
> 为每个Invoker创建replicaNumber（160）个虚拟节点，每个真实节点对应160个虚拟节点，每个虚拟节点都有一个hash值，将每个`虚拟节点hash值---Invoker`的关系存储在TreeMap中。
* 在TreeMap中根据key的hash值找到真实节点：
	* `ceilingEntry`的作用：获取大于等于传入值的首个元素。
```java
Map.Entry<Long, Invoker<T>> entry = virtualInvokers.ceilingEntry(hash);
```
