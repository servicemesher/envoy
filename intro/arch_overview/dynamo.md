# DynamoDB

Envoy 支持一个 HTTP 级 DynamoDB 嗅探过滤器，该过滤器有以下特性：

- DynamoDB API 请求/响应解析。
- DynamoDB 按每操作、每数据库表、每分区以及操作的统计信息。
- 故障类型统计覆盖 4xx 响应、由响应 JSON 解析，例如 ProvisionedThroughputExceededException。
- 批处理操作部分失败的统计信息。

它为应用程序平台以及特定的 AWS SDK 提供了宝贵的对数据无感知的来源。

DynamoDB 过滤器 [配置](../../configuration/http_filters/dynamodb_filter.md#config-http-filters-dynamo)。