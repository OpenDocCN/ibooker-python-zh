# 第十二章。容器编排：Kubernetes

如果您正在尝试使用 Docker，或者在单台机器上运行一组 Docker 容器就是您的全部需求，那么 Docker 和 Docker Compose 就足够满足您的需求。然而，一旦您从数字`1`（单台机器）转移到数字`2`（多台机器），您就需要开始考虑如何在网络中编排这些容器。对于生产场景来说，这是必须的。您至少需要两台机器来实现容错和高可用性。

在我们这个云计算时代，扩展基础设施的推荐方法是“外部”（也称为“水平扩展”），通过向整个系统添加更多实例来实现，而不是通过“上升”（或“垂直扩展”）的旧方法，即向单个实例添加更多 CPU 和内存。一个 Docker 编排平台使用这些许多实例或节点作为原始资源（CPU、内存、网络），然后将这些资源分配给平台内运行的各个容器。这与我们在第十一章中提到的使用容器而不是经典虚拟机（VMs）的优势相关联：您可以更精细地为容器分配这些资源，因此您的基础设施投入将得到更好的利用率。

在为特定目的配置服务器并在每个实例上运行特定软件包（如 Web 服务器软件、缓存软件、数据库软件）的模式已经发生了转变，现在将它们配置为资源分配的通用单元，并在其上运行 Docker 容器，由 Docker 编排平台协调管理。您可能熟悉将服务器视为“宠物”与将它们视为“牲畜”的区别。在基础架构设计的早期阶段，每个服务器都有明确的功能（如邮件服务器），许多时候每个特定功能仅有一个服务器。这些服务器有命名方案（格里格还记得在点博时代使用行星系统的命名方案），花费大量时间进行管理和维护，因此被称为“宠物”。当 Puppet、Chef 和 Ansible 等配置管理工具出现时，通过在每台服务器上使用相同的安装程序，同时轻松配置多个相同类型的服务器（例如 Web 服务器农场）变得更加容易。这与云计算的兴起同时发生，前面提到的水平扩展的概念以及对容错和高可用性的更多关注，这些都是良好设计的系统基础设施的关键属性。这些服务器或云实例被视为牲畜，是可以丢弃的单位，它们在集合中有价值。

容器和无服务器计算时代也带来了另一个称谓，“昆虫”。确实，人们可以将容器的出现和消失视为一种短暂存在，就像是一个短暂的昆虫一样。函数即服务比 Docker 容器更短暂，其存在期间短暂而强烈，与其调用的持续时间相一致。

在容器的情况下，它们的短暂性使得在大规模上实现它们的编排和互操作性变得困难。这正是容器编排平台填补的需求。以前有多个 Docker 编排平台可供选择，如 Mesosphere 和 Docker Swarm，但如今我们可以安全地说 Kubernetes 已经赢得了这场比赛。本章剩余部分将简要概述 Kubernetes，并示例演示如何在 Kubernetes 中运行相同的应用程序，从`docker-compose`迁移到 Kubernetes。我们还将展示如何使用 Helm，一个 Kubernetes 包管理器，来安装名为 charts 的包，用于监控和仪表盘工具 Prometheus 和 Grafana，并如何自定义这些 charts。

# Kubernetes 概念简介

