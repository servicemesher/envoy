# Redis

The Redis health checker is a custom health checker which checks Redis upstream hosts. It sends a Redis PING command and expect a PONG response. The upstream Redis server can respond with anything other than PONG to cause an immediate active health check failure. Optionally, Envoy can perform EXISTS on a user-specified key. If the key does not exist it is considered a passing healthcheck. This allows the user to mark a Redis instance for maintenance by setting the specified [key](../../api-v2/config/health_checker/redis/v2/redis.proto.md#envoy-api-field-config-health-checker-redis-v2-redis-key) to any value and waiting for traffic to drain.

- [v2 API reference](../../api-v2/api/v2/core/health_check.proto.md#envoy-api-msg-core-healthcheck-customhealthcheck)