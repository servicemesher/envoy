# gRPC-JSON 转码

- gRPC [架构简述](../../intro/arch_overview/grpc.md#arch-overview-grpc)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/grpc_json_transcoder_filter#config-http-filters-grpc-json-transcoder-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/transcoder/v2/transcoder.proto#envoy-api-msg-config-filter-http-transcoder-v2-grpcjsontranscoder)

这个过滤器可以实现 Restful JSON 风格客户端通过 HTTP 协议向 Envoy 发送请求，透过代理实现对一个 gRPC 服务的访问。 HTTP 与 gRPC 服务之间的映射关系定义在 [自定义选项](https://cloud.google.com/service-management/reference/rpc/google.api#http)。

## 如何创建原型描述集

为了实现代码转换， Envoy 需要知道你的 gRPC 服务的原型描述符。

要为 gRPC 服务创建一个协议描述符，你需要在运行 protoc 之前，先在 GitHub 上把 googleapis 仓库克隆到本地，因为你需要在包含路径中包含有 annotations.proto ，才能定义 HTTP 的映射。

```bash
git clone https://github.com/googleapis/googleapis
GOOGLEAPIS_DIR=<your-local-googleapis-folder>
```

然后运行 protoc 从 bookstore.proto 中生成描述集:

```bash
protoc -I$(GOOGLEAPIS_DIR) -I. --include_imports --include_source_info \
  --descriptor_set_out=proto.pb test/proto/bookstore.proto
```

如果你有多个 proto 源文件，你可以把它们全部传递到一条命令中。
