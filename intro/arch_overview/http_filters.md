# HTTP filter

Much like the [network level filter](network_filters.md#arch-overview-network-filters) stack, Envoy supports an HTTP level filter stack within the connection manager. Filters can be written that operate on HTTP level messages without knowledge of the underlying physical protocol (HTTP/1.1, HTTP/2, etc.) or multiplexing capabilities. There are three types of HTTP level filters:

- **Decoder**: Decoder filters are invoked when the connection manager is decoding parts of the request stream (headers, body, and trailers).
- **Encoder**: Encoder filters are invoked when the connection manager is about to encode parts of the response stream (headers, body, and trailers).
- **Decoder/Encoder**: Decoder/Encoder filters are invoked both when the connection manager is decoding parts of the request stream and when the connection manager is about to encode parts of the response stream.

The API for HTTP level filters allows the filters to operate without knowledge of the underlying protocol. Like network level filters, HTTP filters can stop and continue iteration to subsequent filters. This allows for more complex scenarios such as health check handling, calling a rate limiting service, buffering, routing, generating statistics for application traffic such as DynamoDB, etc. Envoy already includes several HTTP level filters that are documented in this architecture overview as well as the [configuration reference](../../configuration/http_filters/http_filters.md#config-http-filters).