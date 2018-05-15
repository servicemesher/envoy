# Buffer

The buffer filter is used to stop filter iteration and wait for a fully buffered complete request. This is useful in different situations including protecting some applications from having to deal with partial requests and high network latency.

- [v1 API reference](../../api-v1/http_filters/buffer_filter.md#config-http-filters-buffer-v1)
- [v2 API reference](../../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-msg-config-filter-http-buffer-v2-buffer)

## Per-Route Configuration

The buffer filter configuration can be overridden or disabled on a per-route basis by providing a[BufferPerRoute](../../api-v2/config/filter/http/buffer/v2/buffer.proto.md#envoy-api-msg-config-filter-http-buffer-v2-bufferperroute) configuration on the virtual host, route, or weighted cluster.

## Statistics

The buffer filter outputs statistics in the *http.<stat_prefix>.buffer.* namespace. The [stat prefix](../../api-v1/network_filters/http_conn_man.md#config-http-conn-man-stat-prefix)comes from the owning HTTP connection manager.

| Name       | Type    | Description                                              |
| ---------- | ------- | -------------------------------------------------------- |
| rq_timeout | Counter | Total requests that timed out waiting for a full request |