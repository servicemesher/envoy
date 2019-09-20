# 参考配置

源代码分发包里为以下三种 Envoy 的主要部署类型准备了一整套的配置案例模版：

- [服务间](../intro/deployment_types/service_to_service.md#deployment-type-service-to-service)
- [前端代理](../intro/deployment_types/front_proxy.md#deployment-type-front-proxy)
- [双向代理](../intro/deployment_types/double_proxy.md#deployment-type-double-proxy)

这一套的配置案例模版主要用来展现 Envoy 在复杂部署下的全部能力。并不是所有的功能点都能适用于所有的应用场景。
详细文档可查看[配置参考](../configuration/configuration.md#config)。

## 配置生成器

Envoy 的配置开始变得越发复杂。 在 Lyft 我们用 [jinja](http://jinja.pocoo.org/) 模版让生产以及管理配置的工作变得轻松一些。
源代码分发包里便包含了其中一个版本的配置生成器，这个版本的配置生成器非常接近我们在 Lyft 使用的版本。
我们同时为上述的三种场景都准备了相应的范例配置模版。

- 脚本生成器: [configs/configgen.py](https://github.com/envoyproxy/envoy/blob/master/configs/configgen.py)
- 服务间模版: [configs/envoy_service_to_service.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_service_to_service_v2.template.yaml)
- 前端代理模版: [configs/envoy_front_proxy.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_front_proxy_v2.template.yaml)
- 双向代理模版: [configs/envoy_double_proxy.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_double_proxy_v2.template.yaml)

可以从 repo 的根目录运行以下命令以生成相关的范例配置：

```bash
mkdir -p generated/configs
bazel build //configs:example_configs
tar xvf $PWD/bazel-genfiles/configs/example_configs.tar -C generated/configs
```

上面的命令将生成三个完全可扩展配置，而配置中所使用到的其中某些变量被定义在 configgen.py 里。可以查看 configgen.py 里的注释以学习如何让不同的扩展工作。

关于范例配置，在此分享一些笔记：

- 一个假定运行在 discovery.yourcompany.net 上的[服务发现](../intro/arch_overview/service_discovery.md#arch-overview-service-discovery-types-sds)实例。
- yourcompany.net 的 DNS 做了许多配置。 可在配置模版中查找基于其实现的各种实例。
- 为 [LightStep](http://lightstep.com/) 而配置的追踪。 为了禁止或启用 Zipkin <http://zipkin.io> 追踪，而删除或改变相应的[追踪配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/tracing#config-tracing-v1) 。
- 用于演示如何使用[全局速率限制服务](../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)的示例配置。 可以通过删除[速率限制配置](../configuration/rate_limit.md#config-rate-limit-service)以禁用此服务。
- 为服务间参考配置而设定的[路由发现服务](../configuration/http_conn_man/rds.md#config-http-conn-man-rds)，此服务假定运行在 rds.yourcompany.net 上。
- 为服务间参考配置而设定的[集群发现服务](../configuration/cluster_manager/cds.md#config-cluster-manager-cds)，此服务假定运行在 cds.yourcompany.net 上。
