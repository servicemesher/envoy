#  入门指南

This section gets you started with a very simple configuration and provides some example configurations.

Envoy does not currently provide separate pre-built binaries, but does provide Docker images. This is the fastest way to get started using Envoy. Should you wish to use Envoy outside of a Docker container, you will need to [build it](../install/building.md#building).

These examples use the [v2 Envoy API](../api-v2/api.md#envoy-api-reference), but use only the static configuration feature of the API, which is most useful for simple requirements. For more complex requirements [Dynamic Configuration](../intro/arch_overview/dynamic_configuration.md#arch-overview-dynamic-config) is supported.

## 快速开始运行简单示例

These instructions run from files in the Envoy repo. The sections below give a more detailed explanation of the configuration file and execution steps for the same configuration.

A very minimal Envoy configuration that can be used to validate basic plain HTTP proxying is available in [configs/google_com_proxy.v2.yaml](https://github.com/envoyproxy/envoy/blob/master/configs/google_com_proxy.v2.yaml). This is not intended to represent a realistic Envoy deployment:

```bash
$ docker pull envoyproxy/envoy:latest
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy:latest
$ curl -v localhost:10000
```

The Docker image used will contain the latest version of Envoy and a basic Envoy configuration. This basic configuration tells Envoy to route incoming requests to *.google.com.

## 简单的配置

Envoy can be configured using a single YAML file passed in as an argument on the command line.

The [admin message](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-msg-config-bootstrap-v2-admin) is required to configure the administration server. The address key specifies the listening [address](../api-v2/api/v2/core/address.proto.md#envoy-api-file-envoy-api-v2-core-address-proto) which in this case is simply 0.0.0.0:9901.

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
```

The [static_resources](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-field-config-bootstrap-v2-bootstrap-static-resources) contains everything that is configured statically when Envoy starts, as opposed to the means of configuring resources dynamically when Envoy is running. The [v2 API Overview](../configuration/overview/v2_overview.md#config-overview-v2) describes this.

```yaml
static_resources:
```

The specification of the [listeners](../api-v2/api/v2/listener/listener.proto.md#envoy-api-file-envoy-api-v2-listener-listener-proto).

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

```
FROM envoyproxy/envoy:latest
COPY envoy.yaml /etc/envoy/envoy.yaml
```

Build the Docker image that runs your configuration using:

```bash
$ docker build -t envoy:v1
```

And now you can execute it with:

```bash
$ docker run -d --name envoy -p 9901:9901 -p 10000:10000 envoy:v1
```

And finally test is using:

```bash
$ curl -v localhost:10000
```

If you would like to use envoy with docker-compose you can overwrite the provided configuration file by using a volume.

## Sandbox

We’ve created a number of sandboxes using Docker Compose that set up different environments to test out Envoy’s features and show sample configurations. As we gauge peoples’ interests we will add more sandboxes demonstrating different features. The following sandboxes are available:

- [Front Proxy](sandboxes/front_proxy.md)
- [Zipkin Tracing](sandboxes/zipkin_tracing.md)
- [Jaeger Tracing](sandboxes/jaeger_tracing.md)
- [Jaeger Native Tracing](sandboxes/jaeger_native_tracing.md)
- [gRPC Bridge](sandboxes/grpc_bridge.md)

## 其他用例

In addition to the proxy itself, Envoy is also bundled as part of several open source distributions that target specific use cases.

- [Envoy as an API Gateway in Kubernetes](distro/ambassador.md)