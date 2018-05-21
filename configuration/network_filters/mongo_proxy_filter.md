# Mongo 代理

- MongoDB [架构概述](../../intro/arch_overview/mongo.md#arch-overview-mongo)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/mongo_proxy_filter#config-network-filters-mongo-proxy-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/mongo_proxy/v2/mongo_proxy.proto#envoy-api-msg-config-filter-network-mongo-proxy-v2-mongoproxy)

## 故障注入

Mongo 代理过滤器支持故障注入。可以查看 V1 以及 V2 的 API 参考文档以了解如何进行相关配置。

## 统计

每一个配置的 MongoDB 代理过滤器的统计信息都以 *mongo.<stat_prefix>.* 开头，其统计信息如下：


| 名称                             | 类型    | 描述                                                  |
| -------------------------------- | ------- | ------------------------------------------------------------ |
| decoding_error                   | Counter | MongoDB 协议解码错误的数量                   |
| delay_injected                   | Counter | 延迟被注入的次数                            |
| op_get_more                      | Counter | OP_GET_MORE 消息的数量                               |
| op_insert                        | Counter | OP_INSERT 消息的数量                                 |
| op_kill_cursors                  | Counter | OP_KILL_CURSORS 消息的数量                           |
| op_query                         | Counter | OP_QUERY 消息的数量                                  |
| op_query_tailable_cursor         | Counter | 设置了 tailable cursor flag 的 OP_QUERY 消息的数量               |
| op_query_no_cursor_timeout       | Counter | 没有设置 cursor timeout flag 的 OP_QUERY 消息的数量              |
| op_query_await_data              | Counter | 设置了 await data flag 的 OP_QUERY 消息的数量                    |
| op_query_exhaust                 | Counter | 设置了 exhaust flag 的 OP_QUERY 消息的数量                       |
| op_query_no_max_time             | Counter | 没有设置 maxTimeMS 的查询数量                      |
| op_query_scatter_get             | Counter | 分散获取查询的数量                               |
| op_query_multi_get               | Counter | 多重查询的次数                                  |
| op_query_active                  | Gauge   | 活跃查询的数量                                    |
| op_reply                         | Counter | OP_REPLY 消息的数量                                  |
| op_reply_cursor_not_found        | Counter | 设置了 cursor not found flag 的 OP_REPLY 消息的数量             |
| op_reply_query_failure           | Counter | 设置了 query failure flag 的 OP_REPLY 消息的数量              |
| op_reply_valid_cursor            | Counter | 拥有有效的游标的 OP_REPLY 消息的数量                       |
| cx_destroy_local_with_active_rq  | Counter | 拥有一个活跃查询并被本地破坏的连接总数           |
| cx_destroy_remote_with_active_rq | Counter | 拥有一个活跃查询并被远程破坏的连接总数          |
| cx_drain_close                   | Counter | Connections gracefully closed on reply boundaries during server drain |
| cx_drain_close                   | Counter | 在服务器关闭期间，在回复边界被优雅关闭的连接总数 |

### 分散获取查询

Envoy defines a *scatter get* as any query that does not use an *_id* field as a query parameter. Envoy looks in both the top level document as well as within a *$query* field for *_id*.

### 多重查询

Envoy defines a *multi get* as any query that does use an *_id* field as a query parameter, but where *_id* is not a scalar value (i.e., a document or an array). Envoy looks in both the top level document as well as within a *$query* field for *_id*.

### $comment 解析

If a query has a top level *$comment* field (typically in addition to a *$query* field), Envoy will parse it as JSON and look for the following structure:

```
{
  "callingFunction": "..."
}
```

- callingFunction

  *(required, string)* the function that made the query. If available, the function will be used in [callsite](#config-network-filters-mongo-proxy-callsite-stats) query statistics.

### 按命令统计

MongoDB 过滤器将在 *mongo.<stat_prefix>.cmd.<cmd>.* 命名空间为命令收集相应的统计信息。

| 名称            | 类型      | 描述                         |
| -------------- | --------- | ---------------------------- |
| total          | Counter   | Number of commands           |
| reply_num_docs | Histogram | Number of documents in reply |
| reply_size     | Histogram | Size of the reply in bytes   |
| reply_time_ms  | Histogram | Command time in milliseconds |

### 按集合查询统计

MongoDB 过滤器将在 *mongo.<stat_prefix>.collection.<collection>.query.* 命名空间为查询收集相应的统计信息。

| 名称            | 类型      | 描述                         |
| -------------- | --------- | ---------------------------- |
| total          | Counter   | Number of queries            |
| scatter_get    | Counter   | Number of scatter gets       |
| multi_get      | Counter   | Number of multi gets         |
| reply_num_docs | Histogram | Number of documents in reply |
| reply_size     | Histogram | Size of the reply in bytes   |
| reply_time_ms  | Histogram | Query time in milliseconds   |

### 按集合与现场查询统计

If the application provides the [calling function](#config-network-filters-mongo-proxy-comment-parsing) in the *$comment* field, Envoy will generate per callsite statistics. These statistics match the [per collection statistics](#config-network-filters-mongo-proxy-collection-stats) but are found in the *mongo.<stat_prefix>.collection.<collection>.callsite.<callsite>.query.* namespace.

## 运行时

Mongo 代理过滤器支持如下运行时设置：

- mongo.connection_logging_enabled

  % of connections that will have logging enabled. Defaults to 100. This allows only a % of connections to have logging, but for all messages on those connections to be logged.

- mongo.proxy_enabled

  % of connections that will have the proxy enabled at all. Defaults to 100.

- mongo.logging_enabled

  % of messages that will be logged. Defaults to 100. If less than 100, queries may be logged without replies, etc.

- mongo.mongo.drain_close_enabled

  % of connections that will be drain closed if the server is draining and would otherwise attempt a drain close. Defaults to 100.

- mongo.fault.fixed_delay.percent

  Probability of an eligible MongoDB operation to be affected by the injected fault when there is no active fault. Defaults to the *percent* specified in the config.

- mongo.fault.fixed_delay.duration_ms

  The delay duration in milliseconds. Defaults to the *duration_ms* specified in the config.

## 访问日志格式

访问日志格式不可定制，并具有以下布局：

```
{"time": "...", "message": "...", "upstream_host": "..."}
```

- time

  解析完整信息所需的系统时间，精确度至毫秒。

- message

  文本扩展的消息。 消息是否完全可扩展取决于上下文。 为避免日志超大，有时会提供摘要数据。

- upstream_host

  The upstream host that the connection is proxying to, if available. This is populated if the filter is used along with the [TCP 代理过滤器](tcp_proxy_filter.md#config-network-filters-tcp-proxy).