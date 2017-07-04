<properties
    pageTitle="Azure CLI 脚本示例 - 使用批处理运行作业 | Azure"
    description="Azure CLI 脚本示例 - 使用批处理运行作业"
    services="batch"
    documentationcenter=""
    author="annatisch"
    manager="daryls"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="batch"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="multiple"
    ms.workload="na"
    ms.date="03/20/2017"
    wacn.date="05/15/2017"
    ms.author="antisch"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="7be8a02df7df7373cf477ea619dc4d1f0c44809a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="running-jobs-on-azure-batch-with-azure-cli"></a>使用 Azure CLI 在 Azure 批处理上运行作业

此脚本将创建一个批处理作业，并将一系列任务添加到该作业。 它还演示了如何监视作业及其任务。
运行此脚本时假定已设置批处理帐户，并且已配置池和应用程序。 有关详细信息，请参阅涵盖上述每个主题的[示例脚本](/documentation/articles/batch-cli-samples/)。

如果需要，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 登录到 Azure。

`az login` 默认是登录到 azure global 环境，要登录到 azure China 环境，请先运行 `az configure` 命令，根据提示去修改相应的文件才可以。 Windows 环境下需要修改 `C:\\Users\\userName\\.azure\\config` 文件，将 `name = AzureCloud` 改为 `name = AzureChinaCloud`。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Authenticate Batch account CLI session.
    az batch account login -g myresource group -n mybatchaccount

    # Create a new job to encapsulate the tasks that we want to add.
    # We'll assume a pool has already been created with the ID 'mypool' - for more information
    # see the sample script for managing pools.
    az batch job create --id myjob --pool-id mypool

    # Now we will add tasks to the job.
    # We'll assume an application package has already been uploaded with the ID 'myapp' - for
    # more information see the sample script for adding applications.
    az batch task create \
        --job-id myjob \
        --id task1 \
        --application-package-references myapp#1.0
        --command-line "cmd /c %AZ_BATCH_APP_PACKAGE_MYAPP#1.0%\\myapp.exe"

    # If we want to add many tasks at once - this can be done by specifying the tasks
    # in a JSON file, and passing it into the command. See tasks.json for formatting.
    az batch task create --job-id myjob --json-file tasks.json

    # Now that all the tasks are added - we can update the job so that it will automatically
    # be marked as completed once all the tasks are finished.
    az batch job set --on-all-tasks-complete terminateJob

    # Monitor the status of the job.
    az batch job show --job-id myjob

    # Monitor the status of a task.
    az batch task show --task-id task1

## <a name="clean-up-job"></a>清除作业

运行上述示例脚本后，可运行以下命令以删除作业及其所有任务。 请注意，将需要单独删除池；请参阅[有关管理池的教程](/documentation/articles/batch-cli-sample-manage-pool/)。

    az batch job delete --job-id myjob

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建批处理作业和任务。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az batch account login](https://docs.microsoft.com/zh-cn/cli/azure/batch/account#login) | 针对批处理帐户进行身份验证。  |
| [az batch job create](https://docs.microsoft.com/zh-cn/cli/azure/batch/job#create) | 创建批处理作业。  |
| [az batch job set](https://docs.microsoft.com/zh-cn/cli/azure/batch/job#set) | 更新批处理作业的属性。  |
| [az batch job show](https://docs.microsoft.com/zh-cn/cli/azure/batch/job#show) | 检索指定批处理作业的详细信息。  |
| [az batch task create](https://docs.microsoft.com/zh-cn/cli/azure/batch/task#create) | 将任务添加到指定的批处理作业。  |
| [az batch task show](https://docs.microsoft.com/zh-cn/cli/azure/batch/task#show) | 从指定的批处理作业中检索任务的详细信息。  |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 批处理 CLI 文档](/documentation/articles/batch-cli-samples/)中找到其他批处理 CLI 脚本示例。

