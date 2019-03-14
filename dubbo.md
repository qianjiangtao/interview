####  1.`dubbo支持哪些协议`？都有什么不同？

- dubbo协议

  ​	Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

  ```
  缺省协议，使用基于mina1.1.7+hessian3.2.1的tbremoting交互。
  连接个数：单连接
  连接方式：长连接
  传输协议：TCP
  传输方式：NIO异步传输
  序列化：Hessian二进制序列化
  适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串。
  适用场景：常规远程服务方法调用
  ```

  ```
  为什么要消费者比提供者个数多：
  因dubbo协议采用单一长连接，
  假设网络为千兆网卡(1024Mbit=128MByte)，
  根据测试经验数据每条连接最多只能压满7MByte(不同的环境可能不一样，供参考)，
  理论上1个服务提供者需要20个服务消费者才能压满网卡。
  
  为什么不能传大包：
  因dubbo协议采用单一长连接，
  如果每次请求的数据包大小为500KByte，假设网络为千兆网卡(1024Mbit=128MByte)，每条连接最大7MByte(不同的环境可能不一样，供参考)，
  单个服务提供者的TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。
  单个消费者调用单个服务提供者的TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。
  如果能接受，可以考虑使用，否则网络将成为瓶颈。
  
  为什么采用异步单一长连接：
  因为服务的现状大都是服务提供者少，通常只有几台机器，
  而服务的消费者多，可能整个网站都在访问该服务，
  比如Morgan的提供者只有6台提供者，却有上百台消费者，每天有1.5亿次调用，
  如果采用常规的hessian服务，服务提供者很容易就被压跨，
  通过单一连接，保证单一消费者不会压死提供者，
  长连接，减少连接握手验证等，
  并使用异步IO，复用线程池，防止C10K问题。
  ```

- RMI : RMI协议采用JDK标准的java.rmi.*实现，采用阻塞式短连接和JDK标准序列化方式 

  ```
  Java标准的远程调用协议。
  连接个数：多连接
  连接方式：短连接
  传输协议：TCP
  传输方式：同步传输
  序列化：Java标准二进制序列化
  适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
  适用场景：常规远程服务方法调用，与原生RMI服务互操作
  ```

- hessian : Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现 

  ```
  基于Hessian的远程调用协议。
  
  连接个数：多连接
  连接方式：短连接
  传输协议：HTTP
  传输方式：同步传输
  序列化：Hessian二进制序列化
  适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
  适用场景：页面传输，文件传输，或与原生hessian服务互操作
  ```

- http :采用Spring的HttpInvoker实现  

  ```
  基于http表单的远程调用协议。
  
  连接个数：多连接
  连接方式：短连接
  传输协议：HTTP
  传输方式：同步传输
  序列化：表单序列化（JSON）
  适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
  适用场景：需同时给应用程序和浏览器JS使用的服务。
  ```

  

- webservice : 基于CXF的frontend-simple和transports-http实现  

  ```
  基于WebService的远程调用协议。
  
  连接个数：多连接
  连接方式：短连接
  传输协议：HTTP
  传输方式：同步传输
  序列化：SOAP文本序列化
  适用场景：系统集成，跨语言调用。
  ```

  



####  2.远程服务发布的原理和流程？

