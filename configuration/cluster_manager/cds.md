# 集群发现服务（CDS）

The cluster discovery service (CDS) is an optional API that Envoy will call to dynamically fetch cluster manager members. Envoy will reconcile the API response and add, modify, or remove known clusters depending on what is required.

Note

Any clusters that are statically defined within the Envoy configuration cannot be modified or removed via the CDS API.

- [v1 CDS API](../../api-v1/cluster_manager/cds.md#config-cluster-manager-cds-v1)
- [v2 CDS API](../overview/v2_overview.md#v2-grpc-streaming-endpoints)

## Statistics

CDS has a statistics tree rooted at *cluster_manager.cds.* with the following statistics:

| Name           | Type    | Description                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| config_reload  | Counter | Total API fetches that resulted in a config reload due to a different config |
| update_attempt | Counter | Total API fetches attempted                                  |
| update_success | Counter | Total API fetches completed successfully                     |
| update_failure | Counter | Total API fetches that failed because of schema errors       |
| version        | Gauge   | Hash of the contents from the last successful API fetch      |