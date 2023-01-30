# springcloud

### feign实现原理

```
考察点：动态代理要知道


----------------


openfeign通过包扫描将所有被@FeignClient注解注释的接口扫描出来，并为每个接口注册一个FeignClientFactoryBean<T>实例。FeignClientFactoryBean<T>是一个FactoryBean<T>，当Spring调用FeignClientFactoryBean<T>的getObject方法时，openfeign返回一个Feign生成的动态代理对象，拦截接口的方法执行。


feign会为代理的接口的每个方法Method都生成一个MethodHandler。


当为接口上的@FeignClient注解的url属性配置服务提供者的url时，其实就是不与Ribbon整合，此时由SynchronousMethodHandler实现接口方法远程同步调用，使用默认的Client实现类Default实例发起http请求。


当接口上的@FeignClient注解的url属性不配置时，且会走负载均衡逻辑，也就是需要与Ribbon整合使用。这时候不再是使用默认的Client（Default）调用接口，而是使用LoadBalancerFeignClient调用接口，由LoadBalancerFeignClient实现与Ribbon的整合。

```



### feign与ribbon什么区别

```
考察点 feign ribbon 功能上的区别


都是为了解决负载均衡调用的组件，feign 在 ribbon 基础上再次进行了封装。
1. 启动类使用的注解不同，Ribbon 用的是@RibbonClient，Feign 用的是@EnableFeignClients。 
2. 服务的指定位置不同，Ribbon 是在@RibbonClient 注解上声明，Feign 则是在定义抽象方法的接口中使用@FeignClient 声明。
3. 调用方式不同，Ribbon 需要自己构建 http 请求，模拟 http 请求然后使用 RestTemplate 发送给其他服务。feign采用接口的方式， 只需要创建一个接口，然后在上面添加注解即可 ，将需要调用的其他服务的方法定义成抽象方法即可， 不需要自己构建 http 请求。
```



### feign是如何跟eureka ribbon整合的

```
1.用户调用 Feign 创建的动态代理。
2.Feign 调用 Ribbon 发起调用流程。
2.1 Ribbon 会从 Eureka Client 里获取到对应的服务列表。
2.2 Ribbon 使用负载均衡算法获得使用的服务。
2.3 Ribbon 调用对应的服务。最后，Ribbon 调用 Feign ，而 Feign 调用 HTTP 库最终调用使用的服务。
```



### ribbon有哪些负载均衡算法

```
考察点：常用的规则 ，以及自定义的规则 （比如灰度发布）


RoundRobinRule： 默认轮询的方式
RandomRule： 随机方式
WeightedResponseTimeRule： 根据响应时间来分配权重的方式，响应的越快，分配的值越大。
BestAvailableRule： 选择并发量最小的方式
RetryRule： 在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server
ZoneAvoidanceRule： 根据性能和可用性来选择。
AvailabilityFilteringRule： 过滤掉那些因为一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的的后端server（active connections 超过配置的阈值）
LeastActiveLoadBalance：最小活跃数负载均衡算法
ConsistentHashLoadBalance：一致性hash算法
```



### springboot 启动流程

```
启动流程图https://www.processon.com/view/link/59812124e4b0de2518b32b6e
```



### SPI机制

```
SPI ，全称为 Service Provider Interface，是一种服务发现机制。它通过在ClassPath路径下的META-INF/services文件夹查找文件，自动加载文件里所定义的类。
```



### 扩展机制 starter如何实现

```
可以从源码看出关键功能是@import注解导入自动配置功能类AutoConfigurationImportSelector类，主要方法getCandidateConfigurations()使用了SpringFactoriesLoader.loadFactoryNames()方法加载META-INF/spring.factories的文件（spring.factories声明具体自动配置）
```



### springboot springmvc spring 区别

