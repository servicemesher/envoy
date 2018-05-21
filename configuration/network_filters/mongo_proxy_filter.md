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
| cx_drain_close                   | Counter | 在服务器关闭期间，在回复边界被优雅关闭的连接总数 |

### 分散获取查询

任何不使用  *_id* 作为查询参数的查询，Envoy 将其定义为一个*分散获取查询* 。Envoy 同时在最顶级文档以及 *_id* 的 *$query* 中查找。

### 多重查询

任何使用  *_id* 作为查询参数的查询，且 *_id* 不是一个标量值（如文档或数组）， Envoy 定义其为 *多重查询*。
Envoy 同时在最顶级文档以及 *_id* 的 *$query* 中查找。

### $comment 解析

如果一个查询具有顶级的 *$comment* 字段（通常在 *$query* 字段的基础上添加），Envoy 会将其解析为 JSON 格式并查找以下结构：

```
{
  "callingFunction": "..."
}
```

- callingFunction

  *(required, string)* 执行查询的函数。 在可用的情况下，这个函数将会用来做 [按调用站](#config-network-filters-mongo-proxy-callsite-stats) 查询统计。

### 按命令统计

MongoDB 过滤器将在 *mongo.<stat_prefix>.cmd.<cmd>.* 命名空间为命令收集相应的统计信息。

| 名称            | 类型      | 描述                         |
| -------------- | --------- | ---------------------------- |
| total          | Counter   | 命令数量          |
| reply_num_docs | Histogram | 回复中的文档数量 |
| reply_size     | Histogram | 回复的字节大小   |
| reply_time_ms  | Histogram | 命令时间（毫秒） |

### 按集合查询统计

MongoDB 过滤器将在 *mongo.<stat_prefix>.collection.<collection>.query.* 命名空间为查询收集相应的统计信息。

| 名称            | 类型      | 描述                         |
| -------------- | --------- | ---------------------------- |
| total          | Counter   | 查询数量            |
| scatter_get    | Counter   | 分散获取查询数量       |
| multi_get      | Counter   | 多重查询梳理         |
| reply_num_docs | Histogram | 回复中的文档数量 |
| reply_size     | Histogram | 回复的字节大小   |
| reply_time_ms  | Histogram | 查询时间（毫秒）   |

### 按集合与现场查询统计

如果应用程序在 *$comment* 字段中提供[调用函数](#config-network-filters-mongo-proxy-comment-parsing) ，Envoy 将相应生成按调用站点为维度的统计信息。
这些统计信息与[按集合统计](#config-network-filters-mongo-proxy-collection-stats)相匹配，可在 *mongo.<stat_prefix>.collection.<collection>.callsite.<callsite>.query.* 命名空间中找到相关信息。

## 运行时

Mongo 代理过滤器支持如下运行时设置：

- mongo.connection_logging_enabled

  启用日志记录的连接百分比。 默认为100。 这将只允许将指定百分比的连接做日志记录, 但这些连接上的所有信息将会做日志记录。

- mongo.proxy_enabled

  启用代理的连接百分比。默认为100。

- mongo.logging_enabled

  启用日志记录的消息的百分比。 默认值为100。如果小于100，部分查询可能会在无回复的情况下被记录。

- mongo.mongo.drain_close_enabled

  当服务器正在被删除或尝试做强制关闭时，将被关闭的连接百分比。默认为100。

- mongo.fault.fixed_delay.percent

  当没有活跃故障时，一个合格的 MongoDB 操作受到注入故障影响的可能性。 默认为配置中指定的 *percent* 。

- mongo.fault.fixed_delay.duration_ms
  
  以毫秒为单位的延迟时间。默认为配置中指定的 *duration_ms*。

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

  连接正在被代理的上游主机, 在过滤器配合 [TCP 代理过滤器](tcp_proxy_filter.md#config-network-filters-tcp-proxy)时，此字段将被填充。