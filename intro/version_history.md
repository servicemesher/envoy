# 历史版本

## 1.7.0 (Pending)

- access log: ability to log response trailers
- access log: ability to format START_TIME
- access log: added DYNAMIC_METADATA [access log formatter](../configuration/access_log.md#config-access-log-format).
- access log: added [HeaderFilter](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-headerfilter) to filter logs based on request headers
- admin: added [`GET /config_dump`](../operations/admin.md#get--config_dump) for dumping the current configuration and associated xDS version information (if applicable).
- admin: added [`GET /stats/prometheus`](../operations/admin.md#get--stats-prometheus) as an alternative endpoint for getting stats in prometheus format.
- admin: added [/runtime_modify endpoint](../operations/admin.md#operations-admin-interface-runtime-modify) to add or change runtime values
- admin: mutations must be sent as POSTs, rather than GETs. Mutations include:[`POST /cpuprofiler`](../operations/admin.md#post--cpuprofiler), [`POST /healthcheck/fail`](../operations/admin.md#post--healthcheck-fail), [`POST /healthcheck/ok`](../operations/admin.md#post--healthcheck-ok), [`POST /logging`](../operations/admin.md#post--logging), [`POST /quitquitquit`](../operations/admin.md#post--quitquitquit), [`POST /reset_counters`](../operations/admin.md#post--reset_counters),[`POST /runtime_modify?key1=value1&key2=value2&keyN=valueN`](../operations/admin.md#post--runtime_modify?key1=value1&key2=value2&keyN=valueN),
- admin: removed /routes endpoint; route configs can now be found at the [/config_dump endpoint](../operations/admin.md#operations-admin-interface-config-dump).
- buffer filter: the buffer filter can be optionally [disabled](../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-field-config-filter-http-buffer-v2-bufferperroute-disabled) or [overridden](../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-field-config-filter-http-buffer-v2-bufferperroute-buffer) with route-local configuration.
- cli: added –config-yaml flag to the Envoy binary. When set its value is interpreted as a yaml representation of the bootstrap config and overrides –config-path.
- cluster: Add [option](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-close-connections-on-host-health-failure) to close tcp_proxy upstream connections when health checks fail.
- cluster: Add [option](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-drain-connections-on-host-removal) to drain connections from hosts after they are removed from service discovery, regardless of health status.
- cluster: fixed bug preventing the deletion of all endpoints in a priority
- health check: added ability to set [additional HTTP headers](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-httphealthcheck-request-headers-to-add) for HTTP health check.
- health check: added support for EDS delivered [endpoint health status](../api-v2/api/v2/endpoint/endpoint.proto.md#envoy-api-field-endpoint-lbendpoint-health-status).
- health check: added interval overrides for health state transitions from [healthy to unhealthy](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-unhealthy-edge-interval), [unhealthy to healthy](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-healthy-edge-interval) and for subsequent checks on [unhealthy hosts](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-unhealthy-interval).
- health check http filter: added [generic header matching](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-headers) to trigger health check response. Deprecated the [endpoint option](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-endpoint).
- health check: added support for [custom health check](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-custom-health-check).
- http: filters can now optionally support [virtual host](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-virtualhost-per-filter-config), [route](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-per-filter-config), and [weighted cluster](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-weightedcluster-clusterweight-per-filter-config) local configuration.
- http: added the ability to pass DNS type Subject Alternative Names of the client certificate in the [x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) header.
- listeners: added [tcp_fast_open_queue_length](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-tcp-fast-open-queue-length) option.
- load balancing: added [weighted round robin](arch_overview/load_balancing.md#arch-overview-load-balancing-types-round-robin) support. The round robin scheduler now respects endpoint weights and also has improved fidelity across picks.
- load balancer: [Locality weighted load balancing](arch_overview/load_balancing.md#arch-overview-load-balancer-subsets) is now supported.
- load balancer: ability to configure zone aware load balancer settings [through the API](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-commonlbconfig-zone-aware-lb-config)
- logger: added the ability to optionally set the log format via the [`--log-format`](../operations/cli.md#cmdoption-log-format) option.
- logger: all [logging levels](../operations/admin.md#operations-admin-interface-logging) can be configured at run-time: trace debug info warning error critical.
- sockets: added [capture transport socket extension](../operations/traffic_capture.md#operations-traffic-capture) to support recording plain text traffic and PCAP generation.
- sockets: added IP_FREEBIND socket option support for [listeners](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-freebind) and upstream connections via[cluster manager wide](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-field-config-bootstrap-v2-clustermanager-upstream-bind-config) and [cluster specific](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-upstream-bind-config) options.
- sockets: added IP_TRANSPARENT socket option support for [listeners](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-transparent).
- sockets: added SO_KEEPALIVE socket option for upstream connections [per cluster](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-upstream-connection-options).
- stats: added support for histograms.
- stats: added [option to configure the statsd prefix](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-field-config-metrics-v2-statsdsink-prefix)
- tls: removed support for legacy SHA-2 CBC cipher suites.
- tracing: the sampling decision is now delegated to the tracers, allowing the tracer to decide when and if to use it. For example, if the [x-b3-sampled](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled) header is supplied with the client request, its value will override any sampling decision made by the Envoy proxy.
- websocket: support configuring [idle_timeout and max_connect_attempts](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-websocket-config).

## 1.6.0 (March 20, 2018)

- access log: added DOWNSTREAM_REMOTE_ADDRESS, DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT, and DOWNSTREAM_LOCAL_ADDRESS [access log formatters](../configuration/access_log.md#config-access-log-format). DOWNSTREAM_ADDRESS access log formatter has been deprecated.
- access log: added less than or equal (LE) [comparison filter](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-comparisonfilter).
- access log: added configuration to [runtime filter](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-runtimefilter) to set default sampling rate, divisor, and whether to use independent randomness or not.
- admin: added [/runtime](../operations/admin.md#operations-admin-interface-runtime) admin endpoint to read the current runtime values.
- build: added support for [building Envoy with exported symbols](https://github.com/envoyproxy/envoy/blob/master/bazel#enabling-optional-features). This change allows scripts loaded with the Lua filter to load shared object libraries such as those installed via [LuaRocks](https://luarocks.org/).
- config: added support for sending error details as [grpc.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto) in [DiscoveryRequest](../api-v2/api/v2/discovery.proto.md#envoy-api-msg-discoveryrequest).
- config: added support for [inline delivery](../api-v2/api/v2/core/base.proto.md#envoy-api-msg-core-datasource) of TLS certificates and private keys.
- config: added restrictions for the backing [config sources](../api-v2/api/v2/core/config_source.proto.md#envoy-api-msg-core-configsource) of xDS resources. For filesystem based xDS the file must exist at configuration time. For cluster based xDS the backing cluster must be statically defined and be of non-EDS type.
- grpc: the Google gRPC C++ library client is now supported as specified in the [gRPC services overview](arch_overview/grpc.md#arch-overview-grpc-services) and [GrpcService](../api-v2/api/v2/core/grpc_service.proto.md#envoy-api-msg-core-grpcservice).
- grpc-json: Added support for [inline descriptors](../api-v2/config/filter/http/transcoder/v2/transcoder.proto.md#envoy-api-field-config-filter-http-transcoder-v2-grpcjsontranscoder-proto-descriptor-bin).
- health check: added [gRPC health check](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-grpc-health-check) based on [grpc.health.v1.Health](https://github.com/grpc/grpc/blob/master/src/proto/grpc/health/v1/health.proto) service.
- health check: added ability to set [host header value](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-httphealthcheck-host) for http health check.
- health check: extended the health check filter to support computation of the health check response based on the [percentage of healthy servers in upstream clusters](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-cluster-min-healthy-percentages).
- health check: added setting for [no-traffic interval](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-no-traffic-interval).
- http : added idle timeout for [upstream http connections](../api-v2/api/v2/core/protocol.proto.md#envoy-api-field-core-httpprotocoloptions-idle-timeout).
- http: added support for [proxying 100-Continue responses](../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.md#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-proxy-100-continue).
- http: added the ability to pass a URL encoded PEM encoded peer certificate in the [x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) header.
- http: added support for trusting additional hops in the [x-forwarded-for](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for) request header.
- http: added support for [incoming HTTP/1.0](../api-v2/api/v2/core/protocol.proto.md#envoy-api-field-core-http1protocoloptions-accept-http-10).
- hot restart: added SIGTERM propagation to children to [hot-restarter.py](../operations/hot_restarter.md#operations-hot-restarter), which enables using it as a parent of containers.
- ip tagging: added [HTTP IP Tagging filter](../configuration/http_filters/ip_tagging_filter.md#config-http-filters-ip-tagging).
- listeners: added support for [listening for both IPv4 and IPv6](../api-v2/api/v2/core/address.proto.md#envoy-api-field-core-socketaddress-ipv4-compat) when binding to ::.
- listeners: added support for listening on [UNIX domain sockets](../api-v2/api/v2/core/address.proto.md#envoy-api-field-core-address-pipe).
- listeners: added support for [abstract unix domain sockets](../api-v2/api/v2/core/address.proto.md#envoy-api-msg-core-pipe) on Linux. The abstract namespace can be used by prepending ‘@’ to a socket path.
- load balancer: added cluster configuration for [healthy panic threshold](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-commonlbconfig-healthy-panic-threshold) percentage.
- load balancer: added [Maglev](arch_overview/load_balancing.md#arch-overview-load-balancing-types-maglev) consistent hash load balancer.
- load balancer: added support for [LocalityLbEndpoints](../api-v2/api/v2/endpoint/endpoint.proto.md#envoy-api-msg-endpoint-localitylbendpoints) priorities.
- lua: added headers [replace()](../configuration/http_filters/lua_filter.md#config-http-filters-lua-header-wrapper) API.
- lua: extended to support [metadata object](../configuration/http_filters/lua_filter.md#config-http-filters-lua-metadata-wrapper) API.
- redis: added local PING support to the [Redis filter](arch_overview/redis.md#arch-overview-redis).
- redis: added GEORADIUS_RO and GEORADIUSBYMEMBER_RO to the [Redis command splitter](arch_overview/redis.md#arch-overview-redis) whitelist.
- router: added DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT, DOWNSTREAM_LOCAL_ADDRESS, DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT, PROTOCOL, and UPSTREAM_METADATA [header formatters](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers). The CLIENT_IP header formatter has been deprecated.
- router: added gateway-error [retry-on](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-on) policy.
- router: added support for route matching based on [URL query string parameters](../api-v2/api/v2/route/route.proto.md#envoy-api-msg-route-queryparametermatcher).
- router: added support for more granular weighted cluster routing by allowing the [total_weight](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-weightedcluster-total-weight)to be specified in configuration.
- router: added support for [custom request/response headers](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) with mixed static and dynamic values.
- router: added support for [direct responses](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-direct-response). I.e., sending a preconfigured HTTP response without proxying anywhere.
- router: added support for [HTTPS redirects](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-https-redirect) on specific routes.
- router: added support for [prefix_rewrite](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-prefix-rewrite) for redirects.
- router: added support for [stripping the query string](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-strip-query) for redirects.
- router: added support for downstream request/upstream response [header manipulation](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) in [weighted cluster](../api-v2/api/v2/route/route.proto.md#envoy-api-msg-route-weightedcluster).
- router: added support for [range based header matching](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-headermatcher-range-match) for request routing.
- squash: added support for the [Squash microservices debugger](../configuration/http_filters/squash_filter.md#config-http-filters-squash). Allows debugging an incoming request to a microservice in the mesh.
- stats: added metrics service API implementation.
- stats: added native [DogStatsd](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-msg-config-metrics-v2-dogstatsdsink) support.
- stats: added support for [fixed stats tag values](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-field-config-metrics-v2-tagspecifier-fixed-value) which will be added to all metrics.
- tcp proxy: added support for specifying a [metadata matcher](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-metadata-match) for upstream clusters in the tcp filter.
- tcp proxy: improved TCP proxy to correctly proxy TCP half-close.
- tcp proxy: added [idle timeout](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-idle-timeout).
- tcp proxy: access logs now bring an IP address without a port when using DOWNSTREAM_ADDRESS. Use [DOWNSTREAM_REMOTE_ADDRESS](../configuration/access_log.md#config-access-log-format) instead.
- tracing: added support for dynamically loading an [OpenTracing tracer](../api-v2/config/trace/v2/trace.proto.md#envoy-api-msg-config-trace-v2-dynamicotconfig).
- tracing: when using the Zipkin tracer, it is now possible for clients to specify the sampling decision (using the [x-b3-sampled](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled) header) and have the decision propagated through to subsequently invoked services.
- tracing: when using the Zipkin tracer, it is no longer necessary to propagate the [x-ot-span-context](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-ot-span-context) header. See more on trace context propagation [here](arch_overview/tracing.md#arch-overview-tracing).
- transport sockets: added transport socket interface to allow custom implementations of transport sockets. A transport socket provides read and write logic with buffer encryption and decryption (if applicable). The existing TLS implementation has been refactored with the interface.
- upstream: added support for specifying an [alternate stats name](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-alt-stat-name) while emitting stats for clusters.
- Many small bug fixes and performance improvements not listed.

## 1.5.0 (December 4, 2017)

- access log: added fields for [UPSTREAM_LOCAL_ADDRESS and DOWNSTREAM_ADDRESS](../configuration/access_log.md#config-access-log-format).
- admin: added [JSON output](../operations/admin.md#operations-admin-interface-stats) for stats admin endpoint.
- admin: added basic [Prometheus output](../operations/admin.md#operations-admin-interface-stats) for stats admin endpoint. Histograms are not currently output.
- admin: added `version_info` to the [/clusters admin endpoint](../operations/admin.md#operations-admin-interface-clusters).
- config: the [v2 API](../configuration/overview/v2_overview.md#config-overview-v2) is now considered production ready.
- config: added [`--v2-config-only`](../operations/cli.md#cmdoption-v2-config-only) CLI flag.
- cors: added [CORS filter](../configuration/http_filters/cors_filter.md#config-http-filters-cors).
- health check: added [x-envoy-immediate-health-check-fail](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) header support.
- health check: added [reuse_connection](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-reuse-connection) option.
- http: added [per-listener stats](../configuration/http_conn_man/stats.md#config-http-conn-man-stats-per-listener).
- http: end-to-end HTTP flow control is now complete across both connections, streams, and filters.
- load balancer: added [subset load balancer](arch_overview/load_balancing.md#arch-overview-load-balancer-subsets).
- load balancer: added ring size and hash [configuration options](../api-v2/api/v2/cds.proto.md#envoy-api-msg-cluster-ringhashlbconfig). This used to be configurable via runtime. The runtime configuration was deleted without deprecation as we are fairly certain no one is using it.
- log: added the ability to optionally log to a file instead of stderr via the [`--log-path`](../operations/cli.md#cmdoption-log-path) option.
- listeners: added [drain_type](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-drain-type) option.
- lua: added experimental [Lua filter](../configuration/http_filters/lua_filter.md#config-http-filters-lua).
- mongo filter: added [fault injection](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy-fault-injection).
- mongo filter: added [“drain close”](arch_overview/draining.md#arch-overview-draining) support.
- outlier detection: added [HTTP gateway failure type](arch_overview/outlier.md#arch-overview-outlier-detection). See [DEPRECATED.md](https://github.com/envoyproxy/envoy/blob/master/DEPRECATED.md#version-150) for outlier detection stats deprecations in this release.
- redis: the [redis proxy filter](../configuration/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy) is now considered production ready.
- redis: added [“drain close”](arch_overview/draining.md#arch-overview-draining) functionality.
- router: added [x-envoy-overloaded](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded) support.
- router: added [regex](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routematch-regex) route matching.
- router: added [custom request headers](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) for upstream requests.
- router: added [downstream IP hashing](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-hashpolicy-connection-properties) for HTTP ketama routing.
- router: added [cookie hashing](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-hashpolicy-cookie).
- router: added [start_child_span](../api-v2/config/filter/http/router/v2/router.proto.md#envoy-api-field-config-filter-http-router-v2-router-start-child-span) option to create child span for egress calls.
- router: added optional [upstream logs](../api-v2/config/filter/http/router/v2/router.proto.md#envoy-api-field-config-filter-http-router-v2-router-upstream-log).
- router: added complete [custom append/override/remove support](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) of request/response headers.
- router: added support to [specify response code during redirect](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-response-code).
- router: added [configuration](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-cluster-not-found-response-code) to return either a 404 or 503 if the upstream cluster does not exist.
- runtime: added [comment capability](../configuration/runtime.md#config-runtime-comments).
- server: change default log level ([`-l`](../operations/cli.md#cmdoption-l)) to info.
- stats: maximum stat/name sizes and maximum number of stats are now variable via the[`--max-obj-name-len`](../operations/cli.md#cmdoption-max-obj-name-len) and [`--max-stats`](../operations/cli.md#cmdoption-max-stats) options.
- tcp proxy: added [access logging](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-access-log).
- tcp proxy: added [configurable connect retries](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-max-connect-attempts).
- tcp proxy: enable use of [outlier detector](arch_overview/outlier.md#arch-overview-outlier-detection).
- tls: added [SNI support](../faq/sni.md#faq-how-to-setup-sni).
- tls: added support for specifying [TLS session ticket keys](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-downstreamtlscontext-session-ticket-keys).
- tls: allow configuration of the [min](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-tlsparameters-tls-minimum-protocol-version) and [max](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-tlsparameters-tls-maximum-protocol-version) TLS protocol versions.
- tracing: added [custom trace span decorators](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-decorator).
- Many small bug fixes and performance improvements not listed.

## 1.4.0 (August 24, 2017)

- macOS is [now supported](https://github.com/envoyproxy/envoy/blob/master//bazel#quick-start-bazel-build-for-developers). (A few features are missing such as hot restart and original destination routing).
- YAML is now directly supported for [config files](../configuration/overview/v1_overview.md#config-overview-v1).
- Added /routes admin endpoint.
- End-to-end flow control is now supported for TCP proxy, HTTP/1, and HTTP/2. HTTP flow control that includes filter buffering is incomplete and will be implemented in 1.5.0.
- Log verbosity [compile time flag](https://github.com/envoyproxy/envoy/blob/master//bazel#log-verbosity) added.
- Hot restart [compile time flag](https://github.com/envoyproxy/envoy/blob/master//bazel#hot-restart) added.
- Original destination [cluster](arch_overview/service_discovery.md#arch-overview-service-discovery-types-original-destination) and [load balancer](arch_overview/load_balancing.md#arch-overview-load-balancing-types-original-destination) added.
- [WebSocket](arch_overview/websocket.md#arch-overview-websocket) is now supported.
- Virtual cluster priorities have been hard removed without deprecation as we are reasonably sure no one is using this feature.
- Route [validate_clusters](../api-v1/route_config/route_config.md#config-http-conn-man-route-table-validate-clusters) option added.
- [x-envoy-downstream-service-node](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-downstream-service-node) header added.
- [x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) header added.
- Initial HTTP/1 forward proxy support for [absolute URLs](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-http1-settings) has been added.
- HTTP/2 codec settings are now [configurable](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-http2-settings).
- gRPC/JSON transcoder [filter](../configuration/http_filters/grpc_json_transcoder_filter.md#config-http-filters-grpc-json-transcoder) added.
- gRPC web [filter](../configuration/http_filters/grpc_web_filter.md#config-http-filters-grpc-web) added.
- Configurable timeout for the rate limit service call in the [network](../configuration/network_filters/rate_limit_filter.md#config-network-filters-rate-limit) and [HTTP](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) rate limit filters.
- [x-envoy-retry-grpc-on](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-grpc-on) header added.
- [LDS API](arch_overview/dynamic_configuration.md#arch-overview-dynamic-config-lds) added.
- TLS [require_client_certificate](../api-v1/listeners/listeners.md#config-listener-ssl-context-require-client-certificate) option added.
- [Configuration check tool](../install/tools/config_load_check_tool.md#install-tools-config-load-check-tool) added.
- [JSON schema check tool](../install/tools/schema_validator_check_tool.md#install-tools-schema-validator-check-tool) added.
- Config validation mode added via the [`--mode`](../operations/cli.md#cmdoption-mode) option.
- [`--local-address-ip-version`](../operations/cli.md#cmdoption-local-address-ip-version) option added.
- IPv6 support is now complete.
- UDP [statsd_ip_address](../configuration/overview/v1_overview.md#config-overview-statsd-udp-ip-address) option added.
- Per-cluster [DNS resolvers](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-dns-resolvers) added.
- [Fault filter](../configuration/http_filters/fault_filter.md#config-http-filters-fault-injection) enhancements and fixes.
- Several features are [deprecated as of the 1.4.0 release](https://github.com/envoyproxy/envoy/blob/master//DEPRECATED.md#version-140). They will be removed at the beginning of the 1.5.0 release cycle. We explicitly call out that the HttpFilterConfigFactory filter API has been deprecated in favor of NamedHttpFilterConfigFactory.
- Many small bug fixes and performance improvements not listed.

## 1.3.0 (May 17, 2017)

- As of this release, we now have an official [breaking change policy](https://github.com/envoyproxy/envoy/blob/master//CONTRIBUTING.md#breaking-change-policy). Note that there are numerous breaking configuration changes in this release. They are not listed here. Future releases will adhere to the policy and have clear documentation on deprecations and changes.
- Bazel is now the canonical build system (replacing CMake). There have been a huge number of changes to the development/build/test flow. See [/bazel/README.md](https://github.com/envoyproxy/envoy/blob/master//bazel/README.md) and [/ci/README.md](https://github.com/envoyproxy/envoy/blob/master//ci/README.md) for more information.
- [Outlier detection](arch_overview/outlier.md#arch-overview-outlier-detection) has been expanded to include success rate variance, and all parameters are now configurable in both runtime and in the JSON configuration.
- TCP level [listener](../api-v1/listeners/listeners.md#config-listeners-per-connection-buffer-limit-bytes) and [cluster](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-per-connection-buffer-limit-bytes) connections now have configurable receive buffer limits at which point connection level back pressure is applied. Full end to end flow control will be available in a future release.
- [Redis health checking](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc) has been added as an active health check type. Full Redis support will be documented/supported in 1.4.0.
- [TCP health checking](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-tcp-health-checking) now supports a “connect only” mode that only checks if the remote server can be connected to without writing/reading any data.
- [BoringSSL](https://boringssl.googlesource.com/boringssl) is now the only supported TLS provider. The default cipher suites and ECDH curves have been updated with more modern defaults for both [listener](../api-v1/listeners/listeners.md#config-listener-ssl-context) and [cluster](../api-v1/cluster_manager/cluster_ssl.md#config-cluster-manager-cluster-ssl) connections.
- The header value match [rate limit action](../api-v1/route_config/rate_limits.md#config-http-conn-man-route-table-rate-limit-actions) has been expanded to include an *expect match*parameter.
- Route level HTTP rate limit configurations now do not inherit the virtual host level configurations by default. The [include_vh_rate_limits](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-include-vh) to inherit the virtual host level options if desired.
- HTTP routes can now add request headers on a per route and per virtual host basis via the[request_headers_to_add](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) option.
- The [example configurations](../install/ref_configs.md#install-ref-configs) have been refreshed to demonstrate the latest features.
- [per_try_timeout_ms](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-retry) can now be configured in a route’s retry policy in addition to via the [x-envoy-upstream-rq-per-try-timeout-ms](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-per-try-timeout-ms) HTTP header.
- [HTTP virtual host matching](../api-v1/route_config/vhost.md#config-http-conn-man-route-table-vhost) now includes support for prefix wildcard domains (e.g., *.lyft.com).
- The default for tracing random sampling has been changed to 100% and is still configurable in[runtime](../configuration/http_conn_man/runtime.md#config-http-conn-man-runtime).
- [HTTP tracing configuration](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-tracing) has been extended to allow tags to be populated from arbitrary HTTP headers.
- The [HTTP rate limit filter](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) can now be applied to internal, external, or all requests via the request_type option.
- [Listener binding](../configuration/listeners/listeners.md#config-listeners) now requires specifying an address field. This can be used to bind a listener to both a specific address as well as a port.
- The [MongoDB filter](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy) now emits a stat for queries that do not have $maxTimeMS set.
- The [MongoDB filter](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy) now emits logs that are fully valid JSON.
- The CPU profiler output path is now [configurable](../api-v1/admin.md#config-admin-v1).
- A [watchdog system](../configuration/overview/v1_overview.md#config-overview-v1) has been added that can kill the server if a deadlock is detected.
- A [route table checking tool](../install/tools/route_table_check_tool.md#install-tools-route-table-check-tool) has been added that can be used to test route tables before use.
- We have added an [example repo](../extending/extending.md#extending) that shows how to compile/link a custom filter.
- Added additional cluster wide information related to outlier detection to the [/clusters admin endpoint](../operations/admin.md#operations-admin-interface).
- Multiple SANs can now be verified via the [verify_subject_alt_name](../api-v1/listeners/listeners.md#config-listener-ssl-context) setting. Additionally, URI type SANs can be verified.
- HTTP filters can now be passed [opaque configuration](../api-v1/route_config/route.md#config-http-conn-man-route-table-opaque-config) specified on a per route basis.
- By default Envoy now has a built in crash handler that will print a back trace. This behavior can be disabled if desired via the `--define=signal_trace=disabled` Bazel option.
- Zipkin has been added as a supported [tracing provider](arch_overview/tracing.md#arch-overview-tracing).
- Numerous small changes and fixes not listed here.

## 1.2.0 (March 7, 2017)

- [Cluster discovery service (CDS) API](../configuration/cluster_manager/cds.md#config-cluster-manager-cds).
- [Outlier detection](arch_overview/outlier.md#arch-overview-outlier-detection) (passive health checking).
- Envoy configuration is now checked against a [JSON schema](../configuration/overview/v1_overview.md#config-overview-v1).
- [Ring hash](arch_overview/load_balancing.md#arch-overview-load-balancing-types) consistent load balancer, as well as HTTP consistent hash routing [based on a policy](../api-v1/route_config/route.md#config-http-conn-man-route-table-hash-policy).
- Vastly [enhanced global rate limit configuration](arch_overview/global_rate_limiting.md#arch-overview-rate-limit) via the HTTP rate limiting filter.
- HTTP routing to a cluster [retrieved from a header](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-cluster-header).
- [Weighted cluster](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-config-weighted-clusters) HTTP routing.
- [Auto host rewrite](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-auto-host-rewrite) during HTTP routing.
- [Regex header matching](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers) during HTTP routing.
- HTTP access log [runtime filter](../api-v1/access_log.md#config-http-con-manager-access-log-filters-runtime-v1).
- LightStep tracer [parent/child span association](arch_overview/tracing.md#arch-overview-tracing).
- [Route discovery service (RDS) API](../configuration/http_conn_man/rds.md#config-http-conn-man-rds).
- HTTP router [x-envoy-upstream-rq-timeout-alt-response header](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-timeout-alt-response) support.
- *use_original_dst* and *bind_to_port* [listener options](../configuration/listeners/listeners.md#config-listeners) (useful for iptables based transparent proxy support).
- TCP proxy filter [route table support](../configuration/network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy).
- Configurable [stats flush interval](../configuration/overview/v1_overview.md#config-overview-stats-flush-interval-ms).
- Various [third party library upgrades](../install/building.md#install-requirements), including using BoringSSL as the default SSL provider.
- No longer maintain closed HTTP/2 streams for priority calculations. Leads to substantial memory savings for large meshes.
- Numerous small changes and fixes not listed here.

## 1.1.0 (November 30, 2016)

- Switch from Jannson to RapidJSON for our JSON library (allowing for a configuration schema in 1.2.0).
- Upgrade [recommended version](../install/building.md#install-requirements) of various other libraries.
- [Configurable DNS refresh rate](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-dns-refresh-rate-ms) for DNS service discovery types.
- Upstream circuit breaker configuration can be [overridden via runtime](../configuration/cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime).
- [Zone aware routing support](arch_overview/load_balancing.md#arch-overview-load-balancing-zone-aware-routing).
- Generic [header matching routing rule](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers).
- HTTP/2 [graceful connection draining](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-drain-timeout-ms) (double GOAWAY).
- DynamoDB filter [per shard statistics](../configuration/http_filters/dynamodb_filter.md#config-http-filters-dynamo) (pre-release AWS feature).
- Initial release of the [fault injection HTTP filter](../configuration/http_filters/fault_filter.md#config-http-filters-fault-injection).
- HTTP [rate limit filter](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) enhancements (note that the configuration for HTTP rate limiting is going to be overhauled in 1.2.0).
- Added [refused-stream retry policy](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-on).
- Multiple [priority queues](arch_overview/http_routing.md#arch-overview-http-routing-priority) for upstream clusters (configurable on a per route basis, with separate connection pools, circuit breakers, etc.).
- Added max connection circuit breaking to the [TCP proxy filter](arch_overview/tcp_proxy.md#arch-overview-tcp-proxy).
- Added [CLI](../operations/cli.md#operations-cli) options for setting the logging file flush interval as well as the drain/shutdown time during hot restart.
- A very large number of performance enhancements for core HTTP/TCP proxy flows as well as a few new configuration flags to allow disabling expensive features if they are not needed (specifically request ID generation and dynamic response code stats).
- Support Mongo 3.2 in the [Mongo sniffing filter](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy).
- Lots of other small fixes and enhancements not listed.

## 1.0.0 (September 12, 2016)

Initial open source release.