# 数据类型
## String
* 底层数据结构：
	* int：
	* SDS-简单动态字符串：
		* 使用`len` 属性记录字符串长度
		* 拼接字符串不会造成缓冲区溢出（动态扩容）
* 常用指令：
* 应用场景：
	* 缓存对象：
		* 直接缓存整个对象的JSON格式
		* 将key各属性分离，使用`MSET`、`MGET`
	* 常规计数：
		* 得益于Redis执行命令是单线程的，因此String适合于做计数场景：访问次数、点赞、转发、库存数量等；
	* 分布式锁：
		* `SET lock_key_name global_unique_value NX PX 10000` 
		* NX保证锁的唯一性
		* lock_key_name是锁名称
		* global_unique_value是当前获取锁的线程的全局唯一标识（释放锁时检查）
		* PX保证锁不会因为线程崩溃而永久不释放（一般为10s）
		* 释放锁：
			* `检查锁标识` 和 `释放锁`两个操作必须保证原子性（Lua脚本）
	* 共享Session信息：
		* 分布式系统会将请求随机分配到不同的服务器，此时不适合在服务器端做Session
		* 将Session信息保存为Redis String，保证多个服务器能够共享Session信息
## List
* 底层数据结构：
	* quicklist
* 常用指令：
* 应用场景--消息队列：
	* 消息队列三大基本需求：
	  1. 消息保序：`LPUSH + RPOP` / `RPUSH + LPOP`
	  2. 处理重复的消息：生产者自行生成全局唯一ID
	  3. 保证消息可靠性：`BRPOPLPUSH`，从队列取完之后加入到另一个备份队列，避免消费者处理消息过程中出现宕机导致消息丢失
	* 另外，还可以使用`BRPOP / BLPOP`进行阻塞读取，节省CPU开销
* List作为消息队列的缺点：
	* List类型不支持多消费组（但可以自行实现）
## Hash
* 底层数据结构：
	* listpack
* 常用指令：
* 应用场景：
	* 缓存对象：
		* Hash类型（field，value）可以对应一个对象的（属性名，属性值）
		* 一般对象用String存储，对象中频繁变化的属性可以用Hash存储
	* 购物车：
		* 以用户id为key，商品id为field，商品数量为value可以构成购物车三要素
		* 增加商品数量：`HINCRBY`（注意：Hash没有`HDECRBY`指令）
		* 商品总数：`HLEN` 
		* 删除商品：`HDEL`
		* 获取购物车所有商品：`HGETALL` 
## Set
> Set数据结构无重复元素且无序
* 底层数据结构：
	* 元素都是整数且元素个数小于 `512`：整数集合
	* 不满足上述条件：哈希表
* 常用指令：
* 应用场景：
	* 点赞：
		* 保证一个用户只能点赞一次
		* key是文章id，value是用户id
		* 点赞时，查看文章id对应的Set中是否存在当前用户id--`SISMENBER` 
		* 取消点赞时，将当前用户id从文章id对应的Set中移除--`SREM` 
	* 共同关注、可能认识的人
		* 可用于计算共同关注的好友、公众号等
		* 每个用户对应一个set，用户id时key，所有好友id为value
		* 计算两个用户的共同好友--`SUNION`
		* 可能认识的人--`DIFF`（差集）
	* 抽奖活动：
		* key为抽奖活动名，value为所有抽奖参与者id
		* 允许用户重复中奖：`SRANDMEMBER` 
		* 不允许用户重复中奖：`SPOP` 
