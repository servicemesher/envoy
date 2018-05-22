# Squash

Squash is an HTTP filter which enables Envoy to integrate with Squash microservices debugger. Code: <https://github.com/solo-io/squash>, API Docs: <https://squash.solo.io/>

Squash 是一个 HTTP 过滤器，可以支持 Envoy

## 概述

The main use case for this filter is in a service mesh, where Envoy is deployed as a sidecar. Once a request marked for debugging enters the mesh, the Squash Envoy filter reports its ‘location’ in the cluster to the Squash server - as there is a 1-1 mapping between Envoy sidecars and application containers, the Squash server can find and attach a debugger to the application container. The Squash filter also holds the request until a debugger is attached (or a timeout occurs). This enables developers (via Squash) to attach a native debugger to the container that will handle the request, before the request arrive to the application code, without any changes to the cluster.

## 配置

- [v1 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/squash_filter#config-http-filters-squash-v1)
- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/squash/v2/squash.proto#envoy-api-msg-config-filter-http-squash-v2-squash)

## 如何运行

当 Squash 过滤器遇到 请求的 header中包含 ‘x-squash-debug’ 这样的值时，将会有如下操作：

1. 推迟请求的进入。
2. 链接 Squash 服务并请求创建调试附件
   - 在 Squash 服务端，Squash 服务会尝试将调试器附加到应用程序的 Envoy 代理上。如果成功，它会将调试附件的状态更改为附件
3. Squash 服务将会处于等待状态，直到更新调试附件的状态为附件或出现错误状态
4. 恢复进入的请求