![img](https://gitee.com/qianjiangtao/my-image/raw/master/dubbo/auto-orient) 

说明：

​	`dubbo`的服务发布无非就是Invoker和exporter的转化，简单的流程就是通过spring事件机制通知`ServiceBean`执行export，首先根据注册中心配置将注册信息和协议信息聚合在一个URL中, 再通过`ProxyFactory `将需要发布的服务实现封装成一个invoker（**一个可执行的对象，能够根据方法名称、参数得到相应的执行结果** ）,最后使用Protocol将invoker导出成一个Exporter ，期间做了两件主要的工作：

一，通过`DubboProtocol `将服务导出

```java
首先根据Invoker的url获取ExchangeServer通信对象（负责与客户端的通信模块），以url中的host和port作为key
存至Map<String, ExchangeServer> serverMap中。即可以采用全部服务的通信交给这一个ExchangeServer通信对
象 
```

```
创建一个DubboExporter，封装invoker。然后根据url的port、path（接口的名称）、版本号、分组号作为key，将
DubboExporter存至Map<String, Exporter<?>> exporterMap中
```

> 现在我们要搞清楚我们的目的：通过通信对象获取客户端传来的Invocation invocation参数，然后找到对应的`DubboExporter`（即能够获取到本地Invoker）就可以执行服务了 



二，根据注册中心的`registryUrl`获取注册服务Registry，然后将`serviceUrl`注册到注册中心上,供客户端订阅 

```java
Registry registry = registryFactory.getRegistry(registryUrl);
registry.register(serviceUrl)
```

####  3.远程服务引用原理

第一步：收集配置的参数，参数如下： 

```properties
methods=hello,
timestamp=1443695417847,
dubbo=2.5.3
application=consumer-of-helloService
side=consumer
pid=7748
interface=com.demo.dubbo.service.HelloService
```

第二步：从注册中心引用服务，创建出Invoker对象 

```
根据url中指定的协议（没有指定的话使用默认的dubbo协议）对外公布这个HelloService服务，当客户端根据协议调用这个服务时，将客户端传递过来的Invocation参数交给服务器端的Invoker来执行。
```

而针对客户端，则需要根据服务器开放的协议（服务器端在注册中心注册的url地址中含有该信息）来创建相应的协议的Invoker对象，如

- DubboInvoker
- InjvmInvoker
- `ThriftInvoker`

第三步：使用`ProxyFactory`创建出一个接口的代理对象，该代理对象的方法的执行都交给上述Invoker来执行，代码如下 

#### 4.谈谈`dubbo`中的`负载均衡算法`及特点 

`dubbo`默认有四种负载均衡算法如下：

- `Random LoadBalance` : 根据权重随机均衡

  ```
  它的算法思想很简单。假设我们有一组服务器 servers = [A, B, C]，他们对应的权重为 weights = [5, 3, 
  2]，权重总和为10。现在把这些权重值平铺在一维坐标值上，[0, 5) 区间属于服务器 A，[5, 8) 区间属于服务
  器 B，[8, 10) 区间属于服务器 C。接下来通过随机数生成器生成一个范围在 [0, 10) 之间的随机数，然后计算
  这个随机数会落到哪个区间上
  ```

- `LeastActive LoadBalance`：根据权重，活跃数负载均衡

  ```
  每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加
  1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下
  降的也越快。此时这样的服务提供者能够优先获取到新的服务请求，这就是最小活跃数负载均衡算法的基本思想
  ```

- `RoundRobin LoadBalance`: 加权轮询

  ```
  所谓轮询就是将请求轮流分配给一组服务器。举个例子，我们有三台服务器 A、B、C。我们将第一个请求分配给服
  务器 A，第二个请求分配给服务器 B，第三个请求分配给服务器 C，第四个请求再次分配给服务器 A。这个过程就
  叫做轮询。轮询是一种无状态负载均衡算法，实现简单，适用于每台服务器性能相近的场景下。显然，现实情况
  下，我们并不能保证每台服务器性能均相近。如果我们将等量的请求分配给性能较差的服务器，这显然是不合理
  的。因此，这个时候我们需要加权轮询算法，对轮询过程进行干预，使得性能好的服务器可以得到更多的请求，性
  能差的得到的少一些。每台服务器能够得到的请求数比例，接近或等于他们的权重比。比如服务器 A、B、C 权重比
  为 5:2:1。那么在8次请求中，服务器 A 将获取到其中的5次请求，服务器 B 获取到其中的2次请求，服务器 C 
  则获取到其中的1次请求
  ```

- `ConsistentHash LoadBalance`： 一致性hash

  ```
  它的工作过程是这样的，首先根据 ip 获取其他的信息为缓存节点生成一个 hash，并将这个 hash 投射到 [0, 
  232 - 1] 的圆环上。当有查询或写入请求时，则为缓存项的 key 生成一个 hash 值。然后查找第一个大于或等
  于该 hash 值的缓存节点，并到这个节点中查询或写入缓存项。如果当前节点挂了，则在下一次查询或写入缓存
  时，为缓存项查找另一个大于其 hash 值的缓存节点即可。大致效果如下，每个缓存节点在圆环上占据一个位置。
  如果缓存项的 key 的 hash 值小于缓存节点 hash 值，则到该缓存节点中存储或读取缓存项。比如下面绿色点对
  应的缓存项存储到 cache-2 节点中。由于 cache-3 挂了，原本应该存到该节点中的缓存想最终会存储到 cache-
  4 节点中。
  ```

  ![1552546176928](https://gitee.com/qianjiangtao/my-image/raw/master/dubbo/1552546176928.png)

关于一致性 hash 算法，我这里只做扫盲。具体的细节不讨论，大家请自行补充相关的背景知识。下面来看看一致性 hash 在 Dubbo 中的应用。我们把上图的缓存节点替换成 Dubbo 的服务提供者，于是得到了下图： 

![1552546198382](https://gitee.com/qianjiangtao/my-image/raw/master/dubbo/1552546198382.png)

这里相同颜色的节点均属于同一个服务提供者，比如 Invoker1-1，Invoker1-2，……, Invoker1-160。这样做的目的是通过引入虚拟节点，让 Invoker 在圆环上分散开来，避免数据倾斜问题。所谓数据倾斜是指，由于节点不够分散，导致大量请求落到了同一个节点上，而其他节点只会接收到了少量的请求。比如： 

![1552546270449](https://gitee.com/qianjiangtao/my-image/raw/master/dubbo/1552546252451.png)

#### 5. 画一画服务注册与发现的流程图

 ![1552552778563](https://gitee.com/qianjiangtao/my-image/raw/master/dubbo/1552552778563.png)

####  6.`Dubbo`有哪几种集群容错方案 

- Failover Cluster - 失败自动切换(默认)
- Failfast Cluster - 快速失败，立即报错，只发起一次调用  
- Failsafe Cluster - 失败安全，败安全，出现异常时，直接忽略 
- Failback Cluster - 失败自动恢复，失败自动恢复，记录失败请求，定时重发 
- Forking Cluster - 并行调用多个服务提供者，并行调用多个服务器，只要一个成功即返回 

