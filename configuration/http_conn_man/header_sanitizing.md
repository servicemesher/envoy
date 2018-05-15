# HTTP header sanitizing

For security reasons, Envoy will “sanitize” various incoming HTTP headers depending on whether the request is an internal or external request. The sanitizing action depends on the header and may result in addition, removal, or modification. Ultimately, whether the request is considered internal or external is governed by the [x-forwarded-for](headers.md#config-http-conn-man-headers-x-forwarded-for) header (please read the linked section carefully as how Envoy populates the header is complex and depends on the [use_remote_address](../../api-v1/network_filters/http_conn_man.md#config-http-conn-man-use-remote-address) setting).

Envoy will potentially sanitize the following headers:

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