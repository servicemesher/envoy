# TCP 代理

Since Envoy is fundamentally written as a L3/L4 server, basic L3/L4 proxy is easily implemented. The TCP proxy filter performs basic 1:1 network connection proxy between downstream clients and upstream clusters. It can be used by itself as an stunnel replacement, or in conjunction with other filters such as the [MongoDB filter](mongo.md#arch-overview-mongo) or the [rate limit](../../configuration/network_filters/rate_limit_filter.md#config-network-filters-rate-limit) filter.

The TCP proxy filter will respect the [connection limits](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-connections) imposed by each upstream cluster’s global resource manager. The TCP proxy filter checks with the upstream cluster’s resource manager if it can create a connection without going over that cluster’s maximum number of connections, if it can’t the TCP proxy will not make the connection.

TCP proxy filter [configuration reference](../../configuration/network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy).