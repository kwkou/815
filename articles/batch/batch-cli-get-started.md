<properties
   pageTitle="Azure Batch CLI 入门 | Azure"
   description="Azure CLI 中用于管理 Azure Batch 服务资源的 Batch 命令简介"
   services="batch"
   documentationCenter=""
   authors="mmacy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="batch"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="multiple"
   ms.workload="big-compute"
   ms.date="09/06/2016"
   wacn.date="11/28/2016"
   ms.author="marsma"/>  


# Azure Batch CLI 入门

使用跨平台 Azure 命令行接口 (Azure CLI) 可以在 Linux、Mac 和 Windows 命令 shell 中管理 Batch 帐户和资源，例如池、作业和任务。使用 Azure CLI，可以执行许多与通过 Batch API、Azure 门户预览和 Batch PowerShell cmdlet 执行的相同任务并为其编写脚本。

本文基于 Azure CLI 版本 0.10.3。

## 先决条件

* [安装 Azure CLI](/documentation/articles/xplat-cli-install/)

* [将 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)

* 切换到 **资源管理器模式**：`azure config mode arm`

>[AZURE.TIP] 建议经常更新 Azure CLI 安装，利用服务更新和增强功能。

## 命令帮助

在命令后面追加 `-h` 作为唯一选项，可以在 Azure CLI 中显示每个命令的帮助文本。例如：

* 若要获取 `azure` 命令的帮助，请输入：`azure -h`
* 若要获取 CLI 中所有 Batch 命令的列表，请使用：`azure batch -h`
* 若要获取有关创建 Batch 帐户的帮助，请输入：`azure batch account create -h`

如有疑问，请使用 `-h` 命令行选项获取有关任何 Azure CLI 命令的帮助。

## 创建批处理帐户

用法：

    azure batch account create [options] <name>

示例：

	azure batch account create --location "China North"  --resource-group "resgroup001" "batchaccount001"

使用指定的参数创建新的 Batch 帐户。必须至少指定一个位置、资源组和帐户名。如果没有资源组，请运行 `azure group create` 创建资源组，并为 `--location` 选项指定一个 Azure 区域（例如“China North”）。例如：

	azure group create --name "resgroup001" --location "China North"

> [AZURE.NOTE] Batch 帐户名必须是创建帐户的 Azure 区域内的唯一名称。它只能包含小写字母数字字符，且长度必须为 3-24 个字符。不能在 Batch 帐户名中使用 `-` 或 `_` 等特殊字符。

<a name="linked-storage-account-autostorage"></a>
### 链接存储帐户（自动存储）

（可选）在创建 Batch 帐户时，可以将**常规用途**存储帐户链接到该帐户。与 [Batch 文件约定 .NET](/documentation/articles/batch-task-output/)库一样，Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能在链接的常规用途存储帐户中使用 Blob 存储。这些可选功能可帮助部署 Batch 任务运行的应用程序，以及保存它们生成的数据。

若要在创建新 Batch 帐户时将现有 Azure 存储帐户链接到该帐户，请指定 `--autostorage-account-id` 选项。此选项需要存储帐户的完全限定资源 ID。

首先，显示存储帐户的详细信息：

    azure storage account show --resource-group "resgroup001" "storageaccount001"

然后，为 `--autostorage-account-id` 选项使用 **Url** 值。Url 值以 "/subscriptions/" 开头，包含订阅 ID 和存储帐户的资源路径：

    azure batch account create --location "China North"  --resource-group "resgroup001" --autostorage-account-id "/subscriptions/8ffffff8-4444-4444-bfbf-8ffffff84444/resourceGroups/resgroup001/providers/Microsoft.Storage/storageAccounts/storageaccount001" "batchaccount001"

## 删除批处理帐户

用法：

    azure batch account delete [options] <name>

示例：

	azure batch account delete --resource-group "resgroup001" "batchaccount001"

删除指定的 Batch 帐户。出现提示时，请确认删除帐户（删除帐户可能需要一段时间才能完成）。

## 管理帐户访问密钥

在 Batch 帐户中[创建和修改资源](#create-and-modify-batch-resources)时需要使用访问密钥。

### 列出访问密钥

用法：

	azure batch account keys list [options] <name>

示例：

	azure batch account keys list --resource-group "resgroup001" "batchaccount001"

列出给定 Batch 帐户的帐户密钥。

### 生成新的访问密钥

用法：

    azure batch account keys renew [options] --<primary|secondary> <name>

示例：

	azure batch account keys renew --resource-group "resgroup001" --primary "batchaccount001"

为给定的 Batch 帐户重新生成指定的帐户密钥。

<a name="create-and-modify-batch-resources"></a>
## 创建和修改 Batch 资源

可以使用 Azure CLI 创建、读取、更新和删除 (CRUD) Batch 资源，例如池、计算节点、作业和任务。这些 CRUD 操作需要 Batch 帐户名、访问密钥和终结点。可以使用 `-a`、`-k` 和 `-u` 选项指定这些对象，或者设置 CLI 自动使用的[环境变量](#credential-environment-variables)（如果已填充）。

<a name="credential-environment-variables"></a>
### 凭据环境变量

可以设置 `AZURE_BATCH_ACCOUNT`、`AZURE_BATCH_ACCESS_KEY` 和 `AZURE_BATCH_ENDPOINT` 环境变量，而无需每次执行命令时在命令行上指定 `-a`、`-k` 和 `-u` 选项。Batch CLI 将使用这些变量（如果已设置），因此可以省略 `-a`、`-k` 和 `-u` 选项。本文的余下部分假设使用这些环境变量。

>[AZURE.TIP] 使用 `azure batch account keys list` 列出密钥，使用 `azure batch account show` 显示帐户的终结点。

<a name="json-files"></a>
### JSON 文件

创建 Batch 资源（如池和作业）时，可以指定包含新资源配置的 JSON 文件，而无需将资源的参数作为命令行选项传递。例如：

`azure batch pool create my_batch_pool.json`  


尽管可以使用命令行选项执行许多资源创建操作，但有些功能需要 JSON 格式的文件（包含资源详细信息）。例如，若要指定启动任务的资源文件，必须使用 JSON 文件。

若要查找创建资源所需的 JSON，请参阅 MSDN 上的 [Batch REST API reference][rest_api]（Batch REST API 参考）文档。每个“Add *resource type*”（添加 <资源类型>）主题都包含用于创建资源的示例 JSON，可将它用作 JSON 文件的模板。例如，在 [Add a pool to an account][rest_add_pool]（将池添加到帐户）中可以找到用于创建池的 JSON。

>[AZURE.NOTE] 如果在创建资源时指定 JSON 文件，则会忽略在命令行上为该资源指定的所有其他参数。

## 创建池

用法：

    azure batch pool create [options] [json-file]

示例（虚拟机配置）：

    azure batch pool create --id "pool001" --target-dedicated 1 --vm-size "STANDARD_A1" --image-publisher "Canonical" --image-offer "UbuntuServer" --image-sku "14.04.2-LTS" --node-agent-id "batch.node.ubuntu 14.04"

示例（云服务配置）：

	azure batch pool create --id "pool002" --target-dedicated 1 --vm-size "small" --os-family "4"

在 Batch 服务中创建计算节点的池。

如 [Batch feature overview](/documentation/articles/batch-api-basics/#pool/)（Batch 功能概述）中所述，为池中的节点选择操作系统时，可以使用两个选项：“虚拟机配置”和“云服务配置”。使用 `--image-*` 选项可创建虚拟机配置池，使用 `--os-family` 可创建云服务配置池。不能同时指定 `--os-family` 和 `--image-*` 选项。

可以指定池[应用程序包](/documentation/articles/batch-application-packages/)以及[启动任务](/documentation/articles/batch-api-basics/#start-task/)的命令行。若要指定启动任务的资源文件，必须改用 [JSON 文件](#json-files)。

使用以下命令删除池：

    azure batch pool delete [pool-id]

>[AZURE.TIP] 在[虚拟机映像列表](/documentation/articles/batch-linux-nodes/#list-of-virtual-machine-images/)中检查适合 `--image-*` 选项的值。

## 创建作业

用法：

    azure batch job create [options] [json-file]

示例：

    azure batch job create --id "job001" --pool-id "pool001"

将作业添加到 Batch 帐户，指定执行其任务的池。

使用以下命令删除作业：

    azure batch job delete [job-id]

## 列出池、作业、任务和其他资源

每个 Batch 资源类型都支持 `list` 命令，该命令可查询 Batch 帐户并列出该类型的资源。例如，可以列出帐户中的池以及作业中的任务：

    azure batch pool list
    azure batch task list --job-id "job001"

### 有效列出资源

为了加快查询，可为 `list` 操作指定 **select**、**filter** 和 **expand** 子句选项。使用这些选项限制 Batch 服务返回的数据量。由于所有筛选发生在服务器端，因此只会返回所需的数据。执行 list 操作时，使用这些子句可以节省带宽（因而节省了时间）。

例如，以下命令只返回 ID 以“renderTask”开头的池：

    azure batch task list --job-id "job001" --filter-clause "startswith(id, 'renderTask')"

Batch CLI 支持 Batch 服务所支持的所有三个子句：

* `--select-clause [select-clause]` 返回每个实体的属性子集
* `--filter-clause [filter-clause]` 返回与指定的 OData 表达式匹配的实体
* `--expand-clause [expand-clause]` 通过一个基础 REST 调用获取实体信息。expand 子句目前仅支持 `stats` 属性。

有关这三个子句以及使用它们执行 list 查询的详细信息，请参阅 [Query the Azure Batch service efficiently](/documentation/articles/batch-efficient-list-queries/)（有效查询 Azure Batch 服务）。

## 应用程序包管理

应用程序包提供将应用程序部署到池中计算节点的简化方式。使用 Azure CLI 可以上载应用程序包、管理包版本，以及删除包。

若要创建新应用程序并添加包版本，请执行以下操作：

**创建**应用程序：

    azure batch application create "resgroup001" "batchaccount001" "MyTaskApplication"

**添加**应用程序包：

    azure batch application package create "resgroup001" "batchaccount001" "MyTaskApplication" "1.10-beta3" package001.zip

**激活**包：

    azure batch application package activate "resgroup002" "azbatch002" "MyTaskApplication" "1.10-beta3" zip

### 部署应用程序包

在创建新池时，可以指定一个或多个要部署的应用程序包。如果创建池时指定包，该包将在节点加入池时部署到每个节点。将节点重新启动或重置映像时，也会部署包。

此命令在创建池时指定包，并在每个节点加入新池时部署该包：

    azure batch pool create --id "pool001" --target-dedicated 1 --vm-size "small" --os-family "4" --app-package-ref "MyTaskApplication"

目前无法使用命令行选项指定要部署的包版本。必须先使用 Azure 门户预览设置应用程序的默认版本，才可以将应用程序分配到池。在 [Application deployment with Azure Batch application packages](/documentation/articles/batch-application-packages/)（使用 Azure Batch 应用程序包部署应用程序）中了解如何设置默认版本。但是，如果在创建池时使用 [JSON 文件](#json-files)而不是命令行选项，则可以指定默认版本。

>[AZURE.IMPORTANT] 若要使用应用程序包，必须[将 Azure 存储帐户链接](#linked-storage-account-autostorage)到 Batch 帐户。

## 故障排除提示

本部分旨在提供排查 Azure CLI 问题时可以使用的资源。这些资源不一定能解决所有问题，但可以帮助缩小排查范围，并提供帮助资源的链接。

* 使用 `-h` 获取任何 CLI 命令的**帮助文本**

* 使用 `-v` 和 `-vv` 显示 **verbose** 命令输出；`-vv` 表示“特别”详细，可显示实际的 REST 请求和响应。使用这些开关可以方便地显示完整的错误输出。

* 可以使用 `--json` 选项查看 **JSON 格式的命令输出**。例如，`azure batch pool show "pool001" --json` 以 JSON 格式显示 pool001 的属性。然后，可以复制并修改此输出，以便在 `--json-file` 中使用（请参阅本文前面的 [JSON 文件](#json-files)）。

* [MSDN 上的 Batch 论坛][batch_forum]是一个极佳的帮助资源，受到 Batch 团队成员的密切监视。如果遇到问题或需要特定操作的帮助，请务必在这里发布问题。

* Azure CLI 目前不一定支持所有的 Batch 资源操作。例如，目前无法指定池的应用程序包*版本*，只能指定包 ID。在这种情况下，可能需要为命令提供 `--json-file`，而不是使用命令行选项。请务必安装最新的 CLI 版本，获取将来所做的增强。

## 后续步骤

*  请参阅 [Application deployment with Azure Batch application packages](/documentation/articles/batch-application-packages/)（使用 Azure Batch 应用程序包部署应用程序），了解如何使用此功能来管理和部署 Batch 计算节点上执行的应用程序。

* 有关如何减少项数以及针对 Batch 查询返回的信息类型的详细信息，请参阅 [Query the Batch service efficiently](/documentation/articles/batch-efficient-list-queries/)（有效查询 Batch 服务）。

[batch_forum]: https://social.msdn.microsoft.com/forums/azure/zh-cn/home?forum=azurebatch
[rest_api]: https://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx
[rest_add_pool]: https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx

<!---HONumber=Mooncake_1107_2016-->
