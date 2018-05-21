# 统计

发出一些统计数据来统计系统行为:

| 名称           | 类型     |         描述                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| stats.overflow | Counter   | 由于共享内存不足，Envoy 无法分配统计的总次数             |

## Server

根植与*server.* 与服务器相关的统计信息，统计信息如下:

| 名称                         | 类型  | 描述                                                 |
| ------------------------------ | ----- | ------------------------------------------------------------ |
| uptime                         | Gauge | 当前 server 的启动时间                           |
| memory_allocated               | Gauge | 当前分配的内存大小，以字节为单位 |
| memory_heap_size               | Gauge | 当前堆预留的内存大小，以字节为单位  |
| live                           | Gauge | 1：当前server还没有耗尽；0：其他   |
| parent_connections             | Gauge | 在热启动中所有就的 Envoy 进程的连接数 |
| total_connections              | Gauge | 所有的 Envoy 进程的连接数，包括新的、旧的 |
| version                        | Gauge | 整型表示基于 SCM 修订的版本号 |
| days_until_first_cert_expiring | Gauge | 下一个证书到期的天数 |

## File system

在 *filesystem.* 中发出的与文件系统相关的统计信息，名称空间

| 名称                 | 类型    | 描述                                             |
| -------------------- | ------- | ------------------------------------------------ |
| write_buffered       | Counter | 文件数据被移动到 Envoy 内部缓冲区的总次数        |
| write_completed      | Counter | 文件被写入的总次数                               |
| flushed_by_timer     | Counter | 由于刷新超时而导致内部刷新缓冲区写入文件的总次数 |
| reopen_failed        | Counter | 文件被打开失败的总次数                           |
| write_total_buffered | Gauge   | 当前内部数据缓冲区的总大小，以字节为单位         |
