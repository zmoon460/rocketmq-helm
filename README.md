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
  --set broker.master.jvmMemory="-Xms2g -Xmx2g -Xmn1g" \
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

## Broker 集群架构

### 单 Master 模式

``` yaml
broker:
  size:
    master: 1
    replica: 0
```

### 多 Master 模式

一个集群无Slave，全是Master，例如2个Master或者3个Master，这种模式的优缺点如下：

- 优点：配置简单，单个Master宕机或重启维护对应用无影响，性能最高；
- 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响。

``` yaml
broker:
  size:
    master: 3
    replica: 0
```

### 多 Master 多 Slave 模式

每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下：

- 优点：Master宕机后，消费者仍然可以从Slave消费，而且此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样；
- 缺点：Master宕机，磁盘损坏情况下会丢失少量消息 (已经同步到 Slave 的数据不受影响)

``` yaml
broker:
  size:
    master: 3
    replica: 1
# 3个 master 节点，每个 master 具有1个副节点，共6个 broker 节点
```
