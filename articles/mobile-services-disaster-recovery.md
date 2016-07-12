<properties 
	pageTitle="在发生灾难时恢复移动服务 | Azure"
	description="了解在发生灾难时如何恢复移动服务。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="christopheranderson" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/07/2016"
	wacn.date="03/28/2016"/>

# 在发生灾难时恢复移动服务

当你使用 Azure 移动服务部署应用程序时，可以使用它的内置功能来确保发生可用性问题（例如服务器故障、网络中断、数据丢失和大范围的设施损毁）时保持业务连续性。使用 Azure 移动服务部署应用程序时，你可以像部署传统的本地解决方案一样，利用你必须设计、实施和管理的许多容错与基础结构功能。Azure 可让你花费少量的成本消除大量的潜在故障。

## <a name="prepare"></a>针对可能的灾难做好准备

为了在出现可用性问题时方便进行恢复，你可以提前做好准备：

+ **在 Azure 移动服务 SQL 数据库中备份数据**  

	移动服务应用程序数据存储在 Azure SQL 数据库中。我们建议你根据 [SQL 数据库业务连续性指南]中的说明备份这些数据。
	
+ **备份移动服务脚本**

	我们建议你将移动服务脚本存储在源代码管理系统（例如 [Team Foundation Service] 或 [GitHub]）中，而不只是依赖于移动服务自身中的副本。你可以使用移动服务[源代码管理功能]或[使用 Azure CLI] 通过 Azure 经典门户下载这些脚本。请特别注意 Azure 经典门户中标记为“预览”的功能，因为不保证你能够恢复这些脚本，你可能需要从自己的源代码管理原件进行恢复。
	
+ **保留辅助移动服务**

	当移动服务发生可用性问题时，你可能需要将它重新部署到备用的 Azure 区域。为确保有可用容量（例如，在极少见的情况下，整个区域都会丢失），建议你在备用区域中创建一个辅助移动服务，并将其模式设置为等同于或者高于主服务的模式。（如果主服务处于基本模式，则你可以将辅助服务设置为基本或标准模式。但如果主服务处于标准模式，则辅助服务也必须处于标准模式。）


## <a name="watch"></a>观察问题的迹象

以下情况表示出现了可能需要你执行恢复操作的问题：

+ 连接到移动服务的应用程序长时间无法与移动服务通信。
+ 移动服务状态在 [Azure 管理门户]中显示为“不正常”。
+ 在 Azure 管理门户中，移动服务的每个选项卡顶部显示“不正常”标题，并且管理操作生成错误消息。
+ [Azure 服务仪表板]指示出现了可用性问题。

## <a name="recover"></a>发生灾难后进行恢复

出现问题时，请使用服务仪表板来获取指导和更新。
 
当服务仪表板发出提示时，请执行以下步骤，以便在备用 Azure 区域中将移动服务还原到某种运行状态。如果你提前创建了辅助服务，则会使用它的容量来还原主服务。由于主服务的 URL 和应用程序密钥在恢复后不会更改，因此你不需要更新引用该 URL 和密钥的应用程序。

若要在发生中断后恢复移动服务，请执行以下操作：

1. 在 Azure 管理门户中，确保服务的状态报告为“不正常”。

2. 如果你已保留了辅助移动服务，则可以跳过此步骤。

    如果你尚未保留辅助移动服务，现在请在另一个 Azure 区域中创建一个辅助移动服务。将它的缩放模式设置为等同于主服务的模式。

3. 根据[使用 Azure CLI 自动操作移动服务]中所述，将 Azure 命令行界面 (Azure CLI) 配置为可处理你的订阅。

