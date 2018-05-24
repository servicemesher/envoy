# 如何设置 SNI？

[SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) 仅被 [v2 配置/API](../configuration/overview/v2_overview.md#config-overview-v2) 支持。

目前的实现中要求所有[过滤器链](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.html#envoy-api-msg-listener-filterchain)中的[过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-filterchain-filters) 必须是相同的。在以后的发布中,这个约束将会放宽，我们将可以将SNI运用到完全不同的过滤器链中。 我们还可以将 [域名匹配](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto.md#envoy-api-field-route-virtualhost-domains) 运用到HTTP 连接管理中去选择不同的路由路线。

这是截至目前最常见的 SNI 使用场景。

以下是一个满足上述条件的 YAML 范例。

```yaml
address:
  socket_address: { address: 127.0.0.1, port_value: 1234 }
filter_chains:
- filter_chain_match:
    sni_domains: "example.com"
  tls_context:
    common_tls_context:
      tls_certificates:
        - certificate_chain: { filename: "example_com_cert.pem" }
          private_key: { filename: "example_com_key.pem" }
  filters:
  - name: envoy.http_connection_manager
    config:
      route_config:
        virtual_hosts:
        - routes:
          - match: { prefix: "/" }
            route: { cluster: service_foo }
- filter_chain_match:
    sni_domains: "www.example.com"
  tls_context:
    common_tls_context:
      tls_certificates:
        - certificate_chain: { filename: "www_example_com_cert.pem" }
          private_key: { filename: "www_example_com_key.pem" }
  filters:
  - name: envoy.http_connection_manager
    config:
      route_config:
        virtual_hosts:
        - routes:
          - match: { prefix: "/" }
            route: { cluster: service_foo }
```