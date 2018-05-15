# 速率限制服务

The [rate limit service](../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit) configuration specifies the global rate limit service Envoy should talk to when it needs to make global rate limit decisions. If no rate limit service is configured, a “null” service will be used which will always return OK if called.

- [v1 API reference](../api-v1/rate_limit.md#config-rate-limit-service-v1)
- [v2 API reference](../api-v2/config/ratelimit/v2/rls.proto.md#envoy-api-msg-config-ratelimit-v2-ratelimitserviceconfig)

## gRPC service IDL

Envoy expects the rate limit service to support the gRPC IDL specified in[/source/common/ratelimit/ratelimit.proto](https://github.com/envoyproxy/envoy/blob/master//source/common/ratelimit/ratelimit.proto). See the IDL documentation for more information on how the API works. See Lyft’s reference implementation [here](https://github.com/lyft/ratelimit).