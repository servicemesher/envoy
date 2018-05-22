# 健康检查
健康检查[架构概览](../../intro/arch_overview/health_checking.md#arch-overview-health-checking).

- 如果为集群配置运行状况检查，会有额外的数据发出.他们记录在[here](cluster_stats.md#config-cluster-manager-cluster-stats).
- [v1 API documentation](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-v1).
- [v2 API documentation](../../api-v2/api/v2/core/health_check.proto.md#envoy-api-msg-core-healthcheck).

## TCP 健康检查
> 注意
本节是为v1 API编写的，但这些概念也适用于v2 API.针对V2 API将在未来的正式版中重新定义

执行的匹配类型如下（这是MongoDB运行状况检查请求和响应）：
```json
 {
   "send": [
     {"binary": "39000000"},
     {"binary": "EEEEEEEE"},
     {"binary": "00000000"},
     {"binary": "d4070000"},
     {"binary": "00000000"},
     {"binary": "746573742e"},
     {"binary": "24636d6400"},
     {"binary": "00000000"},
     {"binary": "FFFFFFFF"},
     {"binary": "13000000"},
     {"binary": "01"},
     {"binary": "70696e6700"},
     {"binary": "000000000000f03f"},
     {"binary": "00"}
    ],
    "receive": [
     {"binary": "EEEEEEEE"},
     {"binary": "01000000"},
     {"binary": "00000000"},
     {"binary": "0000000000000000"},
     {"binary": "00000000"},
     {"binary": "11000000"},
     {"binary": "01"},
     {"binary": "6f6b"},
     {"binary": "00000000000000f03f"},
     {"binary": "00"}
    ]
}
```
在每个运行状况检查周期中，所有"send"字节都会发送到目标服务器。每个二进制块的长度可以是任意的，并且在发送时只是连接在一起。（分离成多个块可用于可读性）。

在检查响应时，执行“模糊”匹配，以便每个二进制块必须被找到，并且按照指定的顺序，但不一定是连续的。因此，在上面的示例中，可以在“EEEEEEEE”和“01000000”之间的响应中插入“FFFFFFFF”，并且该检查仍然会通过。这样做是为了支持将非确定性数据（如时间）插入到响应中的协议。

健康检查需要更复杂的模式，如发送/接收/发送/接收目前不可能。

如果“receive ”是一个空数组，则Envoy将执行"connect only " TCP健康检查。在每个周期中，Envoy将尝试连接到上游主机，并且如果连接成功，则认为它是成功的。每个健康检查周期都会创建一个新连接。
