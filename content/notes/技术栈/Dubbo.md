> [Dubbo 官网](https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/brief/)

# Quick Start
> Dubbo可以向用户提供跨进程的 RPC 远程调用能力。
> 
> 通过注册中心，服务调用者可以获取服务提供者的连接方式，从而将服务请求发送给正确的服务提供者。

![[Dubbo背景.png]]



## 本地做一个Dubbo应用示例
### Step 1. 起一个 Zookeeper 服务
```shell
# 直接用 docker 起一个，端口映射到 本地 2181
docker run --name zy-zookeeper  -p 2181:2181 --restart always -d zookeeper
```

### Step 2.定义服务接口
> 服务接口是通信的关键，Provider 和 Consumer 都应该持有服务接口
> Provider 通过实现服务接口，向 Consumer 提供服务；
> Consumer 通过服务接口向 Provider 发起调用（实际的服务接口实现类在 Provider 被动态代理）
```java
public interface GreetingService {  
  
    String sayHi(String name);  
  
}
```

### Step 3.定义服务提供者 Provider
> 服务方需持有服务接口，然后定义该接口的实现类

```java
public class GreetingServiceImpl implements GreetingService {  
  
  
    @Override  
    public String sayHi(String name) {  
       return "Hi, " + name;  
    }  
}
```

> 服务方启动 Dubbo 服务。
> 这里的 registry、protocol、service 可以多次传入，意味着：
> 	1.  服务端可以将信息注册到多个注册中心上；
> 	2.  服务端支持多种协议；
> 	3.  提供多种服务；

```java
public class Application {  
    public static void main(String[] args) {  
       //定义具体的服务  
       ServiceConfig<GreetingService> service = new ServiceConfig<>();  
       service.setInterface(GreetingService.class);//设置服务接口  
       service.setRef(new GreetingServiceImpl());//设置服务实现类实例
  
		//启动 Dubbo
		DubboBootstrap.getInstance()  
	       .application("first-dubbo-provider")  
	       .registry(new RegistryConfig("zookeeper://127.0.0.1:2181")) 
	       .protocol(new ProtocolConfig("dubbo", -1)) 
	       .service(service)
	       .start()  
	       .await();
    }  
}
```
### Step 4.定义服务消费者 Consumer
> 服务消费者通过注册中心拉取服务端的地址，通过服务接口向服务端发起调用；
```java
public class Application {  
    public static void main(String[] args) throws IOException {  
       //定义要订阅的服务（传入接口信息）  
       ReferenceConfig<GreetingService> reference = new ReferenceConfig<>();  
       reference.setInterface(GreetingService.class);  
  
       DubboBootstrap.getInstance()  
             .application("first-dubbo-provider")  
             .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))//  
             .reference(reference);  
  
       GreetingService service = reference.get();//获取动态代理对象  
       String msg = service.sayHi("zy");  
       System.out.println("Receive result ===> " + msg);  
       System.in.read();  
    }  
}
```



# API方式使用 Dubbo
## 常用类
* `ApplicationConfig`：用于配置当前应用的信息。它包含应用程序的名称和其他一些应用级别的配置。
* `RegistryConfig`：用于配置注册中心的信息。注册中心用于服务的注册和发现。
* `ServiceConfig`：用于配置具体的服务。它包含服务的接口、实现类和其他一些服务级别的配置。（服务提供者 通过 ServiceConfig 暴露服务接口，发布服务接口到注册中心。）
* `ReferenceConfig`：用于配置服务引用的配置类。它主要用于消费者端，用来配置和引用远程的服务接口，从而在本地调用远程服务。（服务消费者 通过 ReferenceConfig 引用远程服务，从注册中心订阅服务接口。）
* `DubboBootstrap`：通过 DubboBootstrap API 可以减少重复配置，更好控制启动过程，支持批量发布/订阅服务接口，还可以更好支持 Dubbo3 的应用级服务发现。
* 

# XML 配置方式使用 Dubbo

# 注解方式使用 Dubbo
