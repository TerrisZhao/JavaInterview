# dubbo

### dubbo 调用流程

```
Provider
第 0 步，start 启动服务。
第 1 步，register 注册服务到注册中心。
Consumer
第 2 步，subscribe 向注册中心订阅服务。
注意，只订阅使用到的服务。
再注意，首次会拉取订阅的服务列表，缓存在本地。
【异步】第 3 步，notify 当服务发生变化时，获取最新的服务列表，更新本地缓存。
invoke 调用
Consumer 直接发起对 Provider 的调用，无需经过注册中心。而对多个 Provider 的负载均衡，Consumer 通过 cluster 组件实现。
count 监控
【异步】Consumer 和 Provider 都异步通知监控中心。

```



### 如何做到请求异步转同步

```
考察点


furture


请求id，RequestContext 确认
```



### 注册中心挂了，是否还可以正常通信

```
注册中心中任意一台机器宕机之后，可以切换到另一台主机上。如果所有的主机都宕机了，还可以依赖本地缓存进行通信。
```



### dubbo rpc 如何实现远程调用像本地调用一样

```
考察点 动态代理
```



### dubbo zk 如何平滑部署

```
```

