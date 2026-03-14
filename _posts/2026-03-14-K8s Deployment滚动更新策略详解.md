---
layout:     post
title:      K8s Deployment滚动更新策略详解
subtitle:   深入理解Kubernetes Deployment的滚动更新机制与最佳实践
date:       2026-03-14
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - K8S
    - Kubernetes
    - Deployment
    - 滚动更新
---

> 技术交流请联系,共同进步!!!可通过以下方式找到我
>
> csdn:[点击进入csdn主页](https://blog.csdn.net/leiyuan2580)
>
> 个人网站:[点击进入网站](https://imlcl.store)
>
> 简书:[点击进入简书](https://www.jianshu.com/u/016322e40e1f)
>
> 头条号:[点击进入头条主页](https://www.toutiao.com/c/user/6132192948/#mid=1616456407686158)

## K8s Deployment滚动更新策略详解

#### 前言与回顾

1. 在前几篇文章中，我们学习了Pod调度、Service服务发现以及ConfigMap和Secret配置管理——这些都是运行应用的基础设施
2. 但在实际生产环境中，**应用的更新与发布**是最频繁的操作之一。如何做到零停机、平滑更新，是每个运维工程师必须掌握的技能
3. 本篇将深入讲解Deployment的滚动更新策略，让你的应用更新既安全又高效

#### Deployment基础回顾

Deployment是Kubernetes中最常用的工作负载控制器，它通过管理ReplicaSet来实现Pod的声明式更新。

##### 一个基本的Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

使用`kubectl apply -f`部署后，Deployment会创建一个ReplicaSet，ReplicaSet再创建3个Pod。

#### 两种更新策略

Deployment支持两种更新策略，通过`spec.strategy.type`指定：

##### 1. RollingUpdate（滚动更新）——默认策略

滚动更新会逐步用新版本Pod替换旧版本Pod，整个过程中始终有Pod在提供服务，实现**零停机更新**。

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

##### 2. Recreate（重建）

先删除所有旧Pod，再创建新Pod。更新期间服务会中断，一般仅用于**开发测试环境**或**不支持多版本共存**的场景。

```yaml
spec:
  strategy:
    type: Recreate
```

#### 滚动更新核心参数详解

滚动更新有两个关键参数，理解它们是掌握更新策略的核心：

##### maxSurge（最大超出数）

- **含义**：更新过程中，允许超过期望副本数的最大Pod数量
- **可选值**：整数或百分比（默认25%）
- **示例**：replicas=4, maxSurge=1 → 更新过程中最多存在5个Pod

##### maxUnavailable（最大不可用数）

- **含义**：更新过程中，允许不可用的最大Pod数量
- **可选值**：整数或百分比（默认25%）
- **示例**：replicas=4, maxUnavailable=1 → 更新过程中至少有3个Pod可用

##### 常见组合策略

| 场景 | maxSurge | maxUnavailable | 特点 |
|------|----------|----------------|------|
| 最安全 | 1 | 0 | 始终保持全量可用，速度最慢 |
| 均衡 | 25% | 25% | 默认值，速度与安全兼顾 |
| 最快速 | 50% | 50% | 更新速度快，但需要更多资源 |
| 零停机 | 1 | 0 | 适合生产环境关键服务 |

#### 实战演练：执行滚动更新

##### 第一步：部署初始版本

```bash
# 创建deployment
kubectl apply -f nginx-deployment.yaml

# 查看部署状态
kubectl rollout status deployment/nginx-deployment

# 查看Pod
kubectl get pods -l app=nginx
```

##### 第二步：触发滚动更新

**方式一：修改镜像版本**

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.21
```

**方式二：编辑YAML后apply**

```bash
# 修改image字段后
kubectl apply -f nginx-deployment.yaml
```

**方式三：直接编辑**

```bash
kubectl edit deployment/nginx-deployment
```

##### 第三步：观察更新过程

```bash
# 实时查看更新状态
kubectl rollout status deployment/nginx-deployment

# 查看更新事件
kubectl describe deployment/nginx-deployment

# 观察Pod替换过程
kubectl get pods -l app=nginx -w
```

你会看到类似如下输出：

```
nginx-deployment-6b474476c4-abc12   1/1     Running   0   5m
nginx-deployment-6b474476c4-def34   1/1     Running   0   5m
nginx-deployment-6b474476c4-ghi56   1/1     Running   0   5m
nginx-deployment-7c585476d5-jkl78   0/1     Pending   0   1s
nginx-deployment-7c585476d5-jkl78   1/1     Running   0   3s
nginx-deployment-6b474476c4-abc12   1/1     Terminating   0   5m
...
```

#### 版本回滚

更新出问题？不用慌，Deployment天然支持版本回滚。

##### 查看历史版本

```bash
# 查看所有版本
kubectl rollout history deployment/nginx-deployment

# 查看某个版本的详情
kubectl rollout history deployment/nginx-deployment --revision=2
```

##### 执行回滚

```bash
# 回滚到上一个版本
kubectl rollout undo deployment/nginx-deployment

# 回滚到指定版本
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

##### 保留历史版本数

通过`revisionHistoryLimit`控制保留的ReplicaSet数量（默认10）：

```yaml
spec:
  revisionHistoryLimit: 5
```

#### 暂停与恢复更新

有时需要在更新过程中做多次修改，可以先暂停更新，修改完再恢复：

```bash
# 暂停更新
kubectl rollout pause deployment/nginx-deployment

# 进行多次修改
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=200m,memory=256Mi

# 恢复更新（所有修改一次性生效）
kubectl rollout resume deployment/nginx-deployment
```

#### 配合健康检查实现安全更新

滚动更新的安全性很大程度上依赖于**健康检查**的正确配置：

```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

- **readinessProbe**：新Pod必须通过就绪检查才会接收流量，确保新版本真正可用
- **livenessProbe**：检测Pod是否存活，异常时自动重启

##### minReadySeconds

设置新Pod在Ready之后还需等待多久才被视为Available：

```yaml
spec:
  minReadySeconds: 30
```

这给了新版本Pod一个"观察期"，如果30秒内出问题，更新会暂停。

#### 最佳实践总结

1. **始终配置readinessProbe**：这是滚动更新安全的基石，没有它，有问题的Pod也会被认为是Ready的
2. **生产环境推荐maxSurge=1, maxUnavailable=0**：保证更新过程中服务可用数量不减少
3. **设置合理的minReadySeconds**：给新版本一个观察窗口期
4. **保留足够的revisionHistoryLimit**：方便快速回滚
5. **使用pause/resume进行批量修改**：避免触发多次不必要的滚动更新
6. **更新前先在staging环境验证**：滚动更新不是万能的，代码质量才是根本
7. **配合PodDisruptionBudget（PDB）**：进一步保证高可用

#### 下期预告

下一篇我们将学习K8s的**HPA（水平Pod自动伸缩）**，实现根据CPU/内存等指标自动扩缩容，让集群资源利用更加智能高效。

---

**如果觉得文章对你有帮助，欢迎点赞收藏！有问题也欢迎在评论区讨论。**
