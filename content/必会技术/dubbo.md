# ...


# 配置
```yml
dubbo:  
  application:  
    name: Lottery  
    version: 1.0.0  
  registry:  
    address: N/A # N/A表示 Dubbo 自动分配地址，直连方式配置，不通过注册中心，可以使用zk配置
  protocol:  
    name: dubbo  
    port: 20880 #Dubbo协议默认端口为20880
```

# 常见错误
* Dubbo 接口无论是入参、出参，以及参数中的包括的对象，都必须 implements Serializable；
* Dubbo接口实现使用`@org.apache.dubbo.config.annotation.Service` 不要用 Spring 的 Service；