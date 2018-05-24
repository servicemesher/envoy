# 流量捕获

Envoy 当前提供了一个实验性的[传输套接字扩展](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/base.proto.html#core-transportsocket)，用于拦截流量并写入一个 [protobuf 文件](https://www.envoyproxy.io/docs/envoy/latest/api-v2/extensions/common/tap/v2alpha/capture.proto.html#envoy-api-msg-extensions-common-tap-v2alpha-trace)中。

> **警告**
>
> 这个功能是实验性的，并存在一个已知的问题，当在给定的 socket 上出现很长的跟踪调用的时候会 OOM。 如果担心存在安全问题，也可以在构建时禁用它，请参阅 <https://github.com/envoyproxy/envoy/blob/master/bazel/README.md#disabling-extensions>。

## 配置

捕获行为可以被配置在 [Listener](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.html#envoy-api-field-listener-filterchain-transport-socket) 和 [Cluster](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto.html#envoy-api-field-cluster-transport-socket) 上，提供了一种能力，可以分别针对上行流量和下行流量进行拦截。

要配置流量捕获, 添加一个`envoy.transport_sockets.capture`[配置](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/transport_socket/capture/v2alpha/capture.proto.html#envoy-api-msg-config-transport-socket-capture-v2alpha-capture)到 listener 或 cluster 上. 如：

```yaml
transport_socket:
  name: envoy.transport_sockets.capture
  config:
    file_sink:
      path_prefix: /some/capture/path
    transport_socket:
      name: raw_buffer
```

若支持 TLS, 如：

```yaml
transport_socket:
  name: envoy.transport_sockets.capture
  config:
    file_sink:
      path_prefix: /some/capture/path
    transport_socket:
      name: ssl
      config: <TLS context>
```

这里的 TLS 配置会分别替换现有在 listener 或 cluster 上 [下行流量](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-downstreamtlscontext) 或 [上行流量](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-upstreamtlscontext) 的 TLS 配置.

任一的 socket 实例都会生成一个包含`path`前缀的跟踪文件. 如：/some/capture/path_0.pb

## PCAP 传播

生成的跟踪文件可以被转成 [libpcap format](https://wiki.wireshark.org/Development/LibpcapFileFormat), 可以使用如 [Wireshark](https://www.wireshark.org/) 和 capture2pcap 这样的工具进行分析, 如:

```bash
bazel run @envoy_api//tools:capture2pcap /some/capture/path_0.pb path_0.pcap
tshark -r path_0.pcap -d "tcp.port==10000,http2" -P
  1   0.000000    127.0.0.1 → 127.0.0.1    HTTP2 157 Magic, SETTINGS, WINDOW_UPDATE, HEADERS
  2   0.013713    127.0.0.1 → 127.0.0.1    HTTP2 91 SETTINGS, SETTINGS, WINDOW_UPDATE
  3   0.013820    127.0.0.1 → 127.0.0.1    HTTP2 63 SETTINGS
  4   0.128649    127.0.0.1 → 127.0.0.1    HTTP2 5586 HEADERS
  5   0.130006    127.0.0.1 → 127.0.0.1    HTTP2 7573 DATA
  6   0.131044    127.0.0.1 → 127.0.0.1    HTTP2 3152 DATA, DATA
```
