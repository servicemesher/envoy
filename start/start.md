#  入门指南


本章节会介绍非常简单的配置，并提供一些示例。

Envoy 目前不提供单独的预先编译好的二进制文件，但提供了 Docker 镜像。这是开始使用 Envoy 的最快方式。如果您希望在 Docker 容器外使用 Envoy，则需要[构建它](../install/building.md#building)。

这些示例使用 [v2 Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api#envoy-api-reference)，但仅使用 API 的静态配置功能，这对于简单的需求非常有用。 更复杂的需求是由[动态配置](../intro/arch_overview/dynamic_configuration.md#arch-overview-dynamic-config)来支持的。
## 快速开始运行简单示例


根据 Envoy 存储库中的文件运行这些命令。下面的部分给出了配置文件和执行步骤更详细的解释。

[configs/google_com_proxy.v2.yaml](https://github.com/envoyproxy/envoy/blob/master/configs/google_com_proxy.v2.yaml) 中提供了一个非常简单的可用于验证基于纯 HTTP 代理的 Envoy 配置。 这不表示实际的 Envoy 部署：

```bash
$ docker pull envoyproxy/envoy:latest
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy:latest
$ curl -v localhost:10000
```

使用的 Docker 镜像将包含最新版本的 Envoy 和一个基本的 Envoy 配置。此基本配置告诉 Envoy 将入站请求路由到 *.google.com。

## 简单的配置

Envoy 通过 YAML 文件中传入的参数来进行配置。

[admin message](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-msg-config-bootstrap-v2-admin) 是 administration 服务必须的配置。address 键指定监听[地址](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/address.proto#envoy-api-file-envoy-api-v2-core-address-proto)，下面的例子监听地址是 0.0.0.0:9901。

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
```

[static_resources](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-field-config-bootstrap-v2-bootstrap-static-resources) 包含 Envoy 启动时静态配置的所有内容，而不是 Envoy 在运行时动态配置的资源。[v2 API Overview](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v2_overview#config-overview-v2) 描述了这一点。

```yaml
static_resources:
```

[listeners](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-file-envoy-api-v2-listener-listener-proto) 规范

```yaml
listeners:
- name: listener_0
  address:
    socket_address: { address: 0.0.0.0, port_value: 10000 }
  filter_chains:
  - filters:
    - name: envoy.http_connection_manager
      config:
        stat_prefix: ingress_http
        codec_type: AUTO
        route_config:
          name: local_route
          virtual_hosts:
          - name: local_service
            domains: ["*"]
            routes:
            - match: { prefix: "/" }
              route: { host_rewrite: www.google.com, cluster: service_google }
        http_filters:
        - name: envoy.router
```

[clusters](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#envoy-api-file-envoy-api-v2-cds-proto) 规范

```yaml
clusters:
- name: service_google
  connect_timeout: 0.25s
  type: LOGICAL_DNS
  # Comment out the following line to test on v6 networks
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  hosts: [{ socket_address: { address: google.com, port_value: 443 }}]
  tls_context: { sni: www.google.com }
```

## 使用 Envoy Docker 镜像

创建一个简单的 Dockerfile 来执行 Envoy，假定 envoy.yaml（如上所述）位于本地目录中。您可以参考[命令行选项](../operations/cli.md#operations-cli)。

```
FROM envoyproxy/envoy:latest
COPY envoy.yaml /etc/envoy/envoy.yaml
```

使用以下命令构建您配置的 Docker 镜像：

```bash
$ docker build -t envoy:v1
```

现在您可以执行它：

```bash
$ docker run -d --name envoy -p 9901:9901 -p 10000:10000 envoy:v1
```

最后测试使用：

```bash
$ curl -v localhost:10000
```

如果您想通过 docker-compose 使用 envoy，则可以使用 volume 覆盖提供的配置文件。

## Sandbox

我们使用 Docker Compose 创建了许多 sandbox ，这些 sandbox 设置了不同的环境来测试 Envoy 的功能并显示示例配置。 当我们觉得人们更有兴趣时，将添加和展示更多不同特征的 sandbox。 以下 sandbox 可用：

- [Front Proxy](sandboxes/front_proxy.md)
- [Zipkin Tracing](sandboxes/zipkin_tracing.md)
- [Jaeger Tracing](sandboxes/jaeger_tracing.md)
- [Jaeger Native Tracing](sandboxes/jaeger_native_tracing.md)
- [gRPC Bridge](sandboxes/grpc_bridge.md)

## 其他用例

除代理本身之外， Envoy 还被几个特定用例捆绑为开源发行版的一部分。

- [Envoy as an API Gateway in Kubernetes](distro/ambassador.md)