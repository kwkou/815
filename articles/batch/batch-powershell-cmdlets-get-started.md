<properties
    pageTitle="Azure Batch PowerShell 入门 | Azure"
    description="快速介绍可用于管理 Azure Batch 服务的 Azure PowerShell cmdlet"
    services="batch"
    documentationcenter=""
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f9ad62c5-27bf-4e6b-a5bf-c5f5914e6199"
    ms.service="batch"
    ms.devlang="NA"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="powershell"
    ms.workload="big-compute"
    ms.date="01/23/2017"
    wacn.date="03/16/2017"
    ms.author="tamram" />  


# Azure Batch PowerShell cmdlet 入门
通过 Azure Batch PowerShell cmdlet，可以执行许多通过 Batch API、Azure 门户预览和 Azure 命令行接口 (CLI) 执行的任务并为它们编写脚本。本文将简要介绍可用于管理 Batch 帐户和处理 Batch 资源（例如池、作业和任务）的 cmdlet。

如需 Batch cmdlet 的完整列表和详细的 cmdlet 语法，请参阅 [Azure Batch cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx)。

本文基于 Azure PowerShell 3.0.0 版本中的 cmdlet。建议你经常更新 Azure PowerShell 以利用服务更新和增强功能。

## 先决条件
执行以下操作，使用 Azure PowerShell 管理 Batch 资源。

- [安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)
- 运行可连接到订阅的 **Login-AzureRmAccount** cmdlet（Azure资源管理器模块随附 Azure Batch cmdlet）：
  
    `Login-AzureRmAccount -EnvironmentName AzureChinaCloud`  

- **注册 Batch 提供程序命名空间**。此操作**每个订阅仅需执行一次**。
  
    `Register-AzureRMResourceProvider -ProviderNamespace Microsoft.Batch`  


## 管理 Batch 帐户和密钥
### 创建 Batch 帐户
**New-AzureRmBatchAccount** 可在指定的资源组中创建 Batch 帐户。如果还没有资源组，可创建一个，方法是运行 [New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/mt603739.aspx) cmdlet。在 **Location** 参数中指定一个 Azure 区域，例如“中国北部”。例如：

    New-AzureRmResourceGroup -Name MyBatchResourceGroup -location "China North"

然后，在资源组中创建 Batch 帐户，为 <*account\_name*> 中的帐户指定帐户名，同时指定资源组的位置和名称。创建 Batch 帐户可能需要一些时间才能完成。例如：

    New-AzureRmBatchAccount -AccountName <account_name> -Location "China North" -ResourceGroupName <res_group_name>

> [AZURE.NOTE]
Batch 帐户名在资源组所在的 Azure 区域中必须唯一，长度为 3 到 24 个字符，并且只能包含小写字母和数字。
> 
> 

### 获取帐户访问密钥
**Get-AzureRmBatchAccountKeys** 显示与 Azure Batch 帐户关联的访问密钥。例如，运行以下命令可获取你创建的帐户的主要密钥和辅助密钥。

    $Account = Get-AzureRmBatchAccountKeys -AccountName <account_name>

    $Account.PrimaryAccountKey

    $Account.SecondaryAccountKey

### 生成新的访问密钥
**New-AzureRmBatchAccountKey** 为 Azure Batch 帐户生成新的主要帐户密钥或辅助帐户密钥。例如，若要为 Batch 帐户生成新的主要密钥，请键入：

    New-AzureRmBatchAccountKey -AccountName <account_name> -KeyType Primary

> [AZURE.NOTE]
若要生成新的辅助密钥，请为 **KeyType** 参数指定“Secondary”。必须单独重新生成主要密钥和辅助密钥。
> 
> 

### 删除 Batch 帐户
**Remove-AzureRmBatchAccount** 删除 Batch 帐户。例如：

    Remove-AzureRmBatchAccount -AccountName <account_name>

出现提示时，确认你想要删除该帐户。帐户删除可能需要一段时间才能完成。

## 创建 BatchAccountContext 对象
若要在创建和管理池、作业、任务和其他资源时使用 Batch PowerShell cmdlet 进行身份验证，需先创建 BatchAccountContext 对象来存储你的帐户名和密钥：

    $context = Get-AzureRmBatchAccountKeys -AccountName <account_name>

将 BatchAccountContext 对象传入使用 **BatchContext** 参数的 cmdlet。

