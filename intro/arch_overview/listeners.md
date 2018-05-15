# Listener

The Envoy configuration supports any number of listeners within a single process. Generally we recommend running a single Envoy per machine regardless of the number of configured listeners. This allows for easier operation and a single source of statistics. Currently Envoy only supports TCP listeners.

Each listener is independently configured with some number of network level (L3/L4) [filters](network_filters.md#arch-overview-network-filters). When a new connection is received on a listener, the configured connection local filter stack is instantiated and begins processing subsequent events. The generic listener architecture is used to perform the vast majority of different proxy tasks that Envoy is used for (e.g., [rate limiting](global_rate_limiting.md#arch-overview-rate-limit), [TLS client authentication](ssl.md#arch-overview-ssl-auth-filter), [HTTP connection management](http_connection_management.md#arch-overview-http-conn-man), MongoDB [sniffing](mongo.md#arch-overview-mongo), raw [TCP proxy](tcp_proxy.md#arch-overview-tcp-proxy), etc.).

Listeners are optionally also configured with some number of [listener filters](listener_filters.md#arch-overview-listener-filters). These filters are processed before the network level filters, and have the opportunity to manipulate the connection metadata, usually to influence how the connection is processed later filters or clusters.

Listeners can also be fetched dynamically via the [listener discovery service (LDS)](../../configuration/listeners/lds.md#config-listeners-lds).

Listener [configuration](../../configuration/listeners/listeners.md#config-listeners).