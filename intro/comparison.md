# 与类似系统比较

一言概之,我们相信Envoy有一套独有特色的优秀的功能集合,以支撑现代面向服务的架构. 以下我们将以Envoy与类似的其余系统进行比对. 
虽然在某些特定领域(例如边缘代理,软负载均衡,服务消息传送层),Envoy未如以下的某些系统提供更加完备的支持,但在完整比对后,没有其他任何一个系统
能提供一套例如Envoy一般拥有完备功能,自包含,以及高性能的解决方案.

**NOTE:** 以下的许多项目还处在一个开发活跃期,我们所描述的信息也许会与项目最新的状态脱节,如果您发现这些情况,请立即告知我们,我们将会进行修正.

## [nginx](https://nginx.org/en/)

nginx是一个经典的现代web服务器. 它支持静态内容展现, HTTP L7反向代理 负载均衡,HTTP/2,以及其他的许多特性. 作为一个边缘反向代理,nginx提供了远远多于Envoy的功能特性,
但我们认为大多数的现代面向服务的架构其实不需要用到那么多的特性. 而Envoy在下列边缘代理的特性做得比nginx更为出色:

- 完备的HTTP/2 透明代理支持. Envoy支持HTTP/2 包括上游连接以及下游连接在内的双向通信. 而nginx仅仅支持HTTP/2 下游连接.
- 免费的高级负载功能. 而在nginx的世界,只有付费的nginx plus服务器才能提供类同于Envoy的高级负载功能.
- 可以在每一个服务节点的边界运行同样一套软件来处理事务. 在许多架构体系中,需要使用nginx与haproxy的混合部署架构. 相比之下,一个独立的代理解决方案会更有利于后续的运维维护.

## [haproxy](http://www.haproxy.org/)

haproxy是一个经典的现代软负载均衡服务器. 她提供基本的HTTP反向代理功能. 而Envoy在下列负载均衡的特性做得比haproxy更为出色：

- HTTP/2 支持.
- 可插拔架构.
- 与远程服务发现服务的整合.
- 与远程全局限速服务的整合.
- 提供大量的更为细致的统计分析.

## [AWS ELB](https://aws.amazon.com/elasticloadbalancing/)

Amazon的ELB为对Amazon EC2上运行的程序提供服务发现以及负载均衡服务.而Envoy在下列负载均衡以及服务发现的特性做得比ELB更为出色:

- 统计与日志 (CloudWatch的统计有延迟,并且在一些细节上严重缺乏,而Amazon的日志只能在S3以特定格式获取)
- 稳定性(使用ELB时不时会遇到不稳定事故,并且难以进行debug)
- 更为高级的负载均衡以及节点间的直连功能. Envoy mesh在进行硬件弹性伸缩时并不需要额外的网络跳转. 
负载均衡可以根据区域,金丝雀状态的回馈提供更好的决策以及收集到许多有意思的统计分析结果. 而且负载均衡还支持例如重试这样的高级特性.

AWS最近发布了一个 *application load balancer* 产品. 这个产品增加了HTTP/2支持,可以将HTTP/2与基本的HTTP L7请求一般流转到不同的后端集群.
相比Envoy,这个产品所能提供的功能偏少,且性能与稳定性还未知,但不容置疑的是AWS将会在这个领域持续地进行研发.

## [SmartStack](http://nerds.airbnb.com/smartstack-service-discovery-cloud/)

SmartStack 这个方案特别有意思,它在haproxy的基础上提供了更多的服务发现以及健康检查支持. 抽象地说,SmartStack在大多数目标上与Envoy保持一致(与进程无关的架构,对应用平台的不可知性 等等).
而Envoy在下列负载均衡以及服务发现的特性做得比SmartStack更为出色:

- 上述所提及的做的比haproxy好的所有特性.
- 整合服务发现与积极的健康检查. Envoy将所有的功能都在一个高性能的方案内完整提供.

## [Finagle](https://twitter.github.io/finagle/)

Finagle是Twitter基于 Scala/JVM开发的服务通讯库. 在Twitter以及其他公司的基于JVM的架构中普遍使用.
它提供了服务发现,负载均衡,过滤器等Envoy也具有的功能.
而Envoy在下列负载均衡以及服务发现的特性做得比Finagle更为出色:

- 最终一致的服务发现,基于在分布式的积极健康检查基础上实现
-在各个维度上都有更优越的性能表现(内存占用率,CPU使用率,P99 时延)
- 与进程无关的架构,对应用平台的不可知性的架构. Envoy与不同的应用技术栈都可一起工作.

## [proxygen](https://github.com/facebook/proxygen) 和 [wangle](https://github.com/facebook/wangle)

proxygen是Facebook使用C+11 开发的高性能HTTP代理库, 是基于一个类同Finagle的C++库wangle进行开发.
在代码实现层面,Envoy使用了类同的技术去实现一个高性能的的HTTP 库/代理. 除此之外,两个项目不具备可比性.
Envoy是一个完备的自包含服务,提供了大量的功能点.相比之下,proxygen仅仅是一个库,各个项目还需要基于它继续构建功能.

## [gRPC](http://www.grpc.io/)

gRPC是一个由google研发的跨平台的消息传送系统. 它使用IDL去描述RPC库,然后基于IDL为各种不同的编程语言生成不同的应用运行时.它底层基于HTTP/2去实现传输层.gRPC似乎有一个终极的目标,在将来去实现许多Enovy的功能点(例如负载均衡),但在我们写就这篇文档的时候,还有不少的运行时库还不成熟并且集中在序列以及反序列化上.我们认为gRPS是Envoy的搭档,而不是竞争者.
Envoy如何与gRPC进行整合,您可以查看[此链接](arch_overview/grpc.html#arch-overview-grpc).

## [linkerd](https://github.com/BuoyantIO/linkerd)

linkerd是一个独立的,开源的RPC路由代理,它基于Netty与Finagle (Scala/JVM)研发. linkerd提供了许多Finagle的特性,包括对延时敏感的负载均衡,连接池,断路器,重试预算,截至线,追踪,细粒度的植入,
以及对请求的流量路由层.linkerd提供一个可插拔的服务发现层(Consul以及Zookeeper为此提供标准支持,就如Marathon 以及Kubernetes 的API).

linkerd的内存消耗以及CPU的要求都远远高于Envoy. 对比Enovy,linkerd仅提供了一个最小可用的配置语言, 且不支持热加载,而是用动态配置以及服务抽象的方式变相地
提供类似的功能. linkerd支持HTTP/1.1, Thrift, ThriftMux, HTTP/2 (experimental) 和 gRPC (experimental).

## [nghttp2](https://nghttp2.org/)

nghttp2 是一个包含了不同事物的项目. 首先,它包含了一个库(nghttp2),用来实现HTTP/2协议.Envoy使用这个库(在这个库上做了一层很浅的封装)来做HTTP/2的支持.
这个项目还包含了一个非常有用的压力测试工具(h2load) 以及一个反向代理 (nghttpx). 如果要做个比对, 在nghttp2所实现的众多功能点中,我们觉得Envoy更类似于nghttpx, nghttpx是一个透明的 
HTTP/1 <-> HTTP/2 反向代理, 支持 TLS 终止, 支持gRPC代理. 我们认为nghttpx是一个表现不同代理功能点的杰出范例,而并不是一个健壮的生产可用的解决方案.
Envoy的产品焦点更多地放在可监控性,操作的敏捷性,以及高级的负载均衡功能.