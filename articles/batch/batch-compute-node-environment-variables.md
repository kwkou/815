<properties
    pageTitle="Azure 批处理计算节点环境变量 | Azure"
    caps.latest.revision="14"
    author="tamram"
    manager="timlt" />
<tags
    ms.custom=""
    ms.date="2017-02-01"
    wacn.date="05/15/2017"
    ms.prod="azure"
    ms.reviewer=""
    ms.service="batch"
    ms.suite=""
    ms.tgt_pltfrm=""
    ms.topic="article"
    ms.assetid="3990f0c8-b627-432f-9551-5ce10f9bb0ca"
    ms.author="tamram"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="dd664e6476f7fc6d49c42da990553addfe733425"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="azure-batch-compute-node-environment-variables"></a>Azure 批处理计算节点环境变量
[Azure 批处理服务](/home/features/batch/)在计算节点上设置以下环境变量。 可以在任务命令行中引用这些环境变量，也可在命令行运行的程序和脚本中引用它们。

有关将环境变量用于批处理的其他信息，请参阅[任务的环境设置](/documentation/articles/batch-api-basics#environment-settings-for-tasks/)。

## <a name="environment-variable-visibility"></a>环境变量的可见性

这些环境变量仅在**任务用户**（即执行任务的节点上的用户帐户）的上下文中可见。 如果通过远程桌面协议 (RDP) 或安全外壳 (SSH) [远程连接](/documentation/articles/batch-api-basics/#connecting-to-compute-nodes/)到计算节点并列出环境变量，将*看不到*这些变量。 这是因为，用于远程连接的用户帐户与任务使用的帐户不同。

## <a name="command-line-expansion-of-environment-variables"></a>环境变量的命令行扩展

任务在计算节点上执行的命令行在 shell 下无法运行。 因此这些命令行无法以本机方式利用 shell 功能，例如环境变量扩展（包括 `PATH`）。 若要利用此类功能，必须在命令行中**调用 shell**。 例如，在 Windows 计算节点上启动 `cmd.exe` 或在 Linux 节点上启动 `/bin/sh`：

`cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

`/bin/sh -c MyTaskApplication $MY_ENV_VAR`

## <a name="environment-variables"></a>环境变量

| 变量名称         | 说明                                                              | 可用性 | 示例 |
|-----------------------------------|--------------------------------------------------------------------------|--------------|---------|
| `AZ_BATCH_ACCOUNT_NAME`           | 任务所属的 Batch 帐户名。 | 所有任务。 | `mybatchaccount` |
| `AZ_BATCH_CERTIFICATES_DIR`       | [任务工作目录][files_dirs]内的目录，会在其中为 Linux 计算节点存储证书。 请注意，此环境变量不适用于 Windows 计算节点。 | 所有任务。 | `/mnt/batch/tasks/workitems/batchjob001/job-1/task001/certs` |
| `AZ_BATCH_JOB_ID`                 | 任务所属的作业的 ID。 | 除启动任务以外的所有任务。 | `batchjob001` |
| `AZ_BATCH_JOB_PREP_DIR`           | 节点上的作业准备[任务目录][files_dirs]的完整路径。 | 除启动任务和作业准备任务之外的所有任务。 仅当使用作业准备任务来配置作业时才适用。 | `C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation` |
| `AZ_BATCH_JOB_PREP_WORKING_DIR`   | 节点上的作业准备[任务工作目录][files_dirs]的完整路径。 | 除启动任务和作业准备任务之外的所有任务。 仅当使用作业准备任务来配置作业时才适用。 | `C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation\wd` |
| `AZ_BATCH_NODE_ID`                | 任务分配到的节点的 ID。 | 所有任务。 | `tvm-1219235766_3-20160919t172711z` |
| `AZ_BATCH_NODE_ROOT_DIR`          | 节点上所有[批处理目录][files_dirs]的根目录的完整路径。 | 所有任务。 | `C:\user\tasks` |
| `AZ_BATCH_NODE_SHARED_DIR`        | 节点上[共享目录][files_dirs]的完整路径。 节点上执行的所有任务具有此目录的读取/写入权限。 在其他节点上执行的任务没有对此目录（它不是“共享”的网络目录）的远程访问权限。 | 所有任务。 | `C:\user\tasks\shared` |
| `AZ_BATCH_NODE_STARTUP_DIR`       | 节点上的[启动任务目录][files_dirs]的完整路径。 | 所有任务。 | `C:\user\tasks\startup` |
| `AZ_BATCH_POOL_ID`                | 运行任务的池的 ID。 | 所有任务。 | `batchpool001` |
| `AZ_BATCH_TASK_DIR`               | 节点上的[任务目录][files_dirs]的完整路径。 此目录包含任务的 `stdout.txt` 和 `stderr.txt`，以及 `AZ_BATCH_TASK_WORKING_DIR`。 | 所有任务。 | `C:\user\tasks\workitems\batchjob001\job-1\task001` |
| `AZ_BATCH_TASK_ID`                | 当前任务的 ID。 | 除启动任务以外的所有任务。 | `task001` |
| `AZ_BATCH_TASK_WORKING_DIR`       | 节点上的[任务工作目录][files_dirs]的完整路径。 当前正在运行的任务具有对此目录的读取/写入权限。 | 所有任务。 | `C:\user\tasks\workitems\batchjob001\job-1\task001\wd` |
| `CCP_NODES`                       | 分配给[多实例任务][multi_instance]的节点和每个节点的内核数的列表。 使用 `numNodes<space>node1IP<space>node1Cores<space>` 格式列出了节点和内核<br/>`node2IP<space>node2Cores<space> ...`，其中节点数后跟一个或多个节点 IP 地址和每个节点的内核数。 |  多实例主要和子任务。 |`2 10.0.0.4 1 10.0.0.5 1` |
| `AZ_BATCH_NODE_LIST`              | 使用 `nodeIP;nodeIP` 格式列出了分配给[多实例任务][multi_instance]的节点的列表。 | 多实例主要和子任务。 | `10.0.0.4;10.0.0.5` |
| `AZ_BATCH_HOST_LIST`              | 使用 `nodeIP,nodeIP` 格式列出了分配给[多实例任务][multi_instance]的节点的列表。 | 多实例主要和子任务。 | `10.0.0.4,10.0.0.5` |
| `AZ_BATCH_MASTER_NODE`            | 在其上运行[多实例任务][multi_instance]的主要任务的计算节点的 IP 地址和端口。 | 多实例主要和子任务。 | `10.0.0.4:6000`|
| `AZ_BATCH_TASK_SHARED_DIR` | [多实例任务][multi_instance]的主要任务和每个子任务的目录路径相同。 路径存在于运行多实例任务的每个节点上，并且在该节点上运行的任务命令（[协调命令][coord_cmd]和[应用程序命令][app_cmd]）对其具有读取/写入权限。 在其他节点上执行的子任务或主要任务不具有对此目录（它不是“共享”的网络目录）的远程访问权限。 | 多实例主要和子任务。 | `C:\user\tasks\workitems\multiinstancesamplejob\job-1\multiinstancesampletask` |
| `AZ_BATCH_IS_CURRENT_NODE_MASTER` | 指定当前节点是否为[多实例任务][multi_instance]的主节点。 可能的值为 `true` 和 `false`。| 多实例主要和子任务。 | `true` |


[files_dirs]:/documentation/articles/batch-api-basics/#files-and-directories/
[multi_instance]: /documentation/articles/batch-mpi/
[coord_cmd]:/documentation/articles/batch-mpi/#coordination-command/
[app_cmd]:/documentation/articles/batch-mpi/#application-command/

