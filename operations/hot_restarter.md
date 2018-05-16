热重启 Python 包装器
==========================

通常情况下，Envoy 将会以[热重启](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart)的方式进行配置变更和二进制更新。但是，在很多情况下，用户会希望使用标准进程管理器，例如 monit、runit 等。我们提供 [/restarter/hot-restarter.py](https://github.com/envoyproxy/envoy/blob/master//restarter/hot-restarter.py) 来使这个过程简单明了。

调用重启程序方式如下:

```bash
hot-restarter.py start_envoy.sh
```

start_envoy.sh 可能像这样定义（使用 salt/jinja 类似的语法）：

```bash
#!/bin/bash

ulimit -n {{ pillar.get('envoy_max_open_files', '102400') }}
exec /usr/sbin/envoy -c /etc/envoy/envoy.cfg --restart-epoch $RESTART_EPOCH --service-cluster {{ grains['cluster_name'] }} --service-node {{ grains['service_node'] }} --service-zone {{ grains.get('ec2_availability-zone', 'unknown') }}
```

在每次重启时，*RESTART_EPOCH* 环境变量是由重启程序设置，并且可以传递给 [`--restart-epoch`](cli.md#cmdoption-restart-epoch) 选项

重启程序可以处理以下信号：

- **SIGTERM**：将干净地终止所有子进程并退出。
- **SIGHUP**：将重新调用作为第一个参数传递给热重启程序的脚本，来进行热重启。
- **SIGCHLD**：如果任何子进程意外关闭，那么重启脚本将关闭所有内容并退出以避免处于意外状态。随后，控制进程管理器应该重新启动重启脚本以再次启动Envoy。
- **SIGUSR1**：将作为重新打开所有访问日志的信号，转发给Envoy。可用于原子移动以及重新打开日志轮转。
