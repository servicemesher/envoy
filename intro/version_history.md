# 历史版本

## 1.7.0 (日期待定)

- 访问日志：能够将响应尾存入日志
- 访问日志：能够格式化 START_TIME
- 访问日志：添加了 DYNAMIC_METADATA [访问日志格式化器](../configuration/access_log.md#config-access-log-format)。
- 访问日志：添加了 [HeaderFilter](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-headerfilter) 以基于请求头过滤日志
- 管理: 添加了 [`GET /config_dump`](../operations/admin.md#get--config_dump) 用于保存当前配置和相关 xDS 版本信息 (如果可用)。
- 管理: 添加了 [`GET /stats/prometheus`](../operations/admin.md#get--stats-prometheus) ，作为以 prometheus 格式获得状态的一个替代的端点。
- 管理: 添加了 [/runtime_modify 端点](../operations/admin.md#operations-admin-interface-runtime-modify) 以添加或改变运行时值
- 管理: 突变必须作为 POST 而不是 GET 发送。突变包括:[`POST /cpuprofiler`](../operations/admin.md#post--cpuprofiler), [`POST /healthcheck/fail`](../operations/admin.md#post--healthcheck-fail), [`POST /healthcheck/ok`](../operations/admin.md#post--healthcheck-ok), [`POST /logging`](../operations/admin.md#post--logging), [`POST /quitquitquit`](../operations/admin.md#post--quitquitquit), [`POST /reset_counters`](../operations/admin.md#post--reset_counters),[`POST /runtime_modify?key1=value1&key2=value2&keyN=valueN`](../operations/admin.md#post--runtime_modify?key1=value1&key2=value2&keyN=valueN)。
- 管理: 移除了 /routes 端点；路由配置现在可以在 [/config_dump 端点](../operations/admin.md#operations-admin-interface-config-dump)找到。
- 缓存过滤器:缓存过滤可以被可选地用路由本地配置[关闭](../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-field-config-filter-http-buffer-v2-bufferperroute-disabled) 或者 [被撤销](../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-field-config-filter-http-buffer-v2-bufferperroute-buffer)。
- cli: 添加了 –config-yaml 标志到 Envoy 二进制文件。当将其值被解释为引导配置和撤销  –config-path 的一个 yaml 表示。
- 集群: 添加 [选项](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-close-connections-on-host-health-failure) 在健康状况检查失败时关闭 tcp_proxy 上游连接。
- 集群: 添加 [选项](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-drain-connections-on-host-removal) 在来自主机的连接被从服务发现中移除后耗尽它们，与健康状态无关。
- 集群: 修复了阻止删除同一优先级的所有端点的缺陷。
- 健康状况检查: 添加了设置[额外 HTTP 头](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-httphealthcheck-request-headers-to-add)用于 HTTP 健康状况检查的能力。
- 健康状况检查: 添加了支持 EDS 发送的 [端点健康状况](../api-v2/api/v2/endpoint/endpoint.proto.md#envoy-api-field-endpoint-lbendpoint-health-status)。
- 健康状况检查: 添加了健康状态转换的区间撤销，[健康到不健康](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-unhealthy-edge-interval)， [不健康到健康](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-healthy-edge-interval) 并后续对[不健康的主机](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-unhealthy-interval)的检查。
- 健康状况检查 http 过滤器: 添加了 [通用头匹配](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-headers) 以触发健康状况检查响应。弃用了[端点选项](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-endpoint)。
- 健康状况检查: 添加了支持 [定制健康状况检查](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-custom-health-check)。
- http: 过滤器现在可以可选地支持[虚拟主机](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-virtualhost-per-filter-config)，[路由](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-per-filter-config)，和 [加权集群](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-weightedcluster-clusterweight-per-filter-config) 本地配置。
- http: 添加了传送在 [x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) 头中的客户端证书 DNS 类型主题替代名 (Subject Alternative Names) 的能力。
- 监听器: 添加了 [tcp_fast_open_queue_length](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-tcp-fast-open-queue-length) 选项。
- 负载均衡器: 添加了 [加权轮询](arch_overview/load_balancing.md#arch-overview-load-balancing-types-round-robin) 支持。轮询调度群现在关心端点权重并且也提升了跨选择的保真度。
- 负载均衡器: [本地化加权负载均衡](arch_overview/load_balancing.md#arch-overview-load-balancer-subsets)现在被支持。
- 负载均衡器: 能够[通过 API](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-commonlbconfig-zone-aware-lb-config) 设置配置域感知负载均衡器。
- logger: 添加了通过 [`--log-format`](../operations/cli.md#cmdoption-log-format) 选项可选地设置日志格式的能力。
- logger: 所有 [日志级别](../operations/admin.md#operations-admin-interface-logging) 可以在运行时配置: trace debug info warning error critical。
- sockets: 添加了[捕捉传输套接字扩展](../operations/traffic_capture.md#operations-traffic-capture) 以支持记录纯文本流量和 PCAP 生成。
- sockets: 添加了 IP_FREEBIND 套接字选项，通过[集群管理器范围](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-field-config-bootstrap-v2-clustermanager-upstream-bind-config) 和 [集群特定的](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-upstream-bind-config) 选项，支持 [监听器](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-freebind) 和上游连接。
- sockets: 添加了 IP_TRANSPARENT 套接字选项支持 [监听器](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-transparent)。
- sockets: 添加了 SO_KEEPALIVE 套接字选项用于[每个集群](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-upstream-connection-options)的上游连接。
- stats: 添加了支持柱状图。
- stats: 添加了[选项配置 statsd 前缀](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-field-config-metrics-v2-statsdsink-prefix)。
- tls: 移除了遗留的 SHA-2 CBC 密码套件。
- 追踪: 采样决策现在授予了追踪器，允许追踪器决定何时以及是否使用它。例如，如果 [x-b3-sampled](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled) 头被以客户端请求提供，它的值会覆盖由 Envoy 代理做的任何决策。
- websocket: 支持配置[空闲和 and max_connect_attempts](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-websocket-config)。

## 1.6.0 (3月20日, 2018)

- 访问日志：添加了 DOWNSTREAM_REMOTE_ADDRESS, DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT, 和 DOWNSTREAM_LOCAL_ADDRESS [访问日志格式化器](../configuration/access_log.md#config-access-log-format).。DOWNSTREAM_ADDRESS 访问日志格式化器被弃用。
- 访问日志：添加了小于或等于 (LE) [比较过滤器](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-comparisonfilter)。
- 访问日志：添加了对[运行时过滤器](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-runtimefilter)的配置，用于设置默认采样速率，因子，和是否使用独立随机。
- 管理: 添加了 [/runtime](../operations/admin.md#operations-admin-interface-runtime) admin 端点读取当前运行时的值。
- 构建: 添加了支持[用输出的符合构建 Envoy](https://github.com/envoyproxy/envoy/blob/master/bazel#enabling-optional-features)。这个变化运行脚本用 Lua 过滤器加载，以加载共享对象库，如那些通过[LuaRocks](https://luarocks.org/)安装的库。
- 配置: 添加了支持在 [DiscoveryRequest](../api-v2/api/v2/discovery.proto.md#envoy-api-msg-discoveryrequest)中发送错误细节，如 [grpc.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)。
- 配置: 添加了支持[内置交付](../api-v2/api/v2/core/base.proto.md#envoy-api-msg-core-datasource) TLS 证书和私钥。
- 配置: 添加了xDS 源恢复[配置源](../api-v2/api/v2/core/config_source.proto.md#envoy-api-msg-core-configsource)的限制。对基于 xDS 的文件系统，文件必须在配置时存在。对于基于 xDS 的集群，恢复集群必须被静态定义并有非 EDS 类型。
- grpc: 谷歌 gRPC C++ 库客户端现在作为在 [gRPC 服务概览](arch_overview/grpc.md#arch-overview-grpc-services) 和 [GrpcService] (../api-v2/api/v2/core/grpc_service.proto.md#envoy-api-msg-core-grpcservice)中指定支持。
- grpc-json: 添加了支持[内置描述符](../api-v2/config/filter/http/transcoder/v2/transcoder.proto.md#envoy-api-field-config-filter-http-transcoder-v2-grpcjsontranscoder-proto-descriptor-bin)。
- 健康状况检查: 添加了基于 [grpc.health.v1.Health](https://github.com/grpc/grpc/blob/master/src/proto/grpc/health/v1/health.proto) 服务的 [gRPC 健康状况检查](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-grpc-health-check)。
- 健康状况检查: 添加了设置 [主机头值](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-httphealthcheck-host) 用于健康状况检查的能力。
- 健康状况检查: 扩展了健康状况检查过滤器以支持基于[上游集群中健康的服务器百分比](../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-cluster-min-healthy-percentages)的健康状况检查计算。
- 健康状况检查: 添加了设置 [非流量区间](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-no-traffic-interval)。
- http : 为[上游 http 连接](../api-v2/api/v2/core/protocol.proto.md#envoy-api-field-core-httpprotocoloptions-idle-timeout)添加了空闲超时。
- http: 添加了支持[代理 100-Continue 响应](../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.md#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-proxy-100-continue)。
- http: 添加了在 [x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) 头中传递一个 URL 编码的 PEM 对等证书的能力。
- http: 添加了支持在 [x-forwarded-for](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for) 请求头中信任额外的跳步。
- http: 添加了支持[进来的 HTTP/1.0](../api-v2/api/v2/core/protocol.proto.md#envoy-api-field-core-http1protocoloptions-accept-http-10)。
- 热重启: 添加了 SIGTERM 传递给子进程到 [hot-restarter.py](../operations/hot_restarter.md#operations-hot-restarter)，这使得可以作为容器的一个父进程使用它。
- ip 标签: 添加了 [HTTP IP 标签过滤器](../configuration/http_filters/ip_tagging_filter.md#config-http-filters-ip-tagging)。
- 监听器: 添加了支持在绑定到 :: 时 [监听IPv4 和 IPv6](../api-v2/api/v2/core/address.proto.md#envoy-api-field-core-socketaddress-ipv4-compat)。
- 监听器: 添加了 support for listening on [UNIX domain sockets](../api-v2/api/v2/core/address.proto.md#envoy-api-field-core-address-pipe)。
- 监听器: 添加了Linux 系统上的 [抽象 unix 域套接字 sockets](../api-v2/api/v2/core/address.proto.md#envoy-api-msg-core-pipe) 支持。抽象命名空间可通过在一个套接字前面加上‘@’ 使用。
- 负载均衡器: 添加了对 [健康崩溃阈值](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-commonlbconfig-healthy-panic-threshold) 百分比的集群配置。
- 负载均衡器: 添加了 [Maglev](arch_overview/load_balancing.md#arch-overview-load-balancing-types-maglev) 一致哈希负载均衡器。
- 负载均衡器: 添加了支持 [LocalityLbEndpoints](../api-v2/api/v2/endpoint/endpoint.proto.md#envoy-api-msg-endpoint-localitylbendpoints) priorities。
- lua: 添加了头 [replace()](../configuration/http_filters/lua_filter.md#config-http-filters-lua-header-wrapper) API。
- lua: 扩展支持[元数据对象](../configuration/http_filters/lua_filter.md#config-http-filters-lua-metadata-wrapper) API。
- redis: 添加了对 [Redis 过滤器](arch_overview/redis.md#arch-overview-redis)的本地 PING 支持。
- redis: 添加了 GEORADIUS_RO and GEORADIUSBYMEMBER_RO to the [Redis command splitter](arch_overview/redis.md#arch-overview-redis) whitelist。
- 路由器: 添加了 DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT, DOWNSTREAM_LOCAL_ADDRESS, DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT, PROTOCOL, and UPSTREAM_METADATA [头格式化器](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers)。CLIENT_IP 头格式化器被弃用。
- 路由器: 添加了网关错误 [retry-on](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-on) 策略。
- 路由器: 添加了对基于 [URL 查询字符串参数](../api-v2/api/v2/route/route.proto.md#envoy-api-msg-route-queryparametermatcher)的路由匹配支持。
- 路由器: 添加了通过允许 [total_weight](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-weightedcluster-total-weight) 在配置中指定支持更细粒度的加权集群路由。
- 路由器: 添加了支持带有混合静态和动态值的[定制请求/响应头](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers) 。
- 路由器: 添加了对[直接响应](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-direct-response)的支持。即，发送一个预配置的 HTTP 响应而无需任何代理。
- 路由器: 添加了在特定路由上对[HTTPS 重定向](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-https-redirect)的支持。
- 路由器: 添加了用于重定向的[prefix_rewrite](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-prefix-rewrite)支持。
- 路由器: 添加了对重定向的[剥离查询字符串](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-strip-query)的支持。
- 路由器: 添加了在[加权集群](../api-v2/api/v2/route/route.proto.md#envoy-api-msg-route-weightedcluster)中支持下游请求/上游响应[头操作](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers)。
- 路由器: 添加了支持用于请求路由的[基于范文的头匹配](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-headermatcher-range-match)。
- squash: 添加了支持 [Squash 微服务调试程序](../configuration/http_filters/squash_filter.md#config-http-filters-squash)。允许调试一个对网络中某个微服务的请求。
- stats: 添加了度量服务 API 实现。
- stats: 添加了原生 [DogStatsd](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-msg-config-metrics-v2-dogstatsdsink) 支持。
- stats: 添加了支持 [固定状态标签值](../api-v2/config/metrics/v2/stats.proto.md#envoy-api-field-config-metrics-v2-tagspecifier-fixed-value)，这将被添加到所有度量标准中。
- tcp 代理: 添加了支持在 tcp 过滤器中为上游集群指定一个[元数据匹配器](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-metadata-match)。
- tcp 代理: 改善了 TCP 代理以正确代理 TCP 半关闭。
- tcp 代理: 添加了 [空闲超时](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-idle-timeout)。
- tcp 代理: 访问日志现在在使用 DOWNSTREAM_ADDRESS 时带来一个 IP address 而没有端口。使用 [DOWNSTREAM_REMOTE_ADDRESS](../configuration/access_log.md#config-access-log-format) 替代。
- 追踪: 添加了对动态加载 [OpenTracing 追踪器](../api-v2/config/trace/v2/trace.proto.md#envoy-api-msg-config-trace-v2-dynamicotconfig)的支持。
- 追踪: 当使用 Zipkin 追踪器时，现在客户端程序可以指定取样决策 (使用  [x-b3-sampled](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled) 头) 并将决策传递给后续被启动的服务。
- 追踪: 当使用 Zipkin 追踪器时，不再需要传递 [x-ot-span-context](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-ot-span-context) 头。关于追踪上下文传递的更多内容，参见[这里](arch_overview/tracing.md#arch-overview-tracing)。
- 传输套接字: 添加了传输套接字接口以允许传输套接字的定制实现。一个传输套接字提供带缓冲加密和解密(如果可用)的读和写逻辑。现存的 TLS 实现已被使用该接口重构。
- 上游: 添加了支持在为集群放出状态时指定一个 [替代状态名](../api-v2/api/v2/cds.proto.md#envoy-api-field-cluster-alt-stat-name)。
- 很多小的缺陷修复和性能提高未列出。

## 1.5.0 (12月4日, 2017)

- 访问日志：为 [UPSTREAM_LOCAL_ADDRESS 和 DOWNSTREAM_ADDRESS](../configuration/access_log.md#config-access-log-format)增加了字段。
- 管理: 为stats admin 端点添加 [JSON 输出](../operations/admin.md#operations-admin-interface-stats) 。
- 管理: 为stats admin 端点添加基本 [Prometheus 输出](../operations/admin.md#operations-admin-interface-stats)。当前柱状图不输出。
- 管理: 添加 `version_info` 到 [/clusters admin 端点](../operations/admin.md#operations-admin-interface-clusters)。
- 配置: [v2 API](../configuration/overview/v2_overview.md#config-overview-v2) 现在被认为是可用于生产系统的。
- 配置: 添加了 [`--v2-config-only`](../operations/cli.md#cmdoption-v2-config-only) CLI 标志。
- cors: 添加了 [CORS 过滤器](../configuration/http_filters/cors_filter.md#config-http-filters-cors)。
- 健康状况检查: 添加了 [x-envoy-immediate-health-check-fail](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) 头支持。
- 健康状况检查: 添加了 [reuse_connection](../api-v2/api/v2/core/health_check.proto.md#envoy-api-field-core-healthcheck-reuse-connection) 选项。
- http: 添加了 [单个监听器状态](../configuration/http_conn_man/stats.md#config-http-conn-man-stats-per-listener)。
- http: 端到端 HTTP 流控现在跨连接、流和过滤器是完整的。
- 负载均衡器: 添加了 [子集负载均衡器](arch_overview/load_balancing.md#arch-overview-load-balancer-subsets)。
- 负载均衡器: 添加了 ring size and hash [configuration options](../api-v2/api/v2/cds.proto.md#envoy-api-msg-cluster-ringhashlbconfig). This used to be configurable via runtime. The runtime configuration was deleted without deprecation as we are fairly certain no one is using it。
- 日志: 添加了通过 [`--log-path`](../operations/cli.md#cmdoption-log-path) 选项可选地将日志发到一个文件而不是标准输出的能力。
- 监听器: 添加了 [drain_type](../api-v2/api/v2/lds.proto.md#envoy-api-field-listener-drain-type) option。
- lua: 添加了 试验性 [Lua 过滤器](../configuration/http_filters/lua_filter.md#config-http-filters-lua)。
- mongo 过滤器: 添加了 [故障注入](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy-fault-injection)。
- mongo 过滤器: 添加了 [“耗尽关闭”](arch_overview/draining.md#arch-overview-draining) 支持。
- 异常值检测: 添加了 [HTTP 网关失败类型](arch_overview/outlier.md#arch-overview-outlier-detection)。对这个发布中的异常值检测状态弃用，参见 [DEPRECATED.md](https://github.com/envoyproxy/envoy/blob/master/DEPRECATED.md#version-150)。
- redis: [redis 代理过滤器](../configuration/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy) 现在被认为可用于生产系统。
- redis: 添加了 [“耗尽关闭 close”](arch_overview/draining.md#arch-overview-draining) 功能。
- 路由器: 添加了 [x-envoy-overloaded](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded) 支持。
- 路由器: 添加了 [正则](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routematch-regex) 路由匹配。
- 路由器: 为上游请求添加了 [定制请求头](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers)。
- 路由器: 为 HTTP ketama 路由添加了 [下游 IP 哈希](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-hashpolicy-connection-properties)。
- 路由器: 添加了 [cookie 哈希](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-hashpolicy-cookie)。
- 路由器: 添加了 [start_child_span](../api-v2/config/filter/http/router/v2/router.proto.md#envoy-api-field-config-filter-http-router-v2-router-start-child-span) 选项为呼出调用创建子范围。
- 路由器: 添加了可选的[上游日志](../api-v2/config/filter/http/router/v2/router.proto.md#envoy-api-field-config-filter-http-router-v2-router-upstream-log)。
- 路由器: 添加了请求/响应头的完整的[定制附加/覆盖/移除 支持](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers)。
- 路由器: 添加了对[在重定向期间指定响应代码](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-redirectaction-response-code)的支持。
- 路由器: 添加了 [配置](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-routeaction-cluster-not-found-response-code) ，如果上游集群不存在，返回 404 或 503。
- 运行时: 添加了 [评论能力](../configuration/runtime.md#config-runtime-comments)。
- 服务器: 改变默认日志级别为([`-l`](../operations/cli.md#cmdoption-l)) info。
- stats: 最大 stat/名字尺寸和stats最大数现在可通过[`--max-obj-name-len`](../operations/cli.md#cmdoption-max-obj-name-len) 和 [`--max-stats`](../operations/cli.md#cmdoption-max-stats) 选项变化。
- tcp 代理: 添加了 [访问日志](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-access-log)。
- tcp 代理: 添加了 [可配置连接重试](../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-field-config-filter-network-tcp-proxy-v2-tcpproxy-max-connect-attempts)。
- tcp 代理: 启用[异常值检测器](arch_overview/outlier.md#arch-overview-outlier-detection)。
- tls: 添加了 [SNI 支持](../faq/sni.md#faq-how-to-setup-sni)。
- tls: 添加了 支持指定 [TLS 会话票键](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-downstreamtlscontext-session-ticket-keys)。
- tls: 运行配置 [min](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-tlsparameters-tls-minimum-protocol-version) 和 [max](../api-v2/api/v2/auth/cert.proto.md#envoy-api-field-auth-tlsparameters-tls-maximum-protocol-version) TLS 协议版本。
- 追踪: 添加了 [定制追踪范围装饰器](../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-decorator)。
- 很多小的缺陷修复和性能提高未列出。

## 1.4.0 (8月24日, 2017)

- macOS 被 [支持](https://github.com/envoyproxy/envoy/blob/master//bazel#quick-start-bazel-build-for-developers)。 (不少特性没有，如热重启和最初目的地路由)。
- 现在直接支持 [配置文件](../configuration/overview/v1_overview.md#config-overview-v1)的YAML。
- 添加 /routes admin 端点。
- 现在支持对 TCP 代理，HTTP/1，和 HTTP/2 的端到端流控。包含过滤器缓存的 HTTP 流控不完整并将在 1.5.0 中实现。
- 添加日志详细程度 [编译时间标志]。(https://github.com/envoyproxy/envoy/blob/master//bazel#log-verbosity)。
- 添加热重启[编译时间标志](https://github.com/envoyproxy/envoy/blob/master//bazel#hot-restart)。
- 添加独创的目的地 [集群](arch_overview/service_discovery.md#arch-overview-service-discovery-types-original-destination) 和 [负载均衡器](arch_overview/load_balancing.md#arch-overview-load-balancing-types-original-destination)。
- 现在支持[WebSocket](arch_overview/websocket.md#arch-overview-websocket)。
- 虚拟集群优先级被硬性移除而没有过渡，因为我们有理由确信没有人使用这一特性。
- 添加路由 [validate_clusters](../api-v1/route_config/route_config.md#config-http-conn-man-route-table-validate-clusters) 选项。
- 添加[x-envoy-downstream-service-node](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-downstream-service-node) 头。
- 添加[x-forwarded-client-cert](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-client-cert) 头。
- 第一次为[绝对 URLs](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-http1-settings) 添加了 HTTP/1 转发代理支持。
- HTTP/2 编解码器设置现在是[可配置的](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-http2-settings)。
- 添加gRPC/JSON 转码 [过滤器](../configuration/http_filters/grpc_json_transcoder_filter.md#config-http-filters-grpc-json-transcoder)。
- 添加gRPC web [过滤器](../configuration/http_filters/grpc_web_filter.md#config-http-filters-grpc-web)。
-  [网络](../configuration/network_filters/rate_limit_filter.md#config-network-filters-rate-limit) 中的速率限制服务调用和 [HTTP](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) 速率限制过滤器中的可配置超时。
- 添加 [x-envoy-retry-grpc-on](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-grpc-on) 头。
- 添加 [LDS API](arch_overview/dynamic_configuration.md#arch-overview-dynamic-config-lds)。
- 添加TLS [require_client_certificate](../api-v1/listeners/listeners.md#config-listener-ssl-context-require-client-certificate) 选项。
- 添加[Configuration 检查工具](../install/tools/config_load_check_tool.md#install-tools-config-load-check-tool)。
- 添加[JSON schema 检查工具](../install/tools/schema_validator_check_tool.md#install-tools-schema-validator-check-tool)。
- 通过 [`--mode`](../operations/cli.md#cmdoption-mode) 选项添加配置验证模式。
- 添加 [`--local-address-ip-version`](../operations/cli.md#cmdoption-local-address-ip-version) 选项。
- IPv6 现在被完全支持。
- 添加UDP [statsd_ip_address](../configuration/overview/v1_overview.md#config-overview-statsd-udp-ip-address) 选项。
- 添加针对每个集群的 [DNS 解析器](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-dns-resolvers)。
- [故障过滤器](../configuration/http_filters/fault_filter.md#config-http-filters-fault-injection) 增强和修复。
- 几个特性 [在 1.4.0 发布中不再支持](https://github.com/envoyproxy/envoy/blob/master//DEPRECATED.md#version-140)。它们将在 1.5.0 发布周期的开始被移除。我们将明确声明 HttpFilterConfigFactory 过滤器 API 被 NamedHttpFilterConfigFactory 代替。
- 很多小的缺陷修复和性能提高未列出。

## 1.3.0 (5月17日, 2017)

- 从这个发布起，我们有了一个官方的 [打破改变策略](https://github.com/envoyproxy/envoy/blob/master//CONTRIBUTING.md#breaking-change-policy)。请注意，在这个发布中有无数的打破配置改变。这里没有列出。将来的发布会遵循这一策略并有关于废除和修改的清晰文档。
- Bazel 现在是正式的构建系统 (取代 CMake)。开发/构建/测试流程有大量的变化。更多信息参见 [/bazel/README.md](https://github.com/envoyproxy/envoy/blob/master//bazel/README.md) 和 [/ci/README.md](https://github.com/envoyproxy/envoy/blob/master//ci/README.md)。
- [异常值检测](arch_overview/outlier.md#arch-overview-outlier-detection) 被扩展包括成功率变化，以及现在所有在运行时和 JSON 配置中可配置的参数。
- TCP 层 [监听器](../api-v1/listeners/listeners.md#config-listeners-per-connection-buffer-limit-bytes) 和 [集群](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-per-connection-buffer-limit-bytes) 连接现在有了可配置的接收缓冲区限制，在这个点，连接层回压被应用。完整的端对端流控将在未来的发布中可用。
- [Redis 健康状况检查](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc) 被作为一个主动健康检查类型加入。完整的 Redis 支持将在 1.4.0 中被记录和支持。
- [TCP 健康状况检查](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-tcp-health-checking) 现在支持 “仅连接” 模式，只检查远端的服务器是否可以被连接而不需要读/写任何数据。
- [BoringSSL](https://boringssl.googlesource.com/boringssl) 是现在唯一支持的 TLS 提供者。默认的加密套件和 ECDH 曲线已经为 [监听器](../api-v1/listeners/listeners.md#config-listener-ssl-context) 和 [集群](../api-v1/cluster_manager/cluster_ssl.md#config-cluster-manager-cluster-ssl) 连接更新了更现代的默认值。
- 头值匹配 [速率限制行为](../api-v1/route_config/rate_limits.md#config-http-conn-man-route-table-rate-limit-actions) 被扩展包括 *expect match*参数。
- 路由层 HTTP 速率限制配置现在默认不继承虚拟主机层配置。如果想要，[include_vh_rate_limits](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-include-vh) 继承虚拟主机层选项。
- HTTP 路由现在可以通过 [request_headers_to_add](../configuration/http_conn_man/headers.md#config-http-conn-man-headers-custom-request-headers)  选项对每个路由和每个虚拟主机添加请求头。
- [配置例子](../install/ref_configs.md#install-ref-configs) 已经被更新以展示最新的特性。
- [per_try_timeout_ms](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-retry) 除了通过[x-envoy-upstream-rq-per-try-timeout-ms](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-per-try-timeout-ms) HTTP 头，现在可以被配置在一个路由项的策略中。
- [HTTP 虚拟主机匹配](../api-v1/route_config/vhost.md#config-http-conn-man-route-table-vhost) 现在包括支持前缀通配符域名 (如, *.lyft.com)。
- 对于追踪随机取样的默认值被改为 100% 而且仍然是可以在[运行时](../configuration/http_conn_man/runtime.md#config-http-conn-man-runtime)中配置。
- [HTTP 追踪配置](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-tracing) 被扩展以允许标签从任意 HTTP 头迁移。
- [HTTP 速率限制过滤器](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) 现在可以通过 request_type  被应用于内部、外部，或者任意请求。
- [监听器绑定](../configuration/listeners/listeners.md#config-listeners) 现在需要指定一个地址与。 这也被用于绑定一个监听器到一个特定地址以及一个端口。
- [MongoDB 过滤器](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy) 现在为没有 $maxTimeMS 集合的查询输出一个状态。
- [MongoDB 过滤器](../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy) 现在输出完全合法的 JSON 格式日志。
- The CPU profiler output path is now [configurable](../api-v1/admin.md#config-admin-v1).
- 添加了一个 [看门狗系统](../configuration/overview/v1_overview.md#config-overview-v1)，如果检测到死锁可以杀掉服务器。
- 添加了一个[路由表检查工具](../install/tools/route_table_check_tool.md#install-tools-route-table-check-tool)，用于在使用前测试路由表。
- 我们已经添加了一个 [示例代码库](../extending/extending.md#extending)，展示如何编译/链接一个定制过滤器。
- 添加了额外的与异常值检测相关的集群范围信息到 [/clusters admin 端点](../operations/admin.md#operations-admin-interface)。
- 多个 SAN 现在可以通过 [verify_subject_alt_name](../api-v1/listeners/listeners.md#config-listener-ssl-context) 设置验证。另外，URI 类型 SAN 可以被验证。
- HTTP 过滤器现在可以被传递给按照每个路由指定的[不透明的配置](../api-v1/route_config/route.md#config-http-conn-man-route-table-opaque-config)。
- Envoy 现在默认有一个内建的崩溃处理程序，它将打印一条堆栈信息。如果需要，这个行为可以通过`--define=signal_trace=disabled` Bazel 选项关掉。
- 添加了 Zipkin 作为一个支持的 [追踪提供者](arch_overview/tracing.md#arch-overview-tracing)。
- 此处未列出的数量庞大的小的变化和修复。

## 1.2.0 (3月7日, 2017)

- [集群发现服务 (CDS) API](../configuration/cluster_manager/cds.md#config-cluster-manager-cds)。
- [异常值检测](arch_overview/outlier.md#arch-overview-outlier-detection) (主动健康检查)。
- Envoy 配置现在针对 [JSON schema](../configuration/overview/v1_overview.md#config-overview-v1) 检查。
- [环哈希](arch_overview/load_balancing.md#arch-overview-load-balancing-types) 一致性负载均衡器，以及 [基于策略](../api-v1/route_config/route.md#config-http-conn-man-route-table-hash-policy)  的HTTP 一致性哈希路由。
- 提供 HTTP 速率限制过滤器极大地 [增强的全局速率限制配置](arch_overview/global_rate_limiting.md#arch-overview-rate-limit) 。
- HTTP 路由到一个 [从头获取的](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-cluster-header)集群。
- [加权的集群](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-config-weighted-clusters) HTTP 路由。
- 在 HTTP 路由过程中[自动主机重写](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-auto-host-rewrite) 。
- 在 HTTP 路由过程中[正则头部匹配 header matching](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers) 。
- HTTP 访问日志 [运行时过滤器](../api-v1/access_log.md#config-http-con-manager-access-log-filters-runtime-v1)。
- LightStep 追踪器 [父/子跨关联](arch_overview/tracing.md#arch-overview-tracing)。
- [路由发现服务 (RDS) API](../configuration/http_conn_man/rds.md#config-http-conn-man-rds)。
- HTTP 路由器 [x-envoy-upstream-rq-timeout-alt-response header](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-timeout-alt-response) 支持。
- *use_original_dst* 和 *bind_to_port* [监听器选项](../configuration/listeners/listeners.md#config-listeners) (对基于iptables 的透明代理支持有用)。
- TCP 代理过滤器 [路由表支持](../configuration/network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy)。
- 可配置 [状态写入磁盘间隔](../configuration/overview/v1_overview.md#config-overview-stats-flush-interval-ms)。
- 各种 [第三方库升级](../install/building.md#install-requirements)，包括使用 BoringSSL 作为默认的 SSL 提供者。
- 不再为优先级计算维护关闭的 HTTP/2 流。导致对大型网络实质性的内存节省。
- 此处未列出的数量庞大的小的变化和修复。

## 1.1.0 (9月30日, 2016)

- 为了我们的 JSON 库从 Jannson 转换为 RapidJSON  (在 1.2.0 中允许一个配置模式)。
- 升级各种其他库的 [建议版本](../install/building.md#install-requirements)。
- 用于 DNS 发现类型的[可配置 DNS 刷新速率](../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-dns-refresh-rate-ms) 。
- 上游电路断路器配置可 [通过运行时撤销](../configuration/cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime)。
- [域感知路由支持](arch_overview/load_balancing.md#arch-overview-load-balancing-zone-aware-routing)。
- 通用[头匹配路由规则](../api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers)。
- HTTP/2 [优雅连接耗尽](../api-v1/network_filters/http_conn_man.md#config-http-conn-man-drain-timeout-ms) (双 GOAWAY)。
- DynamoDB 过滤器 [按片统计](../configuration/http_filters/dynamodb_filter.md#config-http-filters-dynamo) (预发布 AWS 特性)。
- [故障注入 HTTP 过滤器](../configuration/http_filters/fault_filter.md#config-http-filters-fault-injection)的第一版发布。
- HTTP [速率限制过滤器](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit) 增强 (注意HTTP 速率限制的配置讲座 1.2.0 中被修正)。
- 添加了 [被拒绝流重试策略](../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-on)。
- 用于上游集群的多 [优先级队列](arch_overview/http_routing.md#arch-overview-http-routing-priority)  (可针对每个路由基础，用各自的连接池，电路断路器，等等，配置)。
- 添加了最大连接电路断路器到 [TCP 带理过滤器](arch_overview/tcp_proxy.md#arch-overview-tcp-proxy)。
- 添加了 [CLI](../operations/cli.md#operations-cli) 选项用于设置日志文件写入磁盘的时间间隔以及在热重启期间耗尽/关机时间。
- 非常大数量的核心 HTTP/TCP 流性能提升，以及不少新的配置标志以允许关掉昂贵的特性，如果它们不需要 (特别是请求 ID 生成和动态响应代码状态)。
- 在 [Mongo 嗅探过滤器] (../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy) 中支持 Mongo 3.2。
- 很多其它没有列出的小的修复和强化。

## 1.0.0 (9月12日, 2016)

第一版开源发布。
