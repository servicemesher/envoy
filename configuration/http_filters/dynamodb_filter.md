# DynamoDB

- DynamoDB [架构概述](../../intro/arch_overview/dynamo.html#arch-overview-dynamo)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/dynamodb_filter#config-http-filters-dynamo-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-field-config-filter-network-http-connection-manager-v2-httpfilter-name)

## 统计

DynamoDB 过滤器在 *http.<stat_prefix>.dynamodb.* 命名空间输出统计信息。[统计前缀](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-stat-prefix) 来自所拥有的 HTTP 连接管理器。

可以在 *http.<stat_prefix>.dynamodb.operation.<operation_name>.* 命名空间找到按操作为维度的统计信息。

| 名称                  | 类型      | 描述                                                  |
| --------------------- | --------- | ------------------------------------------------------------ |
| upstream_rq_total     | Counter   | 以 <operation_name> 命名的请求总数               |
| upstream_rq_time      | Histogram | 在 <operation_name> 上耗费的时间                               |
| upstream_rq_total_xxx | Counter   | 以 <operation_name> 命名的请求总数，以响应代码为维度统计(503、2xx等) |
| upstream_rq_time_xxx  | Histogram | 在 <operation_name> 上耗费的时间，以响应代码为维度统计(400、3xx等) |

可以在 *http.<stat_prefix>.dynamodb.table.<table_name>.* 命名空间找到按数据库表为维度的统计信息。
大多数的 DynamoDB 操作仅仅涉及一个数据库表，而 BatchGetItem 以及 BatchWriteItem 会包括多个数据库表。
只有在该批次的所有操作中使用的数据库表都是相同时，Envoy 才会跟踪每个数据库表的统计数据。

| 名称                  | 类型      | 描述                                                  |
| --------------------- | --------- | ------------------------------------------------------------ |
| upstream_rq_total     | Counter   | 对数据库表 <table_name> 的请求总数               |
| upstream_rq_time      | Histogram | 在数据库表 <table_name> 上耗费的时间                             |
| upstream_rq_total_xxx | Counter   | 对数据库表 <table_name> 的请求总数，以响应代码为维度统计(503、2xx等) |
| upstream_rq_time_xxx  | Histogram | 在数据库表 <table_name> 上耗费的时间，以响应代码为维度统计(400、3xx等) |

*免责声明：请注意，这是尚未广泛使用的预发布 Amazon DynamoDB 功能。* 按每个分区及操作为维度的统计信息可以在 *http.<stat_prefix>.dynamodb.table.<table_name>.* 命名空间中找到。 对于批量操作，只有在该批次的所有操作中使用的数据库表都是相同时，只有在该批次的所有操作中使用的数据库表都是相同时，Envoy 才会按分区及操作追踪统计信息。

| 名称                  | 类型      | 描述                                                  |
| ------------------------------------------------------------ | ------- | ------------------------------------------------------------ |
| capacity.<operation_name>.__partition_id=<last_seven_characters_from_partition_id> | Counter | 指定分区 <partition_id> 上的 数据库表 <table_name> 上的操作 <operation_name> 的总容量   |

其他详细统计信息:

- 对于4xx响应和不完整的批处理操作失败，将在 *http.<stat_prefix>.dynamodb.error.<table_name>.* 命名空间内对指定数据库表以及故障的失败总数进行追踪。

| 名称                  | 类型      | 描述                                                  |
| --------------------------- | ------- | ------------------------------------------------------------ |
| <error_type>                | Counter | 对指定数据库表 <table_name> 发生故障  <error_type> 的总次数 |
| BatchFailureUnprocessedKeys | Counter | 对指定数据库表 <table_name> 发生不完整的批处理操作失败的次数 |

## 运行时

DynamoDB 过滤器支持以下运行时设置:

- dynamodb.filter_enabled

  启用过滤器的请求的百分比。 默认值是100％。
