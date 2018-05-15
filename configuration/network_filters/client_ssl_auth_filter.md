# 客户端 TLS 身份验证

- Client TLS authentication filter [architecture overview](../../intro/arch_overview/ssl.md#arch-overview-ssl-auth-filter)
- [v1 API reference](../../api-v1/network_filters/client_ssl_auth_filter.md#config-network-filters-client-ssl-auth-v1)
- [v2 API reference](../../api-v2/config/filter/network/client_ssl_auth/v2/client_ssl_auth.proto.md#envoy-api-msg-config-filter-network-client-ssl-auth-v2-clientsslauth)

## Statistics

Every configured client TLS authentication filter has statistics rooted at *auth.clientssl.<stat_prefix>.*with the following statistics:

| Name                 | Type    | Description                                          |
| -------------------- | ------- | ---------------------------------------------------- |
| update_success       | Counter | Total principal update successes                     |
| update_failure       | Counter | Total principal update failures                      |
| auth_no_ssl          | Counter | Total connections ignored due to no TLS              |
| auth_ip_white_list   | Counter | Total connections allowed due to the IP white list   |
| auth_digest_match    | Counter | Total connections allowed due to certificate match   |
| auth_digest_no_match | Counter | Total connections denied due to no certificate match |
| total_principals     | Gauge   | Total loaded principals                              |

## REST API

- `GET /v1/certs/list/approved`

  The authentication filter will call this API every refresh interval to fetch the current list of approved certificates/principals. The expected JSON response looks like:`{   "certificates": [] } `certificates*(required, array)* list of approved certificates/principals.Each certificate object is defined as:`{   "fingerprint_sha256": "...", } `fingerprint_sha256*(required, string)* The SHA256 hash of the approved client certificate. Envoy will match this hash to the presented client certificate to determine whether there is a digest match.