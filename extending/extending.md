# 为自定义用例扩展 Envoy

The Envoy architecture makes it fairly easily extensible via both [network filters](../intro/arch_overview/network_filters.md#arch-overview-network-filters) and [HTTP filters](../intro/arch_overview/http_filters.md#arch-overview-http-filters).

An example of how to add a network filter and structure the repository and build dependencies can be found at [envoy-filter-example](https://github.com/envoyproxy/envoy-filter-example).