#  入门指南


本章节会介绍非常简单的配置，并提供一些示例。

Envoy 目前不提供单独的预先编译好的二进制文件，但提供了 Docker 镜像。这是开始使用 Envoy 的最快方式。如果您希望在 Docker 容器外使用 Envoy，则需要[构建它](../install/building.md#building)。

这些示例使用[v2 Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api#envoy-api-reference)，但仅使用 API 的静态配置功能，这对于简单的需求非常有用。 更复杂的需求是由[动态配置](../intro/arch_overview/dynamic_configuration.md#arch-overview-dynamic-config)来支持的。
## 快速开始运行简单示例

These instructions run from files in the Envoy repo. The sections below give a more detailed explanation of the configuration file and execution steps for the same configuration.

这些命令从 Envoy 存储库中的文件运行。下面的部分给出了相同配置的配置文件和执行步骤的更详细的解释。

A very minimal Envoy configuration that can be used to validate basic plain HTTP proxying is available in [configs/google_com_proxy.v2.yaml](https://github.com/envoyproxy/envoy/blob/master/configs/google_com_proxy.v2.yaml). This is not intended to represent a realistic Envoy deployment:

configs / google_com_proxy.v2.yaml中提供了一个可用于验证基本纯HTTP代理的非常简单的Envoy配置。 这不是为了代表一个现实的特使部署：

```bash
$ docker pull envoyproxy/envoy:latest
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy:latest
$ curl -v localhost:10000
```

The Docker image used will contain the latest version of Envoy and a basic Envoy configuration. This basic configuration tells Envoy to route incoming requests to *.google.com.

使用的Docker镜像将包含最新版本的Envoy和一个基本的Envoy配置。 此基本配置告诉Envoy将传入请求路由到* .google.com。

## 简单的配置

Envoy can be configured using a single YAML file passed in as an argument on the command line.

Envoy可以使用作为参数在命令行中传入的单个YAML文件进行配置。

The [admin message](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-msg-config-bootstrap-v2-admin) is required to configure the administration server. The address key specifies the listening [address](../api-v2/api/v2/core/address.proto.md#envoy-api-file-envoy-api-v2-core-address-proto) which in this case is simply 0.0.0.0:9901.

管理员消息是配置管理服务器所必需的。 地址码指定监听地址，在这种情况下，监听地址只是0.0.0.0:9901。

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
```

The [static_resources](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-field-config-bootstrap-v2-bootstrap-static-resources) contains everything that is configured statically when Envoy starts, as opposed to the means of configuring resources dynamically when Envoy is running. The [v2 API Overview](../configuration/overview/v2_overview.md#config-overview-v2) describes this.

static_resources包含Envoy启动时静态配置的所有内容，而不是在Envoy运行时动态配置资源的方式。 v2 API概述描述了这一点。

```yaml
static_resources:
```

The specification of the [listeners](../api-v2/api/v2/listener/listener.proto.md#envoy-api-file-envoy-api-v2-listener-listener-proto).

听众的规格

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

The specification of the [clusters](../api-v2/api/v2/cds.proto.md#envoy-api-file-envoy-api-v2-cds-proto).

集群的规格

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

Create a simple Dockerfile to execute Envoy, which assumes that envoy.yaml (described above) is in your local directory. You can refer to the [Command line options](../operations/cli.md#operations-cli).

创建一个简单的Dockerfile来执行Envoy，它假定envoy.yaml（如上所述）位于本地目录中。 您可以参考命令行选项。

```
FROM envoyproxy/envoy:latest
COPY envoy.yaml /etc/envoy/envoy.yaml
```

Build the Docker image that runs your configuration using:

构建使用以下命令运行配置的Docker镜像：

```bash
$ docker build -t envoy:v1
```

And now you can execute it with:

现在你可以执行它：

```bash
$ docker run -d --name envoy -p 9901:9901 -p 10000:10000 envoy:v1
```

And finally test is using:

最后测试使用：

```bash
$ curl -v localhost:10000
```

If you would like to use envoy with docker-compose you can overwrite the provided configuration file by using a volume.

如果您想通过docker-compose使用envoy，则可以使用卷覆盖提供的配置文件。

## Sandbox

We’ve created a number of sandboxes using Docker Compose that set up different environments to test out Envoy’s features and show sample configurations. As we gauge peoples’ interests we will add more sandboxes demonstrating different features. The following sandboxes are available:

我们使用Docker Compose创建了许多沙箱，这些沙箱设置了不同的环境来测试Envoy的功能并显示示例配置。 当我们衡量人们的兴趣时，我们将添加更多展示不同特征的沙箱。 以下沙箱可用：

- [Front Proxy](sandboxes/front_proxy.md)
- [Zipkin Tracing](sandboxes/zipkin_tracing.md)
- [Jaeger Tracing](sandboxes/jaeger_tracing.md)
- [Jaeger Native Tracing](sandboxes/jaeger_native_tracing.md)
- [gRPC Bridge](sandboxes/grpc_bridge.md)

## 其他用例

In addition to the proxy itself, Envoy is also bundled as part of several open source distributions that target specific use cases.

除代理本身之外，Envoy还被捆绑为几个针对特定用例的开源发行版的一部分。

- [Envoy as an API Gateway in Kubernetes](distro/ambassador.md)