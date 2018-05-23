# HTTP 标头操作

HTTP 连接管理器在解码时（当接受到请求时）以及编码时（当发送响应时）操作多种 HTTP 标头。

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

### user-agent

当启用 [add_user_agent](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-add-user-agent) 选项后，连接管理器可能会在解码时设定  user-agent 标头。 
标头只有在尚未设置的情况下才会被修改。 如果连接管理器确实设置了标头，则该值由 `--service-cluster` 命令行选项确定。

### server

server 标头将在编码期间被设置为 [server_name](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-server-name) 选项中的值。

### x-client-trace-id

如果外部客户端设置了该标头, Envoy 会将提供的 trace ID 与内部生产的 [x-request-id](#x-request-id) 连接起来。x-client-trace-id 需要保持全局的唯一性，并且我们推荐以 uuid4 的方式生成 id 。如果设置了此标头，它与  [x-envoy-force-trace](#x-envoy-force-trace)有类似的效果。 请参看 [tracing.client_enabled](../runtime.md#config-http-conn-man-runtime-client-enabled) 运行时设置。

### x-envoy-downstream-service-cluster

内部服务通常想知道哪个服务正在调用它们。 外部请求的这个标头被清洗，而内部请求将包含调用者的服务器集群信息。
请注意在当前的实现中，这被视为调用者设置的提示，同时容易被任意内部的实体进行欺诈。将来，Envoy 将支持相互认证的 TLS 网格，从而让这个标头完全具有安全性。类似 user-agent，该值由 `--service-cluster` 命令行决定。
为启用此功能，你需要设置 [user_agent](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-add-user-agent) 选项为 true。

### x-envoy-downstream-service-node

内部服务可能会想知道请求来自哪个下游节点。这个标头非常类似 [x-envoy-downstream-service-cluster](#x-envoy-downstream-service-cluster)，除了它的值是来自 `--service-node` 选项。

### x-envoy-external-address

服务希望根据原始客户端的IP地址做分析，这是一种常见的需求。 
然后根据对 [XFF](#x-forwarded-for) 的冗长讨论，这事情可以变得非常复杂，因此，在请求来自外部客户端时，Envoy 通过将 x-envoy-external-address 设置为可信客户端地址来简化此操作。 内部请求未设置或覆盖 x-envoy-external-address 。 
为了达到分析目的，可以在内部服务之间安全地转发此头文件，而无需处理复杂的 XFF。

### x-envoy-force-trace

如果内部请求设置了这个标头，Envoy会修改生成的  [x-request-id](#x-request-id)，这样它就会强制收集跟踪信息。 这也迫使  [x-request-id](#x-request-id) 在响应标头中返回。 如果此请求标识随后传播到其他主机，那么这些主机上也会收集跟踪，由此这些主机将为整个请求流提供一致的跟踪。
请参看  [tracing.global_enabled](../runtime.md#config-http-conn-man-runtime-global-enabled) 与 [tracing.random_sampling](../runtime.md#config-http-conn-man-runtime-random-sampling) 运行时设置。

### x-envoy-internal

服务想知道请求是否来自内部来源是一种常见的情况。 Envoy 使用 [XFF](#x-forwarded-for) 来确定这一点，然后将标头值设置为 true 。
这有利于避免解析和理解 XFF。

### x-forwarded-client-cert

*x-forwarded-client-cert* (XFCC)是一个代理标头，代表从客户端到服务器的路径中的部分或全部客户端以及代理服务器的证书信息。
 代理可以选择在代理请求之前清理/追加/转发 XFCC 标头。

XFCC 标头值是逗号（“，”）分隔的字符串。 每个子字符串都是 XFCC 元素，它保存由单个代理添加的信息。
代理可以将当前客户端证书信息作为 XFCC 元素附加到请求的 XFCC 头后面的逗号后面。

Each XFCC element is a semicolon “;” separated string. Each substring is a key-value pair, grouped together by an equals (“=”) sign. The keys are 
每个XFCC元素都是分号“;”分隔的字符串。 每个子字符串都是一个键值对，由一个等号（“=”）组成。 密钥不区分大小写，值区分大小写。 如果“，”，“;”或“=”出现在一个值中，则该值应该用双引号。 值中的双引号应该用反斜杠双引号（\"）替换。

支持以下键值：

1. `By` The Subject Alternative Name (URI type) of the current proxy’s certificate.
2. `Hash` The SHA 256 diguest of the current client certificate.
3. `Cert` The entire client certificate in URL encoded PEM format.
4. `Subject` The Subject field of the current client certificate. The value is always double-quoted.
5. `URI` The URI type Subject Alternative Name field of the current client certificate.
6. `DNS` The DNS type Subject Alternative Name field of the current client certificate. A client certificate may contain multiple DNS type Subject Alternative Names, each will be a separate key-value pair.

A client certificate may contain multiple Subject Alternative Name types. For details on different Subject Alternative Name types, please refer RFC 2459.

Some examples of the XFCC header are:

    For one client certificate with only URI type Subject Alternative Name: x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;Subject="/C=US/ST=CA/L=San Francisco/OU=Lyft/CN=Test Client";URI=http://testclient.lyft.com
    For two client certificates with only URI type Subject Alternative Name: x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;URI=http://testclient.lyft.com,By=http://backend.lyft.com;Hash=9ba61d6425303443c0748a02dd8de688468ed33be74eee6556d90c0149c1309e;URI=http://frontend.lyft.com
    For one client certificate with both URI type and DNS type Subject Alternative Name: x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;Subject="/C=US/ST=CA/L=San Francisco/OU=Lyft/CN=Test Client";URI=http://testclient.lyft.com;DNS=lyft.com;DNS=www.lyft.com

How Envoy processes XFCC is specified by the forward_client_cert and the set_current_client_cert_details HTTP connection manager options. If forward_client_cert is unset, the XFCC header will be sanitized by default.

### x-forwarded-for

x-forwarded-for (XFF) is a standard proxy header which indicates the IP addresses that a request has flowed through on its way from the client to the server. A compliant proxy will append the IP address of the nearest client to the XFF list before proxying the request. Some examples of XFF are:

    x-forwarded-for: 50.0.0.1 (single client)
    x-forwarded-for: 50.0.0.1, 40.0.0.1 (external proxy hop)
    x-forwarded-for: 50.0.0.1, 10.0.0.1 (internal proxy hop)

Envoy will only append to XFF if the use_remote_address HTTP connection manager option is set to true. This means that if use_remote_address is false (which is the default), the connection manager operates in a transparent mode where it does not modify XFF.

Attention

In general, use_remote_address should be set to true when Envoy is deployed as an edge node (aka a front proxy), whereas it may need to be set to false when Envoy is used as an internal service node in a mesh deployment.

The value of use_remote_address controls how Envoy determines the #### trusted client address. Given an HTTP request that has traveled through a series of zero or more proxies to reach Envoy, the trusted client address is the earliest source IP address that is known to be accurate. The source IP address of the immediate downstream node’s connection to Envoy is trusted. XFF sometimes can be trusted. Malicious clients can forge XFF, but the last address in XFF can be trusted if it was put there by a trusted proxy.

Envoy’s default rules for determining the trusted client address (before appending anything to XFF) are:

    If use_remote_address is false and an XFF containing at least one IP address is present in the request, the trusted client address is the last (rightmost) IP address in XFF.
    Otherwise, the trusted client address is the source IP address of the immediate downstream node’s connection to Envoy.

In an environment where there are one or more trusted proxies in front of an edge Envoy instance, the xff_num_trusted_hops configuration option can be used to trust additional addresses from XFF:

    If use_remote_address is false and xff_num_trusted_hops is set to a value N that is greater than zero, the trusted client address is the (N+1)th address from the right end of XFF. (If the XFF contains fewer than N+1 addresses, Envoy falls back to using the immediate downstream connection’s source address as trusted client address.)
    If use_remote_address is true and xff_num_trusted_hops is set to a value N that is greater than zero, the trusted client address is the Nth address from the right end of XFF. (If the XFF contains fewer than N addresses, Envoy falls back to using the immediate downstream connection’s source address as trusted client address.)

Envoy uses the trusted client address contents to determine whether a request originated externally or internally. This influences whether the x-envoy-internal header is set.

Example 1: Envoy as edge proxy, without a trusted proxy in front of it

    Settings:
        use_remote_address = true
        xff_num_trusted_hops = 0
    Request details:
        Downstream IP address = 192.0.2.5
        XFF = “203.0.113.128, 203.0.113.10, 203.0.113.1”
    Result:
        Trusted client address = 192.0.2.5 (XFF is ignored)
        X-Envoy-External-Address is set to 192.0.2.5
        XFF is changed to “203.0.113.128, 203.0.113.10, 203.0.113.1, 192.0.2.5”
        X-Envoy-Internal is removed (if it was present in the incoming request)

Example 2: Envoy as internal proxy, with the Envoy edge proxy from Example 1 in front of it

    Settings:
        use_remote_address = false
        xff_num_trusted_hops = 0
    Request details:
        Downstream IP address = 10.11.12.13 (address of the Envoy edge proxy)
        XFF = “203.0.113.128, 203.0.113.10, 203.0.113.1, 192.0.2.5”
    Result:
        Trusted client address = 192.0.2.5 (last address in XFF is trusted)
        X-Envoy-External-Address is not modified
        X-Envoy-Internal is removed (if it was present in the incoming request)

Example 3: Envoy as edge proxy, with two trusted external proxies in front of it

    Settings:
        use_remote_address = true
        xff_num_trusted_hops = 2
    Request details:
        Downstream IP address = 192.0.2.5
        XFF = “203.0.113.128, 203.0.113.10, 203.0.113.1”
    Result:
        Trusted client address = 203.0.113.10 (2nd to last address in XFF is trusted)
        X-Envoy-External-Address is set to 203.0.113.10
        XFF is changed to “203.0.113.128, 203.0.113.10, 203.0.113.1, 192.0.2.5”
        X-Envoy-Internal is removed (if it was present in the incoming request)

Example 4: Envoy as internal proxy, with the edge proxy from Example 3 in front of it

    Settings:
        use_remote_address = false
        xff_num_trusted_hops = 2
    Request details:
        Downstream IP address = 10.11.12.13 (address of the Envoy edge proxy)
        XFF = “203.0.113.128, 203.0.113.10, 203.0.113.1, 192.0.2.5”
    Result:
        Trusted client address = 203.0.113.10
        X-Envoy-External-Address is not modified
        X-Envoy-Internal is removed (if it was present in the incoming request)

Example 5: Envoy as an internal proxy, receiving a request from an internal client

    Settings:
        use_remote_address = false
        xff_num_trusted_hops = 0
    Request details:
        Downstream IP address = 10.20.30.40 (address of the internal client)
        XFF is not present
    Result:
        Trusted client address = 10.20.30.40
        X-Envoy-External-Address remains unset
        X-Envoy-Internal is set to “true”

Example 6: The internal Envoy from Example 5, receiving a request proxied by another Envoy

    Settings:
        use_remote_address = false
        xff_num_trusted_hops = 0
    Request details:
        Downstream IP address = 10.20.30.50 (address of the Envoy instance proxying to this one)
        XFF = “10.20.30.40”
    Result:
        Trusted client address = 10.20.30.40
        X-Envoy-External-Address remains unset
        X-Envoy-Internal is set to “true”

A few very important notes about XFF:

    If use_remote_address is set to true, Envoy sets the x-envoy-external-address header to the trusted client address.

    XFF is what Envoy uses to determine whether a request is internal origin or external origin. If use_remote_address is set to true, the request is internal if and only if the request contains no XFF and the immediate downstream node’s connection to Envoy has an internal (RFC1918 or RFC4193) source address. If use_remote_address is false, the request is internal if and only if XFF contains a single RFC1918 or RFC4193 address.
        NOTE: If an internal service proxies an external request to another internal service, and includes the original XFF header, Envoy will append to it on egress if use_remote_address is set. This will cause the other side to think the request is external. Generally, this is what is intended if XFF is being forwarded. If it is not intended, do not forward XFF, and forward x-envoy-internal instead.
        NOTE: If an internal service call is forwarded to another internal service (preserving XFF), Envoy will not consider it internal. This is a known “bug” due to the simplification of how XFF is parsed to determine if a request is internal. In this scenario, do not forward XFF and allow Envoy to generate a new one with a single internal origin IP.
    Testing IPv6 in a large multi-hop system can be difficult from a change management perspective. For testing IPv6 compatibility of upstream services which parse XFF header values, represent_ipv4_remote_address_as_ipv4_mapped_ipv6 can be enabled in the v2 API. Envoy will append an IPv4 address in mapped IPv6 format, e.g. ::FFFF:50.0.0.1. This change will also apply to x-envoy-external-address.


### x-forwarded-proto

服务想要知道由 前端/边缘 Enovy 终止的连接的始发协议（HTTP 或 HTTPS），这是一种常见的情况。 x-forwarded-proto 包含这些信息。 它将被设置为 http 或 https 。

### x-request-id

The x-request-id header is used by Envoy to uniquely identify a request as well as perform stable access logging and tracing. Envoy will generate an x-request-id header for all external origin requests (the header is sanitized). It will also generate an x-request-id header for internal requests that do not already have one. This means that x-request-id can and should be propagated between client applications in order to have stable IDs across the entire mesh. Due to the out of process architecture of Envoy, the header can not be automatically forwarded by Envoy itself. This is one of the few areas where a thin client library is needed to perform this duty. How that is done is out of scope for this documentation. If x-request-id is propagated across all hosts, the following features are available:

- Stable [access logging] via the [v1 API 运行时过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v1/access_log#config-http-con-manager-access-log-filters-runtime-v1) or the [v2 API 运行时过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/accesslog/v2/accesslog.proto#envoy-api-field-config-filter-accesslog-v2-accesslogfilter-runtime-filter).
- Stable tracing when performing random sampling via the [tracing.random_sampling] runtime setting or via forced tracing using the [x-envoy-force-trace](#x-envoy-force-trace) and [x-client-trace-id](#x-client-trace-id) headers.


### x-ot-span-context

x-ot-span-context HTTP 标头用于在 Envoy 与 LightStep 追踪器间跟踪跨度时建立适当的父-子关系。 
例如，出口跨度是入口跨度的子节点（如果入口跨度存在）。Envoy 在入口请求注入 x-ot-span-context 标头并将其转发给本地服务。
Envoy 依靠应用程序将出口调用的 x-ot-span-context 传播到上游。在[这里](../../intro/arch_overview/tracing.md##arch-overview-tracing)可查看更多信息。

### x-b3-traceid

Envoy 的 Zipkin 追踪器使用 x-b3-traceid HTTP 标头。TraceId 的长度为64字节，并反映追踪的总体 ID。
追踪中的每个跨度都共享此 ID。可在 <https://github.com/openzipkin/b3-propagation> 查阅 zipkin 追踪的更多信息。 

### x-b3-spanid

Envoy 的 Zipkin 追踪器使用 x-b3-spanid HTTP 标头。
SpanId 的长度为64字节，并反映当前操作在追踪树中的位置。该值不应被解释：它可能会或也有可能不会由 TraceId 的值派生出来。
可在 <https://github.com/openzipkin/b3-propagation> 查阅 zipkin 追踪的更多信息。

### x-b3-parentspanid

Envoy 的 Zipkin 追踪器使用 x-b3-parentspanid 标头。 
ParentSpanId 的长度为64字节，并反映父操作在追踪树中的位置。 当 span 为追踪树的根节点时，不存在 ParentSpanId。 可在 <https://github.com/openzipkin/b3-propagation> 查阅 zipkin 追踪的更多信息。

### x-b3-sampled
 
当 Sampled flag 标志未被指定或设置为1时，跨度将被报告给跟踪系统。 一旦 Sampled 设置为0或1，将始终向下游发送同样的值。
可在 <https://github.com/openzipkin/b3-propagation> 查阅 zipkin 追踪的更多信息。

### x-b3-flags

Envoy 的 Zipkin 追踪器使用 x-b3-flags 标头。 编码一个或多个选项。 例如，Debug 被编码为 `X-B3-Flags: 1`。
可在 <https://github.com/openzipkin/b3-propagation> 查阅 zipkin 追踪的更多信息。

### custom-request-response-headers

Custom request/response headers can be added to a request/response at the weighted cluster, route, virtual host, and/or global route configuration level. See the relevant v1 and v2 API documentation.

Headers are appended to requests/responses in the following order: weighted cluster level headers, route level headers, virtual host level headers and finally global level headers.

Envoy supports adding dynamic values to request and response headers. The percent symbol (%) is used to delimit variable names.

Attention

If a literal percent symbol (%) is desired in a request/response header, it must be escaped by doubling it. For example, to emit a header with the value 100%, the custom header value in the Envoy configuration must be 100%%.

Supported variable names are:

%DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT%

    Remote address of the downstream connection. If the address is an IP address the output does not include port.

    Note

    This may not be the physical remote address of the peer if the address has been inferred from proxy proto or x-forwarded-for.
%DOWNSTREAM_LOCAL_ADDRESS%
    Local address of the downstream connection. If the address is an IP address it includes both address and port. If the original connection was redirected by iptables REDIRECT, this represents the original destination address restored by the Original Destination Filter using SO_ORIGINAL_DST socket option. If the original connection was redirected by iptables TPROXY, and the listener’s transparent option was set to true, this represents the original destination address and port.
%DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT%
    Same as %DOWNSTREAM_LOCAL_ADDRESS% excluding port if the address is an IP address.
%PROTOCOL%
    The original protocol which is already added by Envoy as a x-forwarded-proto request header.
%UPSTREAM_METADATA([“namespace”, “key”, …])%
    Populates the header with EDS endpoint metadata from the upstream host selected by the router. Metadata may be selected from any namespace. In general, metadata values may be strings, numbers, booleans, lists, nested structures, or null. Upstream metadata values may be selected from nested structs by specifying multiple keys. Otherwise, only string, boolean, and numeric values are supported. If the namespace or key(s) are not found, or if the selected value is not a supported type, then no header is emitted. The namespace and key(s) are specified as a JSON array of strings. Finally, percent symbols in the parameters do not need to be escaped by doubling them. 




See https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/headers