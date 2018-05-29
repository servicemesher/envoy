# TLS 检查器

TLS 检查器监听器筛选器允许检测传输是 TLS 还是明文，如果是 TLS，它将检测来自客户端的[服务器名称指示](https://en.wikipedia.org/wiki/Server_Name_Indication)。这可以用来通过 [FilterChainMatch](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchainmatch) 的 [sni_domains](../../api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-filterchainmatch-sni-domains) 来选择[过滤器链](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.md#envoy-api-msg-listener-filterchain)。

- [SNI](../../faq/sni.md#faq-how-to-setup-sni)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.html#envoy-api-field-listener-listenerfilter-name)
