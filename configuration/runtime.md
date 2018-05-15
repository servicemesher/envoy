# 运行时

The [runtime configuration](../intro/arch_overview/runtime.md#arch-overview-runtime) specifies the location of the local file system tree that contains re-loadable configuration elements. Values can be viewed at the [/runtime admin endpoint](../operations/admin.md#operations-admin-interface-runtime). Values can be modified and added at the [/runtime_modify admin endpoint](../operations/admin.md#operations-admin-interface-runtime-modify). If runtime is not configured, an empty provider is used which has the effect of using all defaults built into the code, except for any values added via /runtime_modify.

Attention

Use the [/runtime_modify](../operations/admin.md#operations-admin-interface-runtime-modify) endpoint with care. Changes are effectively immediately. It is **critical**that the admin interface is [properly secured](../operations/admin.md#operations-admin-interface-security).

- [v1 API reference](../api-v1/runtime.md#config-runtime-v1)
- [v2 API reference](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-msg-config-bootstrap-v2-runtime)

## File system layout

Various sections of the configuration guide describe the runtime settings that are available. For example, [here](cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime) are the runtime settings for upstream clusters.

Assume that the folder `/srv/runtime/v1` points to the actual file system path where global runtime configurations are stored. The following would be a typical configuration setting for runtime:

- *symlink_root*: `/srv/runtime/current`
- *subdirectory*: `envoy`
- *override_subdirectory*: `envoy_override`

Where `/srv/runtime/current` is a symbolic link to `/srv/runtime/v1`.

Each ‘.’ in a runtime key indicates a new directory in the hierarchy, rooted at *symlink_root* +*subdirectory*. For example, the *health_check.min_interval* key would have the following full file system path (using the symbolic link):

`/srv/runtime/current/envoy/health_check/min_interval`

The terminal portion of a path is the file. The contents of the file constitute the runtime value. When reading numeric values from a file, spaces and new lines will be ignored.

The *override_subdirectory* is used along with the [`--service-cluster`](../operations/cli.md#cmdoption-service-cluster) CLI option. Assume that [`--service-cluster`](../operations/cli.md#cmdoption-service-cluster) has been set to `my-cluster`. Envoy will first look for the*health_check.min_interval* key in the following full file system path:

`/srv/runtime/current/envoy_override/my-cluster/health_check/min_interval`

If found, the value will override any value found in the primary lookup path. This allows the user to customize the runtime values for individual clusters on top of global defaults.

## Comments

Lines starting with `#` as the first character are treated as comments.

Comments can be used to provide context on an existing value. Comments are also useful in an otherwise empty file to keep a placeholder for deployment in a time of need.

## Updating runtime values via symbolic link swap

There are two steps to update any runtime value. First, create a hard copy of the entire runtime tree and update the desired runtime values. Second, atomically swap the symbolic link root from the old tree to the new runtime tree, using the equivalent of the following command:

```
/srv/runtime:~$ ln -s /srv/runtime/v2 new && mv -Tf new current
```

It’s beyond the scope of this document how the file system data is deployed, garbage collected, etc.

## Statistics

The file system runtime provider emits some statistics in the *runtime.* namespace.

| Name                    | Type    | Description                                                  |
| ----------------------- | ------- | ------------------------------------------------------------ |
| load_error              | Counter | Total number of load attempts that resulted in an error      |
| override_dir_not_exists | Counter | Total number of loads that did not use an override directory |
| override_dir_exists     | Counter | Total number of loads that did use an override directory     |
| load_success            | Counter | Total number of load attempts that were successful           |
| num_keys                | Gauge   | Number of keys currently loaded                              |