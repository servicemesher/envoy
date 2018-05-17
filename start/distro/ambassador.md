# Envoy 作为 Kubernetes 的 API 网关

使用 Ambassador 的一个常见场景是将其部署为 Kubernetes 的 edge 服务（ API 网关）。[Ambassador](https://www.getambassador.io/) 是开源 Envoy 的分布式版本，专门为 kubernetes 设计的。

本例将介绍如何通过 Ambassador 在 Kubernetes 上部署 Ambassador 。

## 部署 Ambassador

Ambassador 的设置是通过 kubernetes 部署的。为了在 kubernetes 安装 Ambassador/Envoy，如果你的集群启动了 RBAC：

```bash
kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-rbac.yaml
```

如果您没启动 RBAC：

```bash
kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-no-rbac.yaml
```

上面的 YAML 将会为 Ambassador 创建 kubernetes 部署，包含 readiness 和 liveness 检查。默认，将会创建3个 Ambassador 实例。每一个 Ambassador 实例包含一个 Envoy 代理以及一个 Ambassador 控制器。

我们现在需要创建一个 Kubernetes 服务来指向 Ambassador 的部署，我们将使用 ` LoadBalancer` 服务。如果你的集群不支持 ` LoadBalancer` 服务，你需要改成 `NodePort` 或者 `ClusterIP`。

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    service: ambassador
  name: ambassador
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    service: ambassador
```

将上面的 YAML 文件保存成`ambassador-svc.yaml`文件。然后将这个服务部署到 kubernetes：

```bash
kubectl apply -f ambassador-svc.yaml
```

这时候 Envoy 和 Ambassador 控制器已经在你的集群上运行。

## 配置 Ambassador

Ambassador 使用 Kubernetes 注解来添加或删除配置。这个示例 YAML 将添加一条到 Google 的路由，类似于[入门指南](https://github.com/xieydd/envoy/blob/master/start/start.md#start)中的基本配置示例。

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: google
  annotations:
    getambassador.io/config: |
      ---
      apiVersion: ambassador/v0
      kind:  Mapping
      name:  google_mapping
      prefix: /google/
      service: https://google.com:443
      host_rewrite: www.google.com
spec:
  type: ClusterIP
  clusterIP: None
```

保存上面的文件，命名为 `google.yaml`。然后运行:

```bash
kubectl apply -f google.yaml
```

Ambassador 将发现您的 Kubernetes 注解的更改，并添加到 Envoy 的路由。注意，我们在这个例子中使用了一个虚拟服务;通常，您会将注解与真正的 Kubernetes 服务关联起来。

## 测试映射

您可以通过获得 Ambassador 服务的外部 IP 地址来测试这个映射，然后通过`curl`发送请求：

```bash
$ kubectl get svc ambassador
NAME         CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
ambassador   10.19.241.98   35.225.154.81   80:32491/TCP   15m
$ curl -v 35.225.154.81/google/
```

## 更多

Ambassador 在上公开了多个 Envoy  的特性映射，比如 CORS 、加权循环调度算法、gRPC、TLS 和超时设定。要了解更多信息，请阅读[配置文档](https://www.getambassador.io/reference/configuration)。