```
spring boot 就是把 spring spring mvc spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，spring boot 就是来解决可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，开箱即用


 spring 框架有超多的延伸产品例如 boot security jpa etc... 但它的基础就是 spring 的 ioc 和 aop ioc 提供了依赖注入的容器 aop 解决了面向横切面的编程 然后在此两者的基础上实现了其他延伸产品的高级功能 


Spring MVC 呢是基于 Servlet 的一个 MVC 框架 主要解决 WEB 开发的问题 


因为 Spring 的配置太复杂了 各种 XML JavaConfig hin 麻烦 于是推出了 Spring boot 约定优于配置 简化了 spring 的配置流程.

```



### springboot 里 如何修改tomcat端口号

```
server.port=8081
```



### hystrix隔离策略

```
Hystrix目前是有两种隔离策略，分别是线程池隔离和信号量隔离。


1.区别
2.使用场景


服务隔离机制
服务隔离有两种方式：线程池隔离和信号量隔离。一般在高并发情况下，都是使用线程池隔离的。




隔离策略
线程池隔离
如其名，他的隔离是通过线程池来做到的，也就是说他的隔离粒度是线程池。一个请求进来都经过一个线程池。
当前端发起请求过来到服务A或者B之后，服务A和服务B是通过线程池隔离的。服务A是否熔断，是否正常都和服务B无关。
他其实是一个异步编程，用线程池将后面的服务包裹了起来，至于服务内部tomcate的线程运行怎么样是无关的。他适合于绝大多数的场景，对于一些超时的场景都非常好用。但是既然是通过线程池来操作的，不可避免的就是线程之间的计算开销，以及线程上下文的切换，调度消耗。


信号量隔离
如其名，他的隔离是通过信号量来做到的。他其实是一个计数器。一个请求进来就会减少一个信号，一个请求完成就会增加一个信号。
信号量的调用时同步的，也就是说他会阻塞直到请求回来。所以他自身是不能实现超时的，因此这里的超时只能依靠协议的超时来做，否则是无法释放的(比如socket超时等等)。所以当此服务不对外部服务依赖同时自身没有大量的计算或者说经过这个服务的时间比较短，则非常适合信号量，比如说spring cloud的zuul的gateway网关。

```



### 什么是hystrix断路器

```
在高并发情况下，防止用户一直处于等待状态（在tomcat中已经没有线程可以处理用户请求的时候，不应该让用户一直loading等待），使用服务降级的方式，返回一个友好的提示给客户端，不去处理请求，调用fallBack。比如秒杀场景，请求过多，就只返回“当前请求人数过多，请稍后再试”。




熔断的原理 todo


自己实现熔断的话，如何做到资源隔离 todo

```



### 什么是hystrix服务降级

```
在高并发情况下，防止用户一直处于等待状态（在tomcat中已经没有线程可以处理用户请求的时候，不应该让用户一直loading等待），使用服务降级的方式，返回一个友好的提示给客户端，不去处理请求，调用fallBack。比如秒杀场景，请求过多，就只返回“当前请求人数过多，请稍后再试”。
```



### 为什么要网关服务？

```
一、什么是服务网关


服务网关 = 路由转发 + 过滤器


1、路由转发：接收一切外界请求，转发到后端的微服务上去；


2、过滤器：在服务网关中可以完成一系列的横切功能，例如权限校验、限流以及监控等，


这些都可以通过过滤器完成（其实路由转发也是通过过滤器实现的）。


二、为什么需要服务网关


上述所说的横切功能（以权限校验为例）可以写在三个位置：


每个服务自己实现一遍


写到一个公共的服务中，然后其他所有服务都依赖这个服务


写到服务网关的前置过滤器中，所有请求过来进行权限校验


第一种，缺点太明显，基本不用；


第二种，相较于第一点好很多，代码开发不会冗余，但是有两个缺点：


由于每个服务引入了这个公共服务，那么相当于在每个服务中都引入了相同的权限校验的代码，使得每个服务的jar包大小无故增加了一些，尤其是对于使用docker镜像进行部署的场景，jar越小越好；


由于每个服务都引入了这个公共服务，那么我们后续升级这个服务可能就比较困难，而且公共服务的功能越多，升级就越难，而且假设我们改变了公共服务中的权限校验的方式，想让所有的服务都去使用新的权限校验方式，我们就需要将之前所有的服务都重新引包，编译部署。


而服务网关恰好可以解决这样的问题：


将权限校验的逻辑写在网关的过滤器中，后端服务不需要关注权限校验的代码，所以服务的jar包中也不会引入权限校验的逻辑，不会增加jar包大小；


如果想修改权限校验的逻辑，只需要修改网关中的权限校验过滤器即可，而不需要升级所有已存在的微服务。


所以，需要服务网关！！！

```



