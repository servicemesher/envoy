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

每个XFCC元素都是分号“;”分隔的字符串。 每个子字符串都是一个键值对，由一个等号（“=”）组成。 密钥不区分大小写，值区分大小写。 如果“，”，“;”或“=”出现在一个值中，则该值应该用双引号。 值中的双引号应该用反斜杠双引号（\\"）替换。

支持以下键值：

1. `By` 当前代理证书的主题备用名称（ URI 类型）。
2. `Hash` The SHA 256 diguest of the current client certificate.
3. `Cert` 当前客户端证书的 SHA 256 摘要。
4. `Subject` 当前客户端证书的主题字段。 该值总是用双引号括起来。
5. `URI` 当前客户端证书的主题备用名称（ URI 类型）。
6. `DNS` 当前客户端证书的主题备用名称字段（URI 类型）。 客户端证书可能包含多个DNS类型的主题备用名称，每个名称都将是一个单独的键值对。

客户端证书可能包含多个主题备用名称类型。关于不同主题备用名称类型的详细信息，请参阅 [RFC 2459](https://tools.ietf.org/html/rfc2459#section-4.2.1.7)。

以下为 XFCC 标头的一些例子：

1. 对于只有URI类型的客户端证书使用主题备用名称：
`x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;Subject="/C=US/ST=CA/L=San Francisco/OU=Lyft/CN=Test Client";URI=http://testclient.lyft.com`
2. 对于只有URI类型的两个客户端证书使用替代名称：
`x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;URI=http://testclient.lyft.com,By=http://backend.lyft.com;Hash=9ba61d6425303443c0748a02dd8de688468ed33be74eee6556d90c0149c1309e;URI=http://frontend.lyft.com`    
3. 对于同时具有URI类型和DNS类型的一个客户端证书使用替代名称：
`x-forwarded-client-cert: By=http://frontend.lyft.com;Hash=468ed33be74eee6556d90c0149c1309e9ba61d6425303443c0748a02dd8de688;Subject="/C=US/ST=CA/L=San Francisco/OU=Lyft/CN=Test Client";URI=http://testclient.lyft.com;DNS=lyft.com;DNS=www.lyft.com`

Envoy 处理 XFCC 的方式由 [forward_client_cert](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-forward-client-cert) 和[set_current_client_cert_details](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-set-current-client-cert-details) HTTP 连接管理器选项指定。 如果未设置 *forward_client_cert*，则默认情况下会对 XFCC 标头进行清理。

### x-forwarded-for

x-forwarded-for (XFF) 是一个标准的代理标头，它表示请求在从客户端到服务器的路上经过的 IP 地址。
 在代理请求之前，兼容代理会将最近客户端的 IP 地址附加到 XFF 列表中。 XFF 的一些例子是：

1. `x-forwarded-for: 50.0.0.1` (单客户端)
2. `x-forwarded-for: 50.0.0.1, 40.0.0.1` (外部代理跳)
3. `x-forwarded-for: 50.0.0.1, 10.0.0.1` (内部代理跳)

仅在 [use_remote_address](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-use-remote-address) HTTP 连接管理选项设置为 true 时，Envoy 才会追加到 XFF 。 这意味着如果 *use_remote_address* 为 false（这是默认值），则连接管理器将以不修改 XFF 的透明模式运行。

> **注意**
>
> 通常，当 Envoy 作为边缘节点（又名前端代理）进行部署时，应将 *use_remote_address* 设置为 true，而将 Envoy 用作网格部署中的内部服务节点时，可能需要将其设置为false。

use_remote_address 的值控制 Envoy 如何确定可信客户端地址。 如果 HTTP 请求已经通过一系列代理(零个或多个)传到 Envoy，则可信的客户端地址是已知准确的最早的源IP地址。 直接下游节点与 Envoy 连接的源 IP 地址是可信的。 XFF 有时可以被信任。恶意客户可以伪造 XFF，但如果 XFF 中的最后一个地址由可信代理放在那里，则可以信任它。

Envoy 用于确定可信客户端地址的默认规则（在向 XFF 添加任何内容之前）是：

- 如果 use_remote_address 为 false 且包含至少一个 IP 地址的 XFF 出现在请求中，则可信客户端地址是 XFF 中的最后（最右边）IP 地址。
- 否则，可信客户端地址是直接下游节点与 Envoy 连接的源 IP 地址。

在边缘 Envoy 实例前有一个或多个可信代理的环境中，xff_num_trusted_hops 配置选项可以用于信任来自 XFF 的其他地址：

- 如果 use_remote_address 为 false 且 xff_num_trusted_hops 设置为大于零的值N，则可信客户端地址为距XFF右端第（N+1）个地址。 
（如果 XFF 包含的地址少于 N+1 个，Envoy 就会使用直接下行连接的源地址作为可信客户端地址。）

- 如果 use_remote_address 为 true 并且 xff_num_trusted_hops 设置为大于零的值 N，则可信客户端地址是 XFF 右端的第 N 个地址。
（如果 XFF 包含的地址少于 N 个，Envoy 就会使用直接下行连接的源地址作为可信客户端地址。）

Envoy 使用可信的客户端地址内容来确定请求是发起于外部还是内部。 这会影响是否设置了 x-envoy-internal 标头。

示例 1: Envoy 作为边缘代理，在它前面没有可信代理

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

示例 2: Envoy 作为内部代理，在它前面有一个如示例1一般的边缘代理

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

示例 3: Envoy 作为边缘代理，在它前面有两个信任的外部代理

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

示例 4: Envoy 作为内部代理, 它前面有一个如示例3一般的边缘代理

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

示例 5: Envoy 作为内部代理，接收来自一个内部客户的请求

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

示例 6: 来自示例5的内部 Envoy，接收由另外一个 Envoy 代理的请求

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

关于 XFF 的一些非常重要的点:

1. 如果 use_remote_address 设置为 true，Envoy会将 x-envoy-external-address 标头设置为受信任的客户端地址。

2. XFF 是Envoy 用来确定请求是内部源还是外部源的。 
如果 use_remote_address 设置为 true，当且仅当请求不包含 XFF 并且直接下游节点与 Envoy 的连接具有内部（RFC1918或RFC4193）源地址时，该请求为内部请求。
如果 use_remote_address 为 false，则当且仅当 XFF 包含单个 RFC1918 或 RFC4193 地址时，该请求才是内部请求。

- 注意：如果内部服务代理到另一个内部服务的外部请求，并且包含原始 XFF 头，则在设置了 [use_remote_address](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-use-remote-address) 的情况下，Envoy 将在出口附加它。 这会导致对方认为请求是外部的。 一般来说，这是 XFF 被转发的意图。 如果没有这个意图，请不要转发 XFF，而是转发 [x-envoy-internal](#x-envoy-internal)。

- 注意： 如果内部服务调用转发到其他内部服务（保留XFF），Envoy 将不会认为这是一个内部服务。 这是一个已知的 "bug"，缘自 XFF 将解析以及判定一个请求是否来自内部的工作进行了简化。在此场景下，请不要将 XFF 转发并允许 Envoy 使用一个内部原始 IP 生成一个新的。

3. 由变更管理的角度来看，在大型多跳系统中测试 IPv6 可能非常困难。为了测试解析 XFF 标头的上游服务的 IPv6 兼容性，可以在 v2 API 中启用
[represent_ipv4_remote_address_as_ipv4_mapped_ipv6](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-represent-ipv4-remote-address-as-ipv4-mapped-ipv6)。Envoy 将以映射的 IPv6 格式附加 IPv4 地址，例如： ::FFFF:50.0.0.1 。此更改同样适用于[x-envoy-external-address](#x-envoy-external-address)。

### x-forwarded-proto

服务想要知道由 前端/边缘 Enovy 终止的连接的始发协议（HTTP 或 HTTPS），这是一种常见的情况。 x-forwarded-proto 包含这些信息。 它将被设置为 http 或 https 。

### x-request-id

Envoy 使用 x-request-id 头来唯一标识请求并执行稳定的访问日志记录和跟踪。Envoy将为所有外部来源请求生成一个 x-request-id 头（标头被清理）。

它还会为没有 x-request-id 头的内部请求生成一个 x-request-id 头。

这意味着 x-request-id 能且应该在客户端应用程序间传播， 以便在整个网格中拥有一个稳定的 ID。

由于 Envoy 的与流程无关的架构设计，Envoy 本身不能自动地转发标头。这是少数瘦客户端库需要做的工作之一。如何去做，这个话题超出了本文档的范围。

如 x-request-id 跨所有主机传播，则可使用如下功能：

- 稳定的 [访问记录](../../configuration/access_log.md#config-access-log) 通过 [v1 API 运行时过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v1/access_log#config-http-con-manager-access-log-filters-runtime-v1) 或 [v2 API 运行时过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/accesslog/v2/accesslog.proto#envoy-api-field-config-filter-accesslog-v2-accesslogfilter-runtime-filter)。
- 进行随机抽样时稳定的追踪，通过开启 [tracing.random_sampling](../../configuration/http_conn_man/runtime.md#config-http-conn-man-runtime-random-sampling) 运行时配置或是使用 [x-envoy-force-trace](#x-envoy-force-trace) 标头以及 [x-client-trace-id](#x-client-trace-id) 标头进行强行追踪。

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

自定义请求/响应标头可以在加权集群、路由、虚拟主机和/或全局路由配置级别添加到请求/响应中。具体可参看相关的 V1 以及 V2 API文档。

标头将按照以下顺序附加到请求/响应中：加权集群级别标头、路由级别标头、虚拟主机级别标头以及全局级别标头。

Envoy 支持将变量添加到请求以及响应标头。百分号(%)用于分割变量名称。

> **注意**
>
> 如果需要在请求/响应标头内增加一个书面的百分比符号（%），则需要重复它以达到转义的效果。

例如，要发送值为100%的标头，Envoy 配置中的自定义标头必须为 100%%。

支持的变量名有：

#### %DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT%

下游连接的远程í地址。如果地址是 IP 地址，则输出不包含端口。

> **注意**
>
> 如果从 [proxy proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-field-listener-filterchain-use-proxy-proto) 或 [x-forwarded-for](#x-forwarded-for) 推断出地址，这非常可能不是对方的物理远程地址。

#### %DOWNSTREAM_LOCAL_ADDRESS%

下游连接的本地地址，如果地址是 IP 地址，则它包括地址和端口。如果原始连接被 iptables REDIRECT 重定向，则表示[原始目标过滤器](../../configuration/listener_filters/original_dst_filter.md#config-listener-filters-original-dst) 使用 SO_ORIGINAL_DST Socket 选项恢复的原始目标地址。如果原始连接被 iptables TPROXY 重定向，且侦听器的透明选项设置为 true，则表示原始目标地址和端口。

#### %DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT% 

与  %DOWNSTREAM_LOCAL_ADDRESS% 类同，但如果地址是 IP 地址时排除端口。

#### %PROTOCOL% 

Envoy 已将其添加为原始协议作为 [x-forwarded-proto](#x-forwarded-proto) 请求标头。

#### %UPSTREAM_METADATA([“namespace”, “key”, …])%

用来自路由器选择的上游主机的 [EDS端点元数据](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto#envoy-api-field-endpoint-lbendpoint-metadata) 填充报头。元数据可以从任何名称空间中选择。通常，元数据值可以是字符串，数字，布尔值，列表，嵌套结构或空值。可以通过指定多个键从嵌套结构中选择上游元数据值。

否则，只支持字符串，布尔值和数值。如果未找到命名空间或键值，或者所选值不是受支持的类型，则不会发出标头。命名空间和键被指定为 JSON 字符串数组。