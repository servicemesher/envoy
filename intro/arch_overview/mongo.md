# MongoDB

Envoy 支持网络层的 MongoDB 嗅探过滤器,并拥有如下特性：

- MongoDB wire 格式的 BSON 转换器。
- 考虑周全的 MongoDB 查询/操作统计分析报告，包括了时间以及路由集群中的散列/多次获取次数。
- 查询记录。
- 通过 $comment 参数做每个调用点的统计分析报告。
- 故障注入。

MongoDB 过滤器是表现 Envoy 扩展性以及核心抽象能力的典范案例。在 Lyft 我们将这个过滤器应用在所有的应用以及数据库中。
在不了解应用平台以及所使用 MongoDB 驱动的情况下，它依然提供了不可估量的数据源。

MongoDB 代理过滤器[参考配置](../../configuration/network_filters/mongo_proxy_filter。md#config-network-filters-mongo-proxy)。