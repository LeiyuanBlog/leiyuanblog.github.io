---
layout:     post
title:      K8s Job与CronJob任务调度详解
subtitle:   深入理解Kubernetes中一次性任务与定时任务的使用
date:       2026-04-15
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - K8S
    - Kubernetes
    - Job
    - CronJob
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

## K8s Job与CronJob任务调度详解

#### 为什么需要Job和CronJob

1. 在之前的文章中，我们学习的Deployment、StatefulSet等控制器都是用来管理**长期运行的服务**
2. 但实际工作中，有很多任务是**一次性**或**定时执行**的——数据备份、日志清理、报表生成、数据库迁移等
3. 如果用Deployment来跑这类任务，Pod完成后会被不断重启，完全不符合预期
4. Kubernetes提供了**Job**（一次性任务）和**CronJob**（定时任务）来解决这个问题

#### Job与CronJob的定位

| 特性 | Job | CronJob |
|------|-----|---------|
| 执行方式 | 一次性执行 | 按Cron表达式定时执行 |
| 完成后行为 | Pod状态变为Completed，不会重启 | 每次调度创建一个新的Job |
| 典型场景 | 数据迁移、批量计算 | 定时备份、日志清理 |
| 与Deployment区别 | 任务完成即结束 | 周期性创建Job |

**核心理解**：Job保证任务**至少成功运行一次**，CronJob则在此基础上加了定时调度能力

---

### 一、Job详解

#### 1. 最简单的Job示例

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox:1.36
        command: ["echo", "Hello from Kubernetes Job!"]
      restartPolicy: Never
```

**关键点**：
- `restartPolicy`必须设置为`Never`或`OnFailure`，不能用`Always`
- Job完成后Pod不会被删除，方便查看日志

```bash
# 创建Job
kubectl apply -f hello-job.yaml

# 查看Job状态
kubectl get jobs
# NAME        COMPLETIONS   DURATION   AGE
# hello-job   1/1           5s         30s

# 查看Pod
kubectl get pods
# NAME              READY   STATUS      RESTARTS   AGE
# hello-job-abc12   0/1     Completed   0          35s

# 查看日志
kubectl logs hello-job-abc12
# Hello from Kubernetes Job!
```

#### 2. Job的重要参数

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 5        # 需要成功完成的Pod总数
  parallelism: 2        # 最多同时运行的Pod数
  backoffLimit: 3       # 失败重试次数上限
  activeDeadlineSeconds: 300  # Job最长运行时间（秒）
  ttlSecondsAfterFinished: 600  # 完成后自动清理时间（秒）
  template:
    spec:
      containers:
      - name: worker
        image: busybox:1.36
        command: ["sh", "-c", "echo Processing item && sleep 10"]
      restartPolicy: Never
```

**参数说明**：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `completions` | 需要成功完成的Pod数量 | 1 |
| `parallelism` | 并行运行的Pod数量 | 1 |
| `backoffLimit` | 失败重试次数 | 6 |
| `activeDeadlineSeconds` | 超时时间，超过后Job会被终止 | 无限制 |
| `ttlSecondsAfterFinished` | 完成后多久自动删除Job和Pod | 不自动删除 |

#### 3. Job的三种运行模式

##### 模式一：单次任务（默认）

```yaml
spec:
  completions: 1
  parallelism: 1
```

最常见的场景：执行一次数据库迁移、运行一个脚本。

##### 模式二：固定完成数的并行任务

```yaml
spec:
  completions: 10
  parallelism: 3
```

适合批量处理场景：需要处理10个任务，最多同时跑3个。

##### 模式三：工作队列模式

```yaml
spec:
  parallelism: 3
  # 不设置completions
```

配合消息队列使用，Pod自己判断何时退出，队列为空时主动退出。

#### 4. Job失败处理机制

```yaml
spec:
  backoffLimit: 4
  template:
    spec:
      containers:
      - name: worker
        image: busybox:1.36
        command: ["sh", "-c", "exit 1"]  # 模拟失败
      restartPolicy: Never  # Never: 创建新Pod重试
                             # OnFailure: 在原Pod内重启容器
```

**`restartPolicy`的选择**：
- `Never`：失败后创建一个全新的Pod来重试，旧Pod保留（方便排查）
- `OnFailure`：在同一个Pod内重启容器，不会产生大量失败Pod

**生产建议**：大多数情况下用`OnFailure`，避免失败Pod堆积

#### 5. 实战：数据库备份Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mysql-backup
  namespace: production
