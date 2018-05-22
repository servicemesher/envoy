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

当gzip过滤器被置为enabled状态，请求和响应头都被检测，从而决定是否内容需要被压缩。如果请求或响应允许的话，内容被压缩并随后带着相应的http头发送给客户端
When gzip filter is enabled, request and response headers are inspected to determine whether or not the content should be compressed. The content is compressed and then sent to the client with the appropriate headers if either response and request allow.

在*默认*情况下，压缩操作被忽略。这些情况如下所列：
By *default* compression will be *skipped* when:

- 请求不包含头*accept-encoding*
- A request does NOT contain *accept-encoding* header.
- 请求包含头*accept-encoding*，但是不包含“gzip”
- A request includes *accept-encoding* header, but it does not contain “gzip”.
- 响应包含头*content-encoding*
- A response contains a *content-encoding* header.
- 响应包含头*cache-control*，该头的值包含"no-transform"
- A Response contains a *cache-control* header whose value includes “no-transform”.
- 响应包含头*transfer-encoding*，该头的值包含"gzip"
- A response contains a *transfer-encoding* header whose value includes “gzip”.
- 响应的*content-type*值，不和如下的mime-type值相匹配，这些mime-type值包含：*application/javascript*, *application/json*, *application/xhtml+xml*, *image/svg+xml*, *text/css*, *text/html*, *text/plain*, *text/xml*
- A response does not contain a *content-type* value that matches one of the selected mime-types, which default to *application/javascript*, *application/json*, *application/xhtml+xml*, *image/svg+xml*, *text/css*, *text/html*, *text/plain*, *text/xml*.
- 响应中及不包含*content-type*头，也不包含*transfer-encoding*头
- Neither *content-length* nor *transfer-encoding* headers are present in the response.
- 响应长度小于30个字节（仅应用于*transfer-encoding*没有被分成大块）
- Response size is smaller than 30 bytes (only applicable when *transfer-encoding* is not chuncked).

在以下情况，压缩操作被*使用*：
When compression is *applied*:

- *content-length*被从响应头中移除
- The *content-length* is removed from response headers.
- 响应头包含“*transfer-encoding: chunked*” 以及 “*content-encoding: gzip*”
- Response headers contain “*transfer-encoding: chunked*” and “*content-encoding: gzip*”.
- 头“*vary: accept-encoding*” 被插入到了每一个响应中
- The “*vary: accept-encoding*” header is inserted on every response.
