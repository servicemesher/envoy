# 监听器过滤器

在[监听器](listeners.md#arch-overview-listeners)一节中讨论到，监听器过滤器可以用于操纵连接元数据。监听器过滤器的主要目的是来更方便地添加系统集成功能，而无需更改Envoy核心功能，并且还可以让多个此类功能之间的交互更加明确。

监听器过滤器的API相对简单，因为最终这些过滤器在新接收的套接字上操作。链中的过滤器可以停止后面的过滤器并随后继续支持。这允许更复杂的场景，例如调用[限速服务](global_rate_limiting.md#arch-overview-rate-limit)等。Envoy包含多个监听器过滤器，这些过滤器在此架构概述以及[配置参考](../../configuration/listener_filters/listener_filters.md#config-listener-filters)中都有记录。