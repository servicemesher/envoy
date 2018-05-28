# 负载均衡

当过滤器需要获取到上游集群中主机的连接时，集群管理器将使用负载均衡策略来确定选择哪个主机。负载均衡策略是可拔插的，并且在[配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster.htmlhttps://www.envoyproxy.io/docs/envoy/latest/#config-cluster-manager-cluster)中以每个上游集群为基础进行指定。请注意，如果没有为集群[配置](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc)活动的运行状况检查策略，则所有上游集群成员都认为是健康的。

## 支持的负载均衡器

### Round robin

这是一个简单的策略，每个健康的上游主机按循环顺序选择。如果将[权重](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto.html#envoy-api-field-endpoint-lbendpoint-load-balancing-weight)分配给本地的端点，则使用加权 round robin（循环）调度，其中较高权重的端点将更频繁地出现在循环中以实现有效权重。

### 加权最少请求

请求最少的负载均衡器使用 O(1) 算法选择两个随机健康主机，并选择主动请求较少的主机（[研究](http://www.eecs.harvard.edu/~michaelm/postscripts/handbook2001.pdf)表明这种方法几乎与 O(N) 全扫描一样好）。如果集群中的任何主机的负载均衡权重大于1，则负载均衡器将转换为随机选择主机并使用该主机<权重>时间的模式。该算法对于负载测试来说简单且足够。它不应该用于需要真正的加权最小请求的地方（通常请求持续时间可变且较长）。我们可能会在将来添加一个真正的全扫描加权最小请求变体来涵盖此用例。

### 环哈希

Ring/modulo 哈希负载均衡器实现对上游主机的一致性哈希。该算法基于将所有主机映射到一个环上，使得从主机集添加或移除主机的更改仅影响 1/N 个请求。这种技术通常也被称为 “[ketama](https://github.com/RJ/ketama)” 哈希。一致的哈希负载均衡器只有在使用指定哈希值的协议路由时才有效。最小环大小控制环中每个主机的复制因子。例如，如果最小环大小为 1024 并且有 16 个主机，则每个主机将被复制 64 次。环哈希负载均衡器当前不支持加权。

当使用基于优先级的负载均衡时，优先级也通过哈希选择，因此当后端集合稳定时，选定的端点仍将保持一致。

> **注意**
>
> 环哈希负载均衡器不支持所在地加权负载均衡。

### Maglev

Maglev（磁悬浮）负载均衡器对上游主机实施一致性的哈希。它使用[本文](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf)第 3.4 节中描述的算法，固定表大小为65537（参见同一论文的第5.3节）。Maglev 可以用作环哈希负载均衡器的替代品，可以在任何需要一致性哈希的地方使用。就像环哈希负载均衡器一样，只有在使用指定哈希值的协议路由时，一致性哈希负载均衡器才有效。

一般来说，与环形散列（“ketama”）算法相比，Maglev 具有快得多的查表编译时间以及主机选择时间（当使用 256K 条目的大环时大约分别为 10 倍和 5 倍）。Maglev 的缺点是它不像环哈希那样稳定。当主机被移除时，更多的键将移动位置（模拟显示键将移动大约两倍）。据说，对于包括 Redis 在内的许多应用程序来说，Maglev 很可能是环形哈希替代品的一大优势。高级读者可以使用这个 [benchmark](https://github.com/envoyproxy/envoy/blob/master//test/common/upstream/load_balancer_benchmark.cc) 来比较具有不同参数的环形哈希与  Maglev。

### 随机

随机负载均衡器选择一个随机的健康主机。如果没有配置健康检查策略，则随机负载均衡器通常比 round robin 更好。随机选择可以避免主机出现故障后对集合中的主机造成偏见。

### 原始目的地

这是一种专用的负载均衡器，只能与[原始目的集群](service_discovery.md#arch-overview-service-discovery-types-original-destination)一起使用。上游主机是根据下游连接元数据选择的，即连接被打开到与连接重定向到 envoy 之前传入与连接的目标地址相同的地址。新的目标由负载均衡器按需添加到集群，并且集群会[定期](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster#config-cluster-manager-cluster-cleanup-interval-ms)清除集群中未使用的主机。原始目标集群不能使用其他[负载均衡类型](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster#config-cluster-manager-cluster-lb-type)。

## 恐慌阈值

在负载均衡期间，Envoy 通常只会考虑上游集群中健康的主机。但是，如果集群中健康主机的百分比变得过低，envoy 将忽视所有主机中的健康状况和均衡。这被称为*恐慌阈值*。缺省恐慌阈值是 50％。这可以通过运行时以及[集群配置](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#envoy-api-field-cluster-commonlbconfig-healthy-panic-threshold)进行[配置](../../configuration/cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime)。恐慌阈值用于避免在负载增加时主机故障导致整个集群中级联故障的情况。

请注意，恐慌阈值是有*优先级*的。这意味着如果单个优先级中健康节点的百分比低于阈值，该优先级将进入恐慌模式。一般而言，不鼓励将恐慌阈值与优先级结合使用，因为当有足够多的节点的状态不健康触发恐慌阈值时，大部分流量应该已经溢出到下一个优先级。

## 优先级划分

在负载均衡期间，Envoy 通常只考虑配置为最高优先级的主机。对于每个 EDS [LocalityLbEndpoints](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto#envoy-api-msg-endpoint-localitylbendpoints)，还可以指定一个可选的优先级。当最高优先级（P=0）的端点健康时，所有流量将以该优先级落在端点上。由于最高优先级的端点变得不健康，因此流量将开始慢慢降低优先级。

目前，假定每个优先级级别都由因子 1.4（硬编码）过度配置的。因此，如果 80％ 的端点是健康的，那么优先级仍然被认为是健康的，因为 80*1.4>100。当健康端点的数量下降到 72％ 以下时，优先级的健康状况低于100。此时，相当于到 P=0 健康状态的流量的百分比将转到 P=0，剩余流量将流向 P=1。

假设一个简单的设置有 2 个优先级，P=1 100％ 健康。

| P=0 健康端点 | 到 P=0 的流量百分比 | 到 P=1 的流量百分比 |
| ------------ | ------------------- | ------------------- |
| 100%         | 100%                | 0%                  |
| 72%          | 100%                | 0%                  |
| 71%          | 99%                 | 1%                  |
| 50%          | 70%                 | 30%                 |
| 25%          | 35%                 | 65%                 |
| 0%           | 0%                  | 100%                |

如果 P=1 变得不健康，它将继续从 P=0 溢出负载，直到健康 P=0 + P=1 的总和低于 100 为止。此时，健康状态将被放大到 100％ 的“有效”健康状态。

| P=0 健康端点 | P=1 健康端点 | Traffic to P=0 | 到 P=1 的流量 |
| ------------ | ------------ | -------------- | ------------- |
| 100%         | 100%         | 100%           | 0%            |
| 72%          | 72%          | 100%           | 0%            |
| 71%          | 71%          | 99%            | 1%            |
| 50%          | 50%          | 70%            | 30%           |
| 25%          | 100%         | 35%            | 65%           |
| 25%          | 25%          | 50%            | 50%           |

随着更多的优先级被添加，每个级别消耗等于其“缩放”有效健康的负载，所以如果 P=0 + P=1 的组合健康度小于 100，则 P=2 将仅接收流量。

| P=0 健康端点 | P=1 健康端点 | P=2 健康端点 | 到 P=0 的流量 | 到 P=1 的流量 | 到 P=2 的流量 |
| ------------ | ------------ | ------------ | ------------- | ------------- | ------------- |
| 100%         | 100%         | 100%         | 100%          | 0%            | 0%            |
| 72%          | 72%          | 100%         | 100%          | 0%            | 0%            |
| 71%          | 71%          | 100%         | 99%           | 1%            | 0%            |
| 50%          | 50%          | 100%         | 70%           | 30%           | 0%            |
| 25%          | 100%         | 100%         | 35%           | 65%           | 0%            |
| 25%          | 25%          | 100%         | 25%           | 25%           | 50%           |

在伪代码中加和：

```bash
load to P_0 = min(100, health(P_0) * 100 / total_health)
health(P_X) = 140 * healthy_P_X_backends / total_P_X_backends
total_health = min(100, Σ(health(P_0)...health(P_X))
load to P_X = 100 - Σ(percent_load(P_0)..percent_load(P_X-1))
```

## Zone 感知路由

我们使用以下术语：

- **始发/上游集群**：Envoy 将来自始发集群的请求路由到上游集群中。
- **本地 zone**：包含始发和上游集群中主机子集的同一区域。
- **Zone 感知路由**：尽量将请求路由到本地 zone 的上游集群主机上。

当始发和上游集群中的主机部署不同 zone 时，Envoy 执行 zone 感知路由。在执行 zone 感知路由之前有几个先决条件：

- 始发和上游集群都不处于恐慌模式。
- Zone [感知路由已启用](../../configuration/cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime-zone-routing)。
- 始发集群与上游集群具有相同数量的 zone。
- 上游集群有足够多的主机。浏览[此处](../../configuration/cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime-zone-routing)获取更多信息。

Zone 感知路由的目的是尽可能多地向上游集群的本地 zone 中发送流量，同时大致保持上游集群中的所有主机拥有相同的（取决于负载均衡策略）的每秒请求数量。

只要上游集群中每台主机的请求数量保持大致相同，Envoy 就会尝试尽可能多地将流量推送到本地上游区域。Envoy 路由到本地 zone 还是执行跨 zone 路由，这取决于本地 zone 中始发集群和上游集群中健康主机的百分比。关于始发和上游集群之间的本地 zone 百分比关系有以下两种情况：

- 始发集群的本地 zone 百分比大于上游集群中本地 zone 的百分比。在这种情况下，我们无法将来自始发集群本地 zone 的所有请求路由到上游集群的本地 zone，因为这会导致所有上游主机请求不均衡。相反，Envoy 计算可以直接路由到上游集群的本地 zone 的请求的百分比。其余的请求被路由到跨 zone。特定 zone 根据 zone 的剩余容量（该 zone 将获得一些本地 zone 流量并且可能具有 Envoy 可用于跨 zone 业务量的额外容量）来选择。
- 始发集群本地 zone 百分比小于上游集群中的本地 zone 百分比。在这种情况下，上游集群的本地 zone 可以获取来自始发集群本地 zone 的所有请求，并且还有一定空间允许来自集群中其他 zone 的流量（如果有必要的话）。

请注意，使用多个优先级时，zone 感知路由当前仅支持 P=0。

## 所在地加权负载均衡

另一种用于确定如何在不同 zone 和地理位置之间分配权重的方式是使用 [LocalityLbEndpoints](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto#envoy-api-msg-endpoint-localitylbendpoints) 消息中通过 EDS 提供的显式权重。这种方式与上述 zone 感知路由相互排斥，因为在所在地感知 LB 的情况下，我们通过管理服务器来提供所在地加权，而不是在 zone 感知路由中使用的 Envoy 侧启发式的方式。

当所有端点健康时，使用加权循环 round-robin 来挑选本地节点，其中地点权重用于加权。当某地的某些端点不健康时，我们通过调整地点的权重来反映这一点。与优先级一样，我们设置了一个过度提供因子（目前硬编码为 1.4），这意味着当一个地区只有少数端点不健康时，我们不会进行任何权重调整。

假设一个简单的设置，包含 2 个地点 X 和 Y，其中 X 的所在地权重为 1，Y 的所在地权重为 2，L=Y 100％ 健康。

| L=X 健康端点 | 到 L=X 的流量百分比 | 到 L=Y 的流量百分比 |
| ------------ | ------------------- | ------------------- |
| 100%         | 33%                 | 67%                 |
| 70%          | 33%                 | 67%                 |
| 69%          | 32%                 | 68%                 |
| 50%          | 26%                 | 74%                 |
| 25%          | 15%                 | 85%                 |
| 0%           | 0%                  | 100%                |

在伪代码中加和：

```bash
health(L_X) = 140 * healthy_X_backends / total_X_backends
effective_weight(L_X) = locality_weight_X * min(100, health(L_X))
load to L_X = effective_weight(L_X) / Σ_c(effective_weight(L_c))
```

请注意，在挑选优先级之后进行所在地加权选取。负载均衡器遵循以下步骤：

1. 挑选优先级别。
2. 从（1）中选择优先级别的所在地（如本节所述）。
3. 从（2）中选择使用集群指定的负载均衡器所在地范围内的端点。

通过在集群配置中设置 [locality_weighted_lb_config](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#envoy-api-field-cluster-commonlbconfig-locality-weighted-lb-config) 并通过 [load_balancing_weight](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto#envoy-api-field-endpoint-localitylbendpoints-load-balancing-weight) 在 [LocalityLbEndpoints](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto#envoy-api-msg-endpoint-localitylbendpoints) 中提供权重来配置所在地加权负载均衡。

此功能与负载均衡器子集设置不兼容，因为将个别子集的所在地级权重与显而易见的权重进行协调并不容易。

## 负载均衡器子集

根据附加在主机上的元数据将上游集群中的主机划分为子集，可以这样来配置 Envoy。然后，路由可以指定主机必须匹配的元数据，有了这些元数据负载均衡器才能选择路由，也可以选择回退到预定义的主机集（包括任何主机）。

子集使用集群指定的负载均衡器策略。原始目的地策略不能与子集一起使用，因为上游主机预先不知道这些策略。子集与 zone 感知路由兼容，但请注意，子集的使用可能很容易违反上述最小主机条件。

如果[已配置的](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#envoy-api-field-cluster-lb-subset-config)子集路由未指定元数据或没有匹配元数据的子集存在，则子集负载均衡器将启动其回退策略。默认策略是`NO_ENDPOINT`，在这种情况下，请求失败，就好像该集群没有主机一样。相反，`ANY_ENDPOINT` 后备策略会在集群中的所有主机上进行负载均衡，而不考虑主机元数据。最后，`DEFAULT_SUBSET` 会导致回退到与特定元数据集匹配的主机之间进行负载均衡。

子集必须被预定义才能让子集负载均衡器有效地选择正确的主机子集。每个定义都是一组密钥，可以转换为零个或多个子集。从概念上讲，具有定义中所有键的元数据值的每个主机都会添加到其键-值对特定的子集。如果所有主机都不拥有密钥，那么定义不会产生子集。可以提供多个定义，并且如果单个主机与多个定义匹配，则可以在多个子集中出现。

在路由期间，路由的元数据匹配配置将用于查找特定的子集。如果存在具有路由指定的确切密钥和值的子集，则该子集将用于负载均衡。否则，使用回退策略。因此，集群的子集配置必须包含一个与给定路由具有相同密钥的定义才能实现子集负载均衡。

此功能只能在 V2 配置 API 中使用。此外，主机元数据仅在集群使用 EDS 发现类型时支持。子集负载均衡的主机元数据必须放在过滤器名称 `"envoy.lb"` 下。同样，路由元数据匹配条件使用 `"envoy.lb"` 过滤器名称。主机元数据可以是分层的（例如，顶级密钥的值可以是结构化值或列表），但子集负载均衡器仅比较顶级密钥和值。因此，在使用结构化值时，如果主机的元数据中出现相同的结构化值，则路由的匹配条件会匹配。

### 示例

我们将使用所有值都是字符串的简单元数据。假设定义了以下主机并将其与集群关联：

| 主机  | 元数据                 |
| ----- | ---------------------- |
| host1 | v: 1.0, stage: prod    |
| host2 | v: 1.0, stage: prod    |
| host3 | v: 1.1, stage: canary  |
| host4 | v: 1.2-pre, stage: dev |

集群可以像这样启用子集负载均衡：

```yaml
---
name: cluster-name
type: EDS
eds_cluster_config:
  eds_config:
    path: '.../eds.conf'
connect_timeout:
  seconds: 10
lb_policy: LEAST_REQUEST
lb_subset_config:
  fallback_policy: DEFAULT_SUBSET
  default_subset:
    stage: prod
  subset_selectors:
  - keys:
    - v
    - stage
  - keys:
    - stage
```

下表描述了一些路由及其对集群的应用结果。通常，匹配标准将与匹配请求的特定方面的路由一起使用，例如路径或 header 信息。

| 匹配标准               | 负载均衡位于 | 原因                              |
| ---------------------- | ------------ | --------------------------------- |
| stage: canary          | host3        | 所选主机的子集                    |
| v: 1.2-pre, stage: dev | host4        | 所选主机的子集                    |
| v: 1.0                 | host1, host2 | 回退：没有仅具有 “v” 的子集选择器 |
| other: x               | host1, host2 | 回退：没有 “other” 子集选择器     |
| (none)                 | host1, host2 | 回退：未请求子集                  |

元数据匹配标准也可以在路由的加权集群上指定。来自选定加权集群的元数据匹配条件将与路由中的条件合并，并覆盖该原路由器中的条件：

| 路由匹配条件        | 加权集群匹配条件      | 最终匹配条件          |
| ------------------- | --------------------- | --------------------- |
| stage: canary       | stage: prod           | stage: prod           |
| v: 1.0              | stage: prod           | v: 1.0, stage: prod   |
| v: 1.0, stage: prod | stage: canary         | v: 1.0, stage: canary |
| v: 1.0, stage: prod | v: 1.1, stage: canary | v: 1.1, stage: canary |
| (none)              | v: 1.0                | v: 1.0                |
| v: 1.0              | (none)                | v: 1.0                |

#### 具有元数据的示例主机

具有主机元数据的EDS `LbEndpoint`：

```yaml
---
endpoint:
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 8888
metadata:
  filter_metadata:
    envoy.lb:
      version: '1.0'
      stage: 'prod'
```

#### 具有元数据匹配标准的示例路由

具有元数据匹配标准的 RDS `Route`：

```yaml
---
match:
  prefix: /
route:
  cluster: cluster-name
  metadata_match:
    filter_metadata:
      envoy.lb:
        version: '1.0'
        stage: 'prod'
```
