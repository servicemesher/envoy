# 参考配置

The source distribution includes a set of example configuration templates for each of the three major Envoy deployment types:

- [Service to service](../intro/deployment_types/service_to_service.md#deployment-type-service-to-service)
- [Front proxy](../intro/deployment_types/front_proxy.md#deployment-type-front-proxy)
- [Double proxy](../intro/deployment_types/double_proxy.md#deployment-type-double-proxy)

The goal of this set of example configurations is to demonstrate the full capabilities of Envoy in a complex deployment. All features will not be applicable to all use cases. For full documentation see the [configuration reference](../configuration/configuration.md#config).

## Configuration generator

Envoy configurations can become relatively complicated. At Lyft we use [jinja](http://jinja.pocoo.org/) templating to make the configurations easier to create and manage. The source distribution includes a version of the configuration generator that loosely approximates what we use at Lyft. We have also included three example configuration templates for each of the above three scenarios.

- Generator script: [configs/configgen.py](https://github.com/envoyproxy/envoy/blob/master/configs/configgen.py)
- Service to service template: [configs/envoy_service_to_service.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_service_to_service.template.json)
- Front proxy template: [configs/envoy_front_proxy.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_front_proxy.template.json)
- Double proxy template: [configs/envoy_double_proxy.template.json](https://github.com/envoyproxy/envoy/blob/master/configs/envoy_double_proxy.template.json)

To generate the example configurations run the following from the root of the repo:

```bash
mkdir -p generated/configs
bazel build //configs:example_configs
tar xvf $PWD/bazel-genfiles/configs/example_configs.tar -C generated/configs
```

The previous command will produce three fully expanded configurations using some variables defined inside of configgen.py. See the comments inside of configgen.py for detailed information on how the different expansions work.

A few notes about the example configurations:

- An instance of [service discovery service](../intro/arch_overview/service_discovery.md#arch-overview-service-discovery-types-sds) is assumed to be running at discovery.yourcompany.net.
- DNS for yourcompany.net is assumed to be setup for various things. Search the configuration templates for different instances of this.
- Tracing is configured for [LightStep](http://lightstep.com/). To disable this or enable Zipkin <http://zipkin.io> tracing, delete or change the [tracing configuration](../api-v1/tracing.md#config-tracing-v1) accordingly.
- The configuration demonstrates the use of a [global rate limiting service](../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit). To disable this delete the [rate limit configuration](../configuration/rate_limit.md#config-rate-limit-service).
- [Route discovery service](../configuration/http_conn_man/rds.md#config-http-conn-man-rds) is configured for the service to service reference configuration and it is assumed to be running at rds.yourcompany.net.
- [Cluster discovery service](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) is configured for the service to service reference configuration and it is assumed that be running at cds.yourcompany.net.