## 简单介绍Netty

* Netty是一个基于NIO的client-server客户端服务器框架，能够快速简单地开发网络应用程序；
* 极大地简化并优化了TCP和UDP网络套接字编程，且性能、安全性在某些方面可能会更好；
* 支持多种协议，如FTP、SMTP、HTTP以及各种二进制和基于文本的传输协议；

* 封装NIO的很多细节，使用更简单；
* 预置了多种编解码功能，支持多种主流协议；
## Netty中的重要组件
* 首先，`ServerBootstrap`和`Bootstrap`分别对应服务端与客户端，属于启动引导类；
* 在服务端会创建两个`NioEventLoopGroup`-事件循环组，一个称之为`bossGroup`，一个称之为`workerGroup`，`bossGroup`只负责关注`Accept`连接事件，当事件发生时，就建立连接，然后将连接交给`workerGroup`来处理，`workGroup`只负责关注`Read/Write`事件，当事件发生时，就对数据进行处理；
* 每个`NioEventLoopGroup`-事件循环组中包含多个`NioEventLoop`，负责监听网络事件并调用事件处理器进行相关I/O操作的处理；
* 每个`NioEventLoop`都有一个`Channel`，是Netty的网络操作抽象类，包括基本的I/O操作，如bind、connect、read、write等；
* 每个`Channel`都有一个`ChannelPipeline`；
* 每个`ChannelPipeline`负责维护一个由多个`ChannelHandler`构成的双向链表，用户可以自定义`ChannelHandler`添加到`ChannelPipeline`中；
* `ChannelFuture`是一个异步返回结果对象，在Netty中，所有I/O操作都是异步的，我们在调用方法后立即返回一个`ChannelFuture`对象，可以在这个`ChannelFuture`对象上通过`addListener()`方法注册一个`ChannelFutureListener`监听器，当操作执行成功或者失败后，监听器会自动触发并执行相应逻辑操作返回结果。

## 为什么用RPC不用HTTP，为什么基于TCP不基于HTTP
* 在传输效率上，HTTP请求中由于有很多我们用不到的字段，开销较大，所以我们使用TCP，将RPC消息转换为自定义的传输协议后放入到TCP的数据部分，使得报文体积更小；
* 在性能消耗上，进行序列化反序列化操作时，HTTP中数据一般是JSON形式的，效率更低，而TCP是基于字节流方式的，效率比较高。

## 线程池
> 在基于Socket方式的RPC中，底层使用了自己配置的线程池；
> [美团线程池实现](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

## 负载均衡实现（一致性哈希）
> 参考Dubbo的负载均衡实现，使用一致性哈希 + 虚拟节点映射的方式来实现负载均衡，具体地：
> [Dubbo负载均衡实现](https://cn.dubbo.apache.org/zh-cn/blog/2019/05/01/dubbo-%e4%b8%80%e8%87%b4%e6%80%a7hash%e8%b4%9f%e8%bd%bd%e5%9d%87%e8%a1%a1%e5%ae%9e%e7%8e%b0%e5%89%96%e6%9e%90/)

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

## 服务发现、服务注册
* 服务发现：
	* 根据RPC请求消息，取出服务名称，在zkClient中寻找该服务下挂的所有子节点；
	* 这些子节点都是属于该服务的节点，将这些子节点组成的List以及RPC请求作为调用负载均衡的参数；
	* 负载算法选出一个节点，并将地址返回给客户端；
* 服务注册：
	* 根据服务名以及节点地址，在zkClient中注册持久性节点信息；

## 序列化
> Kryo是非线程安全的，可以考虑使用ThreadLocal来存储Kryo对象。

* 序列化：传入对象即可；序列化返回一个字节数组；
* 反序列化，传入序列化得到的字节数组、传入要将字节数组反序列化得到的类型Class；反序列化返回一个所需要类型的对象；

## 自定义通信协议
```java

```

* 4字节的魔术字段：用于筛选消息是否符合自定义协议，我的项目中保存的是`z、r、p、c`四个byte字符
* 1字节的版本号字段：我的项目中使用的是1，
* 4字节的消息长度字段：
* 1字节的消息类型字段：用于区分是请求消息、响应消息还是心跳消息（消息类型在RPC消息中保存，取出放到通信协议的头部的消息类型字段中即可）
* 1字节的压缩类型字段：表示使用的是什么压缩算法，本项目统一使用`gzip`压缩
* 1字节的序列化类型字段：表示采用的是什么序列化协议，本项目统一使用`kryo`进行序列化与反序列化；
* 4字节的RPC请求ID字段：用于判断响应和请求是否对应
* 最后是消息内容字段：
## 自定义编解码

> 自定义编码-->`RpcMessageEncoder`：根据RPC消息以及自定义通信协议格式，填写到`ByteBuf`中即可；


> 自定义解码-->`RpcMessageDecoder`：根据`ByteBuf`中的内容以及自定义通信协议，封装一个RPC消息对象即可；

## 心跳机制
> 在客户端的bootStrap中使用`.handler()`方法，向客户端Channel的pipeline中添加一个`IdleStateHandler`，并设置允许其写空闲为5s，即5s内若接收到客户端的数据，那么就触发写空闲事件，具体地，在自定义的Handler里面，重写的`userEventTriggered()`方法中，判断event是否为空闲状态`IdleStateEvent`并为写空闲`WRITE_IDLE`，若是，就发送一个包含PING的心跳RPC消息（服务端会根据接收到的消息是否是心跳消息来回应一个包含PONG的心跳RPC响应消息）


> 在服务端的bootStrap中使用`.childHanlder()`方法，向服务端Channel的pipeline中添加一个`IdleStateHandler`，并设置允许其读空闲为30s，即30s内若没有从客户端读取，那么就会触发读空闲事件，具体地，在自定义的Handler里面，重写的`userEventTriggered()`方法中，判断event是否为空闲状态`IdleStateEvent`并为读空闲`READER_IDLE`，若是，就关闭连接；


## Spring自定义注解完成服务注册


## 获取结果是同步还是异步的
>⽬前是异步的，我使⽤了CompletableFuture对象来包装RpcResponse,然后将他存⼊⼀个名为
>unprocessedRequests的map⾥，该map以请求ID作为key，对应的CompletableFuture作为value，⼀开始future⾥并没有内容，在客户端Handler的channelRead⽅法中，判断出是rpcResponse消息，才使⽤complete⽅法，将CompletableFuture的结果设置为这个Response。

> CompletableFuture类中的complete方法用于手动完成Future，并设置其结果值。调用complete方法将Future标记为已完成状态，并将结果值设置为指定的值。这个方法允许你在异步操作完成之前手动设置结果，而不必等待操作完成。