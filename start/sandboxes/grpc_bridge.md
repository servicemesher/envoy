# gRPC 网桥

## Envoy gRPC

gRPC 网桥沙箱是 Envoy 的 [gRPC 网桥过滤器](../../configuration/http_filters/grpc_http1_bridge_filter.md#config-http-filters-grpc-bridge)的一个实例。包含在沙箱中的是带有 Python HTTP 客户端的gRPC 内存键/值存储。Python客户端通过 Envoy sidecar 进程发出 HTTP/1请求，并将其升级为 HTTP/2 gRPC 请求。然后响应 trailer 被缓冲并作为 HTTP/1 标头的有效载荷发送回客户端。

本例中演示的 Envoy 的另一个功能是通过其路由配置执行权威基础路由。

## 构建 Go 服务

运行下面的命令构建 Go gRPC 服务：

```bash
$ pwd
envoy/examples/grpc-bridge
$ script/bootstrap
$ script/build
```

注意：`build` 命令要求 Envoy 代码库（或其工作副本）位于 `$GOPATH/src/github.com/envoyproxy/envoy`。

## Docker compose

运行 docker compose 文件，同时运行 Python 和 个RPC 容器：

```bash
$ pwd
envoy/examples/grpc-bridge
$ docker-compose up --build
```

## 向键/值存储发送请求

使用 Python 服务，发送 gRPC 请求：

```bash
$ pwd
envoy/examples/grpc-bridge
# set a key
$ docker-compose exec python /client/client.py set foo bar
setf foo to bar

# get a key
$ docker-compose exec python /client/client.py get foo
bar

# modify an existing key
$ docker-compose exec python /client/client.py set foo baz
setf foo to baz

# get the modified key
$ docker-compose exec python /client/client.py get foo
baz
```

在运行的 docker-compose 容器中，您应该可以看到 gRPC 服务打印的活动的记录：

```
grpc_1    | 2017/05/30 12:05:09 set: foo = bar
grpc_1    | 2017/05/30 12:05:12 get: foo
grpc_1    | 2017/05/30 12:05:18 set: foo = baz
```