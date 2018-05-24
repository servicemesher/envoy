# 路由表检查工具

路由表检查工具检查路由返回的路由参数是否符合预期。该工具还可以用于检查路径重定向、路径重写或主机重写是否符合预期。

- 输入

    该工具期望两个输入 JSON 文件：一个路由配置 JSON 文件。在一个工具配置 JSON  文件中找到了路由配置 JSON 文件架构。配置 JSON 文件模板的工具在 [config](https://github.com/servicemesher/envoy/blob/master/configuration/tools/router_check.md#config-tools-router-check-tool) 中。工具配置输入文件指定 URL （由权限和路径组成）以及期望的路由参数值。其他参数（如附加标头）是可选的。
    
- 输出
  
    如果任何测试用例与预期的路由参数值不匹配，那么程序将以状态 EXIT_FAILURE 退出 。`--detail` 选项打印出每个测试的详细信息。第一行表示测试名称。如果测试失败，则打印失败的测试用例的详细信息。第一个字段是预期路由参数值。第二个字段是实际路由参数值。第三个字段表示比较的参数。在下面的例子中，`test_2` 和 `test_5` 失败了而其他测试通过了。在失败的测试用例中，会打印冲突细节。 `Test_1 Test_2 default other virtual_host_name Test_3 Test_4 Test_5 locations ats cluster_name Test_6`  目前不支持使用有效[运行时值](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route)进行测试，这可能会在以后的工作中添加。

- 构建

    工具可以在本地使用 Bazel 构建。`bazel build //test/tools/router_check:router_check_tool` 

- 运行

    该工具接受两个输入 json 文件和一个可选的命令行参数 `--details` 。 命令行参数的预期顺序是：1. 路由配置 json  文件;2. 工具配置 json 文件;3. 可选项 。 `bazel-bin/test/tools/router_check/router_check_tool router_config.json tool_config.json bazel-bin/test/tools/router_check/router_check_tool router_config.json tool_config.json --details`

- 测试

    bash shell 脚本测试可以使用 bazel 运行。该测试比较了使用不同路由和工具配置 json 文件的路由。配置文件可以在 test/tools/router_check/test/config/…  找到。`bazel test //test/tools/router_check/...`。
