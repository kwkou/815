<properties
	pageTitle="在 Azure Batch 中轻松安装和管理应用程序 | Azure"
	description="使用 Azure Batch 的应用程序包功能轻松管理要安装在 Batch 计算节点上的多个应用程序和版本。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />

<tags
	ms.service="batch"
	ms.date="06/30/2016"
	wacn.date="08/08/2016" />

# 使用 Azure Batch 应用程序包部署应用程序

Azure Batch 的应用程序包功能可为池中的计算节点提供简单的应用程序管理和部署能力。通过应用程序包，可以上载及管理多个版本的工作执行应用程序（包括二进制文件和支持文件），然后将一或多个这种类型的应用程序自动部署到池中的计算节点。

在本文中，将了解如何使用 Azure 门户来上载及管理应用程序包，然后使用 [Batch .NET][api_net] 库将它们安装在池的计算节点上。

> [AZURE.NOTE] 此处所述的应用程序包功能替换了旧版服务中的“Batch Apps”功能。

## 应用程序包要求

本文所讨论的应用程序包功能仅能与 2016 年 3 月 10 日后创建的 Batch 池兼容。应用程序包将无法部署到在此日期之前创建r 池中的计算节点。

[Batch REST API][api_rest] 2015-12-01.2.2 版和对应的 [Batch .NET][api_net] 库 3.1.0 版引入了此功能。使用 Batch 时，我们建议始终使用最新的 API 版本。

> [AZURE.IMPORTANT] 当前只有使用 **CloudServiceConfiguration** 创建的池才支持应用程序包。无法在使用 VirtualMachineConfiguration 映像创建的池中使用应用程序包。有关这两种不同配置的详细信息，请参阅 [Provision Linux compute nodes in Azure Batch pools（在 Azure Batch 池中预配 Linux 计算节点）](/documentation/articles/batch-linux-nodes/)的 [Virtual Machine Configuration（虚拟机配置）](/documentation/articles/batch-linux-nodes/#virtual-machine-configuration)部分。

## 关于应用程序和应用程序包

在 Azure Batch 中，**应用程序**是指一组已创建版本的二进制文件，这些文件可自动下载到池中的计算节点。**应用程序包**指的是这些二进制文件中的一组特定组合，其代表应用程序的特定版本。

![应用程序和应用程序包的统括示意图][1]

### 应用程序

Batch 中的每个应用程序都包含一或多个应用程序包。它能指定应用程序的配置选项；例如，要安装在计算节点上的默认应用程序包版本，以及应用程序的包是否可以更新或删除。

### 应用程序包

应用程序包为 ZIP 文件，其中包含应用程序二进制文件和任务执行所需的支持文件。每个应用程序包都代表特定版本的应用程序。在 Batch 服务中创建池时，可以指定一或多个这种类型的应用程序和版本（可选），当节点添加池时，这些应用程序包就自动下载及解压缩到每个节点上。

> [AZURE.IMPORTANT] Batch 帐户中的应用程序和应用程序包数目，以及应用程序包的大小上限有其限制。有关这些限制的详细信息，请参阅 [Quotas and limits for the Azure Batch service（Azure Batch 服务的配额和限制）](/documentation/articles/batch-quota-limit/)。

### 应用程序包的优点

应用程序包可以简化 Batch 解决方案中的代码，也能降低管理任务运行的应用程序所需的开销。

通过应用程序包，池的启动工作不需要指定在节点上安装一长串的单个资源文件。你不需要在 Azure 存储空间中或在节点上手动管理这些文件的多个版本。再者，也不必费心生成 [SAS URL](/documentation/articles/storage-dotnet-shared-access-signature-part-1/) 来提供这些文件在 Azure 存储帐户中的访问权限。

Batch 能在后台处理使用 Azure 存储空间将应用程序包存储及部署到计算节点的详细信息，因此可以简化代码和管理开销。

## 上载和管理应用程序

可以使用 Azure 门户来添加、更新和删除应用程序包，以及配置每个应用程序的默认版本。

在接下来的几个部分中，我们先讨论让存储帐户与 Batch 帐户关联，然后再了解 Azure 门户提供的包管理功能。之后，你将了解如何使用 [Batch .NET][api_net] 库将这些包部署到计算节点。

### <a name="link-a-storage-account"></a>链接存储帐户

若要使用应用程序包，必须先链接 Azure 存储帐户与 Batch 帐户。如果还没有为 Batch 帐户配置存储帐户，Azure 门户在第一次单击 Batch 帐户边栏选项卡中的“应用程序”磁贴时显示警告。

> [AZURE.IMPORTANT] Batch 目前仅支持**常规用途**存储帐户类型，如 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)的 [Create a storage account（创建存储帐户）](/documentation/articles/storage-create-storage-account/#create-a-storage-account)中步骤 5 所述。将某个 Azure 存储帐户链接到你的 Batch 帐户时，只会链接**常规用途**的存储帐户。

![Azure 门户中显示的未配置存储帐户警告][9]

Batch 服务在应用程序包的存储和检索操作中使用关联的存储帐户。一旦链接两个帐户后，Batch 便能将存储在链接之存储帐户中的包自动部署到计算节点。单击“警告”边栏选项卡中的“存储帐户设置”，然后单击“存储帐户”边栏选项卡上的“存储帐户”，以将现有存储帐户链接到 Batch 帐户。

![在 Azure 门户中选择存储帐户边栏选项卡][10]

我们建议你“专门”创建一个存储帐户用作 Batch 帐户，并在此处选择该帐户。有关创建存储帐户的详细信息，请参阅 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)中的“Create a storage account”（创建存储帐户）。创建存储帐户之后，可以使用“存储帐户”边栏选项卡将它链接到 Batch 帐户。

