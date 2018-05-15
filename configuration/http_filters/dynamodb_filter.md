# DynamoDB

- DynamoDB [architecture overview](../../intro/arch_overview/dynamo.html#arch-overview-dynamo)
- [v1 API reference](../../api-v1/http_filters/dynamodb_filter.html#config-http-filters-dynamo-v1)
- [v2 API reference](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpfilter-name)

## Statistics

The DynamoDB filter outputs statistics in the *http.<stat_prefix>.dynamodb.* namespace. The [stat prefix](../../api-v1/network_filters/http_conn_man.html#config-http-conn-man-stat-prefix) comes from the owning HTTP connection manager.

Per operation stats can be found in the *http.<stat_prefix>.dynamodb.operation.<operation_name>.* namespace.

| Name                  | Type      | Description                                                  |
| --------------------- | --------- | ------------------------------------------------------------ |
| upstream_rq_total     | Counter   | Total number of requests with <operation_name>               |
| upstream_rq_time      | Histogram | Time spent on <operation_name>                               |
| upstream_rq_total_xxx | Counter   | Total number of requests with <operation_name> per response code (503/2xx/etc) |
| upstream_rq_time_xxx  | Histogram | Time spent on <operation_name> per response code (400/3xx/etc) |

Per table stats can be found in the *http.<stat_prefix>.dynamodb.table.<table_name>.* namespace. Most of the operations to DynamoDB involve a single table, but BatchGetItem and BatchWriteItem can include several tables, Envoy tracks per table stats in this case only if it is the same table used in all operations from the batch.

| Name                  | Type      | Description                                                  |
| --------------------- | --------- | ------------------------------------------------------------ |
| upstream_rq_total     | Counter   | Total number of requests on <table_name> table               |
| upstream_rq_time      | Histogram | Time spent on <table_name> table                             |
| upstream_rq_total_xxx | Counter   | Total number of requests on <table_name> table per response code (503/2xx/etc) |
| upstream_rq_time_xxx  | Histogram | Time spent on <table_name> table per response code (400/3xx/etc) |

*Disclaimer: Please note that this is a pre-release Amazon DynamoDB feature that is not yet widely available.* Per partition and operation stats can be found in the *http.<stat_prefix>.dynamodb.table.<table_name>.* namespace. For batch operations, Envoy tracks per partition and operation stats only if it is the same table used in all operations.

| Name                                                         | Type    | Description                                                  |
| ------------------------------------------------------------ | ------- | ------------------------------------------------------------ |
| capacity.<operation_name>.__partition_id=<last_seven_characters_from_partition_id> | Counter | Total number of capacity for <operation_name> on <table_name> table for a given <partition_id> |

Additional detailed stats:

- For 4xx responses and partial batch operation failures, the total number of failures for a given table and failure are tracked in the *http.<stat_prefix>.dynamodb.error.<table_name>.*namespace.

| Name                        | Type    | Description                                                  |
| --------------------------- | ------- | ------------------------------------------------------------ |
| <error_type>                | Counter | Total number of specific <error_type> for a given <table_name> |
| BatchFailureUnprocessedKeys | Counter | Total number of partial batch failures for a given <table_name> |

## Runtime

The DynamoDB filter supports the following runtime settings:

- dynamodb.filter_enabled

  The % of requests for which the filter is enabled. Default is 100%.
