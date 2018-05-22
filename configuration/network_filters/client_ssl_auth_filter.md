# 客户端 TLS 身份验证

- 客户端 TLS 认证过滤器[架构概览](../../intro/arch_overview/ssl.md#arch-overview-ssl-auth-filter)
- [v1 API 参考](https://www.envoyproxy.io/api-v1/network_filters/client_ssl_auth_filter.html#config-network-filters-client-ssl-auth-v1)
- [v2 API 参考](https://www.envoyproxy.io/api-v2/config/filter/network/client_ssl_auth/v2/client_ssl_auth.proto.html#envoy-api-msg-config-filter-network-client-ssl-auth-v2-clientsslauth)

## 统计

每个配置的客户端 TLS 认证过滤器均有以 *auth.clientssl.<stat_prefix>.* 开头的统计信息，其统计信息如下所示：

| 名称                 | 类型    | 描述                                          |
| -------------------- | ------- | ---------------------------------------------------- |
| update_success       | Counter | 主体更新成功总次数                     |
| update_failure       | Counter | 主体更新失败总次数                      |
| auth_no_ssl          | Counter | 无 TLS 而被忽略的连接总次数              |
| auth_ip_white_list   | Counter | 由于 IP 白名单而被允许的连接总次数   |
| auth_digest_match    | Counter | 由于证书匹配而被允许的连接总次数   |
| auth_digest_no_match | Counter | 由于证书不匹配而被拒绝的连接总次数 |
| total_principals     | Gauge   | 已加载主体总数                              |

## REST API

`GET /v1/certs/list/approved`

认证过滤器将每隔一段刷新时间调用一次这个API，来获取当前获得批准的证书/主体列表。预期的 JSON 响应如下所示：

```json
{
  "certificates": []
}
```
**certificates**

*(required, array)* 

为批准的证书/主体列表。

每个证书对象定义为：

```json
{
  "fingerprint_sha256": "...",
}
```

**fingerprint_sha256**

*(required, string)* 

为批准的客户端证书的 SHA256 hash 值。Envoy 会将此 hash 与所提供的客户端证书进行匹配，以确定是否存在摘要匹配。