spec:
  backoffLimit: 2
  activeDeadlineSeconds: 3600
  ttlSecondsAfterFinished: 86400
  template:
    spec:
      containers:
      - name: backup
        image: mysql:8.0
        command:
        - /bin/sh
        - -c
        - |
          TIMESTAMP=$(date +%Y%m%d_%H%M%S)
          mysqldump -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD \
            --all-databases --single-transaction \
            > /backup/full_backup_${TIMESTAMP}.sql
          echo "Backup completed: full_backup_${TIMESTAMP}.sql"
        env:
        - name: MYSQL_HOST
          value: "mysql-service"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: backup-volume
          mountPath: /backup
      volumes:
      - name: backup-volume
        persistentVolumeClaim:
          claimName: backup-pvc
      restartPolicy: OnFailure
```

---

### 二、CronJob详解

#### 1. CronJob基础示例

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: log-cleanup
spec:
  schedule: "0 2 * * *"          # 每天凌晨2点执行
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: busybox:1.36
            command:
            - /bin/sh
            - -c
            - |
              echo "Cleaning up logs older than 7 days..."
              find /var/log/app -name "*.log" -mtime +7 -delete
              echo "Cleanup done at $(date)"
          restartPolicy: OnFailure
```

#### 2. Cron表达式速查

```
┌───────────── 分钟 (0-59)
│ ┌───────────── 小时 (0-23)
│ │ ┌───────────── 日 (1-31)
│ │ │ ┌───────────── 月 (1-12)
│ │ │ │ ┌───────────── 星期几 (0-6，0=周日)
│ │ │ │ │
* * * * *
```

**常用示例**：

| 表达式 | 含义 |
|--------|------|
| `*/5 * * * *` | 每5分钟 |
| `0 * * * *` | 每小时整点 |
| `0 2 * * *` | 每天凌晨2点 |
| `0 0 * * 0` | 每周日午夜 |
| `0 9 * * 1-5` | 工作日早上9点 |
| `0 0 1 * *` | 每月1号午夜 |
| `30 8,12,18 * * *` | 每天8:30、12:30、18:30 |

#### 3. CronJob的重要参数

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scheduled-task
spec:
  schedule: "*/30 * * * *"
  concurrencyPolicy: Forbid          # 并发策略
  successfulJobsHistoryLimit: 3      # 保留成功Job数
  failedJobsHistoryLimit: 3          # 保留失败Job数
  startingDeadlineSeconds: 200       # 错过调度的容忍时间
  suspend: false                     # 是否暂停
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 1800
      template:
        spec:
          containers:
          - name: task
            image: busybox:1.36
            command: ["echo", "Running scheduled task"]
          restartPolicy: OnFailure
```

**参数详解**：

| 参数 | 说明 | 建议 |
|------|------|------|
| `concurrencyPolicy` | `Allow`（默认）/`Forbid`/`Replace` | 长时间任务用`Forbid` |
| `successfulJobsHistoryLimit` | 保留多少个成功的Job | 3 |
| `failedJobsHistoryLimit` | 保留多少个失败的Job | 3（方便排查） |
| `startingDeadlineSeconds` | 如果错过了调度时间，在多少秒内还可以补执行 | 根据任务重要性设置 |
| `suspend` | 设为true可以暂停调度，不删除CronJob | 维护时使用 |

#### 4. concurrencyPolicy三种策略

- **Allow**（默认）：允许并发执行，可能同时存在多个Job
- **Forbid**：如果上一个Job还在运行，跳过本次调度
- **Replace**：如果上一个Job还在运行，终止它并启动新的

```
场景：每5分钟执行一次的任务，但某次执行了8分钟

Allow:   [Job1====][Job2====]  ← 两个同时跑
                [Job2====]
Forbid:  [Job1========]       ← 跳过第二次
                    [Job3==]
Replace: [Job1=====X]         ← 终止第一个
              [Job2====]
