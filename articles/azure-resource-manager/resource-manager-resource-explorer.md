<properties
    pageTitle="Azure 资源浏览器 | Azure"
    description="介绍 Azure 资源浏览器以及如何使用它通过 Azure Resource Manager 来查看和更新部署"
    services="azure-resource-manager"
    documentationcenter="na"
    author="stuartleeks"
    manager="ankodu"
    editor="" />  

<tags
    ms.assetid="950f2298-b5dd-4343-b925-8ef28493cacf"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="08/01/2016"
    wacn.date="12/26/2016"
    ms.author="stuartle;tomfitz" />

# 使用 Azure 资源浏览器查看和修改资源
[Azure 资源浏览器](https://resources.azure.com)是一个极佳的工具，可用于查看已在订阅中创建的资源。通过使用此工具，可以了解组织资源的方式，并查看分配给每个资源的属性。你可以了解可用于某种资源类型的 REST API 操作和 PowerShell cmdlet 的相关信息，并可以通过界面发出命令。在创建资源管理器模板时，资源浏览器可能特别有用，因为使用它可以查看现有资源的属性。

资源浏览器工具的源代码可在 [github](https://github.com/projectkudu/ARMExplorer) 上找到，如果你需要在自己的应用程序中实现类似行为，它可提供有价值的参考。

## 查看资源
导航到 [https://resources.azure.com](https://resources.azure.com) 并使用用于 [Azure 门户预览](https://portal.azure.cn)的相同凭据登录。

加载后，使用左侧树视图可以向下钻取到订阅和资源组：

![树视图](./media/resource-manager-resource-explorer/are-01-treeview.png)

钻取到资源组后，你将看到该组中有其资源的提供程序：

![providers](./media/resource-manager-resource-explorer/are-02-treeview-providers.png)

从该处可以开始钻取到资源实例。在下面的屏幕截图中，可以看到树视图中的 `sltest` SQL Server 实例。在右侧，可以看到可用于该资源的 REST API 请求的相关信息。通过导航到资源节点，资源浏览器自动发出 GET 请求以检索有关资源的信息。在 URL 下的大文本区域中，你将看到来自 API 的响应。

随着对 Resource Manager 模板的不断熟悉，正文内容也会开始变得让人熟悉！ 响应的 **properties** 部分将与可以在模板的 **properties** 节中提供的值匹配。

![SQL Server](./media/resource-manager-resource-explorer/are-03-sqlserver-with-response.png)

资源浏览器允许你继续向下钻取以浏览子资源，就 SQL 数据库服务器而言，数据库和防火墙规则等资源有子资源。

浏览一个数据库时将显示该数据库的属性。在下面的屏幕截图中可以看到，数据库 `edition` 是 `Standard`，`serviceLevelObjective`（或数据库层）是 `S1`。

![SQL 数据库](./media/resource-manager-resource-explorer/are-04-database-get.png)

## 更改资源
导航到某个资源后，可以选择“编辑”按钮，使 JSON 内容可编辑。然后，可以使用资源浏览器编辑 JSON 并发送 PUT 请求以更改资源。例如，下图显示数据库层已更改为 `S0`：

![数据库 - PUT 请求](./media/resource-manager-resource-explorer/are-05-database-put.png)

通过选择 **PUT** 可以提交请求。

提交请求后，资源浏览器会再次发出 GET 请求以刷新状态。在此例中，我们可以看到 `requestedServiceObjectiveId` 已更新并且不同于 `currentServiceObjectiveId`，以指示正在进行缩放操作。可以单击 GET 按钮以手动刷新状态。

![数据库 - GET 请求 2](./media/resource-manager-resource-explorer/are-06-database-get2.png)

## 对资源执行操作
使用“操作”选项卡可以查看和执行其他 REST 操作。例如，当你选择了网站资源后，“操作”选项卡将显示可用操作的长列表，其中一些操作如下所示。

![web - POST 请求](./media/resource-manager-resource-explorer/are-web-post.png)

## 通过 PowerShell 调用 API
资源浏览器中的 PowerShell 选项卡显示用于与你正在浏览的资源交互的 cmdlet。根据所选的资源类型，显示的 PowerShell 脚本的范围可以从简单的 cmdlet（如 `Get-AzureRmResource` 和 `Set-AzureRmResource`）到更复杂的 cmdlet（如交换网站上的槽）。

![PowerShell](./media/resource-manager-resource-explorer/are-07-powershell.png)

有关 Azure PowerShell cmdlet 的详细信息，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)

## 摘要
使用 Resource Manager 时，资源浏览器就是一个非常有用的工具。它是想办法使用 PowerShell 查询和进行更改的有效方法。如果你正在使用 REST API，它是在开始编写代码前开始使用和快速测试 API 调用的好办法。如果要编写 ARM 模板，它会是了解资源层次结构和查找在何处放置配置的好办法 - 可以在门户中进行更改，然后在资源浏览器中找到对应条目！

有关详细信息，观看 [Scott Hanselman 和 David Ebbo 协作完成的第 9 频道视频](https://channel9.msdn.com/Shows/Azure-Friday/Azure-Resource-Manager-Explorer-with-David-Ebbo)

<!---HONumber=Mooncake_1219_2016-->