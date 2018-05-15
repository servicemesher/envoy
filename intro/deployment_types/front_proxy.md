# 服务间外加前端代理

![../../images/front_proxy.svg](../../images/front_proxy.svg)

The above diagram shows the [service to service](service_to_service.md#deployment-type-service-to-service) configuration sitting behind an Envoy cluster used as an HTTP L7 edge reverse proxy. The reverse proxy provides the following features:

- Terminates TLS.
- Supports both HTTP/1.1 and HTTP/2.
- Full HTTP L7 routing support.
- Talks to the service to service Envoy clusters via the standard [ingress port](service_to_service.md#deployment-type-service-to-service-ingress) and using the discovery service for host lookup. Thus, the front Envoy hosts work identically to any other Envoy host, other than the fact that they do not run collocated with another service. This means that are operated in the same way and emit the same statistics.

## 配置模板

The source distribution includes an example front proxy configuration that is very similar to the version that Lyft runs in production. See [here](../../install/ref_configs.md#install-ref-configs) for more information.