> [AZURE.WARNING] 由于 Batch 使用 Azure 存储空间来存储应用程序包，因此我们针对块 Blob 数据向[收取正常费用][storage_pricing]。请务必考虑应用程序包的大小和数目，并定期删除过时的包以降低成本。

### 查看当前应用程序

若要查看 Batch 帐户中的应用程序，请单击 Batch 帐户边栏选项卡中的“应用程序”磁贴。

![应用程序磁贴][2]

此时将打开“应用程序”边栏选项卡：

![列出应用程序][3]

“应用程序”边栏选项卡显示帐户中每个应用程序的 ID，以及以下属性：

* **包** - 与此应用程序关联的版本号码。
* **默认版本** – 如果在设置池的应用程序时未指定版本，系统安装此版本。此设置是可选的。
* **允许更新** – 如果将此属性设置为“否”，应用程序的包更新和删除将被禁用，只能添加应用程序包版本。默认值为“是”。

### 查看应用程序详细信息

在“应用程序”边栏选项卡中单击应用程序时，该应用程序的详细信息边栏选项卡随即出现。

![应用程序详细信息][4]

在应用程序详细信息边栏选项卡中，可以配置应用程序的以下设置。

* **允许更新** - 指定是否可更新或删除应用程序的应用程序包（请参阅下文中的“更新或删除应用程序包”）。
* **默认版本** - 指定要部署到计算节点的默认应用程序包。
* **显示名称** - 这是在显示应用程序相关信息时（例如，通过 Batch 提供给客户的服务的 UI 中），Batch 解决方案可以使用的“友好”名称。

### 添加新的应用程序

若要创建新应用程序，请使用新的唯一应用程序 ID 添加应用程序包。使用新应用程序 ID 添加的第一个应用程序包也会创建新的应用程序。

单击“应用程序”边栏选项卡中的“添加”以打开“新建应用程序”边栏选项卡。

![Azure 门户中的新建应用程序边栏选项卡][5]

“新建应用程序”边栏选项卡提供以下字段，供你指定新应用程序和应用程序包的设置。

**应用程序 ID**

指定新应用程序的 ID，需符合标准 Azure Batch ID 验证规则：

* 可以包含字母数字字符（包括连字符和下划线）的任意组合。
* 不能超过 64 个字符。
* 在 Batch 帐户内必须唯一。
* 保留大小写且不区分大小写。

**版本**

指定要上载的应用程序包版本。版本字符串需符合以下验证规则：

* 可以包含字母数字字符（包括连字符、下划线和句点）的任意组合。
* 不能超过 64 个字符。
* 在应用程序内必须唯一。
* 保留大小写且不区分大小写。

**应用程序包**

指定包含应用程序二进制文件和执行应用程序所需的任何支持文件的 ZIP 文件。单击“选择文件”文本框或文件夹图标，以浏览及选择包含应用程序文件的 ZIP 文件。

