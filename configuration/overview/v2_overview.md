# v2 API 概览

The Envoy v2 APIs are defined as [proto3](https://developers.google.com/protocol-buffers/docs/proto3) [Protocol Buffers](https://developers.google.com/protocol-buffers/) in the [data plane API repository](https://github.com/envoyproxy/data-plane-api/tree/master/api). They evolve the existing [v1 APIs and concepts](v1_overview.html#config-overview-v1) to support:

- Streaming delivery of [xDS](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md) API updates via gRPC. This reduces resource requirements and can lower the update latency.
- A new REST-JSON API in which the JSON/YAML formats are derived mechanically via the [proto3 canonical JSON mapping](https://developers.google.com/protocol-buffers/docs/proto3#json).
- Delivery of updates via the filesystem, REST-JSON or gRPC endpoints.
- Advanced load balancing through an extended endpoint assignment API and load and resource utilization reporting to management servers.
- [Stronger consistency and ordering properties](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#eventual-consistency-considerations) when needed. The v2 APIs still maintain a baseline eventual consistency model.

See the [xDS protocol description](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md) for further details on aspects of v2 message exchange between Envoy and the management server.

## Bootstrap 配置

To use the v2 API, it’s necessary to supply a bootstrap configuration file. This provides static server configuration and configures Envoy to access [dynamic configuration if needed](../../intro/arch_overview/dynamic_configuration.html#arch-overview-dynamic-config). As with the v1 JSON/YAML configuration, this is supplied on the command-line via the [`-c`](../../operations/cli.html#cmdoption-c) flag, i.e.:

```bash
./envoy -c <path to config>.{json,yaml,pb,pb_text} --v2-config-only
```

where the extension reflects the underlying v2 config representation. The [`--v2-config-only`](../../operations/cli.html#cmdoption-v2-config-only) flag is not strictly required as Envoy will attempt to autodetect the config file version, but this option provides an enhanced debug experience when configuration parsing fails.

The [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) message is the root of the configuration. A key concept in the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) message is the distinction between static and dynamic resouces. Resources such as a [Listener](../../api-v2/api/v2/lds.proto.html#envoy-api-msg-listener) or [Cluster](../../api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) may be supplied either statically in [static_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-static-resources) or have an xDS service such as [LDS](../listeners/lds.html#config-listeners-lds) or [CDS](../cluster_manager/cds.html#config-cluster-manager-cds)configured in [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources).

## 示例

Below we will use YAML representation of the config protos and a running example of a service proxying HTTP from 127.0.0.1:10000 to 127.0.0.2:1234.

### 静态

A minimal fully static bootstrap config is provided below:

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 127.0.0.1, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: some_service }
          http_filters:
          - name: envoy.router
  clusters:
  - name: some_service
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    hosts: [{ socket_address: { address: 127.0.0.2, port_value: 1234 }}]
```

### 除了动态 EDS 大部分静态

A bootstrap config that continues from the above example with [dynamic endpoint discovery](../../intro/arch_overview/dynamic_configuration.html#arch-overview-dynamic-config-sds) via an[EDS](../../api-v2/api/v2/eds.proto.html#envoy-api-file-envoy-api-v2-eds-proto) gRPC management server listening on 127.0.0.3:5678 is provided below:

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 127.0.0.1, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: some_service }
          http_filters:
          - name: envoy.router
  clusters:
  - name: some_service
    connect_timeout: 0.25s
    lb_policy: ROUND_ROBIN
    type: EDS
    eds_cluster_config:
      eds_config:
        api_config_source:
          api_type: GRPC
          cluster_names: [xds_cluster]
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    hosts: [{ socket_address: { address: 127.0.0.3, port_value: 5678 }}]
```

Notice above that *xds_cluster* is defined to point Envoy at the management server. Even in an otherwise completely dynamic configurations, some static resources need to be defined to point Envoy at its xDS management server(s).

In the above example, the EDS management server could then return a proto encoding of a [DiscoveryResponse](../../api-v2/api/v2/discovery.proto.html#envoy-api-msg-discoveryresponse):

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
  cluster_name: some_service
  endpoints:
  - lb_endpoints:
    - endpoint:
        address:
          socket_address:
            address: 127.0.0.2
            port_value: 1234
```

The versioning and type URL scheme that appear above are explained in more detail in the [streaming gRPC subscription protocol](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#streaming-grpc-subscriptions) documentation.

### 动态

A fully dynamic bootstrap configuration, in which all resources other than those belonging to the management server are discovered via xDS is provided below:

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

dynamic_resources:
  lds_config:
    api_config_source:
      api_type: GRPC
      cluster_names: [xds_cluster]
  cds_config:
    api_config_source:
      api_type: GRPC
      cluster_names: [xds_cluster]

static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    hosts: [{ socket_address: { address: 127.0.0.3, port_value: 5678 }}]
```

The management server could respond to LDS requests with:

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.Listener
  name: listener_0
  address:
    socket_address:
      address: 127.0.0.1
      port_value: 10000
  filter_chains:
  - filters:
    - name: envoy.http_connection_manager
      config:
        stat_prefix: ingress_http
        codec_type: AUTO
        rds:
          route_config_name: local_route
          config_source:
            api_config_source:
              api_type: GRPC
              cluster_names: [xds_cluster]
        http_filters:
        - name: envoy.router
```

The management server could respond to RDS requests with:

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.RouteConfiguration
  name: local_route
  virtual_hosts:
  - name: local_service
    domains: ["*"]
    routes:
    - match: { prefix: "/" }
      route: { cluster: some_service }
```

The management server could respond to CDS requests with:

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.Cluster
  name: some_service
  connect_timeout: 0.25s
  lb_policy: ROUND_ROBIN
  type: EDS
  eds_cluster_config:
    eds_config:
      api_config_source:
        api_type: GRPC
        cluster_names: [xds_cluster]
```

The management server could respond to EDS requests with:

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
  cluster_name: some_service
  endpoints:
  - lb_endpoints:
    - endpoint:
        address:
          socket_address:
            address: 127.0.0.2
            port_value: 1234
```

## 管理服务器

A v2 xDS management server will implement the below endpoints as required for gRPC and/or REST serving. In both streaming gRPC and REST-JSON cases, a [DiscoveryRequest](../../api-v2/api/v2/discovery.proto.html#envoy-api-msg-discoveryrequest) is sent and a[DiscoveryResponse](../../api-v2/api/v2/discovery.proto.html#envoy-api-msg-discoveryresponse) received following the [xDS protocol](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md).

### gRPC streaming 端点

- `POST /envoy.api.v2.ClusterDiscoveryService/StreamClusters`


See [cds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto) for the service definition. This is used by Envoy as a client when

```yaml
cds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

is set in the [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) of the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) config.

- `POST /envoy.api.v2.EndpointDiscoveryService/StreamEndpoints`


See [eds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto) for the service definition. This is used by Envoy as a client when

```yaml
eds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

is set in the [eds_cluster_config](../../api-v2/api/v2/cds.proto.html#envoy-api-field-cluster-eds-cluster-config) field of the [Cluster](../../api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) config.

- `POST ``/envoy.api.v2.ListenerDiscoveryService/StreamListeners`


See [lds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto) for the service definition. This is used by Envoy as a client when

```yaml
lds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

is set in the [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) of the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) config.

- `POST /envoy.api.v2.RouteDiscoveryService/StreamRoutes`


See [rds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto) for the service definition. This is used by Envoy as a client when

```yaml
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

is set in the [rds](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-rds) field of the [HttpConnectionManager](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager) config.

### REST 端点

- `POST /v2/discovery:clusters`


See [cds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto) for the service definition. This is used by Envoy as a client when

```yaml
cds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

is set in the [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) of the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) config.

- `POST /v2/discovery:endpoints`


See [eds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto) for the service definition. This is used by Envoy as a client when

```yaml
eds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

is set in the [eds_cluster_config](../../api-v2/api/v2/cds.proto.html#envoy-api-field-cluster-eds-cluster-config) field of the [Cluster](../../api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) config.

- `POST /v2/discovery:listeners`


See [lds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto) for the service definition. This is used by Envoy as a client when

```yaml
lds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

is set in the [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) of the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) config.

- `POST /v2/discovery:routes`


See [rds.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto) for the service definition. This is used by Envoy as a client when

```yaml
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

is set in the [rds](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-rds) field of the [HttpConnectionManager](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager) config.

## 聚合发现服务

While Envoy fundamentally employs an eventual consistency model, ADS provides an opportunity to sequence API update pushes and ensure affinity of a single management server for an Envoy node for API updates. ADS allows one or more APIs and their resources to be delivered on a single, bidirectional gRPC stream by the management server. Without this, some APIs such as RDS and EDS may require the management of multiple streams and connections to distinct management servers.

ADS will allow for hitless updates of configuration by appropriate sequencing. For example, suppose *foo.com* was mappped to cluster *X*. We wish to change the mapping in the route table to point *foo.com* at cluster *Y*. In order to do this, a CDS/EDS update must first be delivered containing both clusters *X* and *Y*.

Without ADS, the CDS/EDS/RDS streams may point at distinct management servers, or when on the same management server at distinct gRPC streams/connections that require coordination. The EDS resource requests may be split across two distinct streams, one for *X* and one for *Y*. ADS allows these to be coalesced to a single stream to a single management server, avoiding the need for distributed synchronization to correctly sequence the update. With ADS, the management server would deliver the CDS, EDS and then RDS updates on a single stream.

ADS is only available for gRPC streaming (not REST) and is described more fully in [this](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#aggregated-discovery-services-ads) document. The gRPC endpoint is:

- `POST /envoy.api.v2.AggregatedDiscoveryService/StreamAggregatedResources`


See [discovery.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/discovery.proto) for the service definition. This is used by Envoy as a client when

```yaml
ads_config:
  api_type: GRPC
  cluster_names: [some_ads_cluster]
```

is set in the [dynamic_resources](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) of the [Bootstrap](../../api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) config.

When this is set, any of the configuration sources [above](#v2-grpc-streaming-endpoints) can be set to use the ADS channel. For example, a LDS config could be changed from

```yaml
lds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

to

```yaml
lds_config: {ads: {}}
```

with the effect that the LDS stream will be directed to *some_ads_cluster* over the shared ADS channel.

## 管理服务器不可达

When Envoy instance looses connectivity with the management server, Envoy will latch on to the previous configuration while actively retrying in the background to reestablish the connection with the management server.

Envoy debug logs the fact that it is not able to establish a connection with the management server every time it attempts a connection.

[upstream_cx_connect_fail](../cluster_manager/cluster_stats.html#config-cluster-manager-cluster-stats) a cluster level statistic of the cluster pointing to management server provides a signal for monitoring this behavior.

## 状态

All features described in the [v2 API reference](../../api-v2/api.html#envoy-api-reference) are implemented unless otherwise noted. In the v2 API reference and the [v2 API repository](https://github.com/envoyproxy/data-plane-api/tree/master), all protos are *frozen* unless they are tagged as *draft* or *experimental*. Here, *frozen* means that we will not break wire format compatibility.

*Frozen* protos may be further extended, e.g. by adding new fields, in a manner that does not break [backwards compatibility](https://developers.google.com/protocol-buffers/docs/overview#how-do-they-work). Fields in the above protos may be later deprecated, subject to the[breaking change policy](https://github.com/envoyproxy/envoy/blob/master//CONTRIBUTING.md#breaking-change-policy), when their related functionality is no longer required. While frozen APIs have their wire format compatibility preserved, we reserve the right to change proto namespaces, file locations and nesting relationships, which may cause breaking code changes. We will aim to minimize the churn here.

Protos tagged *draft*, meaning that they are near finalized, are likely to be at least partially implemented in Envoy but may have wire format breaking changes made prior to freezing.

Protos tagged *experimental*, have the same caveats as draft protos and may have have major changes made prior to Envoy implementation and freezing.

The current open v2 API issues are tracked [here](https://github.com/envoyproxy/envoy/issues?q=is%3Aopen+is%3Aissue+label%3A%22v2+API%22).