理解构成 Kubernetes 集群的许多部分的最佳起点是[官方 Kubernetes 文档](https://oreil.ly/TYpdE)。

在高层次上，Kubernetes 集群由节点组成，可以等同于运行在云中的裸金属或虚拟机的服务器。节点运行 pod，即 Docker 容器的集合。Pod 是 Kubernetes 中的部署单位。一个 pod 中的所有容器共享同一网络，并且可以像在同一主机上运行一样互相引用。有许多情况下，运行多个容器在一个 pod 中是有利的。通常情况下，您的应用程序容器作为 pod 中的主容器运行，如果需要，您可以运行一个或多个所谓的“sidecar”容器，用于功能，例如日志记录或监视。一个特殊的 sidecar 容器案例是“init 容器”，它保证首先运行，并可用于诸如运行数据库迁移等的日常管理任务。我们将在本章后面进一步探讨这个问题。

应用程序通常会为了容错性和性能而使用多个 pod。负责启动和维护所需 pod 数量的 Kubernetes 对象称为部署（deployment）。为了让 pod 能够与其他 pod 通信，Kubernetes 提供了另一种对象，称为服务（service）。服务通过选择器（selectors）与部署绑定。服务也可以向外部客户端暴露，可以通过在每个 Kubernetes 节点上暴露一个 NodePort 作为静态端口，或者创建对应实际负载均衡器的 LoadBalancer 对象来实现，如果云提供商支持的话。

对于管理诸如密码、API 密钥和其他凭据等敏感信息，Kubernetes 提供了 Secret 对象。我们将看到一个示例，使用 Secret 存储数据库密码。

# 使用 Kompose 从 docker-compose.yaml 创建 Kubernetes Manifests

让我们再次查看讨论 第十一章 中的 Flask 示例应用程序的 *docker_compose.yaml* 文件：

```py
$ cat docker-compose.yaml
version: "3"
services:
  app:
    image: "griggheo/flask-by-example:v1"
    command: "manage.py runserver --host=0.0.0.0"
    ports:
      - "5000:5000"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
  worker:
    image: "griggheo/flask-by-example:v1"
    command: "worker.py"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
  migrations:
    image: "griggheo/flask-by-example:v1"
    command: "manage.py db upgrade"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
    depends_on:
      - db
  db:
    image: "postgres:11"
    container_name: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - dbdata:/var/lib/postgresql/data
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
volumes:
  dbdata:
```

我们将使用一个名为 `Kompose` 的工具来将此 YAML 文件翻译成一组 Kubernetes Manifests。

要在 macOS 机器上获取新版本的 `Kompose`，首先从 [Git 仓库](https://oreil.ly/GUqaq) 下载它，然后将其移动到 */usr/local/bin/kompose*，并使其可执行。请注意，如果您依赖操作系统的软件包管理系统（例如 Ubuntu 系统上的 `apt` 或 Red Hat 系统上的 `yum`）来安装 `Kompose`，可能会获得一个较旧且不兼容这些说明的版本。

运行 `kompose convert` 命令从现有的 *docker-compose.yaml* 文件创建 Kubernetes manifest 文件：

```py
$ kompose convert
INFO Kubernetes file "app-service.yaml" created
INFO Kubernetes file "db-service.yaml" created
INFO Kubernetes file "redis-service.yaml" created
INFO Kubernetes file "app-deployment.yaml" created
INFO Kubernetes file "db-deployment.yaml" created
INFO Kubernetes file "dbdata-persistentvolumeclaim.yaml" created
INFO Kubernetes file "migrations-deployment.yaml" created
INFO Kubernetes file "redis-deployment.yaml" created
INFO Kubernetes file "worker-deployment.yaml" created
```

此时，请删除 *docker-compose.yaml* 文件：

```py
$ rm docker-compose.yaml
```

# 将 Kubernetes Manifests 部署到基于 minikube 的本地 Kubernetes 集群

我们的下一步是将 Kubernetes Manifests 部署到基于 `minikube` 的本地 Kubernetes 集群。

在 macOS 上运行 `minikube` 的先决条件是安装 *VirtualBox*。从其 [下载页面](https://oreil.ly/BewRq) 下载 macOS 版本的 VirtualBox 包，并安装它，然后将其移动到 */usr/local/bin/minikube* 以使其可执行。请注意，此时写作本文时，`minikube` 安装了 Kubernetes 版本为 1.15\. 如果要按照这些示例进行操作，请指定要使用 `minikube` 安装的 Kubernetes 版本：

```py
$ minikube start --kubernetes-version v1.15.0
 minikube v1.2.0 on darwin (amd64)
 Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
 Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
 Downloading kubeadm v1.15.0
 Downloading kubelet v1.15.0
 Pulling images ...
 Launching Kubernetes ...
 Verifying: apiserver proxy etcd scheduler controller dns
 Done! kubectl is now configured to use "minikube"

```

与 Kubernetes 集群交互的主要命令是 `kubectl`。

通过从 [发布页面](https://oreil.ly/f9Wv0) 下载并移动到 */usr/local/bin/kubectl* 并使其可执行，来在 macOS 机器上安装 `kubectl`。

在运行 `kubectl` 命令时，您将使用的一个主要概念是 *context*，它表示您希望与之交互的 Kubernetes 集群。`minikube` 的安装过程已经为我们创建了一个称为 *minikube* 的上下文。指定 `kubectl` 指向特定上下文的一种方法是使用以下命令：

```py
$ kubectl config use-context minikube
Switched to context "minikube".
```

另一种更方便的方法是从 [Git 仓库](https://oreil.ly/SIf1U) 安装 `kubectx` 实用程序，然后运行：

```py
$ kubectx minikube
Switched to context "minikube".
```

###### 提示

另一个在 Kubernetes 工作中很实用的客户端实用程序是 [kube-ps1](https://oreil.ly/AcE32)。对于基于 Zsh 的 macOS 设置，请将以下片段添加到文件 *~/.zshrc* 中：

```py
source "/usr/local/opt/kube-ps1/share/kube-ps1.sh"
PS1='$(kube_ps1)'$PS1
```

这些行将 shell 提示符更改为显示当前 Kubernetes 上下文和命名空间。当您开始与多个 Kubernetes 集群进行交互时，这将帮助您区分生产环境和暂存环境。

现在在本地 `minikube` 集群上运行 `kubectl` 命令。例如，`kubectl get nodes` 命令显示集群中的节点。在本例中，只有一个带有 `master` 角色的节点：

```py
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   2m14s   v1.15.0
```

首先，从 *dbdata-persistentvolumeclaim.yaml* 文件创建持久卷声明（PVC）对象，该文件由 `Kompose` 创建，对应于在使用 `docker-compose` 运行时为 PostgreSQL 数据库容器分配的本地卷：

```py
$ cat dbdata-persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: dbdata
  name: dbdata
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
status: {}
```

要在 Kubernetes 中创建此对象，请使用 `kubectl create` 命令，并使用 `-f` 标志指定清单文件名：

```py
$ kubectl create -f dbdata-persistentvolumeclaim.yaml
persistentvolumeclaim/dbdata created
```

使用 `kubectl get pvc` 命令列出所有 PVC，验证我们的 PVC 是否存在：

```py
$ kubectl get pvc
NAME     STATUS   VOLUME                                     CAPACITY
ACCESS MODES   STORAGECLASS   AGE
dbdata   Bound    pvc-39914723-4455-439b-a0f5-82a5f7421475   100Mi
RWO            standard       1m
```

下一步是为 PostgreSQL 创建 Deployment 对象。使用之前由 `Kompose` 工具创建的清单文件 *db-deployment.yaml*：

```py
$ cat db-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: db
  name: db
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: db
    spec:
      containers:
      - image: postgres:11
        name: postgres
        ports:
        - containerPort: 5432
        resources: {}
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: dbdata
      restartPolicy: Always
      volumes:
      - name: dbdata
        persistentVolumeClaim:
          claimName: dbdata
status: {}
```

要创建部署，请使用 `kubectl create -f` 命令，并指向清单文件：

```py
$ kubectl create -f db-deployment.yaml
deployment.extensions/db created
```

要验证是否已创建部署，请列出集群中的所有部署，并列出作为部署一部分创建的 pod：

```py
$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
db       1/1     1            1           1m

$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
db-67659d85bf-vrnw7   1/1     Running   0          1m
```

接下来，为示例 Flask 应用程序创建数据库。使用类似于 `docker exec` 的命令在运行中的 Docker 容器内运行 `psql` 命令。在 Kubernetes 集群中，命令的形式是 `kubectl exec`：

```py
$ kubectl exec -it db-67659d85bf-vrnw7 -- psql -U postgres
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

postgres=# create database wordcount;
CREATE DATABASE
postgres=# \q

$ kubectl exec -it db-67659d85bf-vrnw7 -- psql -U postgres wordcount
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

wordcount=# CREATE ROLE wordcount_dbadmin;
CREATE ROLE
wordcount=# ALTER ROLE wordcount_dbadmin LOGIN;
ALTER ROLE
wordcount=# ALTER USER wordcount_dbadmin PASSWORD 'MYPASS';
ALTER ROLE
wordcount=# \q
```

下一步是创建与 `db` 部署相对应的 Service 对象，将部署暴露给运行在集群内的其他服务，例如 Redis worker 服务和主应用程序服务。这是 `db` 服务的清单文件：

```py
$ cat db-service.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: db
  name: db
spec:
  ports:
  - name: "5432"
    port: 5432
    targetPort: 5432
  selector:
    io.kompose.service: db
status:
  loadBalancer: {}
```

需要注意的一点是以下部分：

```py
  labels:
    io.kompose.service: db
```

此部分同时出现在部署清单和服务清单中，并确实是将两者联系在一起的方法。服务将与具有相同标签的任何部署关联。

使用 `kubectl create -f` 命令创建 Service 对象：

```py
$ kubectl create -f db-service.yaml
service/db created
```

列出所有服务，并注意已创建的 `db` 服务：

```py
$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
db           ClusterIP   10.110.108.96   <none>        5432/TCP   6s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    4h45m
```

下一个要部署的服务是 Redis。基于由 `Kompose` 生成的清单文件创建 Deployment 和 Service 对象：

```py
$ cat redis-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: redis
  name: redis
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        ports:
        - containerPort: 6379
        resources: {}
      restartPolicy: Always
status: {}

$ kubectl create -f redis-deployment.yaml
deployment.extensions/redis created

$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
db-67659d85bf-vrnw7     1/1     Running   0          37m
redis-c6476fbff-8kpqz   1/1     Running   0          11s

$ kubectl create -f redis-service.yaml
service/redis created

$ cat redis-service.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: redis
  name: redis
spec:
  ports:
  - name: "6379"
    port: 6379
    targetPort: 6379
  selector:
    io.kompose.service: redis
status:
  loadBalancer: {}

$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
db           ClusterIP   10.110.108.96   <none>        5432/TCP   84s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    4h46m
redis        ClusterIP   10.106.44.183   <none>        6379/TCP   10s
```

到目前为止，已部署的两个服务，`db` 和 `redis`，互不影响。应用程序的下一部分是工作进程，需要与 PostgreSQL 和 Redis 进行通信。这就是使用 Kubernetes 服务的优势所在。工作部署可以通过服务名称引用 PostgreSQL 和 Redis 的端点。Kubernetes 知道如何将来自客户端（作为工作部署中的 pod 的一部分运行的容器）的请求路由到服务器（作为 `db` 和 `redis` 部署中的 pod 的一部分运行的 PostgreSQL 和 Redis 容器）。

工作节点部署中使用的环境变量之一是`DATABASE_URL`。 它包含应用程序使用的数据库密码。 不应在部署清单文件中明文显示密码，因为该文件需要检入版本控制。 取而代之，创建一个 Kubernetes Secret 对象。

首先，将密码字符串编码为`base64`：

```py
$ echo MYPASS | base64
MYPASSBASE64
```

然后，创建一个描述要创建的 Kubernetes Secret 对象的清单文件。 由于密码的`base64`编码不安全，请使用`sops`来编辑和保存加密的清单文件*secrets.yaml.enc*：

```py
$ sops --pgp E14104A0890994B9AC9C9F6782C1FF5E679EFF32 secrets.yaml.enc
```

在编辑器中添加这些行：

```py
apiVersion: v1
kind: Secret
metadata:
  name: fbe-secret
type: Opaque
data:
  dbpass: MYPASSBASE64
```

*secrets.yaml.enc* 文件现在可以检入，因为它包含密码的`base64`值的加密版本。

要解密加密文件，请使用`sops -d`命令：

```py
$ sops -d secrets.yaml.enc
apiVersion: v1
kind: Secret
metadata:
  name: fbe-secret
type: Opaque
data:
  dbpass: MYPASSBASE64
```

将`sops -d`的输出导向`kubectl create -f`以创建 Kubernetes Secret 对象：

```py
$ sops -d secrets.yaml.enc | kubectl create -f -
secret/fbe-secret created
```

检查 Kubernetes Secrets 并描述已创建的 Secret：

```py
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-k7652   kubernetes.io/service-account-token   3      3h19m
fbe-secret            Opaque                                1      45s

$ kubectl describe secret fbe-secret
Name:         fbe-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
dbpass:  12 bytes
```

要获取`base64`编码的 Secret，请使用：

```py
$ kubectl get secrets fbe-secret -ojson | jq -r ".data.dbpass"
MYPASSBASE64
```

要在 macOS 机器上获取纯文本密码，请使用以下命令：

```py
$ kubectl get secrets fbe-secret -ojson | jq -r ".data.dbpass" | base64 -D
MYPASS
```

在 Linux 机器上，`base64`解码的正确标志是`-d`，因此正确的命令将是：

```py
$ kubectl get secrets fbe-secret -ojson | jq -r ".data.dbpass" | base64 -d
MYPASS
```

现在可以在工作节点的部署清单中使用该秘密。 修改由`Kompose`实用程序生成的*worker-deployment.yaml*文件，并添加两个环境变量：

+   `DBPASS`是从`fbe-secret` Secret 对象中检索的数据库密码。

+   `DATABASE_URL`是 PostgreSQL 的完整数据库连接字符串，包括数据库密码，并将其引用为`${DBPASS}`。

这是修改后的*worker-deployment.yaml*的版本：

```py
$ cat worker-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: worker
  name: worker
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: worker
    spec:
      containers:
      - args:
        - worker.py
        env:
        - name: APP_SETTINGS
          value: config.ProductionConfig
        - name: DBPASS
          valueFrom:
            secretKeyRef:
              name: fbe-secret
              key: dbpass
        - name: DATABASE_URL
          value: postgresql://wordcount_dbadmin:${DBPASS}@db/wordcount
        - name: REDISTOGO_URL
          value: redis://redis:6379
        image: griggheo/flask-by-example:v1
        name: worker
        resources: {}
      restartPolicy: Always
status: {}
```

通过调用`kubectl create -f`创建工作节点部署对象的方式与其他部署相同：

```py
$ kubectl create -f worker-deployment.yaml
deployment.extensions/worker created
```

列出 pods：

```py
$ kubectl get pods
NAME                      READY   STATUS              RESTARTS   AGE
db-67659d85bf-vrnw7       1/1     Running             1          21h
redis-c6476fbff-8kpqz     1/1     Running             1          21h
worker-7dbf5ff56c-vgs42   0/1     Init:ErrImagePull   0          7s
```

注意，工作节点显示为状态`Init:ErrImagePull`。 要查看有关此状态的详细信息，请运行`kubectl describe`：

```py
$ kubectl describe pod worker-7dbf5ff56c-vgs42 | tail -10
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m51s                default-scheduler
  Successfully assigned default/worker-7dbf5ff56c-vgs42 to minikube

  Normal   Pulling    76s (x4 over 2m50s)  kubelet, minikube
  Pulling image "griggheo/flask-by-example:v1"

  Warning  Failed     75s (x4 over 2m49s)  kubelet, minikube
  Failed to pull image "griggheo/flask-by-example:v1": rpc error:
  code = Unknown desc = Error response from daemon: pull access denied for
  griggheo/flask-by-example, repository does not exist or may require
  'docker login'

  Warning  Failed     75s (x4 over 2m49s)  kubelet, minikube
  Error: ErrImagePull

  Warning  Failed     62s (x6 over 2m48s)  kubelet, minikube
  Error: ImagePullBackOff

  Normal   BackOff    51s (x7 over 2m48s)  kubelet, minikube
  Back-off pulling image "griggheo/flask-by-example:v1"
```

部署尝试从 Docker Hub 拉取`griggheo/flask-by-example:v1`私有 Docker 镜像，并且缺少访问私有 Docker 注册表所需的适当凭据。 Kubernetes 包括一种特殊类型的对象，用于处理这种情况，称为*imagePullSecret*。

使用`sops`创建包含 Docker Hub 凭据和`kubectl create secret`调用的加密文件：

```py
$ sops --pgp E14104A0890994B9AC9C9F6782C1FF5E679EFF32 \
create_docker_credentials_secret.sh.enc
```

文件的内容是：

```py
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=Type your dockerhub username, same as when you `docker login`
DOCKER_EMAIL=Type your dockerhub email, same as when you `docker login`
DOCKER_PASSWORD=Type your dockerhub pw, same as when you `docker login`

kubectl create secret docker-registry myregistrykey \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL
```

使用`sops`解密加密文件并通过`bash`运行它：

```py
$ sops -d create_docker_credentials_secret.sh.enc | bash -
secret/myregistrykey created
```

检查秘密：

```py
$ kubectl get secrets myregistrykey -oyaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJkb2NrZXIuaW8iO
kind: Secret
metadata:
  creationTimestamp: "2019-07-17T22:11:56Z"
  name: myregistrykey
  namespace: default
  resourceVersion: "16062"
  selfLink: /api/v1/namespaces/default/secrets/myregistrykey
  uid: 47d29ffc-69e4-41df-a237-1138cd9e8971
type: kubernetes.io/dockerconfigjson
```

对工作节点部署清单的唯一更改是添加这些行：

```py
      imagePullSecrets:
      - name: myregistrykey
```

在此行后包含它：

```py
     restartPolicy: Always
```

删除工作节点部署并重新创建：

```py
$ kubectl delete -f worker-deployment.yaml
deployment.extensions "worker" deleted

$ kubectl create -f worker-deployment.yaml
deployment.extensions/worker created
```

现在工作节点处于运行状态，并且没有错误：

```py
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
db-67659d85bf-vrnw7       1/1     Running   1          22h
redis-c6476fbff-8kpqz     1/1     Running   1          21h
worker-7dbf5ff56c-hga37   1/1     Running   0          4m53s
```

使用`kubectl logs`命令检查工作节点的日志：

```py
$ kubectl logs worker-7dbf5ff56c-hga37
20:43:13 RQ worker 'rq:worker:040640781edd4055a990b798ac2eb52d'
started, version 1.0
20:43:13 *** Listening on default...
20:43:13 Cleaning registries for queue: default
```

接下来是解决应用程序部署的步骤。当应用程序在第十一章中以`docker-compose`设置部署时，使用单独的 Docker 容器来运行更新 Flask 数据库所需的迁移。这种任务很适合作为同一 Pod 中主应用程序容器的侧车容器运行。侧车容器将在应用程序部署清单中定义为 Kubernetes 的[`initContainer`](https://oreil.ly/80L5L)。此类容器保证在属于其所属的 Pod 内的其他容器启动之前运行。

将此部分添加到由`Kompose`实用程序生成的*app-deployment.yaml*清单文件中，并删除*migrations-deployment.yaml*文件：

```py
      initContainers:
      - args:
        - manage.py
        - db
        - upgrade
        env:
        - name: APP_SETTINGS
          value: config.ProductionConfig
        - name: DATABASE_URL
          value: postgresql://wordcount_dbadmin:@db/wordcount
        image: griggheo/flask-by-example:v1
        name: migrations
        resources: {}

$ rm migrations-deployment.yaml
```

在应用程序部署清单中复用为工作程序部署创建的`fbe-secret` Secret 对象：

```py
$ cat app-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: app
  name: app
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: app
    spec:
      initContainers:
      - args:
        - manage.py
        - db
        - upgrade
        env:
        - name: APP_SETTINGS
          value: config.ProductionConfig
        - name: DBPASS
          valueFrom:
            secretKeyRef:
              name: fbe-secret
              key: dbpass
        - name: DATABASE_URL
          value: postgresql://wordcount_dbadmin:${DBPASS}@db/wordcount
        image: griggheo/flask-by-example:v1
        name: migrations
        resources: {}
      containers:
      - args:
        - manage.py
        - runserver
        - --host=0.0.0.0
        env:
        - name: APP_SETTINGS
          value: config.ProductionConfig
        - name: DBPASS
          valueFrom:
            secretKeyRef:
              name: fbe-secret
              key: dbpass
        - name: DATABASE_URL
          value: postgresql://wordcount_dbadmin:${DBPASS}@db/wordcount
        - name: REDISTOGO_URL
          value: redis://redis:6379
        image: griggheo/flask-by-example:v1
        name: app
        ports:
        - containerPort: 5000
        resources: {}
      restartPolicy: Always
status: {}
```

使用`kubectl create -f`创建应用程序部署，然后列出 Pod 并描述应用程序 Pod：

```py
$ kubectl create -f app-deployment.yaml
deployment.extensions/app created

$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
app-c845d8969-l8nhg       1/1     Running   0          7s
db-67659d85bf-vrnw7       1/1     Running   1          22h
redis-c6476fbff-8kpqz     1/1     Running   1          21h
worker-7dbf5ff56c-vgs42   1/1     Running   0          4m53s
```

将应用程序部署到`minikube`的最后一步是确保为应用程序创建 Kubernetes 服务，并将其声明为类型`LoadBalancer`，以便从集群外访问：

```py
$ cat app-service.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.16.0 (0c01309)
  creationTimestamp: null
  labels:
    io.kompose.service: app
  name: app
spec:
  ports:
  - name: "5000"
    port: 5000
    targetPort: 5000
  type: LoadBalancer
  selector:
    io.kompose.service: app
status:
  loadBalancer: {}
```

###### 注意

与`db`服务类似，`app`服务通过在应用程序部署和服务清单中存在的标签声明与`app`部署关联：

```py
  labels:
    io.kompose.service: app
```

使用`kubectl create`创建服务：

```py
$ kubectl create -f app-service.yaml
service/app created

$ kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
app          LoadBalancer   10.99.55.191    <pending>     5000:30097/TCP   2s
db           ClusterIP      10.110.108.96   <none>        5432/TCP         21h
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          26h
redis        ClusterIP      10.106.44.183   <none>        6379/TCP         21h
```

接下来运行：

```py
$ minikube service app
```

此命令将使用 URL[*http://192.168.99.100:30097/*](http://192.168.99.100:30097/)打开默认浏览器，并显示 Flask 站点的主页。

在下一节中，我们将使用相同的 Kubernetes 清单文件部署我们的应用程序到一个将由 Pulumi 在 Google Cloud Platform（GCP）中配置的 Kubernetes 集群中。

# 使用 Pulumi 在 Google Cloud Platform（GCP）中启动 GKE Kubernetes 集群

在本节中，我们将使用[Pulumi GKE 示例](https://oreil.ly/VGBfF)以及[GCP 设置文档](https://oreil.ly/kRsFA)，因此在继续之前，请使用这些链接获取所需的文档。

首先创建一个新目录：

```py
$ mkdir pulumi_gke
$ cd pulumi_gke
```

使用[macOS 说明](https://oreil.ly/f4pPs)设置 Google Cloud SDK。

使用`gcloud init`命令初始化 GCP 环境。创建一个新的配置和一个名为*pythonfordevops-gke-pulumi*的新项目：

```py
$ gcloud init
Welcome! This command will take you through the configuration of gcloud.

Settings from your current configuration [default] are:
core:
  account: grig.gheorghiu@gmail.com
  disable_usage_reporting: 'True'
  project: pulumi-gke-testing

Pick configuration to use:
 [1] Re-initialize this configuration [default] with new settings
 [2] Create a new configuration
Please enter your numeric choice:  2

Enter configuration name. Names start with a lower case letter and
contain only lower case letters a-z, digits 0-9, and hyphens '-':
pythonfordevops-gke-pulumi
Your current configuration has been set to: [pythonfordevops-gke-pulumi]

Pick cloud project to use:
 [1] pulumi-gke-testing
 [2] Create a new project
Please enter numeric choice or text value (must exactly match list
item):  2

Enter a Project ID. pythonfordevops-gke-pulumi
Your current project has been set to: [pythonfordevops-gke-pulumi].
```

登录到 GCP 帐户：

```py
$ gcloud auth login
```

登录到默认应用程序`pythonfordevops-gke-pulumi`：

```py
$ gcloud auth application-default login
```

运行`pulumi new`命令创建一个新的 Pulumi 项目，指定*gcp-python*作为模板，*pythonfordevops-gke-pulumi*作为项目名称：

```py
$ pulumi new
Please choose a template: gcp-python
A minimal Google Cloud Python Pulumi program
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project name: (pulumi_gke_py) pythonfordevops-gke-pulumi
project description: (A minimal Google Cloud Python Pulumi program)
Created project 'pythonfordevops-gke-pulumi'

stack name: (dev)
Created stack 'dev'

gcp:project: The Google Cloud project to deploy into: pythonfordevops-gke-pulumi
Saved config

Your new project is ready to go! 

To perform an initial deployment, run the following commands:

   1\. virtualenv -p python3 venv
   2\. source venv/bin/activate
   3\. pip3 install -r requirements.txt

Then, run 'pulumi up'.

```

以下文件由`pulumi new`命令创建：

```py
$ ls -la
ls -la
total 40
drwxr-xr-x  7 ggheo  staff  224 Jul 16 15:08 .
drwxr-xr-x  6 ggheo  staff  192 Jul 16 15:06 ..
-rw-------  1 ggheo  staff   12 Jul 16 15:07 .gitignore
-rw-r--r--  1 ggheo  staff   50 Jul 16 15:08 Pulumi.dev.yaml
-rw-------  1 ggheo  staff  107 Jul 16 15:07 Pulumi.yaml
-rw-------  1 ggheo  staff  203 Jul 16 15:07 __main__.py
-rw-------  1 ggheo  staff   34 Jul 16 15:07 requirements.txt
```

我们将使用[Pulumi 示例](https://oreil.ly/SIT-v)GitHub 存储库中的`gcp-py-gke`示例。

从*examples/gcp-py-gke*复制**.py*和*requirements.txt*到当前目录：

```py
$ cp ~/pulumi-examples/gcp-py-gke/*.py .
$ cp ~/pulumi-examples/gcp-py-gke/requirements.txt .
```

配置与 Pulumi 在 GCP 中操作所需的与 GCP 相关的变量：

```py
$ pulumi config set gcp:project pythonfordevops-gke-pulumi
$ pulumi config set gcp:zone us-west1-a
$ pulumi config set password --secret PASS_FOR_KUBE_CLUSTER
```

创建并使用 Python `virtualenv`，安装*requirements.txt*中声明的依赖项，然后通过运行`pulumi up`命令启动在*mainpy*中定义的 GKE 集群：

```py
$ virtualenv -p python3 venv
$ source venv/bin/activate
$ pip3 install -r requirements.txt
$ pulumi up
```

###### 提示

确保通过在 GCP Web 控制台中将其与 Google 计费账户关联来启用 Kubernetes Engine API。

可以在[GCP 控制台](https://oreil.ly/Su5FZ)中看到 GKE 集群。

生成适当的`kubectl`配置以及使用它与新配置的 GKE 集群进行交互。通过 Pulumi 程序将`kubectl`配置方便地导出为`output`：

```py
$ pulumi stack output kubeconfig > kubeconfig.yaml
$ export KUBECONFIG=./kubeconfig.yaml
```

列出组成 GKE 集群的节点：

```py
$ kubectl get nodes
NAME                                                 STATUS   ROLES    AGE
   VERSION
gke-gke-cluster-ea17e87-default-pool-fd130152-30p3   Ready    <none>   4m29s
   v1.13.7-gke.8
gke-gke-cluster-ea17e87-default-pool-fd130152-kf9k   Ready    <none>   4m29s
   v1.13.7-gke.8
gke-gke-cluster-ea17e87-default-pool-fd130152-x9dx   Ready    <none>   4m27s
   v1.13.7-gke.8
```

# 将 Flask 示例应用程序部署到 GKE

使用相同的 Kubernetes 清单文件在 GKE 集群中部署`minikube`示例，通过`kubectl`命令。首先创建`redis`部署和服务：

```py
$ kubectl create -f redis-deployment.yaml
deployment.extensions/redis created

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running   0          5m57s
redis-9946db5cc-8g6zz            1/1     Running   0          20s

$ kubectl create -f redis-service.yaml
service/redis created

$ kubectl get service redis
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
redis   ClusterIP   10.59.245.221   <none>        6379/TCP   18s
```

创建一个 PersistentVolumeClaim，用作 PostgreSQL 数据库的数据卷：

```py
$ kubectl create -f dbdata-persistentvolumeclaim.yaml
persistentvolumeclaim/dbdata created

$ kubectl get pvc
NAME     STATUS   VOLUME                                     CAPACITY
dbdata   Bound    pvc-00c8156c-b618-11e9-9e84-42010a8a006f   1Gi
   ACCESS MODES   STORAGECLASS   AGE
   RWO            standard       12s
```

创建`db`部署：

```py
$ kubectl create -f db-deployment.yaml
deployment.extensions/db created

$ kubectl get pods
NAME                             READY   STATUS             RESTARTS  AGE
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running            0         8m52s
db-6b4fbb57d9-cjjxx              0/1     CrashLoopBackOff   1         38s
redis-9946db5cc-8g6zz            1/1     Running            0         3m15s

$ kubectl logs db-6b4fbb57d9-cjjxx

initdb: directory "/var/lib/postgresql/data" exists but is not empty
It contains a lost+found directory, perhaps due to it being a mount point.
Using a mount point directly as the data directory is not recommended.
Create a subdirectory under the mount point.
```

当尝试创建`db`部署时遇到了问题。GKE 提供了一个已挂载为*/var/lib/postgresql/data*的持久卷，并且根据上述错误消息，该目录并非空。

删除失败的`db`部署：

```py
$ kubectl delete -f db-deployment.yaml
deployment.extensions "db" deleted
```

创建一个新的临时 Pod，用于挂载与 Pod 内部的*/data*中相同的`dbdata` PersistentVolumeClaim，以便检查其文件系统。为了故障排除目的，启动这种类型的临时 Pod 是一个有用的技术手段：

```py
$ cat pvc-inspect.yaml
kind: Pod
apiVersion: v1
metadata:
  name: pvc-inspect
spec:
  volumes:
    - name: dbdata
      persistentVolumeClaim:
        claimName: dbdata
  containers:
    - name: debugger
      image: busybox
      command: ['sleep', '3600']
      volumeMounts:
        - mountPath: "/data"
          name: dbdata

$ kubectl create -f pvc-inspect.yaml
pod/pvc-inspect created

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running   0          20m
pvc-inspect                      1/1     Running   0          35s
redis-9946db5cc-8g6zz            1/1     Running   0          14m
```

使用`kubectl exec`打开 Pod 内部的 shell，以便检查*/data*：

```py
$ kubectl exec -it pvc-inspect -- sh
/ # cd /data
/data # ls -la
total 24
drwx------    3 999      root          4096 Aug  3 17:57 .
drwxr-xr-x    1 root     root          4096 Aug  3 18:08 ..
drwx------    2 999      root         16384 Aug  3 17:57 lost+found
/data # rm -rf lost\+found/
/data # exit
```

注意*/data*中包含一个需要移除的名为*lost+found*的目录。

删除临时 Pod：

```py
$ kubectl delete pod pvc-inspect
pod "pvc-inspect" deleted
```

再次创建`db`部署，在这次操作中成功完成：

```py
$ kubectl create -f db-deployment.yaml
deployment.extensions/db created

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running   0          23m
db-6b4fbb57d9-8h978              1/1     Running   0          19s
redis-9946db5cc-8g6zz            1/1     Running   0          17m

$ kubectl logs db-6b4fbb57d9-8h978
PostgreSQL init process complete; ready for start up.

2019-08-03 18:12:01.108 UTC [1]
LOG:  listening on IPv4 address "0.0.0.0", port 5432
2019-08-03 18:12:01.108 UTC [1]
LOG:  listening on IPv6 address "::", port 5432
2019-08-03 18:12:01.114 UTC [1]
LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2019-08-03 18:12:01.135 UTC [50]
LOG:  database system was shut down at 2019-08-03 18:12:01 UTC
2019-08-03 18:12:01.141 UTC [1]
LOG:  database system is ready to accept connections
```

创建`wordcount`数据库和角色：

```py
$ kubectl exec -it db-6b4fbb57d9-8h978 -- psql -U postgres
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

postgres=# create database wordcount;
CREATE DATABASE
postgres=# \q

$ kubectl exec -it db-6b4fbb57d9-8h978 -- psql -U postgres wordcount
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

wordcount=# CREATE ROLE wordcount_dbadmin;
CREATE ROLE
wordcount=# ALTER ROLE wordcount_dbadmin LOGIN;
ALTER ROLE
wordcount=# ALTER USER wordcount_dbadmin PASSWORD 'MYNEWPASS';
ALTER ROLE
wordcount=# \q
```

创建`db`服务：

```py
$ kubectl create -f db-service.yaml
service/db created
```

```py
$ kubectl describe service db
Name:              db
Namespace:         default
Labels:            io.kompose.service=db
Annotations:       kompose.cmd: kompose convert
                   kompose.version: 1.16.0 (0c01309)
Selector:          io.kompose.service=db
Type:              ClusterIP
IP:                10.59.241.181
Port:              5432  5432/TCP
TargetPort:        5432/TCP
Endpoints:         10.56.2.5:5432
Session Affinity:  None
Events:            <none>
```

根据数据库密码的`base64`值创建 Secret 对象。密码的明文值存储在使用`sops`加密的文件中：

```py
$ echo MYNEWPASS | base64
MYNEWPASSBASE64

$ sops secrets.yaml.enc

apiVersion: v1
kind: Secret
metadata:
  name: fbe-secret
type: Opaque
data:
  dbpass: MYNEWPASSBASE64

$ sops -d secrets.yaml.enc | kubectl create -f -
secret/fbe-secret created

kubectl describe secret fbe-secret
Name:         fbe-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
===
dbpass:  21 bytes
```

创建另一个表示 Docker Hub 凭据的 Secret 对象：

```py
$ sops -d create_docker_credentials_secret.sh.enc | bash -
secret/myregistrykey created
```

考虑到正在考虑的情景是将应用程序部署到 GKE 的生产类型部署，将*worker-deployment.yaml*中的`replicas`设置为`3`以确保始终运行三个 worker Pod：

```py
$ kubectl create -f worker-deployment.yaml
deployment.extensions/worker created
```

确保有三个 worker Pod 正在运行：

```py
$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running   0          39m
db-6b4fbb57d9-8h978              1/1     Running   0          16m
redis-9946db5cc-8g6zz            1/1     Running   0          34m
worker-8cf5dc699-98z99           1/1     Running   0          35s
worker-8cf5dc699-9s26v           1/1     Running   0          35s
worker-8cf5dc699-v6ckr           1/1     Running   0          35s

$ kubectl logs worker-8cf5dc699-98z99
18:28:08 RQ worker 'rq:worker:1355d2cad49646e4953c6b4d978571f1' started,
 version 1.0
18:28:08 *** Listening on default...
```

类似地，在*app-deployment.yaml*中将`replicas`设置为两个：

```py
$ kubectl create -f app-deployment.yaml
deployment.extensions/app created
```

确保有两个应用程序 Pod 正在运行：

```py
$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
app-7964cff98f-5bx4s             1/1     Running   0          54s
app-7964cff98f-8n8hk             1/1     Running   0          54s
canary-aqw8jtfo-f54b9749-q5wqj   1/1     Running   0          41m
db-6b4fbb57d9-8h978              1/1     Running   0          19m
redis-9946db5cc-8g6zz            1/1     Running   0          36m
worker-8cf5dc699-98z99           1/1     Running   0          2m44s
worker-8cf5dc699-9s26v           1/1     Running   0          2m44s
worker-8cf5dc699-v6ckr           1/1     Running   0          2m44s
```

创建`app`服务：

```py
$ kubectl create -f app-service.yaml
service/app created
```

注意到创建了一个类型为 LoadBalancer 的服务：

```py
$ kubectl describe service app
Name:                     app
Namespace:                default
Labels:                   io.kompose.service=app
Annotations:              kompose.cmd: kompose convert
                          kompose.version: 1.16.0 (0c01309)
Selector:                 io.kompose.service=app
Type:                     LoadBalancer
IP:                       10.59.255.31
LoadBalancer Ingress:     34.83.242.171
Port:                     5000  5000/TCP
TargetPort:               5000/TCP
NodePort:                 5000  31305/TCP
Endpoints:                10.56.1.6:5000,10.56.2.12:5000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
Type    Reason                Age   From                Message
----    ------                ----  ----                -------
Normal  EnsuringLoadBalancer  72s   service-controller  Ensuring load balancer
Normal  EnsuredLoadBalancer   33s   service-controller  Ensured load balancer
```

测试应用程序，访问基于`LoadBalancer Ingress`对应的 IP 地址的端点 URL：[*http://34.83.242.171:5000*](http://34.83.242.171:5000)。

我们演示了如何从原始 Kubernetes 清单文件创建 Kubernetes 对象（如 Deployments、Services 和 Secrets）。随着应用程序变得更加复杂，此方法的局限性开始显现，因为定制这些文件以适应不同环境（例如，分阶段、集成和生产环境）将变得更加困难。每个环境都将有其自己的环境值和秘密，您需要跟踪这些内容。一般而言，跟踪在特定时间安装了哪些清单将变得越来越复杂。Kubernetes 生态系统中存在许多解决此问题的方案，其中最常见的之一是使用[Helm](https://oreil.ly/duKVw)包管理器。把 Helm 视为 `yum` 和 `apt` 包管理器的 Kubernetes 等价物。

下一节将展示如何使用 Helm 在 GKE 集群内安装和自定义 Prometheus 和 Grafana。

# 安装 Prometheus 和 Grafana Helm Charts

在当前版本（截至本文撰写时为 v2），Helm 具有一个名为 Tiller 的服务器端组件，需要在 Kubernetes 集群内具有特定权限。

为 Tiller 创建一个新的 Kubernetes Service Account，并赋予适当的权限：

```py
$ kubectl -n kube-system create sa tiller

$ kubectl create clusterrolebinding tiller \
  --clusterrole cluster-admin \
  --serviceaccount=kube-system:tiller

$ kubectl patch deploy --namespace kube-system \
tiller-deploy -p  '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

从官方[Helm 发布](https://oreil.ly/sPwDO)页面下载并安装适用于您操作系统的 Helm 二进制文件，然后使用`helm init`命令安装 Tiller：

```py
$ helm init
```

创建一个名为 `monitoring` 的命名空间：

```py
$ kubectl create namespace monitoring
namespace/monitoring created
```

在 `monitoring` 命名空间中安装 Prometheus [Helm chart](https://oreil.ly/CSaSo)：

```py
$ helm install --name prometheus --namespace monitoring stable/prometheus
NAME:   prometheus
LAST DEPLOYED: Tue Aug 27 12:59:40 2019
NAMESPACE: monitoring
STATUS: DEPLOYED
```

列出 `monitoring` 命名空间中的 pods、services 和 configmaps：

```py
$ kubectl get pods -nmonitoring
NAME                                             READY   STATUS    RESTARTS AGE
prometheus-alertmanager-df57f6df6-4b8lv          2/2     Running   0        3m
prometheus-kube-state-metrics-564564f799-t6qdm   1/1     Running   0        3m
prometheus-node-exporter-b4sb9                   1/1     Running   0        3m
prometheus-node-exporter-n4z2g                   1/1     Running   0        3m
prometheus-node-exporter-w7hn7                   1/1     Running   0        3m
prometheus-pushgateway-56b65bcf5f-whx5t          1/1     Running   0        3m
prometheus-server-7555945646-d86gn               2/2     Running   0        3m

$ kubectl get services -nmonitoring
NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP  PORT(S)
   AGE
prometheus-alertmanager         ClusterIP   10.0.6.98     <none>       80/TCP
   3m51s
prometheus-kube-state-metrics   ClusterIP   None          <none>       80/TCP
   3m51s
prometheus-node-exporter        ClusterIP   None          <none>       9100/TCP
   3m51s
prometheus-pushgateway          ClusterIP   10.0.13.216   <none>       9091/TCP
   3m51s
prometheus-server               ClusterIP   10.0.4.74     <none>       80/TCP
   3m51s

$ kubectl get configmaps -nmonitoring
NAME                      DATA   AGE
prometheus-alertmanager   1      3m58s
prometheus-server         3      3m58s
```

通过`kubectl port-forward`命令连接到 Prometheus UI：

```py
$ export PROMETHEUS_POD_NAME=$(kubectl get pods --namespace monitoring \
-l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")

$ echo $PROMETHEUS_POD_NAME
prometheus-server-7555945646-d86gn

$ kubectl --namespace monitoring port-forward $PROMETHEUS_POD_NAME 9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
Handling connection for 9090
```

在浏览器中转到 localhost:9090 并查看 Prometheus UI。

在 `monitoring` 命名空间中安装 Grafana [Helm chart](https://oreil.ly/--wEN)：

```py
$ helm install --name grafana --namespace monitoring stable/grafana
NAME:   grafana
LAST DEPLOYED: Tue Aug 27 13:10:02 2019
NAMESPACE: monitoring
STATUS: DEPLOYED
```

列出 `monitoring` 命名空间中与 Grafana 相关的 pods、services、configmaps 和 secrets：

```py
$ kubectl get pods -nmonitoring | grep grafana
grafana-84b887cf4d-wplcr                         1/1     Running   0

$ kubectl get services -nmonitoring | grep grafana
grafana                         ClusterIP   10.0.5.154    <none>        80/TCP

$ kubectl get configmaps -nmonitoring | grep grafana
grafana                   1      99s
grafana-test              1      99s

$ kubectl get secrets -nmonitoring | grep grafana
grafana                                     Opaque
grafana-test-token-85x4x                    kubernetes.io/service-account-token
grafana-token-jw2qg                         kubernetes.io/service-account-token
```

检索 Grafana Web UI 中 `admin` 用户的密码：

```py
$ kubectl get secret --namespace monitoring grafana \
-o jsonpath="{.data.admin-password}" | base64 --decode ; echo

SOMESECRETTEXT
```

通过`kubectl port-forward`命令连接到 Grafana UI：

```py
$ export GRAFANA_POD_NAME=$(kubectl get pods --namespace monitoring \
-l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")

$ kubectl --namespace monitoring port-forward $GRAFANA_POD_NAME 3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

在浏览器中转到 localhost:3000 并查看 Grafana UI。使用上述检索到的密码以 admin 用户身份登录。

使用 `helm list` 列出当前已安装的 charts。安装 chart 后，当前的安装称为 “Helm release”：

```py
$ helm list
NAME        REVISION  UPDATED                   STATUS    CHART
    APP VERSION NAMESPACE
grafana     1         Tue Aug 27 13:10:02 2019  DEPLOYED  grafana-3.8.3
    6.2.5       monitoring
prometheus. 1         Tue Aug 27 12:59:40 2019  DEPLOYED  prometheus-9.1.0
    2.11.1      monitoring

```

大多数情况下，您需要自定义一个 Helm chart。如果您从本地文件系统下载 chart 并使用 `helm` 安装，则会更容易进行此操作。

使用 `helm fetch` 命令获取最新稳定版本的 Prometheus 和 Grafana Helm charts，该命令将下载这些 chart 的 `tgz` 归档文件：

```py
$ mkdir charts
$ cd charts
$ helm fetch stable/prometheus
$ helm fetch stable/grafana
$ ls -la
total 80
drwxr-xr-x   4 ggheo  staff    128 Aug 27 13:59 .
drwxr-xr-x  15 ggheo  staff    480 Aug 27 13:55 ..
-rw-r--r--   1 ggheo  staff  16195 Aug 27 13:55 grafana-3.8.3.tgz
-rw-r--r--   1 ggheo  staff  23481 Aug 27 13:54 prometheus-9.1.0.tgz
```

解压 `tgz` 文件，然后删除它们：

```py
$ tar xfz prometheus-9.1.0.tgz; rm prometheus-9.1.0.tgz
$ tar xfz grafana-3.8.3.tgz; rm grafana-3.8.3.tgz
```

默认情况下，模板化的 Kubernetes 清单存储在 chart 目录下名为 *templates* 的目录中，因此在此例中，这些位置将分别是 *prometheus/templates* 和 *grafana/templates*。给定 chart 的配置值在 chart 目录下的 *values.yaml* 文件中声明。

作为 Helm 图表定制的示例，让我们向 Grafana 添加一个持久卷，这样当重启 Grafana pods 时就不会丢失数据。

修改文件 *grafana/values.yaml*，并在该部分将 `persistence` 父键下的 `enabled` 子键的值设置为 `true`（默认为 `false`）。

```py
## Enable persistence using Persistent Volume Claims
## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
##
persistence:
  enabled: true
  # storageClassName: default
  accessModes:
    - ReadWriteOnce
  size: 10Gi
  # annotations: {}
  finalizers:
    - kubernetes.io/pvc-protection
  # subPath: ""
  # existingClaim:
```

使用 `helm upgrade` 命令来升级现有的 `grafana` Helm 发布。命令的最后一个参数是包含图表的本地目录的名称。在 *grafana* 图表目录的父目录中运行此命令：

```py
$ helm upgrade grafana grafana/
Release "grafana" has been upgraded. Happy Helming!
```

验证在 `monitoring` 命名空间中为 Grafana 创建了 PVC：

```py
kubectl describe pvc grafana -nmonitoring
Name:        grafana
Namespace:   monitoring
StorageClass:standard
Status:      Bound
Volume:      pvc-31d47393-c910-11e9-87c5-42010a8a0021
Labels:      app=grafana
             chart=grafana-3.8.3
             heritage=Tiller
             release=grafana
Annotations: pv.kubernetes.io/bind-completed: yes
             pv.kubernetes.io/bound-by-controller: yes
             volume.beta.kubernetes.io/storage-provisioner:kubernetes.io/gce-pd
Finalizers:  [kubernetes.io/pvc-protection]
Capacity:    10Gi
Access Modes:RWO
Mounted By:  grafana-84f79d5c45-zlqz8
Events:
Type    Reason                 Age   From                         Message
----    ------                 ----  ----                         -------
Normal  ProvisioningSucceeded  88s   persistentvolume-controller  Successfully
provisioned volume pvc-31d47393-c910-11e9-87c5-42010a8a0021
using kubernetes.io/gce-pd
```

另一个 Helm 图表定制的示例是修改 Prometheus 中存储数据的默认保留期，从 15 天改为其他。

在 *prometheus/values.yaml* 文件中将 `retention` 值更改为 30 天：

```py
  ## Prometheus data retention period (default if not specified is 15 days)
  ##
  retention: "30d"
```

通过运行 `helm upgrade` 命令来升级现有的 Prometheus Helm 发布。在 *prometheus* 图表目录的父目录中运行此命令：

```py
$ helm upgrade prometheus prometheus
Release "prometheus" has been upgraded. Happy Helming!
```

验证保留期已更改为 30 天。运行 `kubectl describe` 命令针对 `monitoring` 命名空间中运行的 Prometheus pod，并查看输出的 `Args` 部分：

```py
$ kubectl get pods -nmonitoring
NAME                                            READY   STATUS   RESTARTS   AGE
grafana-84f79d5c45-zlqz8                        1/1     Running  0          9m
prometheus-alertmanager-df57f6df6-4b8lv         2/2     Running  0          87m
prometheus-kube-state-metrics-564564f799-t6qdm  1/1     Running  0          87m
prometheus-node-exporter-b4sb9                  1/1     Running  0          87m
prometheus-node-exporter-n4z2g                  1/1     Running  0          87m
prometheus-node-exporter-w7hn7                  1/1     Running  0          87m
prometheus-pushgateway-56b65bcf5f-whx5t         1/1     Running  0          87m
prometheus-server-779ffd445f-4llqr              2/2     Running  0          3m

$ kubectl describe pod prometheus-server-779ffd445f-4llqr -nmonitoring
OUTPUT OMITTED
      Args:
      --storage.tsdb.retention.time=30d
      --config.file=/etc/config/prometheus.yml
      --storage.tsdb.path=/data
      --web.console.libraries=/etc/prometheus/console_libraries
      --web.console.templates=/etc/prometheus/consoles
      --web.enable-lifecycle
```

# 销毁 GKE 集群

如果不再需要，记得删除用于测试目的的任何云资源，因为这真的很“贵”。否则，月底收到云服务提供商的账单时，可能会有不愉快的惊喜。

通过 `pulumi destroy` 销毁 GKE 集群：

```py
$ pulumi destroy

Previewing destroy (dev):

     Type                            Name                            Plan
 -   pulumi:pulumi:Stack             pythonfordevops-gke-pulumi-dev  delete
 -   ├─ kubernetes:core:Service      ingress                         delete
 -   ├─ kubernetes:apps:Deployment   canary                          delete
 -   ├─ pulumi:providers:kubernetes  gke_k8s                         delete
 -   ├─ gcp:container:Cluster        gke-cluster                     delete
 -   └─ random:index:RandomString    password                        delete

Resources:
    - 6 to delete

Do you want to perform this destroy? yes
Destroying (dev):

     Type                            Name                            Status
 -   pulumi:pulumi:Stack             pythonfordevops-gke-pulumi-dev  deleted
 -   ├─ kubernetes:core:Service      ingress                         deleted
 -   ├─ kubernetes:apps:Deployment   canary                          deleted
 -   ├─ pulumi:providers:kubernetes  gke_k8s                         deleted
 -   ├─ gcp:container:Cluster        gke-cluster                     deleted
 -   └─ random:index:RandomString    password                        deleted

Resources:
    - 6 deleted

Duration: 3m18s
```

# 练习

+   在 GKE 中不再运行 PostgreSQL 的 Docker 容器，而是使用 Google Cloud SQL for PostgreSQL。

+   使用 [AWS 云开发工具包](https://aws.amazon.com/cdk) 来启动 Amazon EKS 集群，并将示例应用部署到该集群中。

+   在 EKS 中不再运行 PostgreSQL 的 Docker 容器，而是使用 Amazon RDS PostgreSQL。

+   尝试使用 [Kustomize](https://oreil.ly/ie9n6) 作为管理 Kubernetes 清单 YAML 文件的 Helm 替代方案。