选择文件后，单击“确定”以开始上载到 Azure 存储空间。上载操作完成时，系统将发出通知，且边栏选项卡会关闭。请注意，因上载之文件的大小和网络连接速度不尽相同，此操作可能需要花费一些时间。

> [AZURE.WARNING] 上载操作完成之前，请勿关闭“新建应用程序”边栏选项卡，否则上载过程将会中止。

### 添加新应用程序包

若要添加现有应用程序的新应用程序包版本，请在“应用程序”边栏选项卡中选择应用程序，再依序单击“包”和“添加”以显示“添加包”边栏选项卡。

![Azure 门户中的添加应用程序包边栏选项卡][8]

如你所见，除了“应用程序 ID”文本框已禁用之外，字段与“新建应用程序”边栏选项卡中的字段匹配。如上所述，指定新包的“版本”，浏览到“应用程序包”ZIP 文件，然后单击“确定”以上载包。

### 更新或删除应用程序包

若要更新或删除现有的应用程序包，请打开应用程序的详细信息边栏选项卡，单击“包”以显示“包”边栏选项卡，单击要修改之应用程序包数据列中的**省略号**，然后选择要执行的操作。

![在 Azure 门户中更新或删除包][7]

**更新**

当你单击“更新”时，“更新包”边栏选项卡随即出现。此边栏选项卡与“新建应用程序包”边栏选项卡相似，只不过包选择字段已启用，因此可以指定要上载的新 ZIP 文件。

![Azure 门户中的更新包边栏选项卡][11]

**删除**

当你单击“删除”时，系统会请求你确认是否要删除包版本，随后 Batch 从 Azure 存储空间中删除该包。如果删除应用程序的默认版本，系统删除应用程序的默认版本设置。

![删除应用程序][12]

## 将应用程序安装在计算节点上

既然我们已探讨如何使用 Azure 门户来上载及管理应用程序包，接下来我们可以讨论如何将它们实际部署到计算节点，以及如何使用 Batch 任务运行它们。

若要将应用程序包安装在池中的计算节点上，需要为池指定一个或多个应用程序包“引用”。在 Batch .NET 中，可以在创建池时指定一个或多个 [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]，或将它们添加现有池。

[ApplicationPackageReference][net_pkgref] 类指定要安装在池的计算节点上的应用程序 ID 和版本。

csharp
		
		// Create the unbound CloudPool
		CloudPool myCloudPool =
		    batchClient.PoolOperations.CreatePool(
		        poolId: "myPool",
		        targetDedicated: "1",
		        virtualMachineSize: "small",
		        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "4"));
		
		// Specify the application and version to install on the compute nodes
		myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
		{
		    new ApplicationPackageReference {
		        ApplicationId = "litware",
		        Version = "1.1001.2b" }
		};
		
		// Commit the pool so that it's created in the Batch service. As the nodes join
		// the pool, the specified application package will be installed on each.
		await myCloudPool.CommitAsync();
		
将每个计算节点加入池以及该节点重新启动或重置映像时，为池指定的应用程序包将安装在该节点上。如果应用程序包部署出于任何原因而失败，Batch 服务会将该节点标记为 [unusable][net_nodestate]，并且不会在该节点上计划执行任何任务。在这种情况下，你应该**重新启动**该节点，以重新启动包部署（重新启动节点也会在节点上重新启用任务计划）。

## 执行安装的应用程序

当每个计算节点添加池（或者重新启动或重置映像），指定的包下载及解压缩到节点上 `AZ_BATCH_ROOT_DIR` 内指定的目录中。Batch 还为任务命令行创建环境变量，以在调用应用程序二进制文件时使用；此变量遵守以下命名方案：

`AZ_BATCH_APP_PACKAGE_appid#version`

例如，如果指定要安装应用程序 blender 的 2.7 版，任务可以通过在其命令行中引用以下环境变量来访问应用程序二进制文件：

`AZ_BATCH_APP_PACKAGE_BLENDER#2.7`

如果应用程序指定默认版本，你可以引用不含版本字符串后缀的环境变量。例如，如果已在 Azure 门户中指定应用程序 blender 的默认版本 2.7，任务可以引用以下环境变量：

`AZ_BATCH_APP_PACKAGE_BLENDER`

以下代码片段演示当已指定应用程序 blender 的默认版本时如何配置任务。

