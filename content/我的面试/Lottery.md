# 项目相关


## 分支顺序
```text
lottery-init-project
lottery-build-rpc-framework
lottery-design-tables
lottery-coding-strategy
lottery-config-activity
```

## domain层各领域
```text
strategy---抽奖策略，根据具体抽奖请求进行抽奖，得到一个奖品信息
award---奖品方法，根据奖品信息，执行奖品发放（包括落库等等）
activity---活动配置，包括活动创建、活动领取、活动的状态变更
support--提供ID生成，包括雪花算法、随机数、日期拼接三种ID生成方式
```
## 库表设计
- 活动配置，activity：提供活动的基本配置
- 策略配置，strategy：用于配置抽奖策略，概率、玩法、库存、奖品
- 策略明细，strategy_detail：抽奖策略的具体明细配置
- 奖品配置，award：用于配置具体可以得到的奖品
- 用户参与活动记录表，user_take_activity：每个用户参与活动都会记录下他的参与信息，时间、次数
- 用户活动参与次数表，user_take_activity_count：用于记录当前参与了多少次
- 用户策略计算结果表，user_strategy_export_001~004：最终策略结果的一个记录，也就是奖品中奖信息的内容

> 抽奖系统包含活动玩法、积分消耗、奖品方法等操作，因此要拆分出：活动表、策略表、策略详情表、奖品表、规则树表、用户参与活动记录表、用户抽奖计算结果表、等等；

> 策略表strategy不依赖于活动表activity，即策略表中不存在跟活动相关的字段，但活动表中存在策略id字段，实现活动的策略可配置。





## 策略模式
```
策略模式（Strategy Pattern）和工厂模式（Factory Pattern）是两种常见的设计模式，它们的主要区别在于解决的问题和实现方式：

1. **解决的问题**：
   - 策略模式主要解决的是在运行时根据不同的情况选择不同的算法或策略。它将算法封装成独立的对象，使得算法可以相互替换，而且可以独立于客户端而变化。
   - 工厂模式主要解决的是创建对象的问题。它提供了一个创建对象的接口，但将具体创建对象的逻辑延迟到子类中去实现，从而使客户端代码与具体创建的对象类型解耦。

2. **实现方式**：
   - 策略模式通常包含一个策略接口和多个具体策略类，客户端根据具体情况选择不同的策略类进行使用。
   - 工厂模式通常包含一个工厂接口和多个具体工厂类，每个具体工厂类负责创建一类对象，客户端通过工厂接口获取具体的工厂类，并使用工厂类创建对象。

3. **用途**：
   - 策略模式适用于需要在运行时根据不同情况切换算法或策略的场景，例如排序算法、支付方式等。
   - 工厂模式适用于需要根据不同情况创建不同类型对象的场景，例如数据库连接、日志记录器等。

总的来说，策略模式强调的是算法的选择和切换，而工厂模式强调的是对象的创建。在实际应用中，它们常常结合使用，工厂模式用于创建具体的策略对象。
```

* `0x61c88647`：黄金分割点`0.618` * 2^32 的结果，使用黄金分割点作为hash函数取模

## 工厂模式

## 模版模式

## 状态模式


## ID算法


## 分库分表
* 分库分表
* 分库不分表

* 水平拆分
* 垂直拆分

## Redis滑块锁
我这里使用的Redis滑块锁主要是针对于某个活动的活动库存进行加锁的，如果是独占锁的话那么就是对活动ID进行加锁，使用独占锁会影响系统的性能，而对于滑块锁，我们这里对总的活动库存预热到Redis中，然后后续的所有扣减库存操作都在一个新的从0开始的Redis_key上进行，当用户进行扣减库存时，实际上是对这个新的Redis_key进行incr，并返回当前该Redis_key的值，如果Redis_key的值大于活动库存，说明超卖了则需要恢复并返回扣减失败的信息。扣减成功的话还需要对当前活动号加上库存量为key使用setNx加锁，表示当前活动的当前库存已经被占用了，并发情况下对同一个库存的扣减操作将会setNx失败。

## 分布式事务解决
对于跨库的事务处理：
* 分布式事务
* MQ + 任务调度补偿保证最终一致性：redis扣减库存，写入记录，发送MQ消息，接收MQ消息更新数据库，并使用XXL-JOB设置定时任务扫描库表对未发送MQ消息的、MQ消息发送失败的进行补偿处理；

## 幂等性
> 幂等性：用户对于某一操作发起多次重复的请求，但结果应当是和只发一次请求是一样的，即不会因处理多次请求而导致异常。

* 本项目中用户虽然只发起一次抽奖请求，但可能会由于MQ重发消息，导致给用户多次奖品（如上条消息实际上被消费了，但由于某些原因，导致了重发，那么会导致实际消费了多次）
* 解决方案：
	* 使用Redis对MQ消息做消费记录，使用业务唯一ID作为key，那么消费前如果缓存没有记录，可以进行消费，消费过后进行缓存记录；
	* 数据库设置唯一索引进行防重兜底；
* 我的项目中使用是在数据库中添加一个字段`使用全局唯一ID + 唯一索引`的方式进行幂等性校验的，当重复的消息被消费时，若记录已经存在了那么就会插入失败；
* 另外还可以在Redis上做一层消费记录，消费时使用setNx Redis进行记录，数据库使用唯一索引进行兜底；

## DDD  VS MVC
* MVC：MVC分层结构是一种贫血模型设计，它将“状态”和“行为“分离到不同的包结构中进行开发使用，MVC中所有的业务逻辑都在service层，当要处理的业务多起来时，代码结构就会变得混乱，
* DDD：DDD架构分层中，domain模块是最重要的一个，其他的模块都需要围绕着domain模块转。domain模块下的每个子领域模块，都包含一组完整的`model-模型对象`、`service-服务对象`、`repository-仓储服务`，