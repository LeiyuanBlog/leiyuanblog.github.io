---
layout:     post
title:      K8s StatefulSet有状态应用详解
subtitle:   深入理解Kubernetes中有状态应用的部署与管理
date:       2026-03-19
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - K8S
    - Kubernetes
    - StatefulSet
    - 有状态应用
    - 运维
---

> 不怕别人比你优秀，就怕比你优秀的人比你更努力!!!既然选择了远方，便只顾风雨兼程
>
> csdn:[点击进入csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人站:[点击进入](https://imlcl.store)
>
> 简书:[点击进入](https://www.jianshu.com/u/016322e40e1f)
>
> 今日头条:[点击关注头条号](https://www.toutiao.com/c/user/6132192948/#mid=1616456407686158)

## K8s StatefulSet有状态应用详解

#### 为什么需要StatefulSet

1. 在前面的文章中，我们已经学习了Deployment来管理无状态应用
2. 但实际生产中，很多应用是有状态的——**它们需要稳定的网络标识、持久化存储和有序的部署/扩缩**
3. 典型的有状态应用包括：数据库（MySQL、PostgreSQL）、分布式存储（Elasticsearch、Etcd）、消息队列（Kafka、ZooKeeper）等
4. Deployment无法满足这些需求，因为Pod重建后名称和IP会变化，存储也无法自动绑定

#### StatefulSet与Deployment的区别

| 特性 | Deployment | StatefulSet |
|------|-----------|-------------|
| Pod名称 | 随机后缀（如web-5d8f6b） | 有序编号（如web-0, web-1） |
| 网络标识 | 不稳定，重建后变化 | 稳定，通过Headless Service固定 |
| 存储 | 共享或临时 | 每个Pod独立PVC，重建后自动绑定 |
| 部署顺序 | 并行创建 | 有序创建（0→1→2） |
| 删除顺序 | 并行删除 | 逆序删除（2→1→0） |
| 扩缩容 | 随机增减 | 有序增减 |

**核心理解**：StatefulSet的每个Pod都有自己的"身份"，这个身份在Pod重建后依然保持不变

#### StatefulSet的三大核心特性

##### 1. 稳定的网络标识

每个Pod获得一个固定的DNS名称，格式为：

```
<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local
```

例如：
- `mysql-0.mysql-headless.default.svc.cluster.local`
- `mysql-1.mysql-headless.default.svc.cluster.local`

即使Pod被删除重建，DNS名称不变，其他服务可以稳定地访问它

##### 2. 稳定的持久化存储

通过`volumeClaimTemplates`，每个Pod自动创建独立的PVC：

- Pod `web-0` → PVC `data-web-0`
- Pod `web-1` → PVC `data-web-1`

Pod重建后会自动重新绑定到原来的PVC，数据不会丢失

##### 3. 有序的部署和扩缩

- **创建时**：按编号从小到大依次创建（0→1→2），前一个Ready后才创建下一个
- **删除时**：按编号从大到小依次删除（2→1→0）
- **更新时**：按逆序滚动更新

#### 前置条件：Headless Service

StatefulSet必须搭配Headless Service使用。Headless Service的特点是`clusterIP: None`，它不会分配虚拟IP，而是直接将DNS解析到Pod IP：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None          # 关键！Headless Service
  selector:
    app: nginx
```

> **说明**：普通Service的DNS解析到ClusterIP，而Headless Service的DNS直接解析到后端Pod的IP列表

#### 基础示例：Nginx StatefulSet

##### 创建StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: nginx-headless      # 必须指定Headless Service
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
          image: nginx:1.24
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:            # 自动为每个Pod创建PVC
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 1Gi
```

##### 关键字段说明

| 字段 | 说明 | 注意事项 |
|------|------|----------|
| serviceName | 关联的Headless Service名称 | 必填，用于生成Pod DNS |
| replicas | 副本数 | Pod编号从0到replicas-1 |
| volumeClaimTemplates | PVC模板 | 每个Pod自动创建独立PVC |
| selector | Pod选择器 | 必须与template.labels匹配 |

##### 观察创建过程

```bash
# 创建资源
kubectl apply -f nginx-statefulset.yaml

# 观察Pod有序创建
kubectl get pods -w -l app=nginx

# 输出示例：
# NAME    READY   STATUS              RESTARTS   AGE
# web-0   0/1     ContainerCreating   0          2s
# web-0   1/1     Running             0          5s
# web-1   0/1     ContainerCreating   0          1s    ← web-0 Ready后才创建
# web-1   1/1     Running             0          4s
# web-2   0/1     ContainerCreating   0          1s    ← web-1 Ready后才创建
# web-2   1/1     Running             0          3s
```

#### 验证网络标识稳定性

##### 查看Pod的DNS记录

```bash
# 启动一个临时Pod来做DNS查询
kubectl run dns-test --image=busybox:1.36 --restart=Never --rm -it -- \
  nslookup nginx-headless

# 查询单个Pod的DNS
kubectl run dns-test --image=busybox:1.36 --restart=Never --rm -it -- \
  nslookup web-0.nginx-headless

# 输出示例：
# Name:    web-0.nginx-headless.default.svc.cluster.local
# Address: 10.244.1.15
```

##### 验证重建后DNS不变

```bash
# 删除web-1
kubectl delete pod web-1

# 观察重建
kubectl get pods -w -l app=nginx

# web-1会被自动重建，名称仍然是web-1
# DNS名称 web-1.nginx-headless 依然有效（虽然IP可能变化）
```

#### 验证存储稳定性

```bash
# 查看自动创建的PVC
kubectl get pvc

# 输出示例：
# NAME         STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS
# data-web-0   Bound    pv-xxx     1Gi        RWO            standard
# data-web-1   Bound    pv-yyy     1Gi        RWO            standard
# data-web-2   Bound    pv-zzz     1Gi        RWO            standard

# 向web-0写入数据
kubectl exec web-0 -- sh -c 'echo "Hello from web-0" > /usr/share/nginx/html/index.html'

# 删除web-0
kubectl delete pod web-0

# 等待重建后验证数据还在
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
# 输出: Hello from web-0    ← 数据持久化成功！
```

> **重要**：删除StatefulSet时，关联的PVC不会自动删除，需要手动清理

#### 实战：部署MySQL主从集群

这是一个常见的生产场景——使用StatefulSet部署MySQL一主两从：

##### ConfigMap（MySQL配置）

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  primary.cnf: |
    [mysqld]
    server-id=1
    log-bin=mysql-bin
    binlog-format=ROW
  replica.cnf: |
    [mysqld]
    server-id=2
    super-read-only=ON
    log-bin=mysql-bin
    binlog-format=ROW
```

##### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  ports:
    - port: 3306
      name: mysql
  clusterIP: None
  selector:
    app: mysql
---
# 普通Service用于读请求（负载均衡到所有Pod）
apiVersion: v1
kind: Service
metadata:
  name: mysql-read
spec:
  ports:
    - port: 3306
      name: mysql
  selector:
    app: mysql
```

##### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
        - name: init-mysql
          image: mysql:8.0
          command:
            - bash
            - -c
            - |
              # 根据Pod编号生成server-id
              ordinal=$(echo $HOSTNAME | grep -o '[0-9]*$')
              echo "[mysqld]" > /mnt/conf.d/server-id.cnf
              echo "server-id=$((100 + ordinal))" >> /mnt/conf.d/server-id.cnf
              # 第一个Pod是主节点
              if [ "$ordinal" = "0" ]; then
                cp /mnt/config-map/primary.cnf /mnt/conf.d/
              else
                cp /mnt/config-map/replica.cnf /mnt/conf.d/
              fi
          volumeMounts:
            - name: conf
              mountPath: /mnt/conf.d
            - name: config-map
              mountPath: /mnt/config-map
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "your-password"       # 生产环境应使用Secret
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
            - name: conf
              mountPath: /etc/mysql/conf.d
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: 500m
              memory: 1Gi
          livenessProbe:
            exec:
              command: ["mysqladmin", "ping", "-uroot", "-p${MYSQL_ROOT_PASSWORD}"]
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command: ["mysql", "-uroot", "-p${MYSQL_ROOT_PASSWORD}", "-e", "SELECT 1"]
            initialDelaySeconds: 10
            periodSeconds: 5
      volumes:
        - name: conf
          emptyDir: {}
        - name: config-map
          configMap:
            name: mysql-config
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 10Gi
```

##### 访问方式

```bash
# 写操作 → 连接主节点
mysql -h mysql-0.mysql-headless -uroot -p

# 读操作 → 通过Service负载均衡
mysql -h mysql-read -uroot -p

# 指定连接某个从节点
mysql -h mysql-1.mysql-headless -uroot -p
```

#### 更新策略

StatefulSet支持两种更新策略：

##### RollingUpdate（默认）

逆序滚动更新，从最后一个Pod开始：

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0          # 编号 >= partition 的Pod才会被更新
```

**partition的妙用——金丝雀发布**：

```yaml
# 只更新web-2（编号>=2的Pod）
spec:
  updateStrategy:
    rollingUpdate:
      partition: 2

# 验证web-2没问题后，改为partition: 1，再更新web-1
# 最后partition: 0，完成全部更新
```

##### OnDelete

手动控制更新——只有Pod被删除后，重建时才使用新的模板：

```yaml
spec:
  updateStrategy:
    type: OnDelete
```

适用于需要精确控制更新节奏的有状态应用

#### 扩缩容

```bash
# 扩容：会按顺序创建web-3、web-4
kubectl scale statefulset web --replicas=5

# 缩容：会逆序删除web-4、web-3
kubectl scale statefulset web --replicas=3
```

> **注意**：缩容时PVC不会自动删除，下次扩容回来时会重新绑定原来的PVC

#### Pod管理策略

默认的`OrderedReady`策略是严格有序的，如果不需要顺序保证，可以使用`Parallel`：

```yaml
spec:
  podManagementPolicy: Parallel    # 并行创建/删除
```

适用于不依赖启动顺序的有状态应用（如Elasticsearch集群中的Data节点）

#### 常用命令

```bash
# 查看StatefulSet
kubectl get statefulsets
kubectl get sts               # 简写

# 查看详细信息
kubectl describe sts web

# 查看关联的PVC
kubectl get pvc -l app=nginx

# 扩缩容
kubectl scale sts web --replicas=5

# 查看有序创建/删除过程
kubectl get pods -w -l app=nginx

# 查看Pod的DNS记录
kubectl run dns-test --image=busybox:1.36 --restart=Never --rm -it -- nslookup nginx-headless

# 导出StatefulSet的YAML
kubectl get sts web -o yaml

# 删除StatefulSet（不删除PVC）
kubectl delete sts web

# 删除StatefulSet及其PVC
kubectl delete sts web
kubectl delete pvc -l app=nginx

# 级联删除：删除StatefulSet但不删除Pod（孤儿Pod）
kubectl delete sts web --cascade=orphan
```

#### 常见问题排查

##### 问题1：Pod一直处于Pending状态

```bash
# 1. 检查PVC是否绑定成功
kubectl get pvc

# 2. 检查是否有可用的PV或StorageClass
kubectl get pv
kubectl get sc

# 3. 查看Pod事件
kubectl describe pod web-0
```

常见原因：
- 没有可用的PV或StorageClass未配置动态供给
- 存储容量不足
- 节点资源不足

##### 问题2：Pod创建卡住，后续Pod不创建

```bash
# 检查卡住的Pod状态
kubectl describe pod web-0

# 常见原因：
# - 镜像拉取失败
# - 健康检查未通过
# - Init容器执行失败
```

由于OrderedReady策略，前一个Pod必须Ready后才会创建下一个

##### 问题3：删除StatefulSet后PVC残留

这是设计行为，不是Bug！防止误删数据：

```bash
# 手动清理PVC
kubectl delete pvc data-web-0 data-web-1 data-web-2

# 或按标签批量清理
kubectl delete pvc -l app=nginx
```

#### 最佳实践

1. **始终配置Headless Service**，这是StatefulSet正常工作的前提
2. **合理设置资源请求**，有状态应用通常对资源更敏感，避免因资源不足导致OOM
3. **配置健康检查**，readinessProbe尤其重要，它决定了有序部署的节奏
4. **使用PodDisruptionBudget**，防止维护时过多Pod同时不可用
5. **谨慎缩容**，缩容前确保数据已迁移或同步完成
6. **善用partition做金丝雀发布**，逐步验证更新，降低风险
7. **生产环境使用反亲和性**，将Pod分散到不同节点，提高可用性
8. **定期备份PVC数据**，PV虽然持久，但不等于安全

##### 反亲和性配置示例

```yaml
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - mysql
                topologyKey: kubernetes.io/hostname
```

##### PodDisruptionBudget配置示例

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: mysql-pdb
spec:
  minAvailable: 2           # 至少保持2个Pod可用
  selector:
    matchLabels:
      app: mysql
```

#### 总结

| 概念 | 说明 | 关键点 |
|------|------|--------|
| StatefulSet | 管理有状态应用的控制器 | 稳定标识 + 持久存储 + 有序部署 |
| Headless Service | 无ClusterIP的Service | 为Pod提供固定DNS |
| volumeClaimTemplates | PVC自动创建模板 | 每个Pod独立存储 |
| 有序部署 | Pod按编号顺序创建/删除 | 前一个Ready才创建下一个 |
| 稳定网络标识 | Pod重建后DNS名不变 | pod-name.service-name格式 |
| partition更新 | 金丝雀发布能力 | 只更新编号>=partition的Pod |
| PodManagementPolicy | 控制创建/删除策略 | OrderedReady或Parallel |

StatefulSet是Kubernetes中管理有状态应用的核心控制器，掌握好StatefulSet能够让你**自信地在K8s上运行数据库、消息队列等关键有状态服务**。核心要记住三个"稳定"：稳定的网络标识、稳定的存储、稳定的部署顺序。

下一篇我们将学习**K8s DaemonSet守护进程**，了解如何在每个节点上运行系统级服务，敬请期待！
