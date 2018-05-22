# 管理接口

Envoy 在本地提供了一个管理界面，可以使用这一界面查询或修改服务器的各种数据。

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/admin.html#config-admin-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-admin)

## 注意
目前管理界面可以进行破坏性操作（例如关闭服务器），也可能暴露私有数据（例如统计数据、集群名称、证书信息等）。将管理界面限制到只能在安全网络之内进行访问是 **非常必要** 的。同时还要注意，提供管理界面服务的网络接口只接入到安全网络之中（防止 CSRF 攻击等目的）。要实现这些目标，可以进行相应的防火墙设置，或者只允许 localhost 访问。可以使用如下的 v2 配置来完成：

```yaml
admin:
access_log_path: /tmp/admin_access.log
address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }
```

未来还会在管理界面中加入更多的安全相关的选项。这部分工作的进度在 [Github Issue #2763](https://github.com/envoyproxy/envoy/issues/2763) 上进行跟踪。
所有的变更操作都应该通过 HTTP POST 方式进行。一段时间之内还是允许 HTTP GET 访问的，但是会有一条 Warning 日志。

## `GET /`

渲染一个 HTML 主页，其中包含指向所有可用选项的链接。

## `GET /help`

以字符表格的形式输出所有可用选项。

## `GET /certs`

列出所有载入的 TLS 认证，包括文件名、序列号以及过期时间。

## `GET /clusters`

  列出[集群管理器](../intro/arch_overview/cluster_manager.md#arch-overview-cluster-manager)中配置的所有集群。这种信息中包含了已被发现的每个集群中的所有上游主机，以及每个主机的统计信息。如果服务发现出现问题需要除错，这些信息就很有帮助了。

### 集群管理器信息

`version_info`：string，最近载入的 [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) 的版本信息字符串。如果 Envoy 没有设置 [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds)，则会输出来自于 `version_info::static`。

### 集群层信息

- 各优先级的[断路器](../configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers)设置。
- 如果设置了[外部检测](../intro/arch_overview/outlier.md#arch-overview-outlier-detection)，则显示相关信息。目前包含了[平均成功率](../intro/arch_overview/outlier.md#arch-overview-outlier-detection-ejection-event-logging-cluster-success-rate-average)以及[驱逐阈值](../intro/arch_overview/outlier.md#arch-overview-outlier-detection-ejection-event-logging-cluster-success-rate-ejection-threshold)的检测器。上一个检测周期中所获取的数据量不足，这些变量的值会被设置为`-1`。
- `added_via_api` 标志：如果是静态配置添加的集群，这个项目的值就是 `false`；如果是使用 [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) API 添加的集群，这个值就会设置为 `true`。

### 主机统计数据

|名称|类型|描述|
|---|---|---|
|cx_total|Counter|连接总数|
|cx_active|Gauge|活动连接总数|
|cx_connect_fail|Counter|连接失败总数|
|rq_total|Counter|请求总数|
|rq_timeout|Counter|超时请求总数|
|rq_success|Counter|收到非 5xx 响应的请求总数|
|rq_error|Counter|收到 5xx 响应的请求总数|
|rq_active|Gauge|活动请求总数|
|healthy|String|主机健康状态，下面会有讲解|
|weight|Integer|负载均衡权重（0-100）|
|zone|String|服务区域|
|canary|Boolean|本主机是否为金丝雀|
|success_rate|Double|请求成功率（0 - 100）。如果统计周期内没有足够的请求量进行运算，则返回 -1|

### 主机健康状态

- 如果主机是健康的，那么 `healthy` 的输出为 `healthy`。
- 如果主机不健康，则 `healthy` 返回的是下面几个状态之一：
  - `/failed_active_hc`：[主动健康监测](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc)失败。
  - `/failed_eds_health`：EDS 标记该主机不健康。
  - `/failed_outlier_check`：外部检测失败。

## `GET /config_dump`

用 JSON 序列化格式从多种 Envoy 组件中导出当前的配置。可以延伸阅读 [response definition](https://www.envoyproxy.io/docs/envoy/latest/api-v2/admin/v2alpha/config_dump.proto#envoy-api-msg-admin-v2alpha-configdump) 的内容，来获得更详细的信息。

## `POST /cpuprofiler`

启用或停用 CPU Profiler。编译时需要启用 gperftools。

## `POST /healthcheck/fail`

设置健康状况为失败。需要配合 HTTP [健康检查过滤器](../configuration/http_filters/health_check_filter.md#config-http-filters-health-check)来使用。这一功能可以停用一个服务器而无需进行关闭或者重启动操作。这个命令执行之后，不论过滤器如何设置，都会把全局范围内的健康检查设为失败。

## `POST /healthcheck/ok`

`POST /healthcheck/fail` 的逆向操作。同样需要配合 HTTP [健康检查过滤器](../configuration/http_filters/health_check_filter.md#config-http-filters-health-check)来使用。

## `GET /hot_restart_version`

参考 [`--hot-restart-version`](../operations/cli#cmdoption-hot-restart-version)。

## `POST /logging`

在不同的子组件上启用或者禁用不同级别的日志。一般只会在开发期间使用。

## `POST /quitquitquit`

完全退出服务。

## `POST /reset_counters`

把所有的计数器复位为 0。这在使用 `GET /stats` 协助调试的时候是很有用的功能。注意这个功能并不会删除任何发送给 statsd 的数据，它只会对 `GET /stats` 命令的输出造成影响。

## `GET /server_info`

输出关于服务器的信息，内容格式类似：

  envoy 267724/RELEASE live 1571 1571 0

其中的字段包括：

- 进程名称
- 编译的 SHA 以及 Build 类型
- 当前热启动周期的启动时间
- 总的启动时间（包括所有的热启动周期）
- 当前的热启动周期

## `GET /stats`

按需输出所有的统计内容。这个命令对于本地调试非常有用。Histograms 能够计算分位数并进行输出，即 P0，P25，P50，P75，P90，P99，P99.9和P100。每个分位数都是一种（区间值，累计值）的形式，区间值代表的是上次刷新以后的数值，累计值代表的是该实例启动以后的总计值。[统计概览](../operations/stats_overview#operations-stats)章节中提供了更多这方面的内容。

### GET /stats?format=json

使用 JSON 格式输出 `/stats`，编程访问统计信息时可以使用这一格式。Counter 和 Gauge 会以（键，值）的形式出现。Histograms 会放在 "histograms" 节点之下，其中包含了 "supported_quantiles" 节点，其中列出了支持的分位数，以及这些分位数的计算结果。只有包含值的 Histograms 才会输出。

如果一个 Histogram 在这一区间没有更新，那么这一区间的所有分位数输出都是空的。

下面是一个输出样例：

```json
{
  "histograms": {
    "supported_quantiles": [
      0, 25, 50, 75, 90, 95, 99, 99.9, 100
    ],
    "computed_quantiles": [
      {
        "name": "cluster.external_auth_cluster.upstream_cx_length_ms",
        "values": [
          {"interval": 0, "cumulative": 0},
          {"interval": 0, "cumulative": 0},
          {"interval": 1.0435787, "cumulative": 1.0435787},
          {"interval": 1.0941565, "cumulative": 1.0941565},
          {"interval": 2.0860023, "cumulative": 2.0860023},
          {"interval": 3.0665233, "cumulative": 3.0665233},
          {"interval": 6.046609, "cumulative": 6.046609},
          {"interval": 229.57333,"cumulative": 229.57333},
          {"interval": 260,"cumulative": 260}
        ]
      },
      {
        "name": "http.admin.downstream_rq_time",
        "values": [
          {"interval": null, "cumulative": 0},
          {"interval": null, "cumulative": 0},
          {"interval": null, "cumulative": 1.0435787},
          {"interval": null, "cumulative": 1.0941565},
          {"interval": null, "cumulative": 2.0860023},
          {"interval": null, "cumulative": 3.0665233},
          {"interval": null, "cumulative": 6.046609},
          {"interval": null, "cumulative": 229.57333},
          {"interval": null, "cumulative": 260}
        ]
      }
    ]
  }
}
```

### `GET /stats?format=prometheus`

或者换个方式 `GET /stats/prometheus`，使用 [Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/) v0.0.4 格式输出统计数据。这样就可以和 Prometheus 进行集成了。当前只有 Counter 和 Gauge 会进行输出。未来的更新中会输出 Histogram。

### `GET /runtime`

使用 JSON 格式按需输出所有运行时数值。[运行时配置](../intro/arch_overview/runtime#arch-overview-runtime)一节中，更详细的讲述了这些值的配置和使用。输出内容包括活动的重载后的运行时数值，以及每个键的堆栈。空字符串代表没有值，来自堆栈的最终有效值会用单独的键来做出标识，例如下面的输出：

```json
{
  "layers": [
    "disk",
    "override",
    "admin",
  ],
  "entries": {
    "my_key": {
      "layer_values": [
        "my_disk_value",
        "",
        ""
      ],
      "final_value": "my_disk_value"
    },
    "my_second_key": {
      "layer_values": [
        "my_second_disk_value",
        "my_disk_override_value",
        "my_admin_override_value"
      ],
      "final_value": "my_admin_override_value"
    }
  }
}
```

## `POST /runtime_modify?key1=value1&key2=value2&keyN=valueN`

通过提交的参数对运行时数值进行添加或修改。要删除一个之前加入的键，只需要使用一个空值即可。注意这种删除操作，只适用于这一端点中使用重载方式加入的值；从磁盘中载入的值是能通过重载进行修改，无法删除。

> **注意**
>
> 使用 /runtime_modify 端点要当心，这一变更是即时生效的。保障管理界面的[安全性](../operations/admin#operations-admin-interface-security)至关重要。