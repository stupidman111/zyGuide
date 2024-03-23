# IoC & AOP
> IoC--控制反转，AOP--面向切面编程
## IoC
> - **使用 IoC 思想的开发方式** ：不通过 new 关键字来创建对象，而是通过 IoC 容器(Spring 框架) 来帮助我们实例化对象。我们需要哪个对象，直接从 IoC 容器里面去取即可。

* **控制**：对象的实例化、管理的权力；
* **反转**：控制权交给外部环境（IoC容器）；
* **好处**：对象之间耦合度降低，资源管理简单；
* **IoC与DI**：DI全名为**依赖注入**，IoC是一种设计思想，DI是IoC的实现方式。

## AOP
> AOP（Aspect Oriented Programming）即面向切面编程，AOP 是 OOP（面向对象编程）的一种延续，二者互补，并不对立。

* AOP 的目的是将横切关注点（如日志记录、事务管理、权限控制、接口限流、接口幂等等）从核心业务逻辑中分离出来，通过动态代理、字节码操作等技术，实现代码的复用和解耦，提高代码的可维护性和可扩展性。
* OOP 的目的是将业务逻辑按照对象的属性和行为进行封装，通过类、对象、继承、多态等概念，实现代码的模块化和层次化（也能实现代码的复用），提高代码的可读性和可维护性。

* 横切关注点：多个类或对象中的公共行为（如日志记录、事务管理、权限控制、接口限流、接口幂等等）；
* 切面：
* 连接点：
* 通知：
* 切点：
* 织入：

# Spring事务
> 程序是否支持事务首先取决于数据库，比如MySQL若使用的是InnoDB引擎，就支持事务，而MyISAM引擎就不支持事务。(MySQL的InnoDB通过`undolog`回滚日志实现回滚)

## 四大特性
* 原子性：
* 一致性：
* 隔离性：
* 持久性

## Spring支持两种方式的事务管理
### 编程式事务管理
> 通过`TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用。

```java

@Autowired
private TransactionTemplate transactionTemplate;

public void testTransaction() {
	
}
```
### 声明式事务管理
> 通过AOP实现，基于`@Transactional`的全注解方式使用最多。
```java

```

## Spring事务属性
### 事务传播行为
* `TransactionDefinition.PROPAGATION_REQUIRED`：
	* 如果外部方法没有开启事务，那么`PROPAGATION_REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰；
	* 如果外部方法开启了事务，且传播行为是`PROPAGATION_REQUIRED`的话，所有`PROPAGATION_REQUIRED`修饰的内部方法和外部方法均属于同一个事务，只要一个方法中发生了回滚，整个事务都会回滚。
* `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：
	* 创建一个新的事务，如果当前存在事务，则把当前事务挂起。即不管外部方法是否开启事务，`PROPAGATION_REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
* `TransactionDefinition.PROPAGATION_NESTED`：
	* 
* `TransactionDefinition.PROPAGATION_MANDATORY`

### 事务隔离级别

	



