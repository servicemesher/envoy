# Schema 验证器检查工具

The schema validator tool validates that the passed in JSON conforms to a schema in the configuration. To validate the entire config, please refer to the [config load check tool](config_load_check_tool.md#install-tools-config-load-check-tool). Currently, only [route config](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table) schema validation is supported.

- Input

  The tool expects two inputs:The schema type to check the passed in JSON against. The supported type is:route - for [route configuration](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table) validation.The path to the JSON.

- Output

  If the JSON conforms to the schema, the tool will exit with status EXIT_SUCCESS. If the JSON does not conform to the schema, an error message is outputted detailing what doesn’t conform to the schema. The tool will exit with status EXIT_FAILURE.

- Building

  The tool can be built locally using Bazel.`bazel build //test/tools/schema_validator:schema_validator_tool `

- Running

  The tool takes a path as described above.`bazel-bin/test/tools/schema_validator/schema_validator_tool  --schema-type SCHEMA_TYPE  --json-path PATH`