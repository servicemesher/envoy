# 访问记录

## Configuration

Access logs are configured as part of the [HTTP connection manager config](http_conn_man/http_conn_man.md#config-http-conn-man) or [TCP Proxy](network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy).

- [v1 API reference](../api-v1/access_log.md#config-access-log-v1)
- [v2 API reference](../api-v2/config/filter/accesslog/v2/accesslog.proto.md#envoy-api-msg-config-filter-accesslog-v2-accesslog)

## Format rules

The access log format string contains either command operators or other characters interpreted as a plain string. The access log formatter does not make any assumptions about a new line separator, so one has to specified as part of the format string. See the [default format](#config-access-log-default-format) for an example. Note that the access log line will contain a ‘-‘ character for every not set/empty value.

The same format strings are used by different types of access logs (such as HTTP and TCP). Some fields may have slightly different meanings, depending on what type of log it is. Differences are noted.

The following command operators are supported:

- %START_TIME%

  HTTPRequest start time including milliseconds.TCPDownstream connection start time including milliseconds.START_TIME can be customized using a [format string](http://en.cppreference.com/w/cpp/io/manip/put_time), for example:

```
%START_TIME(%Y/%m/%dT%H:%M:%S%z %s)%
```

- %BYTES_RECEIVED%

  HTTPBody bytes received.TCPDownstream bytes received on connection.

- %PROTOCOL%

  HTTPProtocol. Currently either *HTTP/1.1* or *HTTP/2*.TCPNot implemented (“-“).

- %RESPONSE_CODE%

  HTTPHTTP response code. Note that a response code of ‘0’ means that the server never sent the beginning of a response. This generally means that the (downstream) client disconnected.TCPNot implemented (“-“).

- %BYTES_SENT%

  HTTPBody bytes sent.TCPDownstream bytes sent on connection.

- %DURATION%

  HTTPTotal duration in milliseconds of the request from the start time to the last byte out.TCPTotal duration in milliseconds of the downstream connection.

- %RESPONSE_FLAGS%

  Additional details about the response or connection, if any. For TCP connections, the response codes mentioned in the descriptions do not apply. Possible values are:HTTP and TCP**UH**: No healthy upstream hosts in upstream cluster in addition to 503 response code.**UF**: Upstream connection failure in addition to 503 response code.**UO**: Upstream overflow ([circuit breaking](../intro/arch_overview/circuit_breaking.md#arch-overview-circuit-break)) in addition to 503 response code.**NR**: No [route configured](../intro/arch_overview/http_routing.md#arch-overview-http-routing) for a given request in addition to 404 response code.HTTP only**LH**: Local service failed [health check request](../intro/arch_overview/health_checking.md#arch-overview-health-checking) in addition to 503 response code.**UT**: Upstream request timeout in addition to 504 response code.**LR**: Connection local reset in addition to 503 response code.**UR**: Upstream remote reset in addition to 503 response code.**UC**: Upstream connection termination in addition to 503 response code.**DI**: The request processing was delayed for a period specified via [fault injection](http_filters/fault_filter.md#config-http-filters-fault-injection).**FI**: The request was aborted with a response code specified via [fault injection](http_filters/fault_filter.md#config-http-filters-fault-injection).**RL**: The request was ratelimited locally by the [HTTP rate limit filter](http_filters/rate_limit_filter.md#config-http-filters-rate-limit) in addition to 429 response code.

- %UPSTREAM_HOST%

  Upstream host URL (e.g., <tcp://ip:port> for TCP connections).

- %UPSTREAM_CLUSTER%

  Upstream cluster to which the upstream host belongs to.

- %UPSTREAM_LOCAL_ADDRESS%

  Local address of the upstream connection. If the address is an IP address it includes both address and port.

- %DOWNSTREAM_REMOTE_ADDRESS%

  Remote address of the downstream connection. If the address is an IP address it includes both address and port.NoteThis may not be the physical remote address of the peer if the address has been inferred from [proxy proto](../api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-filterchain-use-proxy-proto) or [x-forwarded-for](http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for).

- %DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT%

  Remote address of the downstream connection. If the address is an IP address the output does*not* include port.NoteThis may not be the physical remote address of the peer if the address has been inferred from [proxy proto](../api-v2/api/v2/listener/listener.proto.md#envoy-api-field-listener-filterchain-use-proxy-proto) or [x-forwarded-for](http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for).

- %DOWNSTREAM_LOCAL_ADDRESS%

  Local address of the downstream connection. If the address is an IP address it includes both address and port. If the original connection was redirected by iptables REDIRECT, this represents the original destination address restored by the [Original Destination Filter](listener_filters/original_dst_filter.md#config-listener-filters-original-dst) using SO_ORIGINAL_DST socket option. If the original connection was redirected by iptables TPROXY, and the listener’s transparent option was set to true, this represents the original destination address and port.

- %DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT%

  Same as **%DOWNSTREAM_LOCAL_ADDRESS%** excluding port if the address is an IP address.

- %REQ(X?Y):Z%

  HTTPAn HTTP request header where X is the main HTTP header, Y is the alternative one, and Z is an optional parameter denoting string truncation up to Z characters long. The value is taken from the HTTP request header named X first and if it’s not set, then request header Y is used. If none of the headers are present ‘-‘ symbol will be in the log.TCPNot implemented (“-“).

- %RESP(X?Y):Z%

  HTTPSame as **%REQ(X?Y):Z%** but taken from HTTP response headers.TCPNot implemented (“-“).

- %TRAILER(X?Y):Z%

  HTTPSame as **%REQ(X?Y):Z%** but taken from HTTP response trailers.TCPNot implemented (“-“).

- %DYNAMIC_METADATA(NAMESPACE:KEY*):Z%

  HTTP[Dynamic Metadata](../api-v2/api/v2/core/base.proto.md#envoy-api-msg-core-metadata) info, where NAMESPACE is the the filter namespace used when setting the metadata, KEY is an optional lookup up key in the namespace with the option of specifying nested keys separated by ‘:’, and Z is an optional parameter denoting string truncation up to Z characters long. Dynamic Metadata can be set by filters using the [RequestInfo](https://github.com/envoyproxy/envoy/blob/master/include/envoy/request_info/request_info.h) API: *setDynamicMetadata*. The data will be logged as a JSON string. For example, for the following dynamic metadata:`com.test.my_filter: {"test_key": "foo", "test_object": {"inner_key": "bar"}}`%DYNAMIC_METADATA(com.test.my_filter)% will log: `{"test_key": "foo", "test_object": {"inner_key": "bar"}}`%DYNAMIC_METADATA(com.test.my_filter:test_key)% will log: `"foo"`%DYNAMIC_METADATA(com.test.my_filter:test_object)% will log: `{"inner_key": "bar"}`%DYNAMIC_METADATA(com.test.my_filter:test_object:inner_key)% will log: `"bar"`%DYNAMIC_METADATA(com.unknown_filter)% will log: `-`%DYNAMIC_METADATA(com.test.my_filter:unknown_key)% will log: `-`%DYNAMIC_METADATA(com.test.my_filter):25% will log (truncation at 25 characters): `{"test_key": "foo", "test`TCPNot implemented (“-“).

## Default format

If custom format is not specified, Envoy uses the following default format:

```
[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%"
%RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION%
%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%"
"%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n
```

Example of the default Envoy access log format:

```
[2016-04-15T20:17:00.310Z] "POST /api/v1/locations HTTP/2" 204 - 154 0 226 100 "10.0.35.28"
"nsq2http" "cc21d9b0-cf5c-432b-8c7e-98aeb7988cd2" "locations" "tcp://10.0.2.1:80"
```