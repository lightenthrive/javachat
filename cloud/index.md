# 微服务

## 什么是服务化？

服务化就是把传统的单机应用中通过JAR包依赖产生的本地方法调用，改造成通过RPC接口产生的远程方法调用。一般在编写业务代码时，对于一些通用的业务逻辑，我会尽力把它抽象并独立成为专门的模块，因为这对于代码复用和业务理解都大有裨益。

在过去的项目经历里，我对此深有体会。以微博系统为例，微博既包含了内容模块，也包含了消息模块和用户模块等。其中消息模块依赖内容模块，消息模块和内容模块又都依赖用户模块。当这三个模块的代码耦合在一起，应用启动时，需要同时去加载每个模块的代码并连接对应的资源。一旦任何模块的代码出现bug，或者依赖的资源出现问题，整个单体应用都会受到影响。

为此，首先可以把用户模块从单体应用中拆分出来，独立成一个服务部署，以RPC接口的形式对外提供服务。微博和消息模块调用用户接口，就从进程内的调用变成远程RPC调用。

这样，用户模块就可以独立开发、测试、上线和运维，可以交由专门的团队来做，与主模块不耦合。进一步的可以再把消息模块也拆分出来作为独立的模块，交由专门的团队来开发和维护。

可见通过服务化，可以解决单体应用膨胀、团队开发耦合度高、协作效率低下的问题。

## 微服务相比于服务化又有什么不同呢？

在我看来，可以总结为以下四点：服务拆分粒度更细。微服务可以说是更细维度的服务化，小到一个子模块，只要该模块依赖的资源与其他模块都没有关系，那么就可以拆分为一个微服务。服务独立部署。每个微服务都严格遵循独立打包部署的准则，互不影响。比如一台物理机上可以部署多个Docker实例，每个Docker实例可以部署一个微服务的代码。服务独立维护。每个微服务都可以交由一个小团队甚至个人来开发、测试、发布和运维，并对整个生命周期负责。服务治理能力要求高。因为拆分为微服务之后，服务的数量变多，因此需要有统一的服务治理平台，来对各个服务进行管理。继续以前面举的微博系统为例，可以进一步对内容模块的功能进行拆分，比如内容模块又包含了feed模块、评论模块和个人页模块。通过微服务化，将这三个模块变成三个独立的服务，每个服务依赖各自的资源，并独立部署在不同的服务池中，可以由不同的开发人员进行维护。当评论服务需求变更时，只需要修改评论业务相关的代码，并独立上线发布；而feed服务和个人页服务不需要变更，也不会受到发布可能带来的变更影响。由此可见，微服务化给服务的发布和部署，以及服务的保障带来了诸多好处。

## 什么是微服务架构？

微服务架构是将复杂臃肿的单体应用进行细粒度的服务化拆分，每个拆分出来的服务各自独立打包部署，并交由小团队进行开发和运维，从而极大地提高了应用交付的周期，并被各大互联网公司所普遍采用。
