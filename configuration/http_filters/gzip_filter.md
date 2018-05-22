# Gzip

Gzip 是一个 http 过滤器。该过滤器允许 Envoy 压缩从上游服务基于客户端请求分发的数据。在大量数据需要传输而响应时间被限定的情况下，压缩操作是有用的。

## Configuration

- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/gzip/v2/gzip.proto.html#envoy-api-msg-config-filter-http-gzip-v2-gzip)

> **注意**
>
> *window bits* 是一个数字。这个数字告诉压缩器，算法应该在文本之前多大范围来寻找重复的字符序列。由于在zlib库里的一个已知bug，*window bits* 的值如果是 8，则这个值不能按照预期运行。因此任何小于 8 的值，应该被自动置为 9。这个问题在库的未来版本将被解决

## 它是如何工作的

当 gzip 过滤器被置为 enabled 状态，请求和响应头都被检测，从而决定是否内容需要被压缩。如果请求或响应允许的话，内容被压缩并随后带着相应的http头发送给客户端

在*默认*情况下，压缩操作被忽略。这些情况如下所列：

- 请求不包含头 *accept-encoding*
- 请求包含头 *accept-encoding*，但是不包含 “gzip”
- 响应包含头 *content-encoding*
- 响应包含头 *cache-control*，该头的值包含 "no-transform"
- 响应包含头 *transfer-encoding*，该头的值包含 "gzip"
- 响应的 *content-type* 值，不和如下的 mime-type 值相匹配，这些 mime-type 值包含：*application/javascript*, *application/json*, *application/xhtml+xml*, *image/svg+xml*, *text/css*, *text/html*, *text/plain*, *text/xml*
- 响应中及不包含 *content-type* 头，也不包含 *transfer-encoding* 头
- 响应长度小于 30 个字节（仅应用于 *transfer-encoding* 没有被分成大块）

在以下情况，压缩操作被*使用*：

- *content-length* 被从响应头中移除
- 响应头包含 “*transfer-encoding: chunked*” 以及 “*content-encoding: gzip*”
- 头 “*vary: accept-encoding*”  被插入到了每一个响应中
