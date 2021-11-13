# 在Docker中运行 Airflow

本快速入门指南将允许您在 Docker 中使用 CeleryExecutor 快速启动 Airflow。这是启动 Airflow 的最快方式。

## 生产准备

:warning: **警告**

不要指望下面的 Docker Compose 足以使用它运行生产就绪的 Docker Compose Airflow 安装。这是真正的`quick-start` docker-compose，让您可以在本地启动和运行 Airflow，并使用 Airflow 弄脏您的手。配置准备用于生产的 Docker-Compose 安装需要具备 Docker Compose 的内在知识、大量定制，甚至可能从头开始编写满足您需求的 Docker Compose 文件。如果您想运行基于 Docker Compose 的部署，这可能没问题，但如果不能成为 Docker Compose 专家，您就不太可能使用它进行健壮的部署。

如果你想获得一个易于配置的基于 Docker 的部署，由 Airflow 社区开发、支持并可以提供部署支持，你应该考虑使用 Kubernetes 并使用官方 [Airflow 社区 Helm Chart](https://airflow.apache.org/docs/helm-chart/stable/index.html)部署 Airflow 。


## 在你开始之前

按照以下步骤安装必要的工具。

在您的工作站上安装Docker 社区版 (CE)。根据操作系统，您可能需要将 Docker 实例配置为使用 4.00 GB 内存让所有容器正常运行。如果使用Docker for Windows或Docker for Mac，请参阅参考资料部分了解更多信息。

在您的工作站上安装Docker Compose v1.29.1 及更新版本。

旧版本`docker-compose`不支持`docker-compose.yaml`文件所需的所有功能，因此请仔细检查您的版本是否满足最低版本要求。

:warning: **警告**

MacOS 上 Docker 可用的默认内存量通常不足以启动和运行 Airflow。如果没有分配足够的内存，可能会导致airflow webserver 不断重启。您应该至少为 Docker 引擎分配 4GB 内存（最好是 8GB）。您可以在资源中检查和更改内存量

您还可以通过运行以下命令来检查您是否有足够的内存：
```
docker run --rm "debian:buster-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'
```

## docker-compose.yaml

要在 Docker Compose 上部署 Airflow，您应该获取[docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml)。
```
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.2.1/docker-compose.yaml'
```

该文件包含几个服务定义：
- airflow-scheduler-调度程序监视所有任务和 DAG，然后在它们的依赖关系完成后触发任务实例。

- airflow-webserver- 网络服务器位于http://localhost:8080。

- airflow-worker - 执行调度程序给出的任务的工人。

- airflow-init - 初始化服务。

flower-用于监测环境的应用程序。它可以在http://localhost:5555。

- postgres - 数据库。

- redis- redis - 将消息从调度程序转发到工作线程的代理。

所有这些服务都允许您使用[CeleryExecutor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html)运行 Airflow 。有关更多信息，请参阅架构概述。

容器中的某些目录是挂载的，这意味着它们的内容在您的计算机和容器之间同步。

- ./dags - 你可以把你的 DAG 文件放在这里。

- ./logs - 包含来自任务执行和调度程序的日志。

- ./plugins- 您可以将自定义插件放在这里。

此文件使用最新的 Airflow 映像 ( apache/airflow )。如果您需要安装新的 Python 库或系统库，您可以构建您的镜像。

## 使用自定义镜像

当您想在本地运行 Airflow 时，您可能需要使用包含一些附加依赖项的扩展映像 - 例如，您可能会添加新的 Python 包，或将气流提供程序升级到更高版本。这可以通过将自定义 Dockerfile 放在您的`docker-compose.yaml`. 然后你可以使用命令来构建你的图像（你只需要做一次）。您还可以将标志添加到您的命令中，以便在运行其他命令时即时重建图像。`docker-compose build--builddocker-composedocker-compose`

可以在构建映像中找到如何使用自定义提供程序、python 包、apt 包等扩展映像的示例。

## 初始化环境

在第一次启动 Airflow 之前，您需要准备好您的环境，即创建必要的文件、目录并初始化数据库。

### 设置正确的 Airflow 用户

在Linux 上，快速入门需要知道您的主机用户 ID 并且需要将组 ID 设置为0。否则，文件创建的`dags`，`logs`并且`plugins`将与创建`root`用户。您必须确保为 `docker-compose` 配置它们：
```
mkdir -p ./dags ./logs ./plugins
echo -e "AIRFLOW_UID=$(id -u)" > .env
```
请参阅Docker Compose 环境变量

### 初始化数据库

在所有操作系统上，您都需要运行数据库迁移并创建第一个用户帐户。要做到这一点，运行。
```
docker-compose up airflow-init
```
初始化完成后，您应该会看到如下所示的消息。

```
airflow-init_1 | 升级完成
airflow-init_1 | 已创建管理员用户气流
airflow-init_1 | 2.2.1
start_airflow-init_1 退出，代码为 0
```
创建的帐户具有登录名`airflow`和密码`airflow`。

## 清理环境

我们准备的 docker-compose 是一个“快速入门”。它不打算在生产中使用，它有许多警告 - 其中之一是从任何问题中恢复的最佳方法是清理它并从头开始。

最好的方法是：

- 在下载文件的目录中运行命令 docker-compose down --volumes --remove-orphansdocker-compose.yaml

- 删除下载docker-compose.yaml文件 的整个目录rm -rf '<DIRECTORY>'

- 重新下载docker-compose.yaml文件

- 按照本指南开头的说明重新开始

## 运行airflow

现在您可以启动所有服务：

```
docker-compose up
```

在第二个终端中，您可以检查容器的状况并确保没有容器处于不健康状态：

```
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                    PORTS                              NAMES
247ebe6cf87a   apache/airflow:2.2.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-worker_1
ed9b09fc84b1   apache/airflow:2.2.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-scheduler_1
65ac1da2c219   apache/airflow:2.2.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    0.0.0.0:5555->5555/tcp, 8080/tcp   compose_flower_1
7cb1fb603a98   apache/airflow:2.2.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    0.0.0.0:8080->8080/tcp             compose_airflow-webserver_1
74f3bbe506eb   postgres:13            "docker-entrypoint.s…"   18 minutes ago   Up 17 minutes (healthy)   5432/tcp                           compose_postgres_1
0bd6576d23cb   redis:latest           "docker-entrypoint.s…"   10 hours ago     Up 17 minutes (healthy)   0.0.0.0:6379->6379/tcp             compose_redis_1
```

## 访问环境
启动 Airflow 后，您可以通过 3 种方式与其交互；

- 通过运行CLI 命令。

- 通过浏览器使用网络界面。

- 使用REST API。

### 运行 CLI 命令
您也可以运行CLI 命令，但您必须在定义的airflow-*服务之一中执行此操作。例如，要运行，请运行以下命令：`airflow info`

```
docker-compose run airflow-worker airflow info
```

如果您有 Linux 或 Mac OS，您可以让您的工作更轻松，并下载一个可选的包装脚本，让您可以使用更简单的命令运行命令。

```
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.2.1/airflow.sh'
chmod +x airflow.sh
```

现在您可以更轻松地运行命令。

```
./airflow.sh info
```

您也可以使用bash作为参数在容器中进入交互式 bash shell 或python进入 python 容器。

```
./airflow.sh bash
./airflow.sh python
```

### 访问网络界面
集群启动后，您可以登录 Web 界面并尝试运行一些任务。

网络服务器位于：http://localhost:8080。默认帐户具有登录名`airflow`和密码`airflow`。

向 REST API 发送请求

### 向 REST API 发送请求
REST API 目前支持基本用户名密码认证，这意味着您可以使用常用工具向 API 发送请求。

网络服务器位于：http://localhost:8080。默认帐户具有登录名airflow和密码airflow。

这是一个示例`curl`命令，它发送请求以检索池列表：

```
ENDPOINT_URL="http://localhost:8080/"
curl -X GET  \
    --user "airflow:airflow" \
    "${ENDPOINT_URL}/api/v1/pools"
```

## 清理
要停止和删除容器、删除包含数据库数据的卷并下载图像，请运行：
```
docker-compose down --volumes --rmi all
```

## 常见问题
`ModuleNotFoundError: No module named 'XYZ'`

Docker Compose 文件使用最新的 Airflow 映像 ( apache/airflow )。如果您需要安装新的 Python 库或系统库，您可以对其进行自定义和扩展。

## 下一步是什么？
从这一点开始，如果您准备好动手，您可以前往教程部分获取更多示例或操作指南部分。

## Docker Compose 支持的环境变量
不要将此处的变量名称与构建图像时设置的构建参数混淆。在 `AIRFLOW_UID`构建ARG默认为`50000` 当镜像被建成，因此被“烤”到镜像。另一方面，可以在容器运行时设置以下环境变量，例如使用命令的结果，这允许使用在构建映像时未知的动态主机运行时用户 ID。`id -u`

|  变量   | 描述  | 默认  | 
|  ----  | ----  |----  |
| AIRFLOW_IMAGE_NAME  | 要使用的airflow 镜像。| 2.2.1
| AIRFLOW_UID  | 运行 Airflow 容器的用户 UID。如果要使用非默认的 Airflow UID，则覆盖（例如，当您从主机映射文件夹时，应将其设置为调用结果。更改时，会在容器内创建一个具有该 UID 的用户和 home 名称）使用的设置为 为了共享安装在那里的 Python 库。这是为了实现 OpenShift 兼容性。在任意 Docker 用户中查看更多信息 `id -u default/airflow/home/` | 50000

:warning: **警告**

在 Airflow 2.2 之前，Docker Compose 也有AIRFLOW_GID参数，但它没有提供任何额外的功能——只是增加了混乱——所以它已被删除。

如果您通过 docker compose 尝试/测试 Airflow 安装，这些附加变量很有用。它们不打算在生产中使用，但它们使环境更快地引导首次使用最常见自定义的用户。

|  变量   | 描述  | 默认  | 
|  ----  | ----  |----  |
| _AIRFLOW_WWW_USER_USERNAME | 管理员 UI 帐户的用户名。如果指定了此值，则会自动创建管理 UI 用户。这仅在您想要为试驾运行 Airflow 并想要启动带有嵌入式开发数据库的容器时才有用。| airflow
| _AIRFLOW_WWW_USER_PASSWORD | 管理员 UI 帐户的密码。仅在_AIRFLOW_WWW_USER_USERNAME设置时使用。| airflow
| _PIP_ADDITIONAL_REQUIREMENTS | 如果不为空，气流容器将尝试安装变量中指定的要求。例如：。在 Airflow 图像 2.1.1 及更高版本中可用。`lxml==4.6.3` `charset-normalizer==1.4.1` |   |