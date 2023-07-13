# RocketMQ Helm Chart

https://github.com/itboon/rocketmq-helm

## 版本兼容性

- Kubernetes 1.18+
- Helm 3.3+
- RocketMQ `>= 4.5` (`5.x` 未测试)

## 添加 helm 仓库

``` shell
## 添加 helm 仓库
helm repo add rocketmq-repo https://helm-charts.itboon.top/rocketmq
helm repo update rocketmq-repo
```

## 部署案例

``` shell
## 部署一个最小化的 rocketmq 集群
## 这里关闭持久化存储，仅演示部署效果
helm upgrade --install rocketmq \
  --namespace rocketmq-demo \
  --create-namespace \
  --set broker.persistence.enabled="false" \
  rocketmq-repo/rocketmq
```

``` shell
## 部署测试集群, 启用 Dashboard (默认已开启持久化存储)
helm upgrade --install rocketmq \
  --namespace rocketmq-demo \
  --create-namespace \
  --set dashboard.enabled="true" \
  --set dashboard.ingress.enabled="true" \
  --set dashboard.ingress.hosts[0].host="rocketmq-demo.example.com" \
  rocketmq-repo/rocketmq
```

``` shell
## 部署高可用集群, 多 Master 多 Slave
## 3个 master 节点，每个 master 具有1个副节点，共6个 broker 节点
helm upgrade --install rocketmq \
  --namespace rocketmq-demo \
  --create-namespace \
  --set broker.size.master="3" \
  --set broker.size.replica="1" \
  --set broker.master.jvmMemory="-Xms2g -Xmx2g" \
  --set broker.master.resources.requests.memory="4Gi" \
  --set nameserver.replicaCount="3" \
  --set dashboard.enabled="true" \
  --set dashboard.ingress.enabled="true" \
  --set dashboard.ingress.hosts[0].host="rocketmq-ha.example.com" \
  rocketmq-repo/rocketmq

```

> 具体资源配额请根据实际环境调整，参考 [examples](https://github.com/itboon/rocketmq-helm/tree/main/examples)

## 镜像仓库

``` yaml
image:
  repository: apache/rocketmq
  tag: 4.9.7
```

### 部署特定版本

``` shell
helm upgrade --install rocketmq \
  --namespace rocketmq-demo \
  --create-namespace \
  --set image.tag="4.9.5" \
  rocketmq-repo/rocketmq
```
