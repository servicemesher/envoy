# Gzip

Gzip是一个http过滤器。该过滤器允许Envoy压缩从上游服务基于客户端请求分发的数据。在大量数据需要传输而响应时间被限定的情况下，压缩操作是有用的。
Gzip is an HTTP filter which enables Envoy to compress dispatched data from an upstream service upon client request. Compression is useful in situations where large payloads need to be transmitted without compromising the response time.

## Configuration

- [v2 API reference](../../api-v2/config/filter/http/gzip/v2/gzip.proto.md#envoy-api-msg-config-filter-http-gzip-v2-gzip)

> **Attention**
> **注意**
>
> *window bits* 是一个数字。这个数字告诉压缩器，算法应该在文本之前多大范围来寻找重复的字符序列。由于在zlib库里的一个已知bug，*window bits*的值如果是8，则这个值不能按照预期运行。因此任何小于8的值，应该被自动置为9。这个问题在库的未来版本将被解决
> The *window bits* is a number that tells the compressor how far ahead in the text the algorithm should be looking for repeated sequence of characters. Due to a known bug in the underlying zlib library, *window bits* with value eight does not work as expected. Therefore any number below that will be automatically set to 9. This issue might be solved in future releases of the library.

## How it works
## 它是如何工作的

当gzip过滤器被置为enabled状态，请求和响应头都被检测，从而决定是否内容需要被压缩。
When gzip filter is enabled, request and response headers are inspected to determine whether or not the content should be compressed. The content is compressed and then sent to the client with the appropriate headers if either response and request allow.

By *default* compression will be *skipped* when:

- A request does NOT contain *accept-encoding* header.
- A request includes *accept-encoding* header, but it does not contain “gzip”.
- A response contains a *content-encoding* header.
- A Response contains a *cache-control* header whose value includes “no-transform”.
- A response contains a *transfer-encoding* header whose value includes “gzip”.
- A response does not contain a *content-type* value that matches one of the selected mime-types, which default to *application/javascript*, *application/json*, *application/xhtml+xml*, *image/svg+xml*, *text/css*, *text/html*, *text/plain*, *text/xml*.
- Neither *content-length* nor *transfer-encoding* headers are present in the response.
- Response size is smaller than 30 bytes (only applicable when *transfer-encoding* is not chuncked).

When compression is *applied*:

- The *content-length* is removed from response headers.
- Response headers contain “*transfer-encoding: chunked*” and “*content-encoding: gzip*”.
- The “*vary: accept-encoding*” header is inserted on every response.
