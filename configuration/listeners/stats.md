# 统计

## 监听器

每个监听器的统计树根在 *`listener.<address>.`* ，有如下统计:

| 名称                      | 类似      | 描述                                  |
| ------------------------- | --------- | ------------------------------------- |
| downstream_cx_total       | Counter   | 连接总数                              |
| downstream_cx_destroy     | Counter   | 销毁连接总数                          |
| downstream_cx_active      | Gauge     | 活动连接总数                          |
| downstream_cx_length_ms   | Histogram | 连接长度，单位毫秒                    |
| ssl.connection_error      | Counter   | TLS 连接错误总数，不包括证书认证失败  |
| ssl.handshake             | Counter   | TLS连接握手成功总数                   |
| ssl.session_reused        | Counter   | TLS会话恢复成功总数                   |
| ssl.no_certificate        | Counter   | 不带客户端证书的TLS连接成功总数       |
| ssl.fail_no_sni_match     | Counter   | 因为缺少SNI匹配而被拒绝的TLS连接总数  |
| ssl.fail_verify_no_cert   | Counter   | 因为缺少客户端证书而失败的TLS连接总数 |
| ssl.fail_verify_error     | Counter   | CA认证失败的TLS连接总数               |
| ssl.fail_verify_san       | Counter   | SAN认证失败的TLS连接总数              |
| ssl.fail_verify_cert_hash | Counter   | 认证pinning认证失败的TLS连接总数      |
| ssl.cipher.<cipher>       | Counter   | 使用 <cipher> 的TLS连接总数           |

## 监听器管理器

监听器管理器的统计树根在 *listener_manager.* ，有以下统计。所有 stats 名称中的 `:` 字符被替换为 `_`。

| 名称                     | 类型    | 描述                                           |
| ------------------------ | ------- | ---------------------------------------------- |
| listener_added           | Counter | 添加的监听器总数（不管是通过静态配置还是 LDS） |
| listener_modified        | Counter | 修改过的监听器总数（通过LDS）                  |
| listener_removed         | Counter | 删除过的监听器总数（通过LDS）                  |
| listener_create_success  | Counter | 成功添加到 workers 的监听器对象总数            |
| listener_create_failure  | Counter | 添加到 workers 失败的监听器对象总数            |
| total_listeners_warming  | Gauge   | 当前热身中的监听器数量                         |
| total_listeners_active   | Gauge   | 当前活动中的监听器数量                         |
| total_listeners_draining | Gauge   | 当前排除中的监听器数量                         |

