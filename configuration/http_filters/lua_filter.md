# Lua

> **注意**
>
> 默认情况下，作为共享对象安装的 Lua 模块需要的符号，Envoy 在构建的时候不会导出它们。Envoy 可能需要建立对导出符号的支持。请参阅 [Bazel 文档](https://github.com/envoyproxy/envoy/blob/master/bazel/README.md)以获取更多信息。

## 概览

HTTP Lua 过滤器允许 [Lua](https://www.lua.org/) 脚本在请求和响应流期间运行。[LuaJIT](http://luajit.org/) 被作为运行时使用。因此，被支持的 Lua 版本大多是带一些 5.2 特性的 5.1版。更多细节参见 [LuaJIT 文档](http://luajit.org/extensions.md)。

该过滤器仅支持加载配置中内置的 Lua 代码。如果需要本地文件系统中的代码，可以使用内置小脚本从本地环境加载剩余的代码。

在高层次对过滤器设计以及 Lua 支持概括如下：

- 所有 Lua 环境都是 [针对工作线程的](../../intro/arch_overview/threading_model.md#arch-overview-threading)。这意味着没有真正的全局数据。任何在加载时创建和存在的全局数据将只在每个工作线程中可见。将来可能会通过添加 API 来支持真正的全局支持。
- 所有脚本作为协程运行。这意味着，即使它们可能执行复杂的异步操作，它们以同步方式编写。这使得脚本相当容易被编写。所有网络/异步处理通过一组 API 被 Envoy 执行。Envoy 将适当退出脚本并在异步任务完成后恢复。
- **不要在脚本中执行阻塞操作。** Envoy API 被用于所有 IO，这一点对性能很关键。

## 当前被支持的高层次特性:

*注意* 预计这个列表将在生产环境不断地使用该过滤器的过程中被扩充。API 的表面被有意地保持很小。以达到编写脚本极度简单和安全的目的。在非常复杂或高性能的使用案例中，默认用原生的 C++ 过滤器 API。

- 在请求流、响应流或二者同时流入时，检查头，正文和尾。
- 修改头和尾部。
- 阻塞并缓存全部请求/响应正文用作检查。
- 执行对外的异步 HTTP 调用至一个上游主机。可以在缓存正文数据时执行这样的调用，因此，当调用结束时，上游头可以被修改。
- 执行一个直接响应并略过更多的过滤器循环。例如，一个脚本可以做一个上游 HTTP 调用做鉴权，并随后直接以一个 403 响应代码响应。

## 配置

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/lua_filter#config-http-filters-lua-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/lua/v2/lua.proto#envoy-api-msg-config-filter-http-lua-v2-lua)

## 脚本示例

本节提供了一些 Lua 脚本的具体例子，作为更温和的介绍和快速开始。关于支持的 API 的更多细节，请参照[流处理 API](#config-http-filters-lua-stream-handle-api)。

```lua
-- 在请求路径上被调用。
function envoy_on_request(request_handle)
  -- 等待整个请求正文并添加一个请求头和正文的大小。
  request_handle:headers():add("request_body_size", request_handle:body():length())
end

-- 在响应路径上被调用。
function envoy_on_response(response_handle)
  -- 等待整个响应正文并添加一个请求头和正文的大小。
  response_handle:headers():add("response_body_size", response_handle:body():length())
  -- 移除一个名为 'foo' 的响应头。
  response_handle:headers():remove("foo")
end
```

```lua
function envoy_on_request(request_handle)
  -- 使用下面的头、正文和超时对上游主机做一个 HTTP 调用。
  local headers, body = request_handle:httpCall(
  "lua_cluster",
  {
    [":method"] = "POST",
    [":path"] = "/",
    [":authority"] = "lua_cluster"
  },
  "hello world",
  5000)

  -- 添加来自 HTTP 调用的信息到将要被发送到过滤器链中下一个过滤器的头。
  request_handle:headers():add("upstream_foo", headers["foo"])
  request_handle:headers():add("upstream_body_size", #body)
end
```

```lua
function envoy_on_request(request_handle)
  -- 做一个 HTTP 调用。
  local headers, body = request_handle:httpCall(
  "lua_cluster",
  {
    [":method"] = "POST",
    [":path"] = "/",
    [":authority"] = "lua_cluster"
  },
  "hello world",
  5000)

  -- 直接响应并从 HTTP 调用设置一个头。没有更多的过滤器循环发生。
  request_handle:respond(
    {[":status"] = "403",
     ["upstream_foo"] = headers["foo"]},
    "nope")
end
```

## 流操作 API

当 Envoy 加载配置中脚本，它寻找脚本定义的两个全局函数:

```lua
function envoy_on_request(request_handle)
end

function envoy_on_response(response_handle)
end
```

一个脚本可以定义这两个函数中的一个或者两个。在请求路径中，Envoy 将作为一个协程运行 *envoy_on_request*，传递一个 API 句柄。在相应路径中，Envoy 将作为一个协程运行 *envoy_on_response*，传递一个 API 句柄。

注意

所有与 Envoy 通过传递的流句柄发生的交互是至关重要的。流句柄不应该被赋值给任何全局变量，而且不应该被用于协程的外部。如果该句柄使用不正确，Envoy 将失败。

下列在流句柄上的方法被支持:

### headers()

```
headers = handle:headers()
```

返回流的头。只要它们还没有被发送到头链中的下一个过滤器，就可以被修改。例如，它们可以在一个 *httpCall()* 或者 *body()* 调用返回后被修改。如果头在任何其他情况下被修改，脚本将失败。

返回一个[头对象](#config-http-filters-lua-header-wrapper)。

### body()

```
body = handle:body()
```

返回流的正文。这个调用将造成 Envoy 退出脚本直到整个正文被缓存。注意，所有缓存必须遵从适当的流控策略。Envoy 将不会缓存比连接管理器允许的更多的数据。

返回一个[缓存对象](#config-http-filters-lua-buffer-wrapper)。

### bodyChunks()

```
iterator = handle:bodyChunks()
```

返回一个迭代器，它可被用于在所有正文块到达时循环访问它们。Envoy 将在块之间退出脚本，但是 *将不会缓存* 它们。这可被一个脚本用于在数据流入时做检查。

```
for chunk in request_handle:bodyChunks() do
  request_handle:log(0, chunk:length())
end
```

迭代器返回的每一块是一个[缓存对象](#config-http-filters-lua-buffer-wrapper)。

### trailers()

```
trailers = handle:trailers()
```

返回流的尾。如果没有尾，可能返回空。尾在被发送到下一个过滤器之前可能被修改。

返回一个[头对象](#config-http-filters-lua-header-wrapper)。

### log*()

```
handle:logTrace(message)
handle:logDebug(message)
handle:logInfo(message)
handle:logWarn(message)
handle:logErr(message)
handle:logCritical(message)
```

使用 Envoy 的应用日志功能保存一条消息。*message* 是被保存的字符串。

### httpCall()

```
headers, body = handle:httpCall(cluster, headers, body, timeout)
```

对一台上游主机做一个 HTTP 调用。Envoy 将推出脚本直到调用结束或者有一个错误。*cluster* 是一个字符串，映射为一个配置好的集群管理器。*headers* 一个要发送的键值对的表。注意，*:method*，*:path* 和 *:authority* 头必须被设置。*body* 是一个可选的要发送的正文数据的字符串。*timeout* 是一个整数，指定以微秒为单位的调用超时。

返回 *headers*，这是响应头的一个表。返回 *body*，这是响应正文的字符串。如果没有正文，则返回空。

### respond()

```
handle:respond(headers, body)
```

立即响应并且不做更多的循环。这个调用*仅在请求流中合法*。另外，只有当请求头还没有被传递给后续过滤器时才可能。这意味着，下面的 Lua 代码是不合法的:

```lua
function envoy_on_request(request_handle)
  for chunk in request_handle:bodyChunks() do
    request_handle:respond(
      {[":status"] = "100"},
      "nope")
  end
end
```

*headers* 一张要发送的键值对的表。注意，*:status* 头必须被设置。*body* 是一个字符串并提供了可选的响应正文，可能为空。

### metadata()

```
metadata = handle:metadata()
```

返回当前路由条目元数据。注意，元数据应该在过滤器名下指定，例如*envoy.lua*。下面是一个 在[路由条目](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route)中 *metadata* 的例子。

```yaml
metadata:
  filter_metadata:
    envoy.lua:
      foo: bar
      baz:
        - bad
        - baz
```

返回一个[元数据对象](#config-http-filters-lua-metadata-wrapper)。

## 头对象 API

### add()

```
headers:add(key, value)
```

添加一个头。*key* 是一个提供头键的字符串。*value*是一个提供头值的字符串。

注意

Envoy 特殊处理特定头。这些被称为 O(1) 或 *inline* 头。一个内建头的列表可以在[这里](https://github.com/envoyproxy/envoy/blob/6b6ce85a24094146c5c225f19b6ecc47b2ca84bf/include/envoy/http/header_map.h#L228)找到。如果一个内建头已经呈现在头映射中，*add()* 将没有效果。如果试图去 *add()* 一个非内建的头，附加的头将会被添加，因此，合成的头包含多个同名的头。如果想要将头换为另一个值，可以考虑使用 *replace* 函数。注意，我们理解这个行为令人迷惑并且我们可能在将来的一版中改变。

### get()

```
headers:get(key)
```

得到一个头。*key* 是一个提供头键的字符串。返回一个头值的字符串，如果没有这个头则返回空。

### __pairs()

```
for key, value in pairs(headers) do
end
```

循环访问每个头。*key* 是一个提供头键的字符串。*value*是一个提供头值的字符串。

注意

在当前的实现中，头不能在循环期间被修改。另外，如果想要在循环后修改头，循环必须完成。这意味着，不用过早使用跳出或其他机制退出循环体。这个要求可能在将来被放松。

### remove()

```
headers:remove(key)
```

移除一个头。 *key* 提供了要移除的头键。

### replace()

```
headers:replace(key, value)
```

替换一个头。*key* 是一个提供头键的字符串。*value*是一个提供头值的字符串。如果头不存在，它用 *add()* 函数添加。

## 缓存 API

### length()

```
size = buffer:length()
```

获得以字节为单位的缓存的大小。返回一个整数。

### getBytes()

```
buffer:getBytes(index, length)
```

从缓存中获得字节。Envoy 默认将不会拷贝所有的缓存数据给 Lua。这将造成一个缓存段被拷贝。*index* 是一个整数并提供了缓存要开始拷贝的索引。*length* 是一个整数并提供了要拷贝的缓存长度。*index* + *length* 必须小于缓存的长度。

## 元数据对象 API

### get()

```
metadata:get(key)
```

获得一个元数据。 *key* 是一个提供了元数据键的字符串。返回给定元数据键对应的值。值的类型可以是: *null*, *boolean*, *number*, *string* 和 *table*。

### __pairs()

```
for key, value in pairs(metadata) do
end
```

循环访问每个 *metadata* 条目。*key* 是一个提供 *metadata* 键的字符串。 *value* 是 *metadata* 条目值。
