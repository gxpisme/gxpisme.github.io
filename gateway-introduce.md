# 网关（接入层）介绍

> 在架构的早期，基本上是没有这个概念的，也没有对应的团队来专门负责这个事情。

>

> 网关是流量进入的第一道关卡，所有的流量都会经过网关之后，再到业务服务。

>

> 有的公司也称网关叫做接入层，许多业务团队都是这么叫的，哈哈。我们公司现在也成一些规模，有专门的一个部门来做这个事情，叫做网关组。

## 网关（接入层）产生的背景

> 在业务不断发展，流量越来越大的过程中，会有下面这些问题。

- 流量进入系统，这些流量有些是异常流量，比如是爬虫的流量。

- 流量还需要根据不同的业务场景还会路由到不同的机器上。

- 流量是否是一些激增的流量，会打垮我们的系统，需要对系统容量做一个保障。

- 一些外部流量进入了内部系统，外部流量能够打开公司后台网页。

- 流量中的请求参数是加密后的，但是系统只能识别解密之后的。

- 有时间某个地域的DNS不好使，导致自己的业务受损。

- 等等。

上面这些问题，都是在网站架构发展阶段遇到的一些问题。对这些问题就行归纳总结抽象，这些问题都是在一个切面上产生的问题。

也就是这些问题都是流量进入时候产生的问题，进入系统后到也是跟这些问题无关了。

这时候就需要在流量接入的时候，处理这些事情，也就产生了接入层，也叫做网关。

## 网关（接入层）目的

提供组件化的能力，能够对这些接入的流量进行统一处理，解决上述提出的这些问题。

## 接入层（网关）具有的能力

网关一般都要具备**可扩展**能力，方便某个组件**可插拔**。

也就是如果需要某个功能，就能够及时开发，上线，将网关能力增强。

如果不需要某个功能，完全可以在网关中把这些功能移除掉。

### 路由

路由就是能够将不同的请求放置到对应的机器上。可以理解为有人寻路，高速这个人怎么走，才能走到这个人的目的地。

也可能会出现奇奇怪怪的请求，这些请求根本没有目的地，可以把他们都指到一个404页面，对用户也友好，也是比较常用的操作。

这也是网关中非常重要的功能。

### 限流

这个主要就是保障系统容量，防止流量冲击将整个系统打垮了。

不仅仅是在网关层，而且在微服务架构情况下，各个微服务相互调用，也是非常重要的手段。

也可以用于下线服务。先将某个业务限流为0，经过一段时间没有反馈什么问题，在操作下线，如有问题，及时把限流开关关闭即可。

限流也是有专门的一些限流算法，这个会单独开篇博文来讲。

### 降级

降级也是保证系统可用的非常重要的手段。

如果某个业务发生问题，不允许外部用户进入，则直接可以启用降级。

### 熔断

这个可以理解为保险丝，当使用多个大功率电器的时候，电线承受不住这样的功率，就会触发保险丝熔断，是保护系统的一种重要方式。

### 协议

协议是双方互相通信的基础，接入层网关只能算作一方，当然也会有很多其他方，相互通信是建立在协议之上的。

有很多其他方，所有需要对不同的协议的支持，使系统更加健壮，能力丰富。

### 反爬虫

一般都会有一个团队叫做反作弊团队来做这件事的，要保证的是，网关具有良好的扩展性。

这样反作弊团队开发的组件能够适用于网关。

### 安全

网络漏洞，也是对系统、用户有着极大的危害，数据泄露事件数不胜数。

接入层需要对数据的传输进行严格的防守，一般采取加密等措施。

### 权限

针对不同网络的请求，要给出不同的权限，最简单的就是外网用户不能访问公司内网后台。

### 监控

每个基础设施，每个系统，其实接入层（网关）也是一种基础设施。

都是需要能够监控的，你才能知道这个基础设施是否健康。

### 配置

这个配置是可以提高生产力的，比如：配置一些数据开始时间结束时间。

可以配置路由，可以配置限流多少，可以配置打到哪个集群，可以配置返回结构等等。