### zuul的实现原理

```
问核心的功能点：比如 核心过滤器


zuul的核心是一系列的filters, 其作用可以类比Servlet框架的Filter，或者AOP。
zuul把Request route到 用户处理逻辑 的过程中，这些filter参与一些过滤处理，比如Authentication，Load Shedding等。Zuul提供了一个框架，可以对过滤器进行动态的加载，编译，运行。
Zuul的过滤器之间没有直接的相互通信，他们之间通过一个RequestContext的静态类来进行数据传递的。RequestContext类中有ThreadLocal变量来记录每个Request所需要传递的数据。
Zuul的过滤器是由Groovy写成，这些过滤器文件被放在Zuul Server上的特定目录下面，Zuul会定期轮询这些目录，修改过的过滤器会动态的加载到Zuul Server中以便过滤请求使用。
下面有几种标准的过滤器类型：
Zuul大部分功能都是通过过滤器来实现的。Zuul中定义了四种标准过滤器类型，这些过滤器类型对应于请求的典型生命周期。
(1) PRE：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
(2) ROUTING：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netfilx Ribbon请求微服务。
(3) POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。
(4) ERROR：在其他阶段发生错误时执行该过滤器。
内置的特殊过滤器
zuul还提供了一类特殊的过滤器，分别为：StaticResponseFilter和SurgicalDebugFilter
StaticResponseFilter：StaticResponseFilter允许从Zuul本身生成响应，而不是将请求转发到源。
SurgicalDebugFilter：SurgicalDebugFilter允许将特定请求路由到分隔的调试集群或主机。
自定义的过滤器
除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。
例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。

```



### gateway实现原理

```
结合开放问题问 （灰度方案）


Gateway的客户端回向Spring Cloud Gateway发起请求，请求首先会被HttpWebHandlerAdapter进行提取组装成网关的上下文，然后网关的上下文会传递到DispatcherHandler。DispatcherHandler是所有请求的分发处理器，DispatcherHandler主要负责分发请求对应的处理器，比如将请求分发到对应RoutePredicateHandlerMapping(路由断言处理器映射器）。路由断言处理映射器主要用于路由的查找，以及找到路由后返回对应的FilteringWebHandler。FilteringWebHandler主要负责组装Filter链表并调用Filter执行一系列Filter处理，然后把请求转到后端对应的代理服务处理，处理完毕后，将Response返回到Gateway客户端。


在Filter链中，通过虚线分割Filter的原因是，过滤器可以在转发请求之前处理或者接收到被代理服务的返回结果之后处理。所有的Pre类型的Filter执行完毕之后，才会转发请求到被代理的服务处理。被代理的服务把所有请求完毕之后，才会执行Post类型的过滤器。

```



### gateway自定义路由策略

```
Spring Cloud Gateway中自定义路由
路由(Route)：id、目标地址uri、断言集、过滤器集构成
断言(Predicate)：java 8 function中的Predicate，用于判断路径是否被拦截。
过滤器(Filter):对拦截的请求或响应做具体操作
自定义路由断言需要继承 AbstractRoutePredicateFactory 工厂类，重写 apply() 方法的逻辑和shortcutFieldOrder方法。
在 apply() 方法中可以通过 serverWebExchange.getRequest() 拿到 ServerHttpRequest 对象，从而可以获取到请求的参数、请求方式、请求头等信息。
apply() 方法的参数是自定义的配置类，在使用的时候配置参数，在 apply 方法中直接获取使用。
命名需要以 RoutePredicateFactory 结尾，比如 UserNameCheckRoutePredicateFactory，那么在使用的时候 UserNameCheck 就是这个路由断言工厂的名称。
自定义路由谓词可以根据业务重写路径匹配规则或请求路径日志跟踪。
```



