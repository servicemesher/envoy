热重启 Python 包装器
==========================

Typically, Envoy will be [hot restarted](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart) for config changes and binary updates. However, in many cases, users will wish to use a standard process manager such as monit, runit, etc. We provide [/restarter/hot-restarter.py](https://github.com/envoyproxy/envoy/blob/master//restarter/hot-restarter.py) to make this straightforward.

The restarter is invoked like so:

```bash
hot-restarter.py start_envoy.sh
```

start_envoy.sh might be defined like so (using salt/jinja like syntax):

```bash
#!/bin/bash

ulimit -n {{ pillar.get('envoy_max_open_files', '102400') }}
exec /usr/sbin/envoy -c /etc/envoy/envoy.cfg --restart-epoch $RESTART_EPOCH --service-cluster {{ grains['cluster_name'] }} --service-node {{ grains['service_node'] }} --service-zone {{ grains.get('ec2_availability-zone', 'unknown') }}
```

The *RESTART_EPOCH* environment variable is set by the restarter on each restart and can be passed to the [`--restart-epoch`](cli.md#cmdoption-restart-epoch) option.

The restarter handles the following signals:

- **SIGTERM**: Will cleanly terminate all child processes and exit.
- **SIGHUP**: Will hot restart by re-invoking whatever is passed as the first argument to the hot restart script.
- **SIGCHLD**: If any of the child processes shut down unexpectedly, the restart script will shut everything down and exit to avoid being in an unexpected state. The controlling process manager should then restart the restarter script to start Envoy again.
- **SIGUSR1**: Will be forwarded to Envoy as a signal to reopen all access logs. This is used for atomic move and reopen log rotation.