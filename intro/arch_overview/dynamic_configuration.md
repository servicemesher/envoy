# 动态配置

Envoy is architected such that different types of configuration management approaches are possible. The approach taken in a deployment will be dependent on the needs of the implementor. Simple deployments are possible with a fully static configuration. More complicated deployments can incrementally add more complex dynamic configuration, the downside being that the implementor must provide one or more external REST based configuration provider APIs. This document gives an overview of the options currently available.

- Top level configuration [reference](../../configuration/configuration.md#config).
- [Reference configurations](../../install/ref_configs.md#install-ref-configs).
- Envoy [v2 API overview](../../configuration/overview/v2_overview.md#config-overview-v2).

## 全静态

In a fully static configuration, the implementor provides a set of [listeners](../../configuration/listeners/listeners.md#config-listeners) (and [filter chains](../../api-v1/listeners/listeners.md#config-listener-network-filters)), [clusters](../../configuration/cluster_manager/cluster_manager.md#config-cluster-manager), and optionally [HTTP route configurations](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table). Dynamic host discovery is only possible via DNS based[service discovery](service_discovery.md#arch-overview-service-discovery). Configuration reloads must take place via the built in [hot restart](hot_restart.md#arch-overview-hot-restart) mechanism.

Though simplistic, fairly complicated deployments can be created using static configurations and graceful hot restarts.

## 仅 SDS/EDS

The [service discovery service (SDS) API](../../api-v1/cluster_manager/sds.md#config-cluster-manager-sds) provides a more advanced mechanism by which Envoy can discover members of an upstream cluster. SDS has been renamed to [Endpoint Discovery Service (EDS)](../../api-v2/api/v2/eds.proto.md#envoy-api-file-envoy-api-v2-eds-proto) in the [v2 API](../../configuration/overview/v2_overview.md#config-overview-v2). Layered on top of a static configuration, SDS allows an Envoy deployment to circumvent the limitations of DNS (maximum records in a response, etc.) as well as consume more information used in load balancing and routing (e.g., canary status, zone, etc.).

## SDS/EDS 和 CDS

The [cluster discovery service (CDS) API](../../configuration/cluster_manager/cds.md#config-cluster-manager-cds) layers on a mechanism by which Envoy can discover upstream clusters used during routing. Envoy will gracefully add, update, and remove clusters as specified by the API. This API allows implementors to build a topology in which Envoy does not need to be aware of all upstream clusters at initial configuration time. Typically, when doing HTTP routing along with CDS (but without route discovery service), implementors will make use of the router’s ability to forward requests to a cluster specified in an [HTTP request header](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-cluster-header).

Although it is possible to use CDS without SDS/EDS by specifying fully static clusters, we recommend still using the SDS/EDS API for clusters specified via CDS. Internally, when a cluster definition is updated, the operation is graceful. However, all existing connection pools will be drained and reconnected. SDS/EDS does not suffer from this limitation. When hosts are added and removed via SDS/EDS, the existing hosts in the cluster are unaffected.

## SDS/EDS、CDS 和 RDS

The [route discovery service (RDS) API](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds) layers on a mechanism by which Envoy can discover the entire route configuration for an HTTP connection manager filter at runtime. The route configuration will be gracefully swapped in without affecting existing requests. This API, when used alongside SDS/EDS and CDS, allows implementors to build a complex routing topology ([traffic shifting](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting), blue/green deployment, etc.) that will not require any Envoy restarts other than to obtain a new Envoy binary.

## SDS/EDS、CDS、RDS 和 LDS

The [listener discovery service (LDS)](../../configuration/overview/v1_overview.md#config-overview-lds) layers on a mechanism by which Envoy can discover entire listeners at runtime. This includes all filter stacks, up to and including HTTP filters with embedded references to [RDS](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds). Adding LDS into the mix allows almost every aspect of Envoy to be dynamically configured. Hot restart should only be required for very rare configuration changes (admin, tracing driver, etc.) or binary updates.