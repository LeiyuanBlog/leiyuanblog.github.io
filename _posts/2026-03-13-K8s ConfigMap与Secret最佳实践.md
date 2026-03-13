---
layout:     post
title:      K8s ConfigMap与Secret最佳实践
subtitle:   配置管理的正确姿势，让你的应用更安全、更灵活
date:       2026-03-13
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - K8S
    - Kubernetes
    - ConfigMap
    - Secret
---

> 多平台同步更新中,感谢大家的支持!!!以下为本篇文章各平台链接
>
> csdn:[我的csdn账号](https://blog.csdn.net/leiyuan2580)
>
> 个人站点:[我的个人站点](https://imlcl.store)
>
> 简书:[我的简书账号](https://www.jianshu.com/u/016322e40e1f)
>
> 头条号:[我的头条账号首页](https://www.toutiao.com/c/user/6132192948/#mid=1616456407686158)

## K8s ConfigMap与Secret最佳实践

#### 为什么写这篇

1. 在前几篇文章中，我们聊了Pod生命周期和Service网络模型。今天来讲讲同样重要但容易被忽视的话题——**配置管理**
2. ConfigMap和Secret是K8s中管理配置的核心手段，用好了事半功倍，用不好就是线上事故的温床

#### ConfigMap基础

ConfigMap用于存储非敏感的配置数据，以键值对的形式存在。

##### 创建ConfigMap的几种方式

**方式一：从字面量创建**

```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql.default.svc \
  --from-literal=DATABASE_PORT=3306 \
  --from-literal=LOG_LEVEL=info
```

**方式二：从文件创建**

```bash
# 假设有一个 nginx.conf 文件
kubectl create configmap nginx-config --from-file=nginx.conf
```

**方式三：YAML声明式（推荐）**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  DATABASE_HOST: "mysql.default.svc"
  DATABASE_PORT: "3306"
  LOG_LEVEL: "info"
  app.properties: |
    server.port=8080
    spring.datasource.url=jdbc:mysql://mysql:3306/mydb
    spring.datasource.pool-size=10
```

#### Secret基础

Secret用于存储敏感信息，如密码、Token、证书等。K8s会对Secret的值进行Base64编码存储。

##### Secret的类型

| 类型 | 说明 | 使用场景 |
| --- | --- | --- |
| Opaque | 默认类型，存储任意数据 | 密码、API Key等 |
| kubernetes.io/tls | TLS证书 | HTTPS证书 |
| kubernetes.io/dockerconfigjson | Docker仓库认证 | 私有镜像拉取 |
| kubernetes.io/basic-auth | 基本认证 | 用户名密码 |
| kubernetes.io/ssh-auth | SSH认证 | SSH密钥 |

##### 创建Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:          # stringData 不需要手动base64编码
  DB_PASSWORD: "my-super-secret-password"
  DB_USERNAME: "admin"
---
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64编码的证书>
  tls.key: <base64编码的私钥>
```

> ⚠️ **注意：** `data`字段需要Base64编码，`stringData`字段不需要。建议使用`stringData`减少出错概率。

#### 在Pod中使用配置

##### 方式一：环境变量注入

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: my-app:latest
      env:
        # 逐个引用
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_HOST
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
      envFrom:
        # 批量引用整个ConfigMap
        - configMapRef:
            name: app-config
          prefix: APP_    # 可选前缀，避免冲突
```

##### 方式二：Volume挂载（推荐）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: my-app:latest
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
          readOnly: true
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: config-volume
      configMap:
        name: app-config
        items:              # 可选：只挂载特定key
          - key: app.properties
            path: application.properties
    - name: secret-volume
      secret:
        secretName: db-secret
        defaultMode: 0400   # 限制文件权限
```

#### 最佳实践

##### 1. 配置与镜像分离

**❌ 反面教材：** 把配置写死在Dockerfile或代码里

```dockerfile
# 千万别这么干
ENV DATABASE_HOST=production-mysql.company.com
ENV DB_PASSWORD=123456
```

**✅ 正确做法：** 通过ConfigMap/Secret在部署时注入

```yaml
# 不同环境使用不同的ConfigMap
# dev-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev
data:
  DATABASE_HOST: "mysql.dev.svc"
  LOG_LEVEL: "debug"
---
# prod-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: prod
data:
  DATABASE_HOST: "mysql.prod.svc"
  LOG_LEVEL: "warn"
```

##### 2. 优先使用Volume挂载而非环境变量

原因：
- **热更新支持：** Volume挂载的ConfigMap更新后，文件内容会自动同步（通常在1-2分钟内），环境变量则需要重启Pod
- **大配置文件支持：** 环境变量有长度限制，Volume没有
- **文件权限控制：** Volume可以设置`defaultMode`限制读写权限

```yaml
# 利用subPath挂载单个文件，不覆盖整个目录
volumeMounts:
  - name: config-volume
    mountPath: /app/config/application.yaml
    subPath: application.yaml
```

> ⚠️ **注意：** 使用`subPath`挂载时，ConfigMap更新不会自动同步到Pod，需要重启。

##### 3. 给Secret加密——不要裸奔

Base64 ≠ 加密！任何人拿到Base64字符串都能解码。生产环境务必做到：

**方案一：启用etcd静态加密**

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64编码的32字节密钥>
      - identity: {}
```

**方案二：使用外部密钥管理（推荐）**

集成HashiCorp Vault、AWS Secrets Manager、Azure Key Vault等：

```yaml
# 使用External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: db-secret
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: secret/data/production/db
        property: password
```

**方案三：Sealed Secrets**

适合GitOps场景，可以安全地把加密后的Secret提交到Git：

```bash
# 加密
kubeseal --format yaml < secret.yaml > sealed-secret.yaml
# sealed-secret.yaml 可以安全提交到Git
```

##### 4. 使用immutable ConfigMap/Secret

对于不需要修改的配置，设置`immutable: true`：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
immutable: true
data:
  DATABASE_HOST: "mysql.prod.svc"
```

好处：
- **性能提升：** kubelet不需要持续watch变更
- **安全防护：** 防止意外修改导致事故
- **版本管理：** 通过名称后缀（如`-v2`）实现版本化

##### 5. RBAC权限控制

限制谁能读取Secret：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["db-secret"]   # 限制到具体Secret
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-db-secret
  namespace: production
subjects:
  - kind: ServiceAccount
    name: my-app-sa
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

##### 6. 配置变更触发滚动更新

ConfigMap/Secret更新后，Deployment不会自动滚动更新。一个常用技巧是在annotation中加入配置的hash：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        # 配置变更时更新这个hash，触发滚动更新
        checksum/config: {{ include (print .Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      containers:
        - name: app
          image: my-app:latest
```

或者使用`Reloader`这样的开源工具自动完成：

```bash
# 安装Reloader
helm install reloader stakater/reloader

# 给Deployment添加annotation
kubectl annotate deployment my-app reloader.stakater.com/auto="true"
```

##### 7. 合理拆分ConfigMap

**❌ 一个巨型ConfigMap塞所有配置**

**✅ 按职责拆分**

```yaml
# 应用配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  SERVER_PORT: "8080"
---
# 中间件配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: middleware-config
data:
  REDIS_HOST: "redis.default.svc"
  KAFKA_BROKERS: "kafka-0:9092,kafka-1:9092"
---
# 完整配置文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    worker_processes auto;
    events { worker_connections 1024; }
    ...
```

#### 常见踩坑总结

| 坑 | 原因 | 解决方案 |
| --- | --- | --- |
| 改了ConfigMap但Pod没生效 | 环境变量不会热更新 | 用Volume挂载，或重启Pod |
| subPath挂载不更新 | K8s的已知限制 | 避免subPath，或用Reloader |
| Secret被提交到Git | Base64不是加密 | 用Sealed Secrets或.gitignore |
| ConfigMap超过1MB | etcd单个对象限制 | 拆分ConfigMap，或用外部存储 |
| 多环境配置混乱 | 缺少配置管理策略 | Kustomize/Helm + 命名空间隔离 |

#### 完整实战示例

一个Spring Boot应用的完整配置管理方案：

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

configMapGenerator:
  - name: spring-config
    files:
      - application.yaml
    options:
      labels:
        app: my-spring-app

secretGenerator:
  - name: spring-secrets
    literals:
      - DB_PASSWORD=change-me
    options:
      labels:
        app: my-spring-app

resources:
  - deployment.yaml
  - service.yaml
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-spring-app
  template:
    metadata:
      labels:
        app: my-spring-app
    spec:
      serviceAccountName: spring-app-sa
      containers:
        - name: app
          image: my-spring-app:1.0.0
          ports:
            - containerPort: 8080
          envFrom:
            - secretRef:
                name: spring-secrets
          volumeMounts:
            - name: config
              mountPath: /app/config
              readOnly: true
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
      volumes:
        - name: config
          configMap:
            name: spring-config
```

#### 总结

| 原则 | 说明 |
| --- | --- |
| 配置与代码分离 | 同一镜像，不同环境通过ConfigMap切换 |
| Secret必须加密 | Base64不是加密，用Vault/Sealed Secrets |
| 优先Volume挂载 | 支持热更新，更灵活 |
| immutable优先 | 性能好，防误改 |
| RBAC最小权限 | 只给需要的人/SA读取权限 |
| 版本化管理 | Kustomize/Helm管理配置变更 |

配置管理看起来简单，但做好了是架构能力的体现。希望这篇文章能帮你在生产环境中少踩几个坑！

---

**K8s系列文章：**
- [K8s Pod生命周期详解：从创建到终止的完整流程](/)
- [K8s Service网络模型深度解析](/)
- K8s ConfigMap与Secret最佳实践（本文）
