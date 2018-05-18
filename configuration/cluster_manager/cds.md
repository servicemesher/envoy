# 集群发现服务（CDS）


集群发现服务（CDS）是一个可选的API，Envoy将调用该API来动态获取集群管理成员。Envoy 还将根据 API 响应协调集群管理，根据需要完成添加，修改或删除已知的集群。


> **注意**
>
> 在 Envoy 配置中静态定义的任何群集都不能通过 CDS API 进行修改或删除。

- [v1 CDS API](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cds.md#config-cluster-manager-cds-v1)
- [v2 CDS API](../overview/v2_overview.md#v2-grpc-streaming-endpoints)

## 统计

CDS 的统计树以 `cluster_manager.cds.` 为根，统计如下：

| 名字           | 类型    | 描述                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| config_reload  | 计数器 | 因配置不同而导致配置重新加载的总次数 |
| update_attempt | 计数器 | 尝试调用配置加载API的总次数                                  |
| update_success | 计数器 | 调用配置加载API成功的总次数                     |
| update_failure | 计数器 | 调用配置加载API失败的总次数（网络或参数错误）       |
| version        | 测量   | 来自上次成功调用配置加载API的内容哈希      |
