# 在本地运行 Airflow
本快速入门指南将帮助您在本地计算机上运行 Airflow 独立实例。

:warning: **注意**
```
目前官方只支持pip安装。

虽然使用 poetry 或 pip-tools 等其他工具取得了成功，但它们与 pip 不共享相同的工作流程——尤其是在约束与需求管理方面。目前不支持通过 Poetry 或 pip-tools 安装。

如果您希望使用这些工具安装Airflow，您应该使用约束文件并将它们转换为您的工具所需的适当格式和工作流程。
```

如果您遵循以下说明，Airflow 的安装将是轻松的。 Airflow 使用约束文件来实现可重复安装，因此建议使用 pip 和约束文件。

```
# Airflow needs a home. `~/airflow` is the default, but you can put it
# somewhere else if you prefer (optional)
export AIRFLOW_HOME=~/airflow

# Install Airflow using the constraints file
AIRFLOW_VERSION=2.2.1
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
# For example: 3.6
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
# For example: https://raw.githubusercontent.com/apache/airflow/constraints-2.2.1/constraints-3.6.txt
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# The Standalone command will initialise the database, make a user,
# and start all components for you.
airflow standalone

# Visit localhost:8080 in the browser and use the admin account details
# shown on the terminal to login.
# Enable the example_bash_operator dag in the home page
```

运行这些命令后，Airflow 将创建 `$AIRFLOW_HOME` 文件夹并使用默认值创建“airflow.cfg”文件，这将使您快速上手。您可以在 `$AIRFLOW_HOME/airflow.cfg` 中或通过 `Admin->Configuration` 菜单中的 UI 检查该文件。 webserver 的 PID 文件将存储在 `$AIRFLOW_HOME/airflow-webserver.pid` 或 `/run/airflow/webserver.pid` 如果由 systemd 启动。

开箱即用，Airflow 使用 SQLite 数据库，由于使用此数据库后端无法进行并行化，因此您应该很快就会淘汰该数据库。它与仅按顺序运行任务实例的 SequentialExecutor 结合使用。虽然这是非常有限的，但它允许您快速启动和运行，并浏览 UI 和命令行实用程序。

随着您的成长和将 Airflow 部署到生产环境，您还需要摆脱我们在此处使用的独立命令来单独运行组件。您可以在[生产部署](https://airflow.apache.org/docs/apache-airflow/stable/production-deployment.html)中阅读更多内容。

以下是一些将触发一些任务实例的命令。运行以下命令时，您应该能够在 `example_bash_operator` DAG 中看到作业的状态更改。

```
# run your first task instance
airflow tasks run example_bash_operator runme_0 2015-01-01
# run a backfill over 2 days
airflow dags backfill example_bash_operator \
    --start-date 2015-01-01 \
    --end-date 2015-01-02
```

如果您想手动运行 Airflow 的各个部分而不是使用多合一的独立命令，您可以运行：
```
airflow db init

airflow users create \
    --username admin \
    --firstname Peter \
    --lastname Parker \
    --role Admin \
    --email spiderman@superhero.org

airflow webserver --port 8080

airflow scheduler
```

# 下一步是什么？
从这一点开始，如果您准备好动手，您可以前往教程部分获取更多示例或操作指南部分。