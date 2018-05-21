# 缓冲区

缓冲区过滤器用于停止过滤器迭代并等待完全被缓冲的完整请求。 这可以在不同场景下发挥作用，包括确保应用程序不必去处理不完整请求以及高网络延迟。

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/buffer_filter#config-http-filters-buffer-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/buffer/v2/buffer.proto#envoy-api-msg-config-filter-http-buffer-v2-buffer)

## 单路由配置

通过在虚拟主机、路由或加权集群上提供 [BufferPerRoute](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/buffer/v2/buffer.proto#envoy-api-msg-config-filter-http-buffer-v2-bufferperroute) 配置，
可达到在单路由的基础上重写或禁用缓冲区过滤器。

## 统计

缓冲区过滤器在 *http.<stat_prefix>.buffer.* 命名空间输出统计信息。 [统计信息前缀](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-stat-prefix) 来自所拥有的 HTTP 连接管理器。

| 名称       | 类型    | 描述                                              |
| ---------- | ------- | -------------------------------------------------------- |
| rq_timeout | Counter | 因超时等待完整请求的总请求数 |