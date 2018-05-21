# HTTP 标头清理

出于安全原因考虑，Envoy 将根据请求是内部请求还是外部请求来 “清理” 各种传入的 HTTP 标头。
清理行为取决于标头，并可能会导致添加、删除或更改。最终，一个请求被认定为内部或是外部请求都是由 [x-forwarded-for](headers.md#config-http-conn-man-headers-x-forwarded-for) 标头决定（请仔细阅读链接部分，因为 Envoy 填充标头的动作是非常复杂的，并取决于 [use_remote_address](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-use-remote-address) 设置）。

Envoy 可能会清理以下标头:

- [x-envoy-decorator-operation](../http_filters/router_filter.md#config-http-filters-router-x-envoy-decorator-operation)
- [x-envoy-downstream-service-cluster](headers.md#config-http-conn-man-headers-downstream-service-cluster)
- [x-envoy-downstream-service-node](headers.md#config-http-conn-man-headers-downstream-service-node)
- [x-envoy-expected-rq-timeout-ms](../http_filters/router_filter.md#config-http-filters-router-x-envoy-expected-rq-timeout-ms)
- [x-envoy-external-address](headers.md#config-http-conn-man-headers-x-envoy-external-address)
- [x-envoy-force-trace](headers.md#config-http-conn-man-headers-x-envoy-force-trace)
- [x-envoy-internal](headers.md#config-http-conn-man-headers-x-envoy-internal)
- [x-envoy-ip-tags](../http_filters/ip_tagging_filter.md#config-http-filters-ip-tagging)
- [x-envoy-max-retries](../http_filters/router_filter.md#config-http-filters-router-x-envoy-max-retries)
- [x-envoy-retry-grpc-on](../http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-grpc-on)
- [x-envoy-retry-on](../http_filters/router_filter.md#config-http-filters-router-x-envoy-retry-on)
- [x-envoy-upstream-alt-stat-name](../http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-alt-stat-name)
- [x-envoy-upstream-rq-per-try-timeout-ms](../http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-per-try-timeout-ms)
- [x-envoy-upstream-rq-timeout-alt-response](../http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-timeout-alt-response)
- [x-envoy-upstream-rq-timeout-ms](../http_filters/router_filter.md#config-http-filters-router-x-envoy-upstream-rq-timeout-ms)
- [x-forwarded-client-cert](headers.md#config-http-conn-man-headers-x-forwarded-client-cert)
- [x-forwarded-for](headers.md#config-http-conn-man-headers-x-forwarded-for)
- [x-forwarded-proto](headers.md#config-http-conn-man-headers-x-forwarded-proto)
- [x-request-id](headers.md#config-http-conn-man-headers-x-request-id)