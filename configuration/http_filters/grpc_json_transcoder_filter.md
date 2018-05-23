# gRPC-JSON 转码

- gRPC [architecture overview](../../intro/arch_overview/grpc.md#arch-overview-grpc)
- [v1 API reference](../../api-v1/http_filters/grpc_json_transcoder_filter.md#config-http-filters-grpc-json-transcoder-v1)
- [v2 API reference](../../api-v2/config/filter/http/transcoder/v2/transcoder.proto.md#envoy-api-msg-config-filter-http-transcoder-v2-grpcjsontranscoder)

这个过滤器可以实现Restful JSON风格客户端通过HTTP协议向Envoy发送请求，透过代理实现对一个gRPC服务的访问。HTTP与gRPC服务之间的映射关系定义在 [自定义选项](https://cloud.google.com/service-management/reference/rpc/google.api#http).

## 如何创建原型描述集

为了实现代码转换，Envoy需要知道你的gRPC服务的原型描述符。

要为gRPC服务创建一个协议描述符，你需要在运行protoc之前，先在GitHub上把googleapis仓库克隆到本地，因为你需要在包含路径中包含有annotations.proto，才能定义HTTP的映射。


```
git clone https://github.com/googleapis/googleapis
GOOGLEAPIS_DIR=<your-local-googleapis-folder>
```

Then run protoc to generate the descriptor set from bookstore.proto:

然后运行protoc从bookstore.proto中生成描述集:

```
protoc -I$(GOOGLEAPIS_DIR) -I. --include_imports --include_source_info \
  --descriptor_set_out=proto.pb test/proto/bookstore.proto
```

如果你有多个proto源文件，你可以把它们全部传递到一条命令中。