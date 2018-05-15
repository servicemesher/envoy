# HTTP header manipulation

The HTTP connection manager manipulates several HTTP headers both during decoding (when the request is being received) as well as during encoding (when the response is being sent).

- [user-agent](#user-agent)
- [server](#server)
- [x-client-trace-id](#x-client-trace-id)
- [x-envoy-downstream-service-cluster](#x-envoy-downstream-service-cluster)
- [x-envoy-downstream-service-node](#x-envoy-downstream-service-node)
- [x-envoy-external-address](#x-envoy-external-address)
- [x-envoy-force-trace](#x-envoy-force-trace)
- [x-envoy-internal](#x-envoy-internal)
- [x-forwarded-client-cert](#x-forwarded-client-cert)
- [x-forwarded-for](#x-forwarded-for)
- [x-forwarded-proto](#x-forwarded-proto)
- [x-request-id](#x-request-id)
- [x-ot-span-context](#x-ot-span-context)
- [x-b3-traceid](#x-b3-traceid)
- [x-b3-spanid](#x-b3-spanid)
- [x-b3-parentspanid](#x-b3-parentspanid)
- [x-b3-sampled](#x-b3-sampled)
- [x-b3-flags](#x-b3-flags)
- [Custom request/response headers](#custom-request-response-headers)

See https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/headers