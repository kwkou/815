<properties
   pageTitle="Azure  批处理( Batch ) PowerShell cmdlet 入门 | Windows Azure"
   description="介绍用于管理 Azure  批处理( Batch )服务的 Azure PowerShell cmdlet"
   services="batch"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.date="07/08/2015"
   wacn.date="10/30/2015"/>

# Azure  批处理( Batch ) PowerShell cmdlet 入门
本文将简要介绍可用于管理 批处理( Batch )帐户的 Azure PowerShell cmdlet，并提供有关 批处理( Batch )作业、任务的信息和其他详细信息。

有关详细的 cmdlet 语法，请键入`get-help <Cmdlet_name>`，或参阅 [Azure 批处理( Batch ) cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx)。

## 先决条件

* **Azure PowerShell** - 有关先决条件以及下载和安装说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。0.8.10 和更高版本中引入了 批处理( Batch ) cmdlet。

## 使用 批处理( Batch ) cmdlet

使用标准过程来启动 Azure PowerShell，然后[连接到你的 Azure 订阅](/documentation/articles/powershell-install-configure#Connect)。此外：

* **选择 Azure 订阅** - 如果你有多个订阅，请选择一个订阅：

    ```
    Select-AzureSubscription -SubscriptionName <SubscriptionName>
    ```

* **切换到 AzureResourceManage 模式** - Azure 资源管理器模块随附了 批处理( Batch ) cmdlet。有关详细信息，请参阅[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。若要使用此模块，请运行 [Switch-AzureMode](https://msdn.microsoft.com/zh-cn/library/dn722470.aspx) cmdlet：

    ```
    Switch-AzureMode -Name AzureResourceManager
    ```

## 管理 批处理( Batch )帐户和密钥

你可以使用 Azure PowerShell cmdlet 来创建和管理 批处理( Batch )帐户与密钥。

### 创建 批处理( Batch )帐户

**New-AzureBatchAccount** 可在指定的资源组中创建新的批处理( Batch )帐户。如果你没有资源组，可以通过运行 [New-AzureResourceGroup](https://msdn.microsoft.com/zh-cn/library/dn654594.aspx) cmdlet 并在 **Location** 参数中指定一个 Azure 区域来创建资源组。（可以通过运行 [Get-AzureLocation](https://msdn.microsoft.com/zh-cn/library/dn654582.aspx) 查找不同 Azure 资源的可用区域。） 例如：

```
New-AzureResourceGroup –Name MyBatchResourceGroup –location "China North"
```

然后，在资源组中创建新的批处理( Batch )帐户，此时还应该为 <*account_name*> 指定帐户名，以及 批处理( Batch )服务可用的位置。帐户创建可能需要几分钟才能完成。例如：

```
New-AzureBatchAccount –AccountName <account_name> –Location "China North" –ResourceGroupName MyBatchResourceGroup
```

> [AZURE.NOTE] 批处理( Batch )帐户名在 Azure 中必须是唯一的，长度为 3 到 24 个字符，并且只能包含小写字母和数字。

### 获取帐户访问密钥
**Get-AzureBatchAccountKeys** 显示与 Azure  批处理( Batch )帐户关联的访问密钥。例如，运行以下命令可获取你创建的帐户的主要密钥和辅助密钥。

```
$Account = Get-AzureBatchAccountKeys –AccountName <accountname>
$Account.PrimaryAccountKey
$Account.SecondaryAccountKey
```

### 生成新的访问密钥
**New-AzureBatchAccountKey** 生成 Azure  批处理( Batch )帐户的新主要帐户密钥或辅助帐户密钥。例如，若要为 批处理( Batch )帐户生成新的主要密钥，请键入：

```
New-AzureBatchAccountKey -AccountName <account_name> -KeyType Primary
```

> [AZURE.NOTE]若要生成新的辅助密钥，请为 **KeyType** 参数指定“Secondary”。必须单独重新生成主要密钥和辅助密钥。

### 删除 批处理( Batch )帐户
**Remove-AzureBatchAccount** 删除 批处理( Batch )帐户。例如：

```
Remove-AzureBatchAccount -AccountName <account_name>
```

出现提示时，确认你想要删除该帐户。帐户删除可能需要一段时间才能完成。

## 作业、任务查询和其他详细信息

使用 **Get-AzureBatchJob**、**Get-AzureBatchTask** 和 **Get-AzureBatchPool** 等 cmdlet 来查询 批处理( Batch )帐户下创建的实体。

若要使用这些 cmdlet，首先需要创建一个 AzureBatchContext 对象用于存储帐户名和密钥：

```
$context = Get-AzureBatchAccountKeys "<account_name>"
```

使用 **BatchContext** 参数将此上下文传入与 批处理( Batch )服务交互的 cmdlet。

> [AZURE.NOTE]默认情况下，帐户的主要密钥用于身份验证，但你可以通过更改 BatchAccountContext 对象的 **KeyInUse** 属性，显式选择要使用的密钥：`$context.KeyInUse = "Secondary"`


### 查询数据

例如，使用 **Get-AzureBatchPools** 可查找池。假设你已将 BatchAccountContext 对象存储在 *$context* 中，则默认情况下，此 cmdlet 将查询帐户下的所有池：

```
Get-AzureBatchPool -BatchContext $context
```
### 使用 OData 筛选器

你可以使用 **Filter** 参数提供一个 OData 筛选器，以便只查找所需的对象。例如，可以查找名称以“myPool”开头的所有池：

```
$filter = "startswith(name,'myPool')"
Get-AzureBatchPool -Filter $filter -BatchContext $context
```

此方法的灵活性不如在本地管道中使用“Where-Object”。但是，该查询将直接发送到 批处理( Batch )服务，因此所有筛选都在服务器端发生，这可以节省 Internet 带宽。

### 使用 Name 参数

OData 筛选器的替代方法是使用 **Name** 参数。若要查询名为“myPool”的特定池：

``` 
Get-AzureBatchWorkItem -Name "myWorkItem" -BatchContext $context 

```
**Name** 参数仅支持全名搜索，而不支持通配符或 OData 样式的筛选器。

### 使用管道

 批处理( Batch ) cmdlet 可以利用 PowerShell 管道在 cmdlet 之间发送数据。这与指定参数的效果相同，但可以更方便地列出多个实体。例如，你可以查找帐户下的所有任务：

```
Get-AzureBatchJob -BatchContext $context | Get-AzureBatchTask -BatchContext $context
```

### 使用 MaxCount 参数

默认情况下，每个 cmdlet 最多返回 1000 个对象。如果达到此限制，你可以优化筛选器以返回更少的对象，或者使用 **MaxCount** 参数显式设置最大值。例如：

```
Get-AzureBatchTask -MaxCount 2500 -BatchContext $context

```

若要去除上限，请 **MaxCount** 设置为 0 或更小。

## 相关主题
* [ 批处理( Batch )技术概述](/documentation/articles/batch-technical-overview)
* [下载 Azure PowerShell](http://go.microsoft.com/p/?linkid=9811175)
* [Azure  批处理( Batch ) cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx)
* [高效的列表查询](/documentation/articles/batch-efficient-list-queries)

<!---HONumber=66-->