# 部署类型

Envoy is usable in a variety of different scenarios, however it’s most useful when deployed as a*mesh* across all hosts in an infrastructure. This section describes three recommended deployment types in increasing order of complexity.

- Service to service only
  - [Service to service egress listener](service_to_service.md#service-to-service-egress-listener)
  - [Service to service ingress listener](service_to_service.md#service-to-service-ingress-listener)
  - [Optional external service egress listeners](service_to_service.md#optional-external-service-egress-listeners)
  - [Discovery service integration](service_to_service.md#discovery-service-integration)
  - [Configuration template](service_to_service.md#configuration-template)
- Service to service plus front proxy
  - [Configuration template](front_proxy.md#configuration-template)
- Service to service, front proxy, and double proxy
  - [Configuration template](double_proxy.md#configuration-template)