csharp

		string taskId = "blendertask01";
		string commandLine = @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -my -command -args";
		CloudTask blenderTask = new CloudTask(taskId, commandLine);
		

> [AZURE.TIP] 有关计算节点环境设置的详细信息，请参阅 [Batch feature overview](/documentation/articles/batch-api-basics/)（Batch 功能概述）中的“Environment settings for tasks”（任务的环境设置）。

## 更新池的应用程序包

如果已配置现有池的应用程序包，你可以指定池的新包。如果为池指定新的包引用，请遵循以下规则：

* 添加池的所有新节点都会安装新指定的包，任何重新启动或重置映像的现有节点也是如此。
* 在更新包引用时，已在池中的计算节点不会自动安装新应用程序包 - 必须将这些节点重新启动或重置映像才能接收新包。
* 部署新包后，创建的环境变量将反映新的应用程序包引用。

在此示例中，现有池已将应用程序 blender 的 2.7 版设置为其中一个 [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]。若要将池的节点更新为 2.76b 版，请将 [ApplicationPackageReference][net_pkgref] 指定为新版本并提交更改。

csharp
		
		string newVersion = "2.76b";
		CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
		boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
		{
		    new ApplicationPackageReference {
		        ApplicationId = "blender",
		        Version = newVersion }
		};
		await boundPool.CommitAsync();


配置新版本后，任何加入池的新节点都将部署 2.76b 版。若要将 2.76b 安装到已在池中的节点上，请重新启动节点（或重置映像）。请注意，重新启动的节点保留前次包部署的文件。

## 列出 Batch 帐户中的应用程序

可以使用 [ApplicationOperations][net_appops].[ListApplicationSummaries][net_appops_listappsummaries] 方法列出 Batch 帐户中的应用程序和应用程序包。

csharp

		// List the applications and their application packages in the Batch account.
		List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
		foreach (ApplicationSummary app in applications)
		{
		    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);
		
		    foreach (string version in app.Versions)
		    {
		        Console.WriteLine("  {0}", version);
		    }
		}


## 总结

通过应用程序包，当客户使用提供的启用 Batch 服务来处理操作时，可以使用更轻松的方式让他们选择操作的应用程序，以及指定要使用的确切版本。你还可以在服务中提供让客户上载及跟踪其应用程序的功能。

## 后续步骤

* [Batch REST API][api_rest] 还提供应用程序包的使用支持。有关示例，请参阅 [Add a pool to an account（将池添加到帐户）][rest_add_pool]中的 [applicationPackageReferences][rest_add_pool_with_packages] 元素，以了解如何使用 REST API 来指定要安装的包。有关使用 Batch REST API 来获取应用程序信息的详细信息，请参阅 [Applications（应用程序）][rest_applications]。

* 了解如何以编程方式[使用 Batch Management .NET 管理 Azure Batch 帐户和配额](/documentation/articles/batch-management-dotnet/)。[Batch Management .NET][api_net_mgmt] 库可以启用 Batch 应用程序或服务的帐户创建和删除功能。

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_mgmt]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[github_samples]: https://github.com/Azure/azure-batch-samples
[storage_pricing]: https://azure.microsoft.com/pricing/details/storage/
[net_appops]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.aspx
[net_appops_listappsummaries]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.listapplicationsummaries.aspx
[net_cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_cloudpool_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.applicationpackagereferences.aspx
[net_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationpackagereference.aspx
[rest_applications]: https://msdn.microsoft.com/library/azure/mt643945.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[rest_add_pool_with_packages]: https://msdn.microsoft.com/library/azure/dn820174.aspx#bk_apkgreference

[1]: ./media/batch-application-packages/app_pkg_01.png 
[2]: ./media/batch-application-packages/app_pkg_02.png 
[3]: ./media/batch-application-packages/app_pkg_03.png 
[4]: ./media/batch-application-packages/app_pkg_04.png 
[5]: ./media/batch-application-packages/app_pkg_05.png 
[7]: ./media/batch-application-packages/app_pkg_07.png 
[8]: ./media/batch-application-packages/app_pkg_08.png 
[9]: ./media/batch-application-packages/app_pkg_09.png 
[10]: ./media/batch-application-packages/app_pkg_10.png 
[11]: ./media/batch-application-packages/app_pkg_11.png 
[12]: ./media/batch-application-packages/app_pkg_12.png 

<!---HONumber=Mooncake_0801_2016-->