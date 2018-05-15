# Network (L3/L4) filter

As discussed in the [listener](listeners.md#arch-overview-listeners) section, network level (L3/L4) filters form the core of Envoy connection handling. The filter API allows for different sets of filters to be mixed and matched and attached to a given listener. There are three different types of network filters:

- **Read**: Read filters are invoked when Envoy receives data from a downstream connection.
- **Write**: Write filters are invoked when Envoy is about to send data to a downstream connection.
- **Read/Write**: Read/Write filters are invoked both when Envoy receives data from a downstream connection and when it is about to send data to a downstream connection.

The API for network level filters is relatively simple since ultimately the filters operate on raw bytes and a small number of connection events (e.g., TLS handshake complete, connection disconnected locally or remotely, etc.). Filters in the chain can stop and subsequently continue iteration to further filters. This allows for more complex scenarios such as calling a [rate limiting service](global_rate_limiting.md#arch-overview-rate-limit), etc. Envoy already includes several network level filters that are documented in this architecture overview as well as the [configuration reference](../../configuration/network_filters/network_filters.md#config-network-filters).