4. 现在，你可以使用辅助服务来恢复主服务。

	> [AZURE.IMPORTANT]除了迁移文件以外，迁移命令还会将主服务的主机名更新为指向辅助服务，因此客户端应用程序不需要更新。但是，主机名解析为新服务最长需要 30 分钟时间。因此，建议只在灾难恢复方案中使用迁移命令。

	> [AZURE.IMPORTANT]当你执行此步骤中的命令时，将会删除辅助服务，以便能够使用它的容量来恢复主服务。我们建议你在运行该命令之前备份脚本和设置（如果你想要保留的话）。

    准备就绪后，执行以下命令：

		azure mobile migrate PrimaryService SecondaryService
		info:    Executing command mobile migrate
		Warning: this action will use the capacity from the mobile service 'SecondaryService' and delete it and the host name for 'PrimaryService' may not resolve for up to 30 minutes. Do you want to migrate the mobile service 'PrimaryService'? [y/n]: y
		+ Performing migration
		+ Migration with id '0123456789abcdef0123456789abcdef' started. The migration may take several minutes to complete.
		+ Cleaning up
		info:    Migration complete. It may take 30 minutes for DNS to resolve to the migrated site.
		info:    mobile migrate command OK

    > [AZURE.NOTE] 完成该命令后，可能需要经过几分钟时间，你才能在 Azure 经典门户中看到更改。

5. 验证是否已正确恢复所有脚本，方法是将其与源代码管理中的原件进行比较。大多数情况下，脚本会自动恢复且不会丢失数据，但如果你发现存在差异，可以手动恢复该脚本。

6. 确保恢复后的服务能够与 Azure SQL 数据库通信。recover 命令将恢复移动服务，但会保留与原始数据库的连接。如果 Azure 主区域中的问题也影响到了数据库，则恢复后的服务可能仍然不会正常运行。你可以使用 Azure 服务仪表板来检查给定区域的数据库状态。如果原始数据库不工作，则你可以恢复它：
	+ 根据 [SQL 数据库业务连续性指南]中所述，将 Azure SQL 数据库恢复到你刚刚在其中恢复了移动服务的 Azure 区域。
	+ 在 Azure 经典门户中，在移动服务的“配置”选项卡上选择“更改数据库”，然后选择刚刚恢复的数据库。

7. 现在，你的移动服务已托管在不同的物理位置。你需要更新发布和/或 git 凭据，以便更新正在运行的站点。
	+ 如果你使用 **.NET 后端**，请按[发布移动服务](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/#publish-your-mobile-service)中所述重新设置发布配置文件。这会更新发布详细信息，以指向新的服务位置。
	+ 如果你使用 **Javascript 后端**并通过门户来管理服务，则不需要执行任何附加的操作。
	+ 如果你使用 **Javascript 后端**并通过节点来管理服务，请更新 git 远程设置以指向新的存储库。为此，请从 git 远程设置中删除 .git 文件路径：

		1. 查找当前的来源远程设置：
				git remote -v
				 origin https://myservice.scm.azure-mobile.net/myservice.git (fetch)
				 origin https://myservice.scm.azure-mobile.net/myservice.git (push)

		3. 使用相同的 url 更新远程设置，但不包含最终 .git 文件路径：
				git remote set-url origin https://myservice.scm.azure-mobile.net
		4. 从来源提取数据以验证它是否正常工作。

现在，你所处的状态应该就是移动服务在新 Azure 区域中恢复到的状态，并且该移动服务正在使用其原始 URL 接收来自应用商店应用程序的流量。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[SQL 数据库业务连续性指南]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh852669.aspx
[Team Foundation Service]: http://tfs.visualstudio.com/
[Github]: https://github.com/
[源代码管理功能]: /documentation/articles/mobile-services-store-scripts-source-control/
[使用 Azure CLI]: /documentation/articles/mobile-services-manage-command-line-interface/
[Azure 经典门户]: http://manage.windowsazure.cn/
[Azure 服务仪表板]: /zh-cn/support/service-dashboard/
[使用 Azure CLI 自动操作移动服务]: /documentation/articles/mobile-services-manage-command-line-interface/

<!---HONumber=Mooncake_0118_2016-->