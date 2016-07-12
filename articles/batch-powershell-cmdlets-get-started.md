<properties
   pageTitle="Azure 批处理( Batch ) PowerShell cmdlet 入门 | Azure"
   description="快速介绍可用于管理 Azure 批处理( Batch )服务的 Azure PowerShell cmdlet"
   services="batch"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.date="04/21/2016"
   wacn.date="06/06/2016"/>

# Azure 批处理 PowerShell cmdlet 入门
本文将简要介绍可用于管理 Batch 帐户和处理 Batch 资源（例如池、作业和任务）的 Azure PowerShell cmdlet。你可以通过 Batch cmdlet 执行许多相同的任务，而 Batch cmdlet 可通过 Batch API 和 Azure 门户来执行。本文基于 Azure PowerShell 1.0 或更高版本中的 cmdlet。

如需 Batch cmdlet 的完整列表和详细的 cmdlet 语法，请参阅 [Azure Batch cmdlet 参考](https://msdn.microsoft.com/library/azure/mt125957.aspx)。


## 先决条件

* **Azure PowerShell** - 有关 Azure PowerShell 的下载和安装说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。 
   
    * 由于 Azure Batch cmdlet 随附在 Azure Resource Manager 模块中，因此需要运行 **Login-AzureRmAccount** cmdlet 才能连接到订阅。 
    
    * 建议你经常更新 Azure PowerShell 以利用服务更新和增强功能。
    
* **注册到 Batch 提供程序命名空间（一次性操作）**- 在使用 Batch 帐户之前，你必须注册到 Batch 提供程序命名空间。此操作只需在订阅时执行一次。运行以下 cmdlet：

    
    	Register-AzureRMResourceProvider -ProviderNamespace Microsoft.Batch
    

## 管理批处理( Batch )帐户和密钥


### 创建 批处理( Batch )帐户

**New-AzureRmBatchAccount** 可在指定的资源组中创建新的批处理( Batch ) 帐户。如果你没有资源组，可以通过运行 [New-AzureRmResourceGroup](https://msdn.microsoft.com/library/azure/mt603739.aspx) cmdlet 并在 **Location** 参数中指定一个 Azure 区域（例如“美国中部”）来创建资源组。例如：


		New-AzureRmResourceGroup –Name MyBatchResourceGroup –location "China North"


然后，在资源组中创建新的批处理( Batch ) 帐户，此时还应该为 <*account\_name*> 指定帐户名，以及批处理 ( Batch ) 服务可用的位置。帐户创建可能需要几分钟才能完成。例如：


		New-AzureRmBatchAccount –AccountName <account_name> –Location "Central US" –ResourceGroupName MyBatchResourceGroup


> [AZURE.NOTE] 批处理( Batch )帐户名在资源组所在的 Azure 区域，长度为 3 到 24 个字符，并且只能包含小写字母和数字。

### 获取帐户访问密钥
**Get-AzureRmBatchAccountKeys** 显示与 Azure 批处理( Batch )帐户关联的访问密钥。例如，运行以下命令可获取你创建的帐户的主要密钥和辅助密钥。
	
	    $Account = Get-AzureRmBatchAccountKeys –AccountName <accountname>
	
	    $Account.PrimaryAccountKey
	
	    $Account.SecondaryAccountKey


### 生成新的访问密钥
**New-AzureRmBatchAccountKey** 为 Azure 批处理( Batch )帐户生成新的主要帐户密钥或辅助帐户密钥。例如，若要为批处理 ( Batch ) 帐户生成新的主要密钥，请键入：


    	New-AzureRmBatchAccountKey -AccountName <account_name> -KeyType Primary


> [AZURE.NOTE]若要生成新的辅助密钥，请为 **KeyType** 参数指定“Secondary”。必须单独重新生成主要密钥和辅助密钥。

### 删除批处理( Batch )帐户
**Remove-AzureRmBatchAccount** 删除批处理( Batch )帐户。例如：


		Remove-AzureRmBatchAccount -AccountName <account_name>


出现提示时，确认你想要删除该帐户。帐户删除可能需要一段时间才能完成。

## 创建 BatchAccountContext 对象

若要在创建和管理池、作业、任务和其他资源时使用 Batch PowerShell cmdlet 进行身份验证，需先创建 BatchAccountContext 对象来存储你的帐户名和密钥：


		$context = Get-AzureRmBatchAccountKeys -AccountName <account_name>


将 BatchAccountContext 对象传入使用 **BatchContext** 参数的 cmdlet。

> [AZURE.NOTE]默认情况下，帐户的主要密钥用于身份验证，但你可以通过更改 BatchAccountContext 对象的 **KeyInUse** 属性，显式选择要使用的密钥：`$context.KeyInUse = "Secondary"`



## 创建和修改 Batch 资源
使用 **New-AzureBatchPool**、**New-AzureBatchJob** 和 **New-AzureBatchTask** 等 cmdlet 在 Batch 帐户下创建资源。可以使用相应的 **Get-** 和 **Set-** cmdlet 来更新现有资源的属性，以及使用 **Remove-** cmdlet 来删除 Batch 帐户下的资源。

### 创建 Batch 池

例如，以下 cmdlet 可创建新的 Batch 池，该池在配置后可以使用包含系列 3 最新操作系统版本 (Windows Server 2012) 映像的小型虚拟机，计算节点的目标数由自动缩放公式而定。在本示例中，公式为 **$TargetDedicated=3**，表示池中的计算节点数最多为 3。**BatchContext** 参数将先前定义的变量 *$context* 指定为 BatchAccountContext 对象。

		
		New-AzureBatchPool -Id "MyAutoScalePool" -VirtualMachineSize "Small" -OSFamily "3" -TargetOSVersion "*" -AutoScaleFormula '$TargetDedicated=3;' -BatchContext $Context

>[AZURE.NOTE]目前，Batch PowerShell cmdlet 仅支持计算节点的云服务配置。这可让你选择要在计算节点上运行的，某个 Azure 来宾 OS 版本的 Windows Server 操作系统。有关 Batch 池的其他计算节点配置选项，请使用 Batch SDK 或 Azure CLI。

## 查询池、作业、任务以及其他详细信息

使用 cmdlet（例如 **Get-AzureBatchPool**、**Get-AzureBatchJob** 和 **Get-AzureBatchTask**）查询在 Batch 帐户下创建的实体。


### 查询数据

例如，使用 **Get-AzureBatchPools** 可查找池。假设你已将 BatchAccountContext 对象存储在 $context 中，则默认情况下，此 cmdlet 将查询帐户下的所有池：


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

### 使用管道

批处理 cmdlet 可以利用 PowerShell 管道在 cmdlet 之间发送数据。这与指定参数的效果相同，但可以更方便地列出多个实体。例如，以下代码可查找你帐户下的所有任务：


		Get-AzureBatchJob -BatchContext $context | Get-AzureBatchTask -BatchContext $context


## 后续步骤
* 有关详细的 cmdlet 语法和示例，请参阅 [Azure Batch cmdlet reference（Azure Batch cmdlet 参考）](https://msdn.microsoft.com/library/azure/mt125957.aspx)。

* 有关如何减少项数和有关针对 Batch 查询返回的信息类型的详细信息，请参阅 [Query the Batch service efficiently（有效地查询 Batch 服务）](batch-efficient-list-queries.md)。

<!---HONumber=Mooncake_0530_2016-->