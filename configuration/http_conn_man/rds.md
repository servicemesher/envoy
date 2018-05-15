# 路由发现服务（RDS）

The route discovery service (RDS) API is an optional API that Envoy will call to dynamically fetch[route configurations](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table). A route configuration includes both HTTP header modifications, virtual hosts, and the individual route entries contained within each virtual host. Each [HTTP connection manager filter](http_conn_man.md#config-http-conn-man) can independently fetch its own route configuration via the API.

- [v1 API reference](../../api-v1/route_config/rds.md#config-http-conn-man-rds-v1)
- [v2 API reference](../overview/v2_overview.md#v2-grpc-streaming-endpoints)

## Statistics

RDS has a statistics tree rooted at *http.<stat_prefix>.rds.<route_config_name>.*. Any `:` character in the `route_config_name` name gets replaced with `_` in the stats tree. The stats tree contains the following statistics:

| Name           | Type    | Description                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| config_reload  | Counter | Total API fetches that resulted in a config reload due to a different config |
| update_attempt | Counter | Total API fetches attempted                                  |
| update_success | Counter | Total API fetches completed successfully                     |
| update_failure | Counter | Total API fetches that failed (either network or schema errors) |
| version        | Gauge   | Hash of the contents from the last successful API fetch      |