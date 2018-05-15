# TLS 检查器

TLS inspector listener filter allows detecting whether the transport appears to be TLS or plaintext, and if it is TLS, it detects the [server name indication](https://en.wikipedia.org/wiki/Server_Name_Indication) from the client. This can be used to select a[FilterChain](../../api-v2/api/v2/listener/listener.proto.md#envoy-api-msg-listener-filterchain) via the [sni_domains](../../api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-filterchainmatch-sni-domains) of a [FilterChainMatch](../../api-v2/api/v2/listener/listener.proto.md#envoy-api-msg-listener-filterchainmatch).

- [SNI](../../faq/sni.md#faq-how-to-setup-sni)
- [v2 API reference](../../api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-listenerfilter-name)