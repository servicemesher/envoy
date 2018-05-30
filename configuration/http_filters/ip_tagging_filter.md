# IP 标签

HTTP IP 标签过滤器使用来自可信地址的 [x-forwarded-for](../http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for)  的值来设置标签头 *x-envoy-ip-tags*。如果没有，则不设置。

IP 标签的实施提供了一种可扩展的方式来高效地将 IP 地址与大量的 CIDR 列表进行范围比较。用于存储标签和 IP 地址子网的基础算法在 S.Nilsson 和 G.Karlsson 的 [IP-address lookup using LC-tries](https://www.nada.kth.se/~snilsson/publications/IP-address-lookup-using-LC-tries/) 论文中阐述。

## 配置

- [v2 API 引用](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/ip_tagging/v2/ip_tagging.proto.html#envoy-api-msg-config-filter-http-ip-tagging-v2-iptagging)

## 统计

IP 标签过滤器会在命名空间 *http.`stat_prefix`.ip_tagging.* 中输出统计信息。`stat_prefix` 来自对应的 HTTP 链接管理器。

| 名称            | 类型     | 描述                                                         |
| -------------- | ------- | ------------------------------------------------------------ |
| `tag_name`.hit | Counter | 已应用 `tag_name` 的请求总数 |
| no_hit         | Counter | 没有适用 IP 标签的请求总数          |
| total          | Counter | IP 标签过滤器运行的请求总数   |

## 运行时

IP 标签过滤器支持以下运行时设置:

- ip_tagging.http_filter_enabled

  启用过滤器的请求的百分比，默认值是100。
