# Redis

Redis 健康检查器是一个用于检查 Redis 上游主机的定制检查器。主要通过发送 PING 命令和接收 PONG 命令进行工作。上游 Redis 主机可以使用 PONG 以外的任何其他响应来导致立即激活的运行状况检查失败。或者，Envoy 可以在用户指定的密钥上执行 EXISTS。如果密钥不存在，则认为它是合格的健康检查。这允许用户通过将执行的密钥设置为任意值并等待流量耗尽来标记 Redis 实例以进行维护。

- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/health_check.proto#envoy-api-msg-core-healthcheck-customhealthcheck)