## ZSet
> ZSet中元素（member）不可重复，但元素的分数(score）可以重复，且ZSet可以排序
* 底层数据结构：
	* listpack
* 常用指令：
* 应用场景：
	* 排行榜：
		* member为文章id，score为文章点赞数
		* 新增一个赞--`ZINCRBY
		* 查看某文章点赞数--`ZSCORE` 
		* 获取点赞数最多的n篇文章--`ZREVRANGE`
	* 电话、姓名排序
		*  ...
## BitMap
> 位图，一串连续的二进制数组（0、1），通过offset定位元素。`0/1`分别表示一种状态。
* 底层数据结构：
	* String
* 常用命令：
* 应用场景：
	* 一个用户的全年/全月签到统计：全年-365bit，全月-31bit
	* 判断用户登录状态：1-在线，0-离线
	* 连续签到用户总数：每天签到的用户记录在一个BitMap里，将多天的BitMap进行`AND`操作
## HyperLogLog

## GEO

## Stream


# Redis 线程模型
> Redis 基于 Reactor 模型开发了一套高效的事件处理模型，对应Redis中的文件事件处理器。

* 单线程为什么快？
	* Redis通过`epoll I/O多路复用`机制监听大量socket，将感兴趣的事件如读、写等注册到内核并监听每个事件是否发生，因此不需要额外的线程来监听大量的客户端连接，降低资源消耗；
	* Redis是基于内存的数据库，大部分操作都在内存中；
	* Redis的单线程避免了多线程之间的竞争，省去了线程切换带来性能消耗；
* Redis为什么不采用多线程？
	* 通常说的Redis是单线程，只是Redis执行命令是单线程的；
	* Redis也会开启三个后台线程来处理对应的任务：
		* 关闭文件任务
		* AOF刷盘任务
		* 释放内存任务
	* 在6.0版本后Redis还引入了多个线程用于处理网络I/O，但执行命令仍然是单线程的
* Redis线程模型
	![](./img/redis单线程模型.png)
	1. 调用`epoll_create()` 创建epoll对象，调用`socket()` 创建服务端socket；
	2. 调用`bind()、listen()` 绑定地址、开启监听；
	3. 调用`epoll_ctl()`将`listen sockert` 加入到epoll，并注册 连接事件处理函数；
	4. 首先，调用**发送队列处理函数** 查看发送队列是否有任务，若有则调用`write()`函数处理，没处理完则注册写事件函数，等待epoll_wait发现可写后再处理；
	5. 进入事件循环函数，调用`epoll_wait()` 等待事件到来：
		1. 如果是`连接事件`，调用连接事件处理函数，该函数会调用`accept()`执行连接并获取该连接的socket，调用`epoll_ctl()`将该socket加入到epoll，并注册 读事件处理函数；
		2. 如果是`读事件`，调用读时间处理函数，该函数调用`read()`获取客户端发送的数据，解析命令、执行命令、将客户端对象添加到发送队列，将执行结果写到发送缓冲区等待发送；
		3. 如果是`写事件`，调用写事件处理函数，该函数调用`write()`将客
		4. 户端发送缓冲区里的数据发生出去，如果这一轮没有发送玩，就会继续注册写事件处理函数，等待`epoll_wait`发现可写后再处理；
# Redis持久化
## AOF日志
> Redis每执行一次写命令，都会将该写命令追加到磁盘中的`AOF`日志文件当中。

* AOF日志的实现：
	1. 命令追加：主线程每执行完一条写命令，都会将该命令追加至`aof缓冲区--server.aof_buf（属于buffer I/O）`；
	2. 文件写入：主线程调用`write()`系统调用函数，将`aof缓冲区`里的内容写入到内核缓冲区`page cache` 中后立即返回；
	3. 文件同步：`aof刷盘后台线程`根据对应的AOF写回策略，向磁盘做同步操作，需要用到`fsync`系统调用函数，该函数针对单个文件操作，对其进行强制硬盘同步，将内核缓冲区中的数据写入到磁盘对应的文件位置，且会阻塞直至写完才返回，保证了数据的持久化；
	4. 文件重写：随着AOF文件越来越大，当达到了所设置的阈值，就会触发`AOF重写机制`，以达到压缩的目的；
	5. 当Redis重启时，可以加载执行AOF文件中的命令以恢复数据。
* AOF写回策略：
	* 当主线程调用完`write()`将命令写入到内核缓冲区后，后台刷盘线程根据AOF写回策略来进行强制同步AOF文件（`fsync()`函数）
	* `appendfsync Always`：主线程每次write，后台线程立即fsync--会降低Redis的性能；
	* `appendfsync EverySec`：后台线程每秒fsync一次；
	* `appendfsync No`：不使用后台线程进行强制刷盘，具体什么时候同步由内核决定，Linux下一般为30s一次。
* AOF重写机制：
	* 当Redis AOF File 文件大小达到所设置的阈值时，Redis会调用`fork()` ，开启一个`子进程`进行AOF重写；
	* 具体地，子进程与主进程会以只读方式共享分页，子进程可以根据Redis当前数据重写构造命令写入到新的AOF文件；
	* 同时Redis主进程不会阻塞，当主进程继续执行写命令时，会触发`写时复制`，于是父子进程就有了独立的数据副本，子进程可以根据副本写新的AOF文件，主进程执行写命令会将命令同时追加到`aof缓冲区`和`aof重写缓冲区`当中；
	* 当子进程重写完毕就发送一个信号给主进程，主进程将`aof重写缓冲区`里的内容写入到新的`aof`文件中，此时新旧两个aof文件的最终结果是一样的，用新的替换旧的，达到压缩文件大小的目的。
## RDB快照
> RDB快照是将某一时刻的Redis数据写入到RDB文件当中，主`进`程阻塞与不阻塞两种实现方法。

* 执行`save`命令，由主进程生成RDB文件，主线程会阻塞；
* 执行`bgsave`命令，会调用`fork()`创建子进程触发`写实复制`由子进程生成RDB文件，主进程不会阻塞
> Redis做RDB快照时，会阻塞主线/进程吗？-
> Redis做RDB快照时，数据能修改吗？-
## 混合持久化
> AOF日志恢复数据太慢，RDB的创建频率不好把握，因此采用混合持久化（结合）
* 当触发AOF日志重写时，`fork()`出来的子进程根据数据副本以生成RDB快照的方式写入新的AOF文件，而主进程此时将写命令写入aof缓冲区和aof重写缓冲区；子进程执行完毕后发信号主进程，主进程将aof重写缓冲区中的内容写入到新的AOF文件，此时AOF文件一部分是RDB快照，一部分是AOF日志格式。
* 好处：
	* 文件以RDB开头，Redis启动更快；
	* AOF降低了数据丢失的风险；
* 缺点：
	* AOF文件可读性变差；
	* 兼容性差；
# Redis过期删除与内存淘汰
## Redis过期删除
> Redis选择【惰性删除】 +【定期删除】两种策略搭配使用。

* 过期字典的概念：Redis中所有设置了过期时间的`key`，都会保存到过期字典当中，value是对应key的过期时间。当查询一个key时，要先查该key是否在过期字典中，不在则正常读取，在则要先判断过期时间是否大于当前时间，若不大于则过期，若大于则正常读取。
* 惰性删除策略：每次访问一个key时，在过期字典中查询到该Key已经过期时才执行删除--CPU友好，内存不友好；
* 定期删除策略：每隔一段时间从库中取出一定量的key，若过期key数量达到某个占比，则循环进行抽取，并删除过期key--难以控制频率；
## Redis持久化时对过期键的处理
> Redis持久化分为【AOF日志】和【RDB快照】两种方式，所以分别从这两个方向去讨论。

* AOF日志持久化时对过期键的处理：
	* 在AOF日志生成阶段：Redis每发现一个过期的key，都向AOF文件中追加一条DEL命令来删除对应的key；
	* 在AOF日志重写阶段：因为是根据数据副本生成AOF日志，所以已经过期的key不会写入到新的AOF文件中；
* RDB快照持久化时对过期键的处理：
	* RDB快照生成阶段：因为是根据数据复制生成RDB快照，所以已过期的key不会写入到新的AOF文件中；
	* RDB加载阶段：根据当前是主/从服务器会有不同：
		* 当前是【主服务器】：加载RDB快照时，会对key做检查，保证过期的key不会加载到内存；
		* 当前是【从服务器】：加载RDB快照时，不做检查，在主从数据同步时，从服务器的键会被清空；
## Redis主从模式中对过期键的处理
从库不进行过期键扫描，即使从库中键过期了，有客户端访问从库时依然可以访问到；
主库在key到期时，向从库同步DEL命令，从库收到后删除过期key。

## Redis内存淘汰策略
> 有八种，可分为【不进行数据淘汰】和【进行数据淘汰】两大类。

* 不进行数据淘汰的策略：`noeviction`，Redis**默认**的淘汰策略，当运行内存超过阈值时，不淘汰任何数据，不再提供服务，直接返回错误；
* 进行数据淘汰的策略：有可分为【在设置了过期时间的数据中进行淘汰】和【在所有数据范围内进行淘汰】：
	* ...

## Redis LRU（最近最少使用）
> Redis没有采用传统的「基于链表的」LRU，而是实现了一套「近似」LRU：
* 在Redis的对象结构体中添加一个额外的字段，用于记录此数据的最后一次访问时间---节省空间，提高性能；
* 当Redis进行内存淘汰时，会使用随机采用抽取多个key，根据所添加的字段，淘汰最久没有被使用的key；
> 无法解决**缓存污染**问题，比如一次读入了大量数据，这些数据大部分只会被访问一次，但会将一些经常访问的数据淘汰掉。

## Redis LFU（最近最不常用）
> 为了避免**缓存污染**问题，Redis引入 LFU算法，根据数据访问次数淘汰数据，核心思想：**如果数据过去被访问多次，那么将来被访问的频率也更高**
* 在Redis的对象结构体中，添加一个额外的24bit的字段，高16bit表示时间戳，低8bit记录key的访问频次。


# Redis缓存设计
## 缓存雪崩
## 缓存击穿
## 缓存穿透
## 缓存更新策略
* 旁路缓存策略：
* 读穿/写穿策略：
* 写回策略：
## 缓存-数据库一致性

# Redis事务

# Redis应用
## 全局唯一ID
## 分布式锁
## 搜索引擎

# Redis性能优化
## 批量操作减少网络传输
## 大Key
## 热Key
## 慢查询命令
## 内存碎片
