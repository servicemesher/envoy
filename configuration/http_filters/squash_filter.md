# Squash

*Squash* 是 *HTTP* 过滤器，可以允许 *Envoy* 整合 Squash 微服务的调试器。具体代码可见：<https://github.com/solo-io/squash>，API Docs：<https://squash.solo.io/>

## 概述

这种过滤器主要使用场景是在 service mesh 中，例如作为将 Envoy 作为一个 sidecar 发布。在 Squash 集群服务中，当一个请求标记为调试模式进入 mesh 中，Squash Envoy 过滤器将会报告请求为 ‘location’ 模式，由于 Envoy sidecar 和应用程序之间会存在 1 比 1 的映射关系，Squash 服务可以将调试器附加到应用程序中。Squash 过滤器也可以保持请求，直到调试器被附加到应用程序容器中或发生超时。在不对集群做任何调整的情况下，允许开发者在请求到达应用程序时将本地调试器附加到请求的容器上。

## 配置

- [v1 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/squash_filter#config-http-filters-squash-v1)
- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/squash/v2/squash.proto#envoy-api-msg-config-filter-http-squash-v2-squash)

## 如何运行

当 Squash 过滤器遇到请求的header中包含 “x-squash-debug” 这样的值时，将会有如下操作：

1. 推迟请求操作
2. 链接 Squash 服务并请求创建调试附件
   - 在 Squash 服务端，Squash 服务会尝试将调试器附加到应用程序的 Envoy 代理上。如果成功，它会将调试附件的状态更改为附件
3. Squash 服务将会处于等待状态，直到更新调试附件的状态为附件或出现错误状态
4. 恢复请求操作