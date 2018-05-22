# Redis

Envoy 可以作为 Redis 的代理，在集群的实例间对命令进行分区。在这种模式下，Envoy 的目标是保持可用性和分区容错的一致性。这是将 Envoy 和 [Redis 集群](https://redis.io/topics/cluster-spec) 进行比较时的重点。Envoy 被设计为尽力而为的缓存，这意味着它不会尝试协调不一致的数据或保持全局一致的集群成员关系视图。

Redis 项目中提供了与 Redis 分区相关的全面参考。请参阅 ”[Partitioning: how to split data among multiple Redis instances](https://redis.io/topics/partitioning)“。

**Envoy Redis 的特点**:

- [Redis 协议](https://redis.io/topics/protocol) 编解码器
- 基于散列的分区
- Ketama 一致性哈希算法
- 详细的命令统计
- 主动和被动的健康检查

**有计划的未来增强功能**:

- 额外的时间统计
- 断路器
- 对分散的命令进行请求折叠
- 复制
- 内置的重试功能
- 跟踪
- 哈希标记

## 配置

有关过滤器配置的详细信息，请参阅 [Redis 代理过滤器配置参考](../../configuration/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy)。

相应的集群定义应该配置为 [环哈希负载均衡](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/load_balancing#arch-overview-load-balancing-types)。

如果需要进行主动健康检查，则应该对集群配置使用 [Redis 健康检查](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc)。

如果需要被动健康检查，还需要配置 [异常检测](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection)。

为了进行被动健康检查，需要将连接超时，命令超时和连接关闭都映射到 5xx 响应，而来自 Redis 的所有其他响应都视为成功。

## 支持的命令

在协议层面，支持流水线，但不支持 MULTI(事务块)。应该尽可能的使用流水线以获得最佳性能。

在命令层面，Envoy 仅支持可能的散列到服务器的命令。PING 命令是唯一的例外，Envoy 将立即使用 PONG 作为回复。对 PING 命令使用参数是不被支持的。所有其他支持的命令都必须包含一个键。受支持的命令在功能上与原始的 Redis 命令相同，除非出现了故障情况。

有关每个命令用法的详细信息，请参阅官方的 [Redis 命令参考](https://redis.io/commands)。

| Command              | Group      |
| -------------------- | ---------- |
| PING                 | Connection |
| DEL                  | Generic    |
| DUMP                 | Generic    |
| EXISTS               | Generic    |
| EXPIRE               | Generic    |
| EXPIREAT             | Generic    |
| PERSIST              | Generic    |
| PEXPIRE              | Generic    |
| PEXPIREAT            | Generic    |
| PTTL                 | Generic    |
| RESTORE              | Generic    |
| TOUCH                | Generic    |
| TTL                  | Generic    |
| TYPE                 | Generic    |
| UNLINK               | Generic    |
| GEOADD               | Geo        |
| GEODIST              | Geo        |
| GEOHASH              | Geo        |
| GEOPOS               | Geo        |
| GEORADIUS_RO         | Geo        |
| GEORADIUSBYMEMBER_RO | Geo        |
| HDEL                 | Hash       |
| HEXISTS              | Hash       |
| HGET                 | Hash       |
| HGETALL              | Hash       |
| HINCRBY              | Hash       |
| HINCRBYFLOAT         | Hash       |
| HKEYS                | Hash       |
| HLEN                 | Hash       |
| HMGET                | Hash       |
| HMSET                | Hash       |
| HSCAN                | Hash       |
| HSET                 | Hash       |
| HSETNX               | Hash       |
| HSTRLEN              | Hash       |
| HVALS                | Hash       |
| LINDEX               | List       |
| LINSERT              | List       |
| LLEN                 | List       |
| LPOP                 | List       |
| LPUSH                | List       |
| LPUSHX               | List       |
| LRANGE               | List       |
| LREM                 | List       |
| LSET                 | List       |
| LTRIM                | List       |
| RPOP                 | List       |
| RPUSH                | List       |
| RPUSHX               | List       |
| EVAL                 | Scripting  |
| EVALSHA              | Scripting  |
| SADD                 | Set        |
| SCARD                | Set        |
| SISMEMBER            | Set        |
| SMEMBERS             | Set        |
| SPOP                 | Set        |
| SRANDMEMBER          | Set        |
| SREM                 | Set        |
| SSCAN                | Set        |
| ZADD                 | Sorted Set |
| ZCARD                | Sorted Set |
| ZCOUNT               | Sorted Set |
| ZINCRBY              | Sorted Set |
| ZLEXCOUNT            | Sorted Set |
| ZRANGE               | Sorted Set |
| ZRANGEBYLEX          | Sorted Set |
| ZRANGEBYSCORE        | Sorted Set |
| ZRANK                | Sorted Set |
| ZREM                 | Sorted Set |
| ZREMRANGEBYLEX       | Sorted Set |
| ZREMRANGEBYRANK      | Sorted Set |
| ZREMRANGEBYSCORE     | Sorted Set |
| ZREVRANGE            | Sorted Set |
| ZREVRANGEBYLEX       | Sorted Set |
| ZREVRANGEBYSCORE     | Sorted Set |
| ZREVRANK             | Sorted Set |
| ZSCAN                | Sorted Set |
| ZSCORE               | Sorted Set |
| APPEND               | String     |
| BITCOUNT             | String     |
| BITFIELD             | String     |
| BITPOS               | String     |
| DECR                 | String     |
| DECRBY               | String     |
| GET                  | String     |
| GETBIT               | String     |
| GETRANGE             | String     |
| GETSET               | String     |
| INCR                 | String     |
| INCRBY               | String     |
| INCRBYFLOAT          | String     |
| MGET                 | String     |
| MSET                 | String     |
| PSETEX               | String     |
| SET                  | String     |
| SETBIT               | String     |
| SETEX                | String     |
| SETNX                | String     |
| SETRANGE             | String     |
| STRLEN               | String     |

## 失败模式

如果 Redis 抛出了错误，我们会将这个错误作为响应传递给命令。Envoy 将来自 Redis 的响应与错误数据类型视为正常响应，并将它传递给调用者。

Envoy 也会产生自己的错误来响应客户端。

| 错误                                 | 含义                                                      |
| ------------------------------------- | ------------------------------------------------------------ |
| 没有上游主机                      | 环哈希负载均衡器在为键的选择的位置上没有可用的主机 |
| 上游主机错误                      | 后端没有在超时期限内进行响应或关闭连接 |
| 无效的请求                       | 由于数据类型或长度的原因，命令在命令拆分器的第一阶段被拒绝 |
| 不支持的命令                   | 该命令未被 Envoy 识别，因此不能被服务，因为它不能被哈希到后端的服务器 |
| 返回了多个错误                | 分段的命令将会组合多个响应(例如 DEL 命令)，将返回接收到的错误总数 |
| 上游协议错误              | 分段命令收到意外的数据类型或后端响应的数据不符合 Redis 协议 |
| 命令参数数量错误 | Envoy 中的命令参数数量检查未通过 |

在 MGET 命令中，每个无法被获取的键都会产生一个错误响应。例如，如果我们获取五个键的指并且其中有两个键出现了后端响应超时的错误，我们将会获得每个值得错误响应信息。

```bash
$ redis-cli MGET a b c d e
1) "alpha"
2) "bravo"
3) (error) upstream failure
4) (error) upstream failure
5) "echo"
```