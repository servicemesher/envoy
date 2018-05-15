# gRPC HTTP/1.1 bridge

- gRPC [architecture overview](../../intro/arch_overview/grpc.md#arch-overview-grpc)
- [v1 API reference](../../api-v1/http_filters/grpc_http1_bridge_filter.md#config-http-filters-grpc-bridge-v1)
- [v2 API reference](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.md#envoy-api-field-config-filter-network-http-connection-manager-v2-httpfilter-name)

This is a simple filter which enables the bridging of an HTTP/1.1 client which does not support response trailers to a compliant gRPC server. It works by doing the following:

- When a request is sent, the filter sees if the connection is HTTP/1.1 and the request content type is *application/grpc*.
- If so, when the response is received, the filter buffers it and waits for trailers and then checks the *grpc-status* code. If it is not zero, the filter switches the HTTP response code to 503. It also copies the *grpc-status* and *grpc-message* trailers into the response headers so that the client can look at them if it wishes.
- The client should send HTTP/1.1 requests that translate to the following pseudo headers:
  - *:method*: POST
  - *:path*: <gRPC-METHOD-NAME>
  - *content-type*: application/grpc
- The body should be the serialized grpc body which is:
  - 1 byte of zero (not compressed).
  - network order 4 bytes of proto message length.
  - serialized proto message.
- Because this scheme must buffer the response to look for the *grpc-status* trailer it will only work with unary gRPC APIs.

This filter also collects stats for all gRPC requests that transit, even if those requests are normal gRPC requests over HTTP/2.

More info: wire format in [gRPC over HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md).

## Statistics

The filter emits statistics in the *cluster.<route target cluster>.grpc.* namespace.

| Name                                 | Type    | Description                           |
| ------------------------------------ | ------- | ------------------------------------- |
| <grpc service>.<grpc method>.success | Counter | Total successful service/method calls |
| <grpc service>.<grpc method>.failure | Counter | Total failed service/method calls     |
| <grpc service>.<grpc method>.total   | Counter | Total service/method calls            |