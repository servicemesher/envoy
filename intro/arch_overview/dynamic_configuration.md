# 动态配置

Envoy 的架构使得使用不同类型的配置管理方法成为可能。部署中采用的方法将取决于实现者的需求。简单部署可以通过全静态配置来实现。更复杂的部署可以递增地添加更复杂的动态配置，缺点是实现者必须提供一个或多个基于外部 REST 配置的提供者 API。本文档概述了当前可用的选项。

- 顶级配置[参考](../../configuration/configuration.md#config)
- [参考配置](../../install/ref_configs.md#install-ref-configs)
- Envoy [v2 API 概述](../../configuration/overview/v2_overview.md#config-overview-v2)

## 全静态

在全静态配置中，实现者提供一组[监听器](../../configuration/listeners/listeners.md#config-listeners)（和[过滤器链](https://www.envoyproxy.io/docs/envoy/latest/api-v1/listeners/listeners#config-listener-network-filters)）、 [集群](../../configuration/cluster_manager/cluster_manager.md#config-cluster-manager)和可选的 [HTTP 路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route_config#config-http-conn-man-route-table)。动态主机发现仅能通过基 于DNS 的[服务发现](service_discovery.md#arch-overview-service-discovery)。配置重载必须通过内置的[热重启](hot_restart.md#arch-overview-hot-restart)机制进行。

虽然简单，但可以使用静态配置和优雅的热重启来创建相当复杂的部署。

## 仅 SDS/EDS

服务发现服务（SDS）API 提供更高级的机制，Envoy 可以通过该机制发现上游集群中的成员。SDS 已在 v2 API 中重命名为 Endpoint Discovery Service（EDS）。 在静态配置之上，SDS 允许 Envoy 部署规避 DNS 的局限性（响应中的最大记录等），并使用更多可用于负载均衡和路由的信息（例如，金丝雀状态、区域等）。

## SDS/EDS 和 CDS

[集群发现服务 (CDS) API](../../configuration/cluster_manager/cds.md#config-cluster-manager-cds) 是 Envoy 的一种机制，在路由期间可以用来发现使用的上游集群。Envoy 将优雅地添加、更新和删除由 API 指定的集群。该API允许实现者构建拓扑，在其中 Envoy 在初始配置时不需要知道所有上游群集。通常，在与 CDS（但没有路由发现服务）一起进行 HTTP 路由时，实现者将利用路由器的能力将请求转发到在 [HTTP 请求头](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-cluster-header)中指定的集群。

尽管可以通过指定全静态集群来使用不带 SDS/EDS 的 CDS，但我们仍建议为通过 CDS 指定的集群使用 SDS/EDS API。 在内部，更新集群定义时，操作是优雅的。但是，所有现有的连接池都将被排空并重新连接。SDS/EDS 不受此限制。当通过 SDS/EDS 添加和删除主机时，集群中的现有主机不受影响。

## SDS/EDS、CDS 和 RDS

[路由发现服务 (RDS) API](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds) 是 Envoy 的一种机制，可以在运行时发现用于HTTP连接管理器过滤器的整个路由配置。路由配置将优雅地交换，而不会影响现有的请求。这个 API 与 SDS/EDS 和 CDS 一起使用时，允许实现者构建复杂的路由拓扑（[流量转移](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting)、蓝绿部署等），除了获取新的 Envoy 二进制文件外，不需要重启 Envoy。

## SDS/EDS、CDS、RDS 和 LDS

[监听器发现服务 (LDS)](../../configuration/overview/v1_overview.md#config-overview-lds) 是 Envoy 的一种机制，可以在运行时发现整个监听器。这包括所有的过滤器堆栈，直到并包括带有内嵌到 [RDS](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds) 的应用的 HTTP 过滤器。将 LDS 添加到混和中，几乎可以动态配置 Envoy 的每个方面。只有非常少见的配置更改（管理员、追踪驱动程序等）或二进制更新时才需要热启动。