### eureka多级缓存原理

```
eureka 服务端缓存


一级缓存 readOnlyCacheMap，数据无过期时间，保存数据信息对外输出。
二级缓存 readWriteCacheMao，包含过期机制 默认180秒过期，当服务下线、过期、注册、状态变更，都会来清除此缓存中的数据。
注册列表：保存全部服务信息
服务client，每隔30s发送一次心跳续约，eurekaServer会有一个线程定时检测心跳故障（心跳超时时间默认90s），默认每60s一次，注册列表。
新服务注册，先更新注册列表信息，清空readWrite缓存，再同步readWrite新的缓存数据。
一个线程会定时从readWrite缓存向readOnly缓存同步，默认30秒。


服务拉取：首先从 readOnly找，没有去readWrite找，如果没有，读取注册列表，拿到之后再填充到各级缓存中。


服务下线：服务下线或被down，同时过期调readWrite缓存中的数据，不影响readOnly缓存中的数据。通过上面定时任务，同步readWrite和readOnly中的数据。


eureka 客户端缓存
eureka Client 启动时会全量拉取服务列表，启动后每隔 30 秒从 Eureka Server 量获取服务列表信息，并保持在本地缓存中

```



### eureka自我保护机制

```


-- 中级


Eureka 自我保护机制
自我保护机制是什么？ 


Eureka Server 在运行期间会去统计心跳失败比例与前一分钟的失败比例对比是否低于 85%（默认值），如果低于 85%，Eureka Server 即会进入自我保护机制。


为什么要有自我保护机制？


Eureka server 会在一定时间内（默认90s）检测某个微服务的心跳来确定是否注销该实例。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，此时不应该做上述操作，而需要自我保护机制。


 -- 高级


如何实现？


每分钟实际收到的心跳数 小于 （期望心跳数【比如5台实例，默认30s发1次 count = 5*2】* 85%）。
会有两个计数器，一个统计上一分钟的实际收到的心跳数，一个统计当前分钟的心跳数，每分钟定时执行把当前分钟的心跳数赋值给上一分钟心跳数，当前分钟清0。
```



### eureka如何实现平滑部署

```
新的应用实例发布过程中，eurekaClient会先于应用初始化成功，这时候要不断重试调用up接口，直至服务启动成功，up接口也就调用成功。
此时再调用down接口，将老的应用实例eureka状态设置为down，等待一段时间(比如1min)，然后将老的应用实例给杀掉。




参数优化：
eureka server端
1.关闭二级缓存
2.集群里eureka节点的变化信息更新的时间间隔缩小
eureka client端
缩短eureka client拉取服务列表间隔
缩小ribbon 本地服务列表缓存时间（默认30s).

```



### eureka nacos zk 区别和设计思想

```
Eureka Server 采用的是Peer to Peer 对等通信。这是一种去中心化的架构，无 master/slave 之分，每一个 Peer 都是对等的。在这种架构风格中，节点通过彼此互相注册来提高可用性，每个节点需要添加一个或多个有效的 serviceUrl 指向其他节点。每个节点都可被视为其他节点的副本。如果某台 Eureka Server 宕机，Eureka Client 的请求会自动切换到新的 Eureka Server 节点上，当宕机的服务器重新恢复后，Eureka 会再次将其纳入到服务器集群管理之中。当节点开始接受客户端请求时，所有的操作都会在节点间进行复制（replicate To Peer）操作，将请求复制到该 Eureka Server 当前所知的其它所有节点中。
问：为什么是AP


Zookeeper采用CP保证数据的一致性的问题，原理采用Zab原子广播协议，当我们的zk领导因为某种原因宕机的情况下，会自动触发重新选一个新的领导角色，整个选举的过程为了保证数据的一致性问题，在选举的过程中整个zk环境是不可使用的可短暂可能无法使用到zk。意味着微服务采用该模式情况下，可能无法实现通讯。
问：为什么是CP


Nacos 默认是采用 AP保证服务可用性。CP 的形式底层集群通过 raft 协议保证数据的一致性的问题。

```



