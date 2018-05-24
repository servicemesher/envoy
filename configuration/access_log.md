# 访问日志

## 配置

访问日志配置是 [HTTP 连接管理器配置](http_conn_man/http_conn_man.md#config-http-conn-man)或者 [TCP 代理配置](network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy) 中的一部分。

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/access_log#config-access-log-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/accesslog/v2/accesslog.proto#envoy-api-msg-config-filter-accesslog-v2-accesslog)

## 格式规则

访问日志格式字符串包含了命令操作符以及其他文本。访问日志的格式化过程并不会对换行符做出什么假设，所以应该将其设置为格式字符串的组成部分。请参阅示例的[缺省格式](#config-access-log-default-format)。要注意的是，访问日志会在未设置、或者空值的位置加入一个字符：“-”。

不同类型的访问日志（例如 HTTP 和 TCP）共用同样的格式字符串。不同类型的日志中，某些字段可能会有不同的含义。

下面是支持的命令：

- *%START_TIME%*

  - *HTTP*

    Request 的开始时间，精确到毫秒。

  - *TCP*

    下游连接的开始时间，精确到毫秒。

    可以用[格式化字符串](http://en.cppreference.com/w/cpp/io/manip/put_time)对 START_TIME 进行定制，例如：`%START_TIME(%Y/%m/%dT%H:%M:%S%z %s)%`

- *%BYTES_RECEIVED%*

  - *HTTP*

    收到的 Body 字节数。

  - *TCP*

    下游连接上收到的字节数。

- *%PROTOCOL%*

  - *HTTP*

    协议。目前取值为 *HTTP/1.1* 或 *HTTP/2*。

  - *TCP*

    未实现 (“-“)。

- *%RESPONSE_CODE%*

  - *HTTP*

    HTTP 响应码。注意如果响应码为 0，说明服务器从未发送过响应，这一般是说明（下游）客户端断开了。

  - *TCP*

    未实现 (“-“)。

- *%BYTES_SENT%*

  - *HTTP*

    发送出去的 Body 字节数。

  - *TCP*

    在连接中向下游发送的字节数。

- *%DURATION%*

  - *HTTP*

    一个请求从开始到最后一个字节传输完成的的持续时间，单位是毫秒。

  - *TCP*

    下游连接的持续时间。

- *%RESPONSE_FLAGS%*

  关于响应或连接的附加细节。这里描述的响应码是不适用于 TCP 连接的。可能的取值包括：

  - *HTTP 和 TCP*
    - *UH*：503 响应码的补充信息，上游集群中没有健康的上游主机。
    - *UF*：503 响应码的补充信息，上游连接失败。
    - *UO*：503 响应码的补充信息，上游溢出（[熔断](../../intro/arch_overview/circuit_breaking.md#arch-overview-circuit-break)状态）。
    - *NR*：404 响应码的补充信息，给定请求没有[路由配置](../../intro/arch_overview/http_routing.md#arch-overview-http-routing)。

  - *HTTP 专用*
    - *LH*：503 响应码的补充信息，本地服务[健康检查](../../intro/arch_overview/health_checking.md#arch-overview-health-checking)失败。
    - *UT*：504 响应码的补充信息，上游请求超时。
    - *LR*：503 响应码的补充信息，连接在本地被复位。
    - *UR*：503 响应码的补充信息，上游复位连接。
    - *UC*：503 响应码的补充信息，上游链接终止。
    - *DI*：该请求受[错误注入](http_filters/fault_filter.md#config-http-filters-fault-injection)功能影响，延迟指定时间。
    - *FI*：这一请求受[错误注入](http_filters/fault_filter.md#config-http-filters-fault-injection)功能影响，返回了指定的状态码。
    - *RL*：429 响应码的补充信息，[HTTP 频率限制过滤器](http_filters/rate_limit_filter#config-http-filters-rate-limit)的配置限制了这一请求。

- *%UPSTREAM_HOST%*

  上游主机的 URL（对 TCP 链接来说就是：`tcp://ip:port`）

- *%UPSTREAM_CLUSTER%*

  上游主机所属的上游集群。

- *%UPSTREAM_LOCAL_ADDRESS%*

  上游连接的本地地址，如果是 Ip 地址，那么其中会包含地址和端口。

- *%DOWNSTREAM_REMOTE_ADDRESS%*

  下游连接的远程地址，如果是 Ip 地址，那么其中会包含地址和端口。

  > 注意：如果地址来自于 [proxy proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-field-listener-filterchain-use-proxy-proto) 或者 [x-forwarded-for](http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for) 这可能不是远端节点的物理地址。

- *%DOWNSTREAM_REMOTE_ADDRESS_WITHOUT_PORT%*

  下游连接的远程地址，如果是 Ip 地址，那么其中不会包含地址和端口。

  > 注意：如果地址来自于 [proxy proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-field-listener-filterchain-use-proxy-proto) 或者 [x-forwarded-for](http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for) 这可能不是远端节点的物理地址。

- *%DOWNSTREAM_LOCAL_ADDRESS%*

  下游连接的本地地址。如果是 Ip 地址，那么其中会包含地址和端口。如果原始连接来自于 iptables 的 REDIRECT，那么这里的原始目标地址会由[原始目标过滤器](listener_filters/original_dst_filter#config-listener-filters-original-dst)通过 `SO_ORIGINAL_DST` 这一 Socket 选项进行获取。如果原始连接是来自 iptables 的 TPROXY，并且监听器的 `transparent` 选项处于开启状态，这里会显示原始的目标地址和端口。

- *%DOWNSTREAM_LOCAL_ADDRESS_WITHOUT_PORT%*

  和 *%DOWNSTREAM_LOCAL_ADDRESS%* 类似，但是不包含端口部分。

- *%REQ(X?Y):Z%*

  - *HTTP*

    一个 HTTP 请求 Header，X 是主要 HTTP 头，Y 是备选 Header，Z 是一个可选的参数，表示截取的字符串的长度。这个值来源于 HTTP Header，首先读取 X，如果 X 没有设置，则读取 Y。如果两个 Header 都没有，在日志中就会写入“-”

  - *TCP*

    未实现 (“-“)。

- *%RESP(X?Y):Z%*

  - *HTTP*

    和 *%REQ(X?Y):Z%* 类似，只不过值来自于响应而非请求。

  - *TCP*

    未实现 (“-“)。

- *%TRAILER(X?Y):Z%*

  - *HTTP*

    和 *%REQ(X?Y):Z%* 类似，只不过值来自于响应尾部。

  - *TCP*

    未实现 (“-“)。

- *%DYNAMIC_METADATA(NAMESPACE:KEY*):Z%*

  - *HTTP*

    [动态元数据](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/base.proto#envoy-api-msg-core-metadata)信息，NAMESPACE 就是设置用于过滤命名空间的参数，KEY 是一个可选参数，用于在命名空间内查询指定的键，可以用 `:` 来表达嵌套关系，Z 是一个可选的参数，表示截断字符串的长度。动态元数据可以使用 [RequestInfo](https://github.com/envoyproxy/envoy/blob/master/include/envoy/request_info/request_info.h) 进行设置：`setDynamicMetadata`。这个数据以 JSON 形式进行记录，例如下面的动态元数据：`com.test.my_filter: {"test_key": "foo", "test_object": {"inner_key": "bar"}}`

    - `%DYNAMIC_METADATA(com.test.my_filter)%` 会记录：`{"test_key": "foo", "test_object": {"inner_key": "bar"}}`
    - `%DYNAMIC_METADATA(com.test.my_filter:test_key)%` 会记录：`"foo"`
    - `%DYNAMIC_METADATA(com.test.my_filter:test_object)%` 会记录： `{"inner_key": "bar"}`
    - ``%DYNAMIC_METADATA(com.test.my_filter:test_object:inner_key)%` 会记录：`"bar"`
    - `%DYNAMIC_METADATA(com.unknown_filter)%` 会记录： `-`
    - `%DYNAMIC_METADATA(com.test.my_filter:unknown_key)%` 会记录： `-`
    - `%DYNAMIC_METADATA(com.test.my_filter):25%` 会记录（截取到 25 个字符）: {"test_key": "foo", "test`

  - *TCP*

    未实现 (“-“)。

## 缺省格式

如果没有设置自定义格式，Envoy 会使用如下的缺省格式：

    [%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%"
    %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION%
    %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%"
    "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n

缺省的访问日志格式示例：

    [2016-04-15T20:17:00.310Z] "POST /api/v1/locations HTTP/2" 204 - 154 0 226 100 "10.0.35.28" "nsq2http" "cc21d9b0-cf5c-432b-8c7e-98aeb7988cd2" "locations" "tcp://10.0.2.1:80"
