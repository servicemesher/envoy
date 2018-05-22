# Schema 验证器检查工具

Schema 验证器工具验证被传入的 JSON 符合配置中的某个 schema。为验证整个配置，请参考[配置负载检查工具](config_load_check_tool.md#install-tools-config-load-check-tool)。当前，仅[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route_config.html#config-http-conn-man-route-table) schema 验证被支持。

- 输入

  工具期望两个输入：检查传入的 JSON 所用的 schema 类型。对于[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route_config.html#config-http-conn-man-route-table) 验证被支持的类型是 :route。JSON 所在的路径。

- 输出

  如果 JSON 符合 schema，工具将以状态 EXIT_SUCCESS 退出。如果 JSON 不符合 schema，会输出一条错误消息告知不符合 schema 的细节。工具将以 EXIT_FAILURE 状态退出。

- 构建

  工具可以在本地使用 Bazel 构建。`bazel build //test/tools/schema_validator:schema_validator_tool `

- 运行

  工具采用上面描述的一条路径。`bazel-bin/test/tools/schema_validator/schema_validator_tool  --schema-type SCHEMA_TYPE  --json-path PATH`
