# TLS

Envoy 同时支持监听器中的 [TLS 终止](https://www.envoyproxy.io/docs/envoy/latest/api-v1/listeners/listeners.html#config-listener-ssl-context)和与上游集群建立连接时的 [TLS 发起](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_ssl#config-cluster-manager-cluster-ssl)。不管是为现代 web 服务提供标准的边缘代理功能，还是同具有高级 TLS 要求（TLS1.2, SNI, 等等）的外部服务建立连接，Envoy 都提供了充分的支持。Envoy 支持以下的 TLS 特性：

- **加密可配置**: 每个 TLS 监听器和客户端都可以指定它支持的加密算法。
- **客户端证书**: 除服务器证书验证外，上游/客户端连接还可以提供一个客户端证书。
- **证书验证和固定**: 证书验证选项包括基本的证书链验证、Subject 名称验证和哈希固定。
- **证书撤销**: 如果[提供]((https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-field-auth-certificatevalidationcontext-crl)了证书撤销列表（CRL），Envoy 可以根据它检查对等证书。
- **ALPN**: TLS 监听器支持 ALPN. HTTP 连接管理器使用这个信息（以及协议接口）来确定客户端使用的是 HTTP/1.1 还是 HTTP/2.
- **SNI**: SNI 同时支持服务端（监听器）连接和客户端（上游）连接。
- **会话恢复**: 服务器连接支持通过 TLS 会话票据恢复之前的会话（参见[RFC 5077](https://www.ietf.org/rfc/rfc5077.txt)）。可以在热重启时和并行 Envoy 实例之间执行恢复（通常在前端代理配置中很有用）。

## 底层实现

目前 Envoy 被编写为使用 [BoringSSL](https://boringssl.googlesource.com/boringssl) 作为 TLS 提供者。

## 启用证书验证

除非验证上下文指定了一个或多个可信授权证书，否则上游和下游连接的证书验证都不会启用。

### 配置示例

```yaml
static_resources:
  listeners:
  - name: listener_0
    address: { socket_address: { address: 127.0.0.1, port_value: 10000 } }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        # ...
      tls_context:
        common_tls_context:
          validation_context:
            trusted_ca:
              filename: /usr/local/my-client-ca.crt
  clusters:
  - name: some_service
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    hosts: [{ socket_address: { address: 127.0.0.2, port_value: 1234 }}]
    tls_context:
      common_tls_context:
        validation_context:
          trusted_ca:
            filename: /etc/ssl/certs/ca-certificates.crt
```

*/etc/ssl/certs/ca-certificates.crt* 是 Debian 系统上 CA 包文件的默认路径。它使得 Envoy 在验证 *127.0.0.2:1234* 的服务器身份时，使用与标准安装的 Debian 系统上的 cURL 等相同的方式。Linux 和 BSD 系统上常见的系统 CA 包文件路径有

- /etc/ssl/certs/ca-certificates.crt (Debian/Ubuntu/Gentoo etc.)
- /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem (CentOS/RHEL 7)
- /etc/pki/tls/certs/ca-bundle.crt (Fedora/RHEL 6)
- /etc/ssl/ca-bundle.pem (OpenSUSE)
- /usr/local/etc/ssl/cert.pem (FreeBSD)
- /etc/ssl/cert.pem (OpenBSD)

对于其他的 TLS 选项，请参见 [UpstreamTlsContexts](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-upstreamtlscontext) 和 [DownstreamTlsContexts](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-downstreamtlscontext) 的参考手册。

## 身份验证过滤器

Envoy 提供了一个网络过滤器，通过从 REST VPN 服务拉取的主体来进行 TLS 客户端身份验证。此过滤器将提供的客户端证书哈希与主体列表进行匹配，以确定是否允许连接。另外也可以配置一个可选的 IP 白名单。此功能可用于为 Web 基础架构构建边缘代理 VPN 支持。

这里是客户端 TSL 验证过滤器的[配置参考](../../configuration/network_filters/client_ssl_auth_filter.md#config-network-filters-client-ssl-auth)。