> [AZURE.NOTE]
默认情况下，帐户的主要密钥用于身份验证，但你可以通过更改 BatchAccountContext 对象的 **KeyInUse** 属性，显式选择要使用的密钥：`$context.KeyInUse = "Secondary"`
> 
> 

## 创建和修改 Batch 资源
使用 **New-AzureBatchPool**、**New-AzureBatchJob** 和 **New-AzureBatchTask** 等 cmdlet 在 Batch 帐户下创建资源。可以使用相应的 **Get-** 和 **Set-** cmdlet 更新现有资源的属性，以及使用 **Remove-** cmdlet 删除 Batch 帐户下的资源。

使用这些 cmdlet 时，除了传递 BatchContext 对象，还需要创建或传递包含详细的资源设置的对象，如以下示例所示。请参阅每个 cmdlet 的详细帮助说明来了解其他示例。

### 创建 Batch 池
创建或更新 Batch 池时，为计算节点上的操作系统选择云服务配置或虚拟机配置（请参阅 [Batch 功能概述](/documentation/articles/batch-api-basics/#pool/)）。用户的选择将决定是使用其中一个 [Azure 来宾 OS 版本](/documentation/articles/cloud-services-guestos-update-matrix/)还是 Azure 应用商店中其中一个受支持的 Linux 或 Windows VM 映像为其计算节点创建映像。

运行 **New-AzureBatchPool** 时，使用 PSCloudServiceConfiguration 或 PSVirtualMachineConfiguration 对象传递操作系统设置。例如，以下 cmdlet 可以在云服务配置中新建包含小型计算节点的 Batch 池，这些节点是使用系列 3 最新操作系统版本 (Windows Server 2012) 映像的。在此，**CloudServiceConfiguration** 参数指定 *$configuration* 变量作为 PSCloudServiceConfiguration 对象。**BatchContext** 参数将先前定义的变量 *$context* 指定为 BatchAccountContext 对象。

    $configuration = New-Object -TypeName "Microsoft.Azure.Commands.Batch.Models.PSCloudServiceConfiguration" -ArgumentList @(4,"*")

    New-AzureBatchPool -Id "AutoScalePool" -VirtualMachineSize "Small" -CloudServiceConfiguration $configuration -AutoScaleFormula '$TargetDedicated=4;' -BatchContext $context

通过自动缩放公式确定新池中的目标计算节点数。在本示例中，公式为 **$TargetDedicated=4**，表示池中的计算节点数最多为 4。

## 查询池、作业、任务以及其他详细信息
使用 **Get-AzureBatchPool**、**Get-AzureBatchJob** 和 **Get-AzureBatchTask** 等 cmdlet 查询在 Batch 帐户下创建的实体。

### 查询数据
例如，使用 **Get-AzureBatchPools** 可查找池。假设你已将 BatchAccountContext 对象存储在 *$context* 中，则默认情况下，此 cmdlet 将查询帐户下的所有池：

    Get-AzureBatchPool -BatchContext $context

### 使用 OData 筛选器
你可以使用 **Filter** 参数提供一个 OData 筛选器，以便只查找所需的对象。例如，可以查找 ID 以“myPool”开头的所有池：

    $filter = "startswith(id,'myPool')"

    Get-AzureBatchPool -Filter $filter -BatchContext $context

此方法的灵活性不如在本地管道中使用“Where-Object”。但是，该查询将直接发送到批处理服务，因此所有筛选都在服务器端发生，这可以节省 Internet 带宽。

### 使用 Id 参数
OData 筛选器的替代方法是使用 **Id** 参数。若要查询 ID 为“myPool”的特定池：

    Get-AzureBatchPool -Id "myPool" -BatchContext $context

**Id** 参数仅支持全 ID 搜索，而不支持通配符或 OData 样式的筛选器。

### 使用 MaxCount 参数
默认情况下，每个 cmdlet 最多返回 1000 个对象。如果达到此限制，可以优化筛选器以返回更少的对象，或者使用 **MaxCount** 参数显式设置最大值。例如：

    Get-AzureBatchTask -MaxCount 2500 -BatchContext $context

若要去除上限，请 **MaxCount** 设置为 0 或更小。

### 使用 PowerShell 管道
Batch cmdlet 可以利用 PowerShell 管道在 cmdlet 之间发送数据。这与指定参数的效果相同，但可以更方便地使用多个实体。

例如，查找和显示帐户下的所有任务：

    Get-AzureBatchJob -BatchContext $context | Get-AzureBatchTask -BatchContext $context

重新启动（重启）池中每个计算节点：

    Get-AzureBatchComputeNode -PoolId "myPool" -BatchContext $context | Restart-AzureBatchComputeNode -BatchContext $context

## 应用程序包管理
应用程序包提供将应用程序部署到池中计算节点的简化方式。使用 Batch PowerShell cmdlet 可以在 Batch 帐户中上载和管理应用程序包，以及将包版本部署到计算节点。

**创建**应用程序：

    New-AzureRmBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"

**添加**应用程序包：

    New-AzureRmBatchApplicationPackage -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -ApplicationVersion "1.0" -Format zip -FilePath package001.zip

设置应用程序的**默认版本**：

    Set-AzureRmBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -DefaultVersion "1.0"

**列出**应用程序的包

    $application = Get-AzureRmBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"

    $application.ApplicationPackages

**删除**应用程序包

    Remove-AzureRmBatchApplicationPackage -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -ApplicationVersion "1.0"

**删除**应用程序

    Remove-AzureRmBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"

> [AZURE.NOTE]
在删除应用程序之前，必须删除该应用程序的所有应用程序包版本。如果尝试删除的应用程序当前存在应用程序包，则会收到“冲突”错误。
> 
> 

### 部署应用程序包
在创建池时，可以指定一个或多个要部署的应用程序包。如果创建池时指定包，该包将在节点加入池时部署到每个节点。将节点重新启动或重置映像时，也会部署包。

在创建池时指定 `-ApplicationPackageReference` 选项，以便在节点加入池时将应用程序包部署到这些节点。首先创建 **PSApplicationPackageReference** 对象，为其配置需要部署到池的计算节点的应用程序 ID 和包版本：

    $appPackageReference = New-Object Microsoft.Azure.Commands.Batch.Models.PSApplicationPackageReference

    $appPackageReference.ApplicationId = "MyBatchApplication"

    $appPackageReference.Version = "1.0"

现在创建池，并将包引用对象指定为 `ApplicationPackageReferences` 选项的参数：

    New-AzureBatchPool -Id "PoolWithAppPackage" -VirtualMachineSize "Small" -CloudServiceConfiguration $configuration -BatchContext $context -ApplicationPackageReferences $appPackageReference

有关应用程序包的详细信息，请参阅[使用 Azure Batch 应用程序包部署应用程序](/documentation/articles/batch-application-packages/)。

> [AZURE.IMPORTANT]
若要使用应用程序包，必须将 Azure 存储帐户链接到 Batch 帐户。
> 
> 

### 更新池的应用程序包
若要更新分配给现有池的应用程序，请首先使用所需属性（应用程序 ID 和包版本）创建 PSApplicationPackageReference 对象：

    $appPackageReference = New-Object Microsoft.Azure.Commands.Batch.Models.PSApplicationPackageReference

    $appPackageReference.ApplicationId = "MyBatchApplication"

    $appPackageReference.Version = "2.0"

接下来，从 Batch 获取池，清除任何现有的包，添加新的包引用，然后使用新的池设置更新 Batch 服务：

    $pool = Get-AzureBatchPool -BatchContext $context -Id "PoolWithAppPackage"

    $pool.ApplicationPackageReferences.Clear()

    $pool.ApplicationPackageReferences.Add($appPackageReference)

    Set-AzureBatchPool -BatchContext $context -Pool $pool

现在已在 Batch 服务中更新池的属性。但是，若要将新的应用程序包实际部署到池中的计算节点，必须重新启动这些节点，或者重置其映像。可以使用以下命令重新启动池中的每个节点：

    Get-AzureBatchComputeNode -PoolId "PoolWithAppPackage" -BatchContext $context | Restart-AzureBatchComputeNode -BatchContext $context

> [AZURE.TIP]
可以将多个应用程序包部署到池中的计算节点。若要*添加*应用程序包而不是更换当前部署的包，请省略上面的 `$pool.ApplicationPackageReferences.Clear()` 行。
> 
> 

## 后续步骤
- 有关详细的 cmdlet 语法和示例，请参阅 [Azure Batch cmdlet reference](https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx)（Azure Batch cmdlet 参考）。
- 有关 Batch 中应用程序和应用程序包的详细信息，请参阅[使用 Azure Batch 应用程序包部署应用程序](/documentation/articles/batch-application-packages/)。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->