```

**生产建议**：
- 数据备份类任务用`Forbid`，避免重复备份占用资源
- 监控告警类任务用`Replace`，确保用最新数据
- 独立无状态任务可以用`Allow`

#### 5. 实战：定时数据库备份CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-daily-backup
  namespace: production
spec:
  schedule: "0 3 * * *"              # 每天凌晨3点
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 7       # 保留7天的成功记录
  failedJobsHistoryLimit: 3
  startingDeadlineSeconds: 600
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 7200
      ttlSecondsAfterFinished: 604800  # 7天后自动清理
      template:
        metadata:
          labels:
            app: mysql-backup
        spec:
          containers:
          - name: backup
            image: mysql:8.0
            command:
            - /bin/sh
            - -c
            - |
              set -e
              TIMESTAMP=$(date +%Y%m%d_%H%M%S)
              BACKUP_FILE="/backup/mysql_backup_${TIMESTAMP}.sql.gz"
              
              echo "[$(date)] Starting backup..."
              mysqldump -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD \
                --all-databases --single-transaction --routines --triggers \
                | gzip > ${BACKUP_FILE}
              
              # 清理30天前的备份
              find /backup -name "mysql_backup_*.sql.gz" -mtime +30 -delete
              
              echo "[$(date)] Backup completed: ${BACKUP_FILE}"
              echo "[$(date)] Backup size: $(du -h ${BACKUP_FILE} | cut -f1)"
            env:
            - name: MYSQL_HOST
              value: "mysql-primary.production.svc.cluster.local"
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-backup-secret
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-backup-secret
                  key: password
            resources:
              requests:
                cpu: 200m
                memory: 256Mi
              limits:
                cpu: "1"
                memory: 1Gi
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-nfs-pvc
          restartPolicy: OnFailure
```

---

### 三、常用运维操作

#### 查看Job/CronJob状态

```bash
# 查看所有Job
kubectl get jobs -A

# 查看CronJob及下次执行时间
kubectl get cronjobs
# NAME                  SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# mysql-daily-backup    0 3 * * *   False     0        6h              30d

# 查看CronJob创建的Job历史
kubectl get jobs --selector=job-name -l app=mysql-backup

# 查看Job详情
kubectl describe job mysql-backup

# 查看Job的Pod日志
kubectl logs job/mysql-backup
```

#### 手动触发CronJob

```bash
# 从CronJob手动创建一次性Job
kubectl create job mysql-backup-manual --from=cronjob/mysql-daily-backup
```

#### 暂停和恢复CronJob

```bash
# 暂停（维护时使用）
kubectl patch cronjob mysql-daily-backup -p '{"spec":{"suspend":true}}'

# 恢复
kubectl patch cronjob mysql-daily-backup -p '{"spec":{"suspend":false}}'
```

#### 清理已完成的Job

```bash
# 删除所有已完成的Job
kubectl delete jobs --field-selector status.successful=1

# 推荐：使用ttlSecondsAfterFinished自动清理
```

---

### 四、生产环境最佳实践

#### 1. 始终设置资源限制

Job也是Pod，不设resource limits可能把节点资源吃光：

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

#### 2. 配置ttlSecondsAfterFinished

避免大量已完成的Job和Pod堆积：

```yaml
spec:
  ttlSecondsAfterFinished: 86400  # 1天后自动清理
```

#### 3. 合理设置activeDeadlineSeconds

防止任务卡住导致资源一直被占用：

```yaml
spec:
  activeDeadlineSeconds: 3600  # 最多跑1小时
```

#### 4. CronJob注意时区

Kubernetes 1.27+ 支持时区设置：

```yaml
spec:
  timeZone: "Asia/Shanghai"    # 指定时区
  schedule: "0 9 * * 1-5"      # 北京时间工作日早9点
```

低版本集群中Cron表达式使用的是**kube-controller-manager所在节点的时区**，需要注意换算。

#### 5. 监控与告警

```yaml
# 结合Prometheus监控Job状态
# kube_job_status_failed: Job失败数
# kube_job_status_succeeded: Job成功数
# kube_cronjob_next_schedule_time: 下次调度时间
```

建议对以下情况设置告警：
- Job连续失败超过N次
- CronJob超过预期时间未执行
- Job执行时间异常过长

---

### 五、总结

| 要点 | 说明 |
|------|------|
| Job用途 | 一次性任务，保证至少成功执行一次 |
| CronJob用途 | 定时周期性任务，底层自动创建Job |
| restartPolicy | 只能用`Never`或`OnFailure` |
| 并发控制 | CronJob通过`concurrencyPolicy`控制 |
| 资源清理 | 用`ttlSecondsAfterFinished`自动清理 |
| 超时保护 | 用`activeDeadlineSeconds`防止任务卡住 |
| 手动触发 | `kubectl create job --from=cronjob/xxx` |

Job和CronJob是Kubernetes任务调度的基础组件。掌握它们，你就能优雅地处理各种批处理和定时任务场景。下一篇我们将继续探索K8s的其他核心资源。
