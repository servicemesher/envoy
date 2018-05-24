# 配置负载检查工具

配置负载检查工具检查 JSON 格式的配置文件是否使用有效的 JSON 编写，并符合 Envoy JSON 模式。
该工具利用 `test/config_test/config_test.cc` 中的配置测试。该测试加载 JSON 配置文件并使用它运行服务器配置初始化。
- 输入

  该工具需要一个指向保存 JSON Envoy 配置文件的根目录的 PATH 变量。该工具将以递归方式遍历文件系统树，并对每个找到的文件运行配置测试。请记住，该工具将尝试加载路径中找到的所有文件。

- 输出

  该工具使用它当前正在测试的配置初始化服务器配置时，将输出 Enovy 日志。 如果存在 JSON 文件格式错误或不符合 Envoy JSON 模式的配置文件，则该工具将以状态 EXIT_FAILURE 退出。 如果该工具成功加载找到的所有配置文件，它将以状态 EXIT_SUCCESS 退出。

- 构建

  我们可以使用 Bazel 在本地构建该工具。 `bazel build //test/tools/config_load_check:config_load_check_tool `

- 运行

  该工具将需要如上所述的 PATH 变量。 `bazel bin/test/tools/config_load_check/config_load